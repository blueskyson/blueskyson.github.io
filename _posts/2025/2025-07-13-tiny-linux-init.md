---
title: "靜態編譯 C 語言 init 程式，搭配 initramfs 啟動自編譯 Linux Kernel"
subtitle: ""
excerpt: "ubuntu 24 linux kernel"
layout: post
author: "blueskyson"
header-style: text
tags:
  - linux
---

本篇文章記錄我在 Ubuntu 24.04 機器上撰寫編譯 init 程式、製作 initramfs 並使用 QEMU 搭配 bzImage 啟動 Linux kernel v6.12。

請先參考 [在 Ubuntu 24 編譯精簡 Linux Kernel](/2025/07/09/boot-tiiny-linux) 文章，完成 Linux Kernel 的編譯。

## 建立 C 專案結構

```bash
mkdir -p my-init/bin
cd my-init
touch init.c
```

編輯 `init.c`：

```c
// init.c
#include <stdio.h>
#include <unistd.h>

int main() {
    while (1) {
        printf("$ ");
        fflush(stdout);

        char input[256];
        if (fgets(input, sizeof(input), stdin) == NULL) break;

        printf("Sorry, I don't know how to do that.\n");
    }
    return 0;
}
```

把所有需要的函式庫一起打包進 ELF 可執行檔，這樣 kernel 不需要提供任何外部環境就能執行它。

```bash
gcc init.c -static -o bin/init
```

## 建立 initramfs 映像

```bash
cd bin
chmod +x init
cd ..
find . -print0 | cpio --null --create --verbose --format=newc | gzip -9 > ../initrd
```

這會產生一個 `initrd`，是 gzip 壓縮過的 `cpio` 檔案，裡面包含你剛剛寫的 `init` 可執行檔。

## 啟用 Kernel 支援

```bash
make menuconfig
```

開啟這幾個選項：

1. General setup ---> Initial RAM filesystem and RAM disk (initramfs/initrd) support
2. Executable file formats ---> Kernel support for ELF binaries
3. Processor type and features ---> 64-bit kernel

再編譯一次。

## 使用 QEMU 啟動

```bash
qemu-system-x86_64 -kernel arch/x86/boot/bzImage -initrd initrd
```

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2025/tiny-linux-init-1.png)

預期輸出：

```non
$ Sorry, I don't know how to do that.
$ Sorry, I don't know how to do that.
...
```

每次輸入都會回應這句話，因為這就是你設計的最小的 shell。

## 參考資料

- [Building a tiny Linux from scratch](https://blinry.org/tiny-linux/)