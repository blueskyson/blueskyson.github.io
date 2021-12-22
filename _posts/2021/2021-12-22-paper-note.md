---
title: "論文翻譯 A Dynamic Priority Assignment Technique for Streams with (m, k)-Firm Deadlines"
subtitle: ""
excerpt: "Dynamic Priority Assignment Technique (m, k)-Firm Deadlines"
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - real-time
---

## 摘要

本文將會解決 **scheduling multiple streams of real-time customers**。首先介紹 (m, k)-firm deadlines 的概念以表達對 real-time stream 計時的限制：任何 k 個連續 customer 中至少有 m 個必須滿足他們的 deadline，則稱此 stream 擁有 (m, k)-firm deadlines；如果少於 m 個 meet deadline，則此 (m, k)-firm deadline stream 會遭遇 dynamic failure。

接下來本論文提出一中基於 priority 的策略，用以在單個 server 上排程 N 個 real-time stream，並減少 dynamic failure 的機率。基本的想法是提高 stream 中故障率較高的 customers 的 priority ，以提高他們 meet deadline 的機會。本論文使用 heuristic 的方式分配優先級，並通過各種 customer arrival 與 service pattern 的模擬來評估，本論文的策略與傳統上所有 customer 都採用相同 priority 、以及用不精確的計算得出的模型進行比較，評估結果表明本論文的策略可以大幅降低 dynamic failure 的機率。

## I. 引言

一個 real-time application 由一組協作的 tasks 組成，這些 task 會頻繁的傳遞 message 來互動，所有 task 都期望在 deadline 前獲得完整的服務。實時應用的例子包括：程序控制、生命維持系統、自動製造系統、機器人、航天飛機和多媒體。在某些情況下，task 或 message 未能在 deadline 前完成會產生災難性後果，因此及時完成非常重要，此類任務被視為 hard deadline。相反的，有些實時應用不需要每個任務都 meet deadline，這類則被視為 soft deadline，如影音播放器。

傳統上我們用 maximum allowable loss percentage 來表達對於 miss deadline 的容忍度，例如在影片串流允許 10% 的丟失率。然而 maximum allowable loss percentage 有一個隱含的假設，就是這些 miss  deadline 的任務盡可能不連續，拿影片串流為例，如果連續好幾幀都丟失，就算整部影片的丟失率為 10%，影片品質還是會受影響。為了解決上述的弊端，我們可以用另一個方法來表達容忍度：在任意 k 個連續幀中至少有 m 個幀滿足它們的 deadline ，即 (m, k)-firm deadline。

本論文解決的問題是調度一組 N 個 real-time customer stream ，其中每個 stream 都有自己的 deadline ，以減少 dynamic failure 的機率，為了達成前述目標，我們給每個 customer 分配 priority 。最基本的想法為 distance-based priority (DBP)，根據最近 miss deadline 的歷史紀錄來分配 priority ，如果某個 stream 中的若干個 customer miss deadline，則此 stream 中的下一個 customer 將會被分配更高的 priority，server 會參考 priority 與其他參數進行排程。我們將 DBP 與 single priority (SP) 進行比較，即所有 customer 都是相同的 priority。結果表明使用 DBP，dynamic failure 的機率大幅降低。由於本論文使用 (m, k)-Firm 來評估，與以往的論文使用 maximum allowable loss percentage 不同，很難互相比較，因為後者容許大量連續的 miss deadline。

此外，(m, k)-firm 也可以套用在 imprecise computation 模型，在此類模型中，customer 被區分為強制性（mandatory）與選擇性（optional）執行，前者必須要被執行，後者則是在資源足夠的情況下被執行，並且後者可以優化前者計算的結果。我們可以規定：在任意連續 k 個 customer 中，至少有 m 個 mandatory 滿足他們的 deadline。雖然這種方法可以顯著降低動態失敗的機率，但代價是更高的 miss deadline 的總體機率。

本文的其餘部分安排如下，第二部分描述了系統模型和 (m, k)-firm 的概念，在第三節中我們描述基於距離分配 priority ，在第四節提出實驗評估的結果，第五節討論了以較少的 priority levels 實施的問題，第六節總結。 

