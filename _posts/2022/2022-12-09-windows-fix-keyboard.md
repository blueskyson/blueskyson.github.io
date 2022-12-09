---
title: "Windows 鍵盤鎖住，打不出字的解法"
subtitle: ""
excerpt: "windows keyboard"
layout: post
author: "blueskyson"
header-style: text
tags:
  - windows
  - others
---

在工具列右下方叫出螢幕小鍵盤，從 `開始` 中透過螢幕小鍵盤搜尋 `cmd`，然後右鍵點擊以系統管理員身分執行命令提示字元，然後透過以下指令重設網路：

```non
sc config i8042prt start= auto
```

接著重新開機，應該有機會解決。