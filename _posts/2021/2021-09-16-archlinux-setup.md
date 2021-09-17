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

```
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

### Hime 輸入法

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

重新登入即可。

### 新酷音輸入法

注意以下方法只有在 xfce 測試過，如果桌面環境是 gnome 或 KDE，不保證以下方法會成功。

透過 `pacman` 安裝 ibus-chewing。

```non
$ sudo pacman -S ibus ibus-chewing libibus libchewing
```

新增一個腳本放在 /etc/profile.d

```non
$ sudo vim /etc/profile.d/ibus.sh
```

輸入以下內容以便開機時自動啟用 ibus：

```
GTK_IM_MODULE=ibus
QT_IM_MODULE=ibus
XMODIFIERS="@im=ibus"
ibus-daemon -drx
```

接著重新開機後，在 xfce 的 `Application Finder` 裡找到 `IBus Preferences` 並雙擊打開，選擇 `Input Method` -> `Add` -> `Chinese` -> `Chewing`，即可完成設定。

## 安裝聲音軟體

最常用的聲音控制軟體應屬 alsa:

```non
$ sudo pacman -S alsa alsa-util
```

開啟混音裝置

```non
$ alsamixer
```

如果要一併啟用 xfce 上方 panel 的音量圖示，需要再執行以下指令：

```non
$ sudo pacman -S pulseaudio pavucontrol
$ pulseaudio --check
$ pulseaudio -D
```

## 啟動 wifi

最簡單的方法是安裝 networkmanager，但是注意要先 disable 目前正在運行的網路管理工具 （如果有的話）。

```non
$ sudo pacman -S networkmanager
$ sudo systemctl enable NetworkManager
$ sudo systemctl start NetworkManager
```

### 打開 wifi

啟動無險網卡：

```non
$ nmcli radio wifi on
```

可以透過 `ip a` 查看網卡狀態，通常無線網卡名稱為 wlan0 或 wlp2s0，狀態會顯示 `<NO-CARRIER,BROADCAST,MULTICAST,UP>>`。

```non
$ ip a
...
wlan0: <NO-CARRIER,BROADCAST,MULTICAST,UP>
...
```

狀態顯示沒有異常的話，就掃描附近的 wifi。

```non
$ nmcli device wifi list
IN-USE  BSSID              SSID           MODE   CHAN  RATE        SIGNAL  BARS  SECURITY  
        40:B0:76:B9:40:77  HSNU-AP        Infra  1     117 Mbit/s  99      ▂▄▆█  WPA2      
        FC:D7:33:01:89:E8  TP-LINK_89E8   Infra  6     270 Mbit/s  69      ▂▄▆_  WPA1 WPA2 
        64:09:80:4F:3F:8B  Xiaomi_3F8A    Infra  1     270 Mbit/s  35      ▂▄__  WPA1 WPA2 
        9A:96:B8:97:B9:0B  AndroidAPE409  Infra  11    130 Mbit/s  22      ▂___  WPA2      
```

以我的手機網路 HSNU-AP 為例，透過以下指令輸入密碼連線：

```non
$ nmcli device wifi connect HSNU-AP password abcd1234
```

檢查是否成功連線，若成功就會顯示類似以下的訊息。若不成功請另行尋找解決方法。

```non
$ nmcli connection
NAME                UUID                                  TYPE      DEVICE 
HSNU-AP             ff86f0fd-2449-4138-9f22-e814d931e424  wifi      wlan0  
Wired connection 1  f70dcdcf-010a-3c84-9484-3ae5e2ce99e1  ethernet  -- 
```

### 關閉 wifi

斷開連接並關閉搜尋附近網路：

```
$ nmcli connection down HSNU-AP
$ nmcli radio wifi off
```

## 好用的軟體

### yay

```non
$ sudo pacman -S --needed base-devel git
$ git clone https://aur.archlinux.org/yay-git.git
$ cd yay
$ makepkg -si
```

### autojump

```non
$ yay -S autojump
```

開啟 ~/.bashrc，加入這一行指令：

```
[[ -s /etc/profile.d/autojump.sh ]] && source /etc/profile.d/autojump.sh
```

### tldr

```non
$ yay -S tldr-cpp-git
```

### chrome

```non
$ yay -S google-chrome
```
