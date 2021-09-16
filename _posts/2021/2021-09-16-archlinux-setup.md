---
title: "Arch Linux 基本設定"
subtitle: ""
excerpt: "virtualBox arch linux 中文 亂碼"
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - linux
---

## 終端機字型

在 Arch 官網有一個 [Font packages](https://wiki.archlinux.org/title/fonts#Font_packages) 列表，從中找一個你喜歡的字型，透過 `pacman` 或 `yay` 下載，我下載 Terminus。

```non
$ sudo pacman -Syu terminus-font
```

然後在 /usr/share/kbd/consolefonts/ 就可以發現許多 ter 開頭的字型包，即為剛剛下載的 Terminus 字型。

```non
$ ls /usr/share/kbd/consolefonts/
```

透過 `setfont`，從剛剛 `ls` 列出的字型包中選擇一個來使用。

```non
$ setfont ter-d24b.psf.gz
```

設定完舒服的字體後，要創建一份設定檔，以便重新開機時系統自動套用字型。

```non
$ sudo vim /etc/vconsole.conf
```

輸入 FONT= 剛剛所套用的字型，然後保存退出就完成了。

```non
FONT=ter-d24b.psf.gz
```

## 安裝桌面

### xfce4

需安裝以下套件：

- **xorg:** 開源圖形化介面架構
- **xfce4:** xfce 核心套件
- **xfce4-goodies:** 美化 xfce 圖形介面的套件
- **lightdm:** light 顯示管理器，又稱登入管理員，用以取代終端機登入
- **lightdm-gtk-greeter:** 基於 lightdm 的登入畫面

```non
$ sudo pacman -Syu
$ sudo pacman -S xorg xfce4 xfce4-goodies lightdm lightdm-gtk-greeter
$ sudo systemctl enable lightdm
```

然後重新啟動電腦即可。

## 在 VirtualBox 中自動調整視窗大小、雙向剪貼簿

此方式須先安裝 xorg 圖形環境。

```non
$ sudo pacman -Syu
$ sudo pacman -S virtualbox-guest-utils
$ sudo systemctl enable vboxservice.service
```

然後重新啟動虛擬機，點選虛擬機上方的`檢視`->`自動調整客體顯示大小`，以及`裝置`->`共用剪貼簿`->`雙向`。

## 解決 Arch Linux 中文亂碼、安裝中文輸入法

Arch 安裝時系統預設沒有中文字體 (至少我安裝時依定會亂碼)，此時只要透過 `pacman` 安裝中文字體即可。[Arch 官網](https://wiki.archlinux.org/title/Localization_(%E6%AD%A3%E9%AB%94%E4%B8%AD%E6%96%87)/Traditional_Chinese_(%E6%AD%A3%E9%AB%94%E4%B8%AD%E6%96%87)#%E4%B8%AD%E6%96%87%E5%AD%97%E9%AB%94)列出若干個字體，我選擇下載 wqy-microhei。

```non
$ sudo pacman -S wqy-microhei
```

透過 `yay` 安裝 hime。

```non
$ yay -S hime-git
```

編輯設定檔。

```non
$ vim ~/.xprofile
```

鍵入以下設置。

```non
export XIM_PROGRAM=hime
export XIM=hime
export GTK\_IM\_MODULE=hime
export QT\_IM\_MODULE=hime
export XMODIFIERS=@im=hime
hime &
```

## 好用的軟體

### yay

```non
$ sudo pacman -S --needed base-devel git
$ git clone https://aur.archlinux.org/yay-git.git
$ cd yay
$ sudo makepkg -si
```

### chrome

```non
$ yay -S google-chrome
```