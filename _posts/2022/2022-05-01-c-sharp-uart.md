---
title: "C# 使用 Serial Port (uart)"
subtitle: ""
excerpt: "c# c sharp serial port uart"
layout: post
author: "blueskyson"
mathjax: true
header-style: text
tags:
  - stm32f
  - csharp
---

環境: wnidows 10, Visual Studio 2019, .NET Core 3.1
測試用開發版: stm32f407g, STM32CubeIDE 1.8.0

## 專案初始化

首先參考[在 stm32f407g 使用 lis302dl 三軸加速規](https://blueskyson.github.io/2022/04/29/stm32f407g-lis302dl/)讓開發版持續輸出三軸加速規的訊息，或是自己準備一個可以讓 COM 連接埠持續齣齣訊息的環境。

接下來打開 Visual Studio 選擇 `.NET Core 3.1` 以及 `Console Application` 初始化一個終端機專案，我將專案命名為 Uart_Console_App。

點選 `工具` -> `NuGet 套件管理員` -> `管理方案的 NuGet 套件`，然後在左上方搜尋欄輸入 SerialPort，然後安裝 `System.IO.Ports`。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/csharp-uart.png)

## 撰寫讀寫 Uart 的 Class

右鍵點選`方案總管`的專案 (即下圖中的 Uart_Console_App)，選擇`加入` -> `新增項目`，在下圖的視窗中選擇 `類別`。

建立好類別後，在`方案總管`將該檔案改名為 Uart.cs，左為我們收發 Uart 訊息的物件。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/csharp-uart-2.png)

在 Uart.cs 中貼入以下程式碼：

```cpp
using System;
using System.Collections.Generic;
using System.Text;
using System.IO.Ports;
using System.Threading;

namespace Uart_Console_App
{
    public class Uart
    {
        private SerialPort serial_port;
        private string port_name = "COM4";
        private int baud_rate = 9600;

        private string serial_buffer = "";

        public Uart() { }

        public Uart(string PortName, int BaudRate)
        {
            port_name = PortName;
            baud_rate = BaudRate;
        }

        public void OpenSerial()
        {
            serial_port = new SerialPort(port_name, baud_rate);

            try {
                serial_port.Open();
                if (!serial_port.IsOpen) {
                    Console.WriteLine("Fail to open " + port_name);
                    return;
                } else {
                    Console.WriteLine("Success to open " + port_name);
                }
            } catch (Exception e) {
                serial_port.Dispose();
                Console.WriteLine(e.Message);
            }
        }

        public void CloseSerial()
        {
            serial_buffer = "";
            serial_port.Dispose();
            Console.WriteLine("Close port: " + port_name);
        }

        public string ReadLines()
        {
            try {
                string s = serial_port.ReadExisting();
                string buffer = "";
                foreach (char c in s) {
                    buffer += c;
                    if (c == '\n') {
                        serial_buffer += buffer;
                        buffer = "";
                    }
                }
                string ret = serial_buffer;
                serial_buffer = buffer;
                if (ret.Length > 0 && ret[ret.Length - 1] == '\n')
                    return ret;
            } catch (Exception e) {
                Console.WriteLine(e.Message);
                return "";
            }
            return "";
        }

        public void Send(string s) {
            try {
                serial_port.Write(s);
            } catch (Exception e) {
                Console.WriteLine(e.Message);
            }
        }

        public void ClearBuffer()
        {
            serial_port.DiscardInBuffer();
        }
    }
}
```

|Method |功能 |
|-------|-----|
|Uart() | 初始化 Uart |
|Uart(string PortName, int BaudRate) | 初始化並自訂連接埠的 PortName 與 BaudRate|
|void OpenSerial() | 打開連接埠|
|void CloseSerial()| 關閉連接埠|
|string ReadLine() | 若讀取到 '\n' 就回傳當前的字串，否則回傳 ""|
|void Send(string s) | 透過 uart 傳入字串|
|void ClearBuffer() | 清空 buffer|

## 測試 Read

複製以下程式碼到主程式 Program.cs：

```cpp
using System;
using System.Threading;

namespace Uart_Console_App
{
    class Program
    {
        static void Main(string[] args)
        {
            Uart u = new Uart("COM4", 9600);
            u.OpenSerial();
            
            for (int i = 0; i < 10; i++) {
                string line = u.ReadLines();
                if (line != "") {
                    Console.Write(line);
                }
                Thread.Sleep(1000);
            }

            u.CloseSerial();
        }
    }
}
```

如此一來就能重複 10 次每隔 1 秒印出以 `"\n"` 結尾的字串。

```non
Success to open COM4
2,3,53
3,3,53
3,3,53
3,3,52
...
```

## 測試 Write

複製以下程式碼到主程式 Program.cs：

```cpp
using System;
using System.Threading;

namespace Uart_Console_App
{
    class Program
    {
        static void Main(string[] args)
        {
            Uart u = new Uart("COM4", 9600);
            u.OpenSerial();
            u.Send("Hello\n");
            Thread.Sleep(1000);
            string line = u.ReadLines();
            if (line != "")
            {
                Console.Write(line);
            }
            u.CloseSerial();
        }
    }
}
```

並且改寫你的開發版的 Uart 程式，使其收到一個字元就傳送一個相同的字元給電腦。

預期輸出：

```non
Success to open COM4
Hello
Close port: COM4     
```
