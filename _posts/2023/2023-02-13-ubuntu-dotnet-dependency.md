---
title: "Ubuntu 上 Microsoft PPA 與 apt 的 dotnet 衝突的解法"
subtitle: ""
excerpt: "ubuntu apt dotnet fxr"
layout: post
author: "blueskyson"
header-style: text
tags:
  - csharp
---

因為最近 dotnet 被加入了 apt 官方的 PPA 中，如果在先前在 Ubuntu 上安裝了 Microsoft PPA 的 dotnet，就會發生衝突：

`A fatal error occurred. The folder [/usr/share/dotnet/host/fxr] does not exist`

此時先刪除所有 dotnet 相關的套件：

```non
$ sudo apt remove dotnet*
$ sudo apt remove aspnetcore*
$ sudo apt remove netstandard*
```

然後在 `/etc/apt/preferences.d` 新增一個 `99microsoft-dotnet.pref` 指名優先使用 Microsoft PPA：

```non
$ sudo vim /etc/apt/preferences.d/99microsoft-dotnet.pref
```

填入以下設定值：

```
Package: *
Pin: origin "packages.microsoft.com"
Pin-Priority: 1001
```

最後重新從 Microsoft PPA 安裝 dotnet 即可：

```non
$ sudo apt update
$ sudo apt install dotnet-sdk-6.0
```