## II. 系統模型與定義問題

考慮一個 real-time customer stream，其中 customer 可以是 task、message 或實時應用程式中的任何其他可調度實體。stream 中的每個 customer 都有一個 deadline，它期望系統在 deadline 內提供完整的服務。(m, k)-firm 的定義如引言所講的，就不多做翻譯。

傳統上期望 stream 裡的每一個 customer 都 meet deadline，可以標記為 (1, 1)-firm deadlines；而 (1, 2)-firm 則表示不可以連續兩個 customer  miss  deadline；同樣的 (4, 6)-firm 表示不可以連續三個 customer  miss  deadline，並且在任連續 6 個 customer 中必有 4 個 meet deadline。我們也可以注意到 (m, k)-firm 有 (k - m)/k 的 maximum allowable loss percentage ，例如 (4, 5)-firm 和 (8, 10)-firm 的 maximum allowable loss percentage 同為 20% (其中 (4, 5)-firm 比 (8, 10)-firm 更為嚴格)。

考慮一個由 N 個 stream 組成的系統 $R_1, R_2, ..., R_N$，其中 $R_j$ 有 $(m_j, k_j)\rm{-firm}\ deadline$，本文解決的問題就是將 customer 調度在單個 server 上，以降低平均 dynamic failure 機率，最直接的方法就是降低 customer miss deadline 的機率。

過往的排程模型大致如下：每個 stream 各自擁有 first-in first-out queue，這些 queue 保證 customer 按照到達的先後順序執行，另外會有一個 service queue 從這些 stream queue 的端點中選擇 customer 執行，如下圖：

