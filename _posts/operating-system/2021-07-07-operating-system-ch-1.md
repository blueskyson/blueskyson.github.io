---
title: "OS Chapter 1"
subtitle: "Introduction"
excerpt: "linear algebra 線代"
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - note
  - os
---

目前普遍的電腦架構如下圖

![](https://raw.githubusercontent.com/blueskyson/image-host/master/note/os/1.png)

CPU 不斷地從記憶體吞吐指令，根據指令操作各種 device，device 跳過 CPU，透過 DMA (Direct Memory Access) 主動向記憶體存取資料，增加電腦效率。CPU、記憶體、devices 互相協同工作下，形成了我們日常使用的電腦系統。

## 多處理器系統 (Multiprocessor systems, parallel systems, multicore systems)

在過去幾年，多處理器系統主導電腦領域，此類系統具有多個緊密通信的處理器，共享 bus，有時還共享 clock、記憶體和外圍設備。 多處理器系統首先出現在伺服器，後來遷移到個人電腦、手機、平板電腦、移動設備等。

優點: 增加吞吐量、將低成本 (相較於多個單核晶片)、提高可靠性 (一個核心故障不會使系統完全停止)

- **Asymmetric multiprocessing**  
  非對稱多處理系統有一個 boss processor 負責分配工作給其他處理器，是 boss–worker 關係。
- **Symmetric multiprocessing (SMP)**  
  對稱多處理器是最常見的架構，所有處理器都是對等的，作業系統負責分配工作給所有處理器，每個處理器有自己的 register 和 cache。實作多處理器系統需要精確控制 I/O 確保資料進入正確的處理器，再來要做好負載平衡，盡量避免一個處理器很忙、其他處理器閒置的狀況。幾乎所有現代作業系統（包括 Windows、Mac OS X 和 Linux）都支援 SMP。

CPU 設計的趨勢是在單個晶片上包含多個計算核心。這種多處理器系統稱為多核。它們比具有單核的多個晶片更高效，因為晶片內通信比晶片間通信更快，與多個單核晶片相比，一個多核晶片的功耗要低得多。

## 電腦叢集 (Clustered System)

另一種類型的多處理器系統是集群系統，由兩個或多個連接在一起的單獨系統（節點）組成，每個節點可以是單處理器系統或多核系統。需要注意的是，叢集的定義並不具體，普遍接受的定義是共享存儲並通過局域網或更快的互連（如 InfiniBand）緊密連接。叢集系統通是高可用性的，叢集裡的節點故障了還是能持續運行。

- **Asymmetric clustering**  
  boss–worker 關係，一個節點處於 hot-standby 作為監督伺服器，負責負仔平衡、分配工作給其他節點，其他節點跑應用程式、容器之類的。
- **Symmetric clustering**  
  所有節點都是 worker，都跑應用程式

## 作業系統

作業系統最重要的特色就是處理多個程式，在適當的時機切換程式運作，讓 CPU 不會閒置。作業系統最主要的目標是讓電腦變得更**方便使用**以及讓電腦**更有效率**，然而這兩個目標往往是互相衝突的，比如進行影片輸出時，我們會希望把資源盡可能用來輸出，但是為了最低限度的操作，作業系統仍然要分配一些資源給鍵盤滑鼠，讓電腦保持可以被操作的狀態，不至於完全卡死。

- **分時系統 (Time-Sharing System)**  
  把電腦執行時間分為多個時間段，並且將這些時間段平均分配給各個程式，輪流執行每個程式一定時間，如此迴圈，直至完成所有任務。此類型作業系統通常有 **job scheduler** 與 **cpu scheduler** 決定每個時間段要把哪個程式分配給哪個 cpu 處理。
- **即時系統 (Real-Time System)**
  如果有一個程式需要執行，即時作業系統會馬上執行該程式，不會有較長的延時，這種特性保證了各個任務的及時執行，通常用於軍事防衛、工廠自動化生產系統。

## Dual-Mode

有些處理器和作業系統會區分 **user-mode** 與 **kernel-mode** (privileged mode)，當程式想要做需要權限的系統操作 **system-call** (例如讀寫檔案) 時，系統會先核對 user 的身分，如果所有條件都 OK，作業系統會先將程式中斷，然後進入 kernal-mode，讓 kernel 做出相應的操作然後回傳 system call，再將程式回復 user-mode。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/note/os/2.png)

## 其他

作業系統需要處理的其他功能包括記**憶體管理**、**檔案系統**、**資訊安全**，這些都會在後面章節比較詳細的介紹。

- **分散式系統 (Distributed Systems)**  
  操作一個叢集就像操作一台電腦一樣
- **Peer-to-Peer Computing**  
  網路上每個節點都提供自己有的資訊，最後拼湊成一份完整的檔案，經典例子如: bittorrent、skype。現在因網路速度快、頻寬高，較不流行 p2p 架構。
- **Virtualization**  
  將作業系統虛擬化，透過安裝多個虛擬機，可以更有效率分配伺服器的資源。近年更發展容器化，將作業系統更輕量化，是現在的趨勢。