---
title: "Oracle Cloud Ubuntu 開放 Port 後仍無法連線：iptables 設定"
subtitle: ""
excerpt: "oracle cloud oci iptables port"
layout: post
author: "blueskyson"
header-style: text
tags:
  - linux
---

在 Oracle Cloud Infrastructure (OCI) 上跑 Ubuntu VM 時，開放特定 Port 的流程看起來很直覺：

1. 進入 Instance details
2. 點選 **Virtual cloud network**
3. 進入 **Security → Default Security List for ...**
4. 在 **Ingress Rules** 新增對應的規則

照著做完之後，卻發現從外部還是連不進來。原因是 OCI 的 Ubuntu image 預設啟用了 `iptables`，會在 OS 層擋掉所有不在白名單內的 inbound 流量，即使 VCN Security List 已經放行，封包還是會被 VM 內部的防火牆丟棄。

## 解法：修改 iptables 規則

SSH 進入 VM 後，編輯 iptables 規則檔：

```bash
sudo vim /etc/iptables/rules.v4
```

加入想要開放的 Port，例如 HTTP (80) 與 HTTPS (443)：

```
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT
```

存檔後套用規則：

```bash
sudo iptables-restore < /etc/iptables/rules.v4
```

套用後再從外部 curl 或 telnet 測試連線是否通了。

參考影片：[YouTube - Oracle Cloud iptables fix](https://www.youtube.com/watch?v=loOzM3M_jF4)
