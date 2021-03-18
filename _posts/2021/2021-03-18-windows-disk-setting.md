---
title: "Windows EFI、ESP 磁區操作"
subtitle: ""
excerpt: "EFI ESP"
layout: post
author: "blueskyson"
header-style: text
tags:
  - others
---

在刪除 linux 雙系統後，常常會在硬碟中殘留 Windows 用不到的 EFI 分割區，可以用以下方式將其清除。

## 刪除 EFI 磁區

首先執行 `C:\Windows\System32\diskpart.exe`

```
DISKPART> list disk
  磁碟 ###  狀態           大小     可用     Dyn  Gpt
  --------  -------------  -------  -------  ---  ---
* 磁碟 0    連線              476 GB      0 B        *
  磁碟 1    連線              476 GB   236 GB

DISKPART> select disk 1

磁碟 1 是所選擇的磁碟。

DISKPART> list partition

  磁碟分割  ###  類型              大小     位移
  -------------  ----------------  -------  -------
  磁碟分割  1    主要                 240 GB  1024 KB
  磁碟分割  2    系統                 512 MB   240 GB
  磁碟分割  0    擴充                 236 GB   240 GB
```

其中`磁碟分割  2`是我想要刪除的 EFI 分割

```
DISKPART> select partition 2

磁碟分割 2 是所選擇的磁碟分割。

DISKPART> delete partition

DiskPart 成功地刪除了選擇的磁碟分割。

DISKPART> list partition

  磁碟分割  ###  類型              大小     位移
  -------------  ----------------  -------  -------
  磁碟分割  1    主要                 240 GB  1024 KB
  磁碟分割  0    擴充                 236 GB   240 GB

```

最後使用磁碟管理員掛載或延伸這塊磁區就行了。

安裝過雙系統後，通常原生的 `ESP` 會被複寫，以利 `grub` 去抓取兩個系統的 loader 供使用者選擇，這個磁區通常在 C 槽的最前面，不可隨意刪除。接下來是如何讓 Windows 掛載 `ESP` 磁區。

## 掛載 ESP 磁區

首先執行 `C:\Windows\System32\diskpart.exe`

```
DISKPART> select disk 0

磁碟 0 是所選擇的磁碟。

DISKPART> list disk

  磁碟 ###  狀態           大小     可用     Dyn  Gpt
  --------  -------------  -------  -------  ---  ---
* 磁碟 0    連線              476 GB      0 B        *
  磁碟 1    連線              476 GB   236 GB

DISKPART> list partition

  磁碟分割  ###  類型              大小     位移
  -------------  ----------------  -------  -------
  磁碟分割  1    系統                 200 MB  1024 KB
  磁碟分割  2    保留                  16 MB   201 MB
  磁碟分割  3    主要                 237 GB   217 MB
  磁碟分割  4    復原                1024 MB   238 GB
  磁碟分割  5    主要                 237 GB   239 GB
```

其中`磁碟分割  1`是 ESP 磁區

```
DISKPART> select partition 1

磁碟分割 1 是所選擇的磁碟分割。

DISKPART> assign

DiskPart 成功地指派了磁碟機代號或掛接點。
```

掛載後，使用系統管理員身分執行`Explorer++`就能檢視 ESP 分割區了