---
title: "Debian 11 筆記"
subtitle: ""
excerpt: ""
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - linux
---

## 將使用者加入 sudoer

剛安裝好 Debian 時，一定會遇到下 `sudo apt ...` 卻被告知權限不足的問題，此時必須手動修改 `/etc/sudoers` 將目前的使用者 (lin) 加入 sudoer。

```non
$ su -
# gpasswd -a lin sudo
# cd /usr/sbin
# ./visudo
```

此時終端機會用 `nano` 開啟 `sudoer.tmp`，將下區塊添加一行 `lin` 的設定：

```bash
# User privilege specification
root  ALL=(ALL:ALL) ALL
lin   ALL=(ALL:ALL) ALL
```

如此便成功將系統權限給予 `lin`。

參考：[How To Add a User to Sudoers On Debian 10 Buster](https://devconnected.com/how-to-add-a-user-to-sudoers-on-debian-10-buster/)

## 安裝注音輸入法

```non
$ sudo apt install ibus-chewing
```

重新啟動系統，到設定裡的 `Region & Language` 的 `Input Source`，新增 `Chinese(chewing)`。

## 在 VituralBox 中設置全螢幕、雙向剪貼

安裝必要的模組

```non
$ sudo apt install build-essential module-assistant dkms
$ sudo m-a prepare
```

接著點選虛擬機視窗上方`裝置`的`插入 Guest Addition CD 映像`，然後透過檔案總管進入 Guest Addition CD 的目錄（通常為 `/media/cdrom0`），執行安裝腳本。

```
$ cd /media/cdrom0
$ sudo sh VBoxLinuxAdditions.run
```

接著重新開機登入，點選虛擬機視窗上方`檢視`的`自動調整客體顯示大小`、點選虛擬機視窗上方`裝置`的`共用剪貼簿`。

參考：[How to Install VirtualBox Guest Additions in Debian 9 Virtual Machine](https://www.linuxbabe.com/debian/install-virtualbox-guest-additions-debian-9-stretch)

## 心得

Debian 預設似乎不支援桌面捷徑，就算把檔案、目錄放在 `~/Desktop` 或是使用 `.desktop` 作為捷徑，也不會顯示在桌面上，當然在桌面按下右鍵也不會有 `Open in terminal` 選項、沒有 `Ctrl+Alt+T` 快捷鍵、另外也沒有常駐的工具列，只能點選桌面左上方的 `Activity` 叫出工具列和所有應用程式，這幾項是跟 Windows 或 Ubuntu 使用邏輯差異最大的地方。Debian 使用的資源較少，閒置時需要消耗約 930 MiB 記憶體，Ubuntu 則需要消耗 1.5 GiB 左右。