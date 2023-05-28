---
title: "透過 date 取得 unix timestamp"
subtitle: ""
excerpt: "date timestamp 10-digit 13-digit"
layout: post
author: "blueskyson"
header-style: text
tags:
  - linux
---

常見的時間戳記形式是 10 位數，例如 Unix 時間戳記，它表示自 1970 年 1 月 1 日 00:00:00 以來的秒數。而 13 位數時間戳記則是在10位數時間戳記的基礎上，增加了毫秒級別的精確度。

在 Linux 終端機可以使用 `date` 指令來獲取現在時間的 timestamp：

```non
$ date +%s
1685251513
```

要獲得指定時間的 timestamp，您可以使用 `-d` 參數：

```non
$ date -d "2023-05-24 12:34:56" +%s
1684902896
```

獲取現在時間的 13-digit timestamp：

```non
$ date +%s%3N
1685251710582
```

獲得指定時間的 13-digit timestamp：

```non
$ date -d "2023-05-24 12:34:56" +%s%3N
1684902896000
```