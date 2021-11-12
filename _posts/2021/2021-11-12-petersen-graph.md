---
title: "Petersen graph 簡介"
subtitle: "What is Petersen graph?"
excerpt: "petersen graph"
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - graph theory
---

**Def:** Petersen graph 是一種 simple graph，其頂點被標記為 5 元素集合的 2 元素子集合 (2-set)，邊為不共元素的 2 元素子集合對。

光看解釋會讓人摸不著頭緒，但其實概念很簡單，首先定義一個 5 元素的集合：$\\{1,2,3,4,5\\}$

接著列出所有 2 元素子集合，根據 $C^5_3=10$，總共有 10 個子集合：

$$\{1,2\},\{1,3\},\{1,4\},\{1,5\}\\
\{2,3\},\{2,4\},\{2,5\}\\
\{3,4\},\{3,5\}\\
\{4,5\}$$

為了簡單標註，把集合的寫法簡化為：12、13、14、15、23、24、25、34、35、45。

接下來畫一個有十個點的圖，並且將不共元素的標籤以邊相連。例如 12 與 34 可以相連；12 與 15 則無法相連，因為 1 這個元素重複了；同理 35 與 45 也無法相連。

