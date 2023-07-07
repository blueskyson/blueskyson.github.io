---
title: "透過 Docker 容器在 Ubuntu 22.04 開發 .NET 3.1"
subtitle: ""
excerpt: ".NET dotnet core 3.1"
layout: post
author: "blueskyson"
header-style: text
tags:
  - csharp
  - docker
  - linux
---

Ubuntu 22.04 僅支持 OpenSSL 3，使用者只能安裝 .NET 6.0，它不支持.NET CORE 3.1 或 2.0。

在新版 Ubuntu 看似無法開發 .NET 3.1，但我想到既然 Docker 可以正常在 Ubuntu 22.04 執行 .NET 3.1 的容器，那我應該也可以透過 Docker 容器來開發 .NET 3.1 的應用程式。幸好微軟官方真的有提供開發用的映像檔，我們甚至不需要自己撰寫 Dockerfile 來設定環境。

## 啟動環境

以下是在本地使用 `devcontainers/dotnet:dev-3.1` 啟動 `dotnet31` 容器的指令：

```non
$ mkdir -p dev-container/dotnet31
$ cd dev-container
$ docker pull mcr.microsoft.com/devcontainers/dotnet:dev-3.1
$ docker run -it --name dotnet31 --volume ./dotnet31:/workspace:cached --network host mcr.microsoft.com/devcontainers/dotnet:dev-3.1 sleep infinity
```

進入容器測試能否編譯、執行程式：

```non
$ docker exec -it dotnet31 bash
# dotnet --list-sdks
3.1.425 [/usr/share/dotnet/sdk]
# mkdir /workspace/test && cd /workspace/test
# dotnet new console
# dotnet run
Hello World!
```

接著使用你喜歡的 IDE 或編輯器進入容器中開發，我個人是使用 VS Code 的 Dev Containers。

## 安裝套件

如果是用 `vscode` 使用者登入的話，需要初始化 root 密碼：

```non
vscode ➜ / $ sudo passwd root
```

接下來透過 `apt` 安裝你想裝的工具

```non
vscode ➜ / $ sudo apt install neofetch
```