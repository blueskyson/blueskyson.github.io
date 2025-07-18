---
title: "在 Ubuntu 24 編譯精簡 Linux Kernel"
subtitle: ""
excerpt: "ubuntu 24 linux kernel"
layout: post
author: "blueskyson"
header-style: text
tags:
  - linux
---

本篇文章記錄我在 Ubuntu 24.04 機器上手動下載、設定並編譯最新 Linux kernel v6.12 的過程。

## 安裝必要工具

Linux Kernel 的編譯需要一些開發工具與函式庫。使用 apt 套件管理器安裝如下：

```bash
sudo apt update
sudo apt install -y git build-essential flex bison bc libncurses-dev libssl-dev libelf-dev
````

各套件用途簡述：

- `build-essential`: 包含 gcc、make 等編譯工具
- `flex`, `bison`: 編譯設定檔選單用
- `bc`: 為 Makefile 中的運算工具
- `libncurses-dev`: 提供 make menuconfig TUI 選單介面
- `libssl-dev`, `libelf-dev`: 某些模組需要

## 下載 Linux Kernel 原始碼

我選擇主線版本 v6.12。以下指令可抓取該版本原始碼：

```bash
git clone --depth=1 --branch=v6.12 git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
cd linux
```

使用 `--depth=1` 可加快下載速度，只取最新提交。

## 使用 tinyconfig 產生最小設定檔

選擇 `tinyconfig` 快速產出一個極簡核心：

```bash
make tinyconfig
```

這會產生 `.config` 設定檔，啟用最小必要功能。

## 編譯 Linux Kernel

開始編譯核心：

```bash
make -j$(nproc)
```

`$(nproc)` 表示使用所有 CPU 核心以加快速度。

成功後，會產出映像檔：

```non
./arch/x86/boot/bzImage
```

`bzImage` 是可開機的 Linux kernel 映像，稱為 "big zImage"。它是壓縮過的自解壓格式，主要用於 x86/x86\_64 架構。

你可以這樣查看格式：

```bash
file arch/x86/boot/bzImage
```

## 用 QEMU 測試啟動核心

安裝 QEMU：

```bash
sudo apt install qemu-system-x86
```

啟動核心：

```bash
qemu-system-x86_64 -kernel arch/x86/boot/bzImage
```

但讓人失望的是，畫面上只顯示了：`Booting from ROM...`。然後就沒有後續了，完全沒有看到任何核心訊息。這通常代表我們的核心缺少基本的輸出驅動，沒辦法輸出文字到畫面。

> 注意：點進 QEMU 視窗後滑鼠會被鎖住，可按 `Ctrl + Alt + G` 放開。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2025/boot-tiny-linux-1.png)

## 設定 menuconfig

關閉 QEMU 視窗後，打開 `menuconfig` 來調整設定（如果是第一次使用，請先安裝 `libncurses-dev`）：

```bash
make menuconfig
```

**啟用 TTY 裝置**

在選單中找到：

```non
Device Drivers  --->
  Character devices  --->
    <*> Enable TTY
```

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2025/boot-tiny-linux-2.png)

> 按 menu 上方說明，使用方向鍵選擇，按 Y 或空白鍵啟用設定。

此選項允許核心支援 TTY 裝置（文字終端或序列埠輸出）。

**啟用 printk**

```non
General setup  --->
  Configure standard kernel features (expert users)  --->
    [*] Enable support for printk
```

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2025/boot-tiny-linux-3.png)

儲存設定後，再次重新編譯執行：

```bash
make -j$(nproc)
qemu-system-x86_64 -kernel arch/x86/boot/bzImage
```

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2025/boot-tiny-linux-4.png)

終端機中出現了核心啟動訊息，雖然最後顯示 `panic - not syncing: No working init found`，但至少我們已經成功啟動了核心並看到輸出。

要解決這個問題，你需要提供一個有效的 `init` 程序（如 `systemd` 或 `busybox`）。可參考另一篇文章 [靜態編譯 C 語言 init 程式，搭配 initramfs 啟動自編譯 Linux Kernel](/2025/07/13/tiny-linux-init)。

如果你想恢復到預設的 `tinyconfig` 設定來做其他實驗，可以使用：

```bash
make mrproper
```

## 參考資料

- [Building a tiny Linux from scratch](https://blinry.org/tiny-linux/)