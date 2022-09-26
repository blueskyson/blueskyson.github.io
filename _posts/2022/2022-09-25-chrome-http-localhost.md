---
title: "Chrome 無法開啟 localhost 的解法"
subtitle: ""
excerpt: "chrome localhost http"
layout: post
author: "blueskyson"
header-style: text
tags:
  - others
---

在學習 vue.js 時遇到了 Chrome 打不開 localhost:8080 的問題，但是用 Edge 卻能夠打開 localhost:8080。實際原因是 Chrome 強制將 http 網站導向 https，遇到這個狀況時，需要到 `chrome://net-internals/#hsts` 將 localhost 的安全設定拿掉。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/chrome-hsts.png)

然後重新打開 Chrome 就 OK 了。