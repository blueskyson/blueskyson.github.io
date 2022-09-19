---
title: "雙系統 Windows 11 wifi 無法使用解決辦法"
subtitle: ""
excerpt: "wifi windows 11 ubuntu 22"
layout: post
author: "blueskyson"
header-style: text
tags:
  - windows
  - others
---

在筆電安裝 Ubuntu 22.04 之後，回到 Windows 11 發現 wifi 顯示「無法使用」，此時以系統管理員身分執行命令提示字元，然後透過以下指令重設網路：

```non
> netsh winsock reset
> netsh int ip reset
```

重新開機後應該就可以使用 wifi 了。