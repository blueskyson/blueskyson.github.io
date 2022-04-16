---
title: "FreeRTOS Lab 1: Task 與 Queue"
subtitle: ""
excerpt: "stm32f stm32f407g freertos task queue"
layout: post
author: "blueskyson"
mathjax: true
header-style: text
tags:
  - FreeRTOS
  - stm32f
  - c
---

環境: wnidows 10, STM32CubeIDE 1.8.0

開發板: stm32f407g

## 環境設定

首先編輯 `.ioc` 檔案

- 將 `PA0` 設為 `GPIO_Input`，並且將 `PA0` 的 Label 設為 `btn_blue`。
- 將 `PD12` 到 `PD15` 設為 `GPIO_Output`，Label 設為 `led_green`、`led_orange`、`led_red`、`led_blue`。
![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/freertos-lab1-1.png)

接下來設置好 FreeRTOS 的執行環境，因為網路很多教學，例如[這篇](https://jasonblog.github.io/note/freertos/[stm32]_4_yi_zhi_freertos.html)，所以就不贅述。

## Lab 目標

- **LED_Task: state 0:** First, only Green LED lights up for 2 seconds,
and then only Red LED lights up for 2 seconds,
and then switches back to the Green LED, then Red, and so on.
- **LED_Task: state 1:** Only ORANGE LED is blinking (1 second ON, 1 second OFF, …).
- **Button_Task:** If the button is pressed, the LED-task will switch to the other state.

## 撰寫主程式

編輯 `Core/Src/main.c`，引入標頭檔：

```c
/* USER CODE BEGIN Includes */
#include "FreeRTOS.h"
#include "task.h"
#include "queue.h"
/* USER CODE END Includes */
```

宣告全域變數 `xQueue` 作為 `LED_Task` 與 `Button_Task` 傳遞訊息的佇列、宣告 `struct TaskMessage` 作為傳遞訊息的資料結構。

```c
TaskHandle_t xHandle=NULL;
QueueHandle_t xQueue;
struct TaskMessage {
	int state;
};
```

撰寫 `Button_Task`，當按鈕被按下時，將當前的 state 傳遞到 `xQueue` 中：

```c
void Button_Task(void *pvParameters) {
	int pushed = 0;
	static struct TaskMessage mesg = {.state = 0};
	for (;;) {
		if (HAL_GPIO_ReadPin(btn_blue_GPIO_Port, btn_blue_Pin)) {
			if (!pushed) {
				mesg.state ^= 1;
				xQueueSend(xQueue, &mesg, 0);
				pushed = 1;
			}
		} else {
			pushed = 0;
		}
	}
}
```

撰寫 `LED_Task`，嘗試從 `xQueue` 抓取當前的 state，並且執行 LED 閃爍的功能：

```c
void LED_Task(void *pvParameters) {
	int msec = 0;
	int state = 0;
	struct TaskMessage mesg;

	for (;;) {
		if (xQueueReceive(xQueue, &mesg, 0) == pdPASS) {
			if (mesg.state == 0) {
				HAL_GPIO_WritePin(led_orange_GPIO_Port, led_orange_Pin, GPIO_PIN_RESET);
				msec = 0;
			} else {
				HAL_GPIO_WritePin(led_red_GPIO_Port, led_red_Pin, GPIO_PIN_RESET);
				HAL_GPIO_WritePin(led_green_GPIO_Port, led_green_Pin, GPIO_PIN_RESET);
			}
			state = mesg.state;
		}

		if (state == 0) {	// state 0
			if (msec < 2000) {
				HAL_GPIO_WritePin(led_green_GPIO_Port, led_green_Pin, GPIO_PIN_SET);
				HAL_GPIO_WritePin(led_red_GPIO_Port, led_red_Pin, GPIO_PIN_RESET);
			} else if (msec < 4000) {
				HAL_GPIO_WritePin(led_green_GPIO_Port, led_green_Pin, GPIO_PIN_RESET);
				HAL_GPIO_WritePin(led_red_GPIO_Port, led_red_Pin, GPIO_PIN_SET);
			} else {
				msec = 0;
				HAL_GPIO_WritePin(led_green_GPIO_Port, led_green_Pin, GPIO_PIN_SET);
				HAL_GPIO_WritePin(led_red_GPIO_Port, led_red_Pin, GPIO_PIN_RESET);
			}
		} else {			// state 1
			if (msec < 1000) {
				HAL_GPIO_WritePin(led_orange_GPIO_Port, led_orange_Pin, GPIO_PIN_SET);
			} else if (msec < 2000) {
				HAL_GPIO_WritePin(led_orange_GPIO_Port, led_orange_Pin, GPIO_PIN_RESET);
			} else {
				msec = 0;
				HAL_GPIO_WritePin(led_orange_GPIO_Port, led_orange_Pin, GPIO_PIN_SET);
			}
		}

		vTaskDelay(1);
		msec++;
	}
}
```

最後在 `main()` 函式創建 `LED_Task` 與 `Button_Task` 讓 FreeRTOS 排程：

```c
/* USER CODE BEGIN 2 */
xQueue = xQueueCreate(3, sizeof(struct TaskMessage));
xTaskCreate(LED_Task, "LED_Task", 128, NULL, 1, &xHandle);
xTaskCreate(Button_Task, "Button_Task", 128, NULL, 1, &xHandle);

vTaskStartScheduler();
/* USER CODE END 2 */
```

Demo

![](https://github.com/blueskyson/image-host/blob/master/2022/freertos-lab1.gif?raw=true)