![](https://i.imgur.com/Yo7Nrvs.png)

第一種基於上述模型的排程方法為 first-in first-out policy：service queue 從所有 stream queue 的端點中選擇**最早到達**的 customer 執行，此方法不管每個 customer 的 deadline 。另一種排程方法為 earliest-deadline-first policy：service queue 選擇**離 deadline 最近**的 customer 先執行。

本論文提出的方法也可以應用在上述模型，為每個 stream queue 端點的 customer 分配 priority ，service queue 會放入 priority 高的 customer。如果 priority 都一樣高，則改用前面提到的兩種 policy 排程。因為過往的研究表示 earliest-deadline-first policy 表現比較好，所以本篇論文提出的方法在 priority 一樣高時會使用 earliest-deadline-first policy。

在進入 III 部分前，還有一個概念需要提點一下：在某些系統中，正在等待的 customer 是否已經 miss deadline 是可以被偵測的，某些應用程式會直接跳過這些遲遲未執行的 customer，又稱為 dropped customer；相反地，如果系統無法偵測 customer 是否已經 miss deadline ，則每個 customer 都必須被執行。我們將在第 IV 部分討論這兩種系統的可能性，而第 III 部分則不用考慮他們的差異。

## III. 基於距離分配 priority 

對於每個 stream，系統會記錄最近一次 met 或是 missed the deadlines 的 state。我們編號各個 streams 的 customers 依據其先後順序，分別為 1, 2, 3, ...。一個 customer 可能是 serviced completely 或是 dropped prior to service。一個 serviced customer 可能 miss 或是 meet the deadline。而所有 dropped customers 則是被視為 miss the deadline。用 $\delta_i^j$ 表示 stream $R_j$ 的 $i$th customer 的狀態, $i\geq1$。用一個 $k_j$-tuple ($\delta_{i-k_j+1}^j$, ..., $\delta_{i - 1}^j$, $\delta_i^j$) 代表一個給定時間 stream $R_j$ 的狀態。如果 stream $R_j$ 呈現一個 meet 數量少於 $m_j$ 的狀態，則稱此狀態是 $R_j$ 的 failing state。

我們目標是防止 streams 進入 failing state，所以想法是越接近 failing state 的 stream，就給此 stream 的下一個 customer 更高的 priority 。本篇論文提出的方法是，assign 一個 priority value 給 customer，這個數值代表這個 stream 從 current state 到 failing state 所需最少的連續 miss 數量。而 costumer 的 priority value 越小，server 就會給其越高的 priority 。

下圖是 (2, 3)-firm deadline stream 的 state transition diagram：

![](https://i.imgur.com/7biPgKG.png)

字母 `M` 和 `m` 分別代表 meet 和 miss，而 state 則是由 3 個字母組合的 string。比如說，`MMm` 代表最近 3 個 customers 分別 miss、meet、meet the deadlines。Edge 則是代表下一個 customer 的 meet/miss 情況。上色的 states 因為 less than 2 meets，表示為 failing states。如果該 stream 是在 failing state，則其下一個 customer 的 priority value 標記為 0。如果該 stream 是在 `MMm`、`MmM` state，代表距離 failing state 為 1，設 priority value 為 1。如果是在 `MMM`、`mMM` state，代表距離 failing state 為 2，設 priority value 為 2。

在一般情況下，只有少部分的 stream 會在 failing state，而這些 stream 就會受益於被賦予高 piority，同時其他 stream 不會受到嚴重的影響。因此 DBP 可以降低 dynamic failure 的機率。

DBP 也有利於一種情況，就是不同 stream 有不同的 deadline 要求時。比如說，有兩個 streams 分別為 (3, 5)-firm deadlines 和 (9, 10)-firm deadlines，其餘因素相同 (same customer service time distribution, same customer interarrival distribution, same customer deadline distribution)。第一個 stream 在連續 5 個 customers 中可以容忍 2 misses，40% loss rate。第二個 stream 在連續 10 個 customers 種只可以容忍 1 miss，10% loss rate。在傳統的 single priority scheme 中，不會考慮到不同 streams 的 requirements，導致兩個 streams 有一樣的 loss rate。但是用 DBP 策略，第二個 stream 會比較常拿到 higher priority。在兩個 streams 都是在 miss-free states 時，第一個 stream 到 failing state 的距離是 3，第二個 stream 到 failing state 的距離是 2。因此在以上情況，(9, 10)-firm deadlines 仍然會有較高的 priority 。這種方式跟靜態得給 deadline requirement 高的 stream 比較高的 priority 也不一樣。DBP 在某些情況 (3, 5)-firm deadlines 也能得到比 (9, 10)-firm deadlines 高的 priority 。比如說，第一個 stream 距離 failing state 只差一個 miss，而第二個 stream 處在 miss-free states。

這種策略可以簡單地用軟、硬體實現。Stream $R_j$ 的 state 可以保存在一個 $k_j$-bit 的 shift register。分別用 bit 0 和 1 代表 miss 和 meet 的情況。當下一個 customer 被服務完後，根據是 miss 還是 meet 從右邊 shift in 0 或是 1。用 $l_j(n, s)$ 表示 stream $R_j$ 的  state $s$ 中，從右邊數來第 n 個 meet (or 1) 的位置。如果 state $s$ 中少於 n 個 1，則 $l_j(n, s) = k_j + 1$。舉例來說，假設 stream $R_1$ 是 (1, 3)-firm deadlines，則 $l_1$(1, MmM) = 1, $l_1$(2, MmM) = 3。如果 n > 2，$l_1$(n, MmM) = $k_1$ + 1 = 4。而 priority assigned to customer i + 1 from stream $R_j$ is given by：

$priority_{i+1}^j = k_j - l_j(m_j, s) + 1$

## IV. 評估 DBP 方案

我們透過模擬器評估 DBP 與 single priority (SP)，在這兩種方案中，如果 customer priority 相同則採用 earliest-deadline-first 排程。

我們使用兩種模式來生成 customer，分別為 Poisson 與 bursty。在 Poisson 模式中，customer 的 interarrival time 為指數分布；bursty 則分為 ON 與 OFF 兩種狀態，狀態為 ON 時會定期產生 customer，OFF 時則不產生 customer，ON 與 OFF 的時長為指數分布，並把 ON 與 OFF 的平均時長記為 $ON_{ave}$ 和 $OFF_{ave}$。bursty 模式常常被用來模擬談話中的聲音，人在交談時為 ON，安靜時為 OFF。

在 A、B 中，我們首先考慮系統中所有 stream 有一樣的 (m,k)-firm，我們也假定只有 meet deadline 的 customer 能夠獲得服務。在 C 中我們在一個不管有沒有 meet deadline，所有 customer 都會獲得服務的系統中，比較 DBP 與 SP。在 D，我們考慮一個異質系統，其中有些 stream 有比較嚴格的 (m,k)-firm。在 E 我們將 DBP 和 SP 與另一個基於此系統的 imprecise model 比較 (在引言倒數第二段有介紹 imprecise model)。最後在 F 我們測試了 stream 數量對 DBP 效能的影響。

### **A. Poisson Streams**

![](https://i.imgur.com/uVa9ofN.png)

Fig. 3 展示了 (1, 2)-firm 和 (3-4)-firm 在兩個一樣的系統中的 dynamic failure 率。系統中有 5 個 Poisson stream、所有 customer 的 service time 固定不變且 deadline 為 service time 的 5 倍、customer 的 interarrival time 為指數分布，透過調整 interarrival time 來讓系統的平均負載介於 0.2 至 0.9。

Fig 3a 和 3b 唯一的差異在於對 miss deadline 的容忍度，3a 允許任 2 個連續 customer 中 1 個 miss  deadline ，而 3b 比較嚴格，只允許任 4 個中 1 個 miss  deadline ，也因此 3b 的 dynamic failure 率較高。

![](https://i.imgur.com/U7uROMM.png)

TABLE I 基於 Fig 3.，以數字表明了在各種負載下，DBP 相較 SP 少了多少 dynamic failure 的機率。在 3a 大致少了 60%、在 3b 少了 40%。 


### **B. Bursty Streams**

![](https://i.imgur.com/pscytOF.png)

Fig. 4 展示了 (1, 2)-firm 和 (3-4)-firm 在兩個一樣的系統中的 dynamic failure 率。系統中有 5 個 bursty stream，$ON_{ave}$ 為 50 微秒、$OFF_{ave}$ 為 100 微秒，因此可以得到尖峰負載為平均負載的三倍。在 ON 狀態下，每個 stream 每 5 微秒會生成一個 customer、 deadline 為 10 微秒、透過拉長 service time 來增加負載。一樣的，實驗結果表明 DBP 較 SP 優。

![](https://i.imgur.com/7B912Lm.png)

TABLE II 基於 Fig 4.，在 4a DBP 的 dynamic failure 率大致少了 95%、在 4b 減少幅度則較小。

比較 Fig 3. 及 Fig 4.，發現 bursty 模式的 dynamic failure 率要比 Poisson 高，這是因為 bursty 的尖峰負載是基於平均負載 (在此實驗中為 3 倍)，尖峰負載時常大於 1，意味著系統常常超過負荷。

### **C. No-Drop Policy (不可以丟掉 customer)**

在 **A**、**B** 實驗中，我們都假設系統已知每個 customer 的 service time，所以可以把必定超過 deadline 的 customer 丟掉，然而不是每一種系統都能預先知道 service time，這類系統必須執行每個 customer，不允許丟掉。

![](https://i.imgur.com/WV8ehZz.png)

Fig 5. 展示了基於 Fig 3. 的系統，但是不允許丟掉 customer 的實驗結果。因為此實驗中，實質上獲得服務的 customer 更多了， dynamic failure 的率也提高，雖然 DBP 相較 SP 減少超過 80% 的 dynamic failure 率，但 dynamic failure 率的數值仍然比 Fig 3. 高。

### **D. 異質系統**

在前面的實驗中，每個 stream 的 (m,k)-firm 都相同。本實驗將每個 stream 設為不同的 (m, k)-firm。分別為 (9, 10)-firm、(3, 4)-firm、(1, 2)-firm、(1, 3)-firm、(1, 4)-firm，其餘條件則同 Fig 3.。

![](https://i.imgur.com/QN46IcO.png)

在 Fig 6. 中，我們單獨挑出負載為 0.5、0.7、0.9 的數據觀察，分別對應 6a、6b、6c。本次實驗除了 SP、DBP 外，還測試了 fixed priority (FP)，在 FP 方案中，時效要求越高的 stream 會被分配越高的 priority ，也就是 (9, 10)-firm stream 的 customer 會被固定為最高 priority ，其次 (3, 4)-firm...，(1, 4)-firm stream 的 customer 為最低 priority 。FP 也符合 earliest-deadline-first policy。

觀察 Fig 6.，在 SP 中，(m, k)-firm 越嚴格則 dynamic failure 率越高；相反的，在 FP 中， priority 越高則 dynamic failure 率則通常偏低。這表明了 (m, k)-firm deadline 與 priority 是影響 dynamic failure 率的兩個因素， dynamic failure 率的高低取決於這兩者的平衡。

最後觀察 DBP 擁有最低的平均 dynamic failure 率，而且在嚴格的 (m, k)-firm 下表現遠超 SP，但是在系統負載偏低以及 (m, k)-firm 較不嚴格的 stream 表現較差。另一方面，DBP 在 (m, k)-firm 較不緊迫的 stream 表現較 FP 好。最後綜觀 5a 5b 5c，DBP 在不同的 (m, k)-firm deadline 下，似乎是較為平衡的方案。

### **E. 與 Imprecise model 比較**

再次複習，Imprecise model 會將 customer 區分為 mandatory 與 optional。mandatory 必須獲得服務，optional 則是拿來優化 mandotary 的結果，而且在系統負載過高時可以被丟棄。

在本實驗中，將 (m, k)-firm 結合 imprecise model，令每個 stream 中，連續 k 個 customer 裡要有 m 個 mandotary customer。本次實驗的系統基於 Fig. 5b. 的由 (3, 4)-firm stream 組成的系統，並將 customer 掛上 mandotary 和 optional 的標籤。接著我們定義一個臨界點 $T$，當在 service queue 中等候的 customer 數量超過 $T$ 時，之後到達的 optional customer 會直接被丟掉。注意，當 $T = 0$ 所有 optional 都會被丟棄；當 $T = \infty$ 則所有 optional 都會獲得服務，排程邏輯等同於 SP。

在 Fig 7. 中，x 軸為 $T$ 的值，介於 0 到 8，左圖 y 軸為 dynamic failure 率，右圖 y 軸為 customer  miss  deadline 的機率。只有 IMPRECISE 會用到 $T$，而 SP 與 DBP 並不會使用到 $T$，因此我們可以看到不管 $T$ 的值為何，SP 與 DBP 的 dynamic failure 率都不變，為兩條水平線。Fig 7. 只抓取平均負載為 0.3、0.6、0.8 來觀察。

![](https://i.imgur.com/1OQfRUs.png)

觀察發現 imprecise model 在合適的 $T$ 下， dynamic failure 率與 DBP 差不多低，例如在負載較高且 $T=0$ 時，左圖 dynamic failure 率低。然而隨之而來的是右圖 customer  miss  deadline 率大幅提升，因為當 $T=0$ 時，所有 optional customer 都會被丟掉，而系統中所有 stream 都是 (3, 4)-firm，所以每四個 customer 必定會丟掉 1 個，導致 miss  deadline 率大於等於 0.25。反觀 DBP 則可以在不拉高 miss  deadline 率的情況下降低 dynamic failure 率。

imprecise model 還有一個問題，$T$ 的值需要依據系統負載來調整，我們可以看到在系統負載為 0.3 時，$T=0$ 反而對 imprecise model 造成很差的效果。

### **F. Stream 數量對效能的影響**

從前面的實驗，我們已經確定根據 stream 的 state 來分配 priority 可以降低 dynamic failure 率，因此系統中 stream 的數量會決定降低的幅度，如果系統中只有一個 stream，DBP 的效果會完全等於 SP。為了研究 stream 數量對 DBP 的影響，這次實驗，我們逐次增加 stream 的數量，並且調整 arrival rate 讓整個系統的平均負載保持相同。

![](https://i.imgur.com/EflxEkB.png)

在 Fig 8. 中，系統的平均負載為 0.7、interarrival time 是指數分布、所有 stream 都是 (1, 2)-firm deadline。8a 中所有 customer 都會獲得服務、8b 中只有趕得上 deadline 的 customer 會得到服務。

我們可以觀察，當 stream 數量為 1，SP 與 DBP 的 dynamic failure 率是一樣的，隨著 stream 數量越多，DBP 的 dynamic failure 率以顯著的幅度下降，在 3a 中，3 個 stream 的狀況下 DBP 較 SP 少了 70% 的 dynamic failure 率。

## V. 限制 priority 數量

DBP 中可供分配的優限度是基於 (m, k)-firm，當 stream $R_j$ 為 $(m_j, k_j)\rm{-firm}$，則總共有 $k_j-m_j+2$ 個 priority 可供分配。例如在 (2, 5)-firm 的情況下， priority 分為 0, 1, 2, 3, 4；在 (4, 5)-firm 的情況下， priority 分為 0, 1, 2。因此，在多個不同 (m, k)-firm stream 的系統，必須支援

$P=\rm{max}\{k_j-m_j+2:j=1,2,...,N\}$

個不同的 priority ，其中 $N$ 為 stream 的數量。

然而現實上，一個系統通常會限制 priority 的數量，將系統最大限制的 priority 數量記為 $P_{max}$ ( priority 介於 0 到 $P_{max}-1$)，在這個章節我們將討論 $P_{max}<P$ 的狀況。

當這個 stream 的 state 與 failing state 的距離大於 $P_{max}-1$，則還是分配 $P_{max}-1$ 給它的 customer，用以下式子表達 $R_j$ 的第 $i+1$ 個 customer 將會分配到的 priority 

$priority^j_{i+1}=\rm{min}\{k_j-l_j(m_j,s)+1,P_{max}-1\}$

其中 s 和 $l_j$ 是我們在第 III 節提到的 $R_j$ 的 state 與右邊數來 meet 的位置。

![](https://i.imgur.com/mJIaTiV.png)

Fig 9. 為一個全為 (2, 5)-firm deadline stream 的系統，系統負載固定為 0.8、interarrival time 為指數分布、所有 customer 的 service time 都一樣。在理想情況下，$P=5$ 且有 0, 1, 2, 3, 4 五種 priority ，但是我們透過調整 $P_{max}$ 觀察 priority 的數量限制對 dynamic failure 率的影響。左圖所有 customer 都會獲得服務；右圖只有 deadline 內可以完成的 customer 可以獲得服務，其餘則被丟掉。

當 $P_{max}=1$ 時，只有一種 priority ，等同於 SP。觀察 Fig 9. 可以發現其實 $P_{max}=3$ 時，DBP 的效果就足夠好了，其中 failing state stream 中的 customer 會被分配 priority  0、近期只有 1 個 customer  miss  deadline 的 stream 則會分配為 1、其餘分配為 2。

![](https://i.imgur.com/YO5fNha.png)

Fig 10. 則展示 $P_{max}$ 對異質系統的影響，五個 stream 分別為 (9, 10)-firm、(3, 4)-firm、(1, 2)-firm、(1, 3)-firm、(1, 4)-firm，其餘條件與 fig 9. 的實驗相同。這裡我們一樣可以發現當 $P_{max}=3$ 時，DBP 就已經展現可觀的效果了。

## VI. 總結

real-time customer 有其時限，根據應用不同，又分為 hard real-time (所有 customer 都要 meet deadline) 與 soft real-time (滿足 miss rate 即可)，前者過於嚴格，後者過於寬鬆。本論文介紹了一個 deadline model 概括了 hard 和 soft real-time 的概念，在這個模型裡每個 stream 有兩個參數 m 和 k，如果 k 個連續客戶中少於 m 個滿足他們的 deadline ，就會發生 dynamic failure ，藉此更精確地表達實時應用程序的要求。

接著本論文提 Distance-Based Priority (DBP)，較接近 failing state 的 stream 會被分配更高的 priority，以提高它的 customer meet deadline 的機會。本方法與 Single Priority、Fix priority 和 IMPRESICE model 進行了比較，實驗結果表明，DBP 大幅降低了 dynamic failure 的概率，同時不會極大地影響整體的 customer miss deadline 的機率。