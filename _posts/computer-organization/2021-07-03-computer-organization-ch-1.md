---
title: "計組 Chapter 1"
subtitle: "Abstractions and Technology"
excerpt: "computer organization 計組"
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - note
  - computer organization
---

## Moore's Law

每經過 18 至 24 個月，同等體積的晶片可以容納兩倍數數量的電晶體 (指數成長)。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/note/co/1.png)

## 8 個重要的想法

![](https://raw.githubusercontent.com/blueskyson/image-host/master/note/co/2.png)

- **Design for Moore's Law**  
  由於晶片設計可能需要數年時間，每顆晶片的可用資源很容易在研發完成時翻倍，架構師必須預測當設計完成時，最先進的技術將在落哪裡，而不是固守剛開始設計時的技術程度。但近幾年摩爾定律遇到瓶頸，可能不那麼適用了。
- **Use Abstraction to Simplify Design**  
  隱藏底層的細節以在高階工具提供更簡單的介面給工程師使用，也就是底層工程師專心在處理底層優化的問題，高階工程師利用介面開發大型、複雜的電路或演算法。
- **Make the Common Case Fast**  
  常見的 case 因為出現頻率高，優化後所帶來的效益是比優化 rare case 倍數增加的，所以我們應該優先優化 common case。
- **Performance via Parallelism**  
  平行運算是很常見的優化方式
- **Performance via Pipeline**  
  流水線是很常使用的優化法，核心概念是將一個大動作拆分成一個一個小步驟，並且一個指令可以在流水線的每一步停駐一陣子，而流水線上的每個步驟可以同時執行，就像製造廠的生產線一樣。
- **Performance via Prediction**  
  預測下一個指令，提早載入下一個資源，可以優化迴圈的執行效率。
- **Hierarchy of Memories**  
  越快的儲存體通常越貴，若整部電腦都用最貴的儲存體，價格會讓一班企業和民眾負荷不了，所以我們將儲存體分層，最貴的作為快取和記憶體，便宜的作為硬碟。執行指令時，將所需資源拉進快取以加速存取，提升效能；不需要該資源就將其存入硬碟永久保存。
- **Dependability via Redundancy**  
  任何物理設備都可能故障，我們需要設計冗餘的組件，確保故障發生時，電腦可以保持最低限度的運轉，就像大型拖車爆胎時，司機可以藉由備用輪胎，趕緊開去維修站修理。

## Below Your Program

電腦程式執行的的流程大致為:  
1. 應用軟體 (以高階語言撰寫)
2. 系統軟體 (包含作業系統、編譯器)
3. 硬體 (處理器、記憶體、輸入輸出裝置)

程式語言區分為:
1. 高階語言 (c, c++, java, python, ... 便於理解、功能強大的語言)
2. 組合語言 (mips, x86, arm, ... 將指令翻譯為人可以理解的語言)
3. 機器語言 (由 0 和 1 組成，只有電腦讀得懂的語言)

## 晶片生產過程

![](https://raw.githubusercontent.com/blueskyson/image-host/master/note/co/3.png)

將矽濃縮提煉成柱狀的**矽晶棒**，接下來將矽晶棒切片變成**矽晶圓**，塗上各種材料後，在上方放**光罩**進行光刻，然後經過測試、分類、封裝便成為電腦裡的晶片。