---
title: "Xfce 基本設定"
subtitle: ""
excerpt: "xfce 新酷音 觸控板"
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - linux
---

### 新酷音輸入法

透過 `pacman` 或 `apt` 安裝 ibus-chewing。

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

### 顯示亮度、電量

```non
$ sudo apt-get install xfce4-power-manager
```

重新開機後右鍵點選上方的工作列，點選 `panel preference`，在 `item` 頁籤加入 `Power Manager Plugin`。

### 筆電觸控板手勢

在 `Application Finder` 開啟 `Mouse and Touchpad`，點開 `Device` 下拉選單，選擇名稱中有 `Touchpad` 的裝置，勾選 `Reverse scroll direction`，這樣觸控手勢便會與我們習慣的觸控裝置一樣。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/touchpad1.png)

另外也可以開啟輕觸點擊。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/touchpad2.png)