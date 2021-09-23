---
title: "C# Asynchronous Socket"
subtitle: ""
excerpt: "c# csharp socket"
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - csharp
---

C# 傳統的 Socket 會在斷線時讓系統卡住十幾秒，就算用 try catch 機制也無法避免這個問題。而非同步連線內部的機制就不會讓系統卡住，以下是非同步連線寫出的函式。

```c#
// Asynchronous Socket
public Socket AsyncConnect(string ip, string port) {
    Socket socket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
    IAsyncResult result = socket.BeginConnect(ip, Int32.Parse(port), null, null);
    result.AsyncWaitHandle.WaitOne(500, true); // Timeout 500 ms
    if (socket.Connected) {
        socket.EndConnect(result);
        return socket;
    } else {
        socket.Close();
        return null;
    }
}
```

以下是調用方式，可以用 `netcat` 作為 server 測試。

```c#
public int SomeFunction() {
    string IP = "127.0.0.1";
    int PORT = 6666;
    int BUFFERSIZE = 1024;

    try {
        // Asynchronous Socket Here
        sock = AsyncConnect(IP, PORT.ToString());
        if (sock == null) {
            return -1;
        } else {
            // Send Data
            sock.Send(Encoding.ASCII.GetBytes("Hello World!"));
            // Receive Data
            byte[] buffer = new byte[BUFFERSIZE];
            int count = sock.Receive(buffer);
            string recv = Encoding.ASCII.GetString(buffer, 0, count - 1);
        }
        sock.Close();
        return 0;
    } catch (Exception ex) {
        if (sock != null)
            sock.Close();
    }
    return -1;
}
```