---
title: "FreeRTOS Lab 2: 走訪 Tasklist 並實作 task monitor"
subtitle: ""
excerpt: "stm32f stm32f407g freertos task"
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

參照 [在 stm32f407g 使用 uart](https://blueskyson.github.io/2022/04/03/stm32f407g-uart/)設置好 uart4。


接下來編輯 `.ioc` 檔案

- 將 `PD12` 到 `PD15` 設為 `GPIO_Output`，Label 設為 `led_green`、`led_orange`、`led_red`、`led_blue`。
![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/freertos-lab2-1.png)

接下來設置好 FreeRTOS 的執行環境，因為網路很多教學，例如[這篇](https://jasonblog.github.io/note/freertos/[stm32]_4_yi_zhi_freertos.html)，所以就不贅述。

## Lab 目標

走訪 `pxReadyTasksLists`、`pxDelayedTaskList`、`pxOverflowDelayedTaskList`，並且透過 uart4 印出所有 task control block 的資訊。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/freertos-lab2-2.png)

TaskList 的資料結構為 Linked List，示意圖如下：

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/freertos-lab2-3.png)

對應的宣告在 [list.h](https://github.com/FreeRTOS/FreeRTOS-Kernel/blob/main/include/list.h)：

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/freertos-lab2-4.png)

Task Control Block 的資料結構宣告在 [tasks.c](https://github.com/FreeRTOS/FreeRTOS-Kernel/blob/main/tasks.c) 的 `tskTaskControlBlock`。 

## 撰寫主程式

在 task.h 的最後面引入必要的函式庫，宣告 `Taskmonitor()` 作為印出 task control block 的資訊的函式：

```c
#include "stm32f4xx_hal.h"
#include <string.h>
#include <stdio.h>
UART_HandleTypeDef huart4;
void Taskmonitor(void);
```

在 tasks.c 的最後面實作 `Taskmonitor()`：

```c
void print_pcb(tskTCB *t, char *state) {
	char Monitor_data[130];
	memset(Monitor_data,'\0',sizeof(Monitor_data));
	sprintf(Monitor_data, "%-19s %3lu/%-19lu 0x%-9lx 0x%-14lx %s\n\r",
            t->pcTaskName, t->uxPriority, t->uxBasePriority, t->pxStack, t->pxTopOfStack, state);
	HAL_UART_Transmit(&huart4, (uint8_t *)Monitor_data, strlen(Monitor_data), HAL_MAX_DELAY);
}

void Taskmonitor(void)
{
	/* Stop scheduler */
	/* Taskmonitor() will block when UART is transmitting data */
	vTaskSuspendAll();

	/* Print title */
    char *Monitor_data = "|Name              |Priority(Base/actual)  |pxStack    |pxTopOfStack    |State    |\n\r";
	HAL_UART_Transmit(&huart4, (uint8_t *)Monitor_data, strlen(Monitor_data), HAL_MAX_DELAY);

	ListItem_t *node;
	/* pxReadyTasksLists */
	for (int i = 0; i < configMAX_PRIORITIES; i++) {
		node = listGET_HEAD_ENTRY(pxReadyTasksLists + i);
		for (uint32_t j = 0; j < listCURRENT_LIST_LENGTH(pxReadyTasksLists + i); j++) {
			print_pcb(listGET_LIST_ITEM_OWNER(node), "Ready");
			node = listGET_NEXT(node);
		}
	}

	/* pxDelayedTaskList*/
	node = listGET_HEAD_ENTRY(pxDelayedTaskList);
	for (uint32_t j = 0; j < listCURRENT_LIST_LENGTH(pxDelayedTaskList); j++) {
		print_pcb(listGET_LIST_ITEM_OWNER(node), "Blocked");
		node = listGET_NEXT(node);
	}

	/* pxOverflowDelayedTaskList */
	node = listGET_HEAD_ENTRY(pxOverflowDelayedTaskList);
	for (uint32_t j = 0; j < listCURRENT_LIST_LENGTH(pxOverflowDelayedTaskList); j++) {
		print_pcb(listGET_LIST_ITEM_OWNER(node), "Overflow");
		node = listGET_NEXT(node);
	}

	/* Resume scheduler */
	xTaskResumeAll();
}
```

編輯 `Core/Src/main.c`，引入標頭檔：

```c
/* USER CODE BEGIN Includes */
#include "FreeRTOS.h"
#include "task.h"
/* USER CODE END Includes */
```

撰寫 `Red_LED_App`、`Green_LED_App`、`Delay_App`、`TaskMonitor_App`：

```c
/* USER CODE BEGIN 0 */
TaskHandle_t xHandle=NULL;

void Red_LED_App(void *pvParameters){
	uint32_t Redtimer = 800;
	for(;;){
		HAL_GPIO_TogglePin(GPIOD, led_red_Pin);
		vTaskDelay(Redtimer);
		Redtimer+=1;
	}
}

void Green_LED_App(void *pvParameters){
	uint32_t Greentimer = 1000;
	for(;;){
		HAL_GPIO_TogglePin(GPIOD, led_green_Pin);
		vTaskDelay(Greentimer);
		Greentimer+=2;
	}
}

void Delay_App(void *pvParameters){
	int delayflag=0;
	uint32_t delaytime;
	while(1){
		if(delayflag==0){
			delaytime = 1000;
			delayflag=1;
		}else{
			delaytime=0xFFFFFFFF;
		}
		vTaskDelay(delaytime);
	}
}

void TaskMonitor_App(void *pvParameters){
	for(;;){
		Taskmonitor();
		vTaskDelay(1000);
	}
}
/* USER CODE END 0 */
```

最後在 `main()` 函式創建這四個 App 讓 FreeRTOS 排程，注意 `TaskMonior_App` 的 Stack Size 要夠大才夠 `sprintf` 處理字串：

```c
/* USER CODE BEGIN 2 */
xTaskCreate(Red_LED_App, "Red_LED_App", 128, NULL, 1, &xHandle);
xTaskCreate(Green_LED_App, "Green_LED_App", 128, NULL, 1, &xHandle);
xTaskCreate(TaskMonitor_App, "TaskMonitor_App", 512, NULL, 3, &xHandle);
xTaskCreate(Delay_App, "Delay_App", 128, NULL, 14, &xHandle);
vTaskStartScheduler();
/* USER CODE END 2 */
```

Demo

![](https://github.com/blueskyson/image-host/blob/master/2022/freertos-lab2-5.gif?raw=true)
