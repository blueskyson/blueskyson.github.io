---
title: "證明 Petersen graph 的圍長為 5"
subtitle: "Prove or disprove that Petersen graph has girth 5."
excerpt: "xfce 新酷音 觸控板"
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - graph theory
---

1. Petersen graph 為 simple graph，故沒有 loop 與 multiple edge，也就是沒有 1-cycle 與 2-cycle，見下圖：
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/loop-multiple-edge.jpg)

2. 形成 3-cycle 需要 3 個所有元素都不重複的 2-set，這個條件無法用 5 個元素的集合來達成。
   ```non
   {1,2}, {3,4}, {5, ?} ====> 無法達成
   ``` 

3. 4-cycle 不管怎麼湊，都會出現兩個一模一樣的 2-set，意味著出現了兩個重複的點，這違反了 Petersen graph 的基礎定義。
   ```non
   {1,2}, {3,4}, {1,5}, {3,4} ====> {3,4} 這個點重複了
   ```

4. 舉一個 5-cycle 的例子：
   ```non
   {1,2}, {3,5}, {5,1}, {2,3}, {4,5} 形成一個 5-girth
   ```

由上述 4 個步驟得證 Petersen graph 的圍長 (girth) 為 5。