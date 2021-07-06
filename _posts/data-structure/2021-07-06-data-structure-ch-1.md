---
title: "資結 Chapter 1"
subtitle: "Basic Concepts"
excerpt: "data structure 資結"
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - note
  - data structure
---

## 演算法 (Algorithm)

以有限的指令完成一個任務，一個演算法必須滿足以下五個條件:  
- **輸入 (Input)：**演算法必須有 0 個或以上的輸入。
- **輸出 (Output)：**演算法應有 1 個或以上輸出。
- **明確性 (Definiteness)：**演算法的描述必須無歧義，以保證實際執行結果精確符合要求。
- **有限性 (Finitense)：**演算法必須在有限個步驟內完成任務。
- **有效性 (Effectiveness)：**演算法中描述的操作可以透過有限次已經實現的基本運算來實現。

## 如何描述一個演算法

以 Selection Sort 為例:

- **口語描述 (statements):**  
  假設有 n 個為排序的整數，編號為 `0` 至 `n - 1`。第 1 輪選取 `0` 到 `n - 1` 中數值最小的數字，將其與編號 `0` 的數字交換位置；第 2 輪選取 `1` 到 `n - 1` 中數值最小的數字，將其與編號 `1` 的數字交換位置；持續直到第 n 輪。
- **偽代碼 (pseudo code):**  
  ```non
  for (i = 0; i < n; i++) {
      Examine list[i] to list[n-1] and
      suppose that the smallest integer is at list [min];
      Interchange list[i] and list[min];
  }
  ```

## 遞迴 (Recursive)

遞迴演算法可以縮短程式碼並且讓人更容易理解程式碼，但通常執行效率較差，因為作業系統會為了分配 function stack 做較多事情。

- **直接遞迴 (Direct Recursion)：**function 自己呼叫自己。
- **間接遞迴 (Indirect Recursion)：**function A 呼叫 function B，然後 function B 再呼叫 function A。

經典例子: 階乘

```non
int factorial(int x)
{
    if (x == 0)
        return 1;
    else 
        return factorial(x - 1) * x;
}
```

## 抽象化 (Data Abstraction)

對於複雜的資料結構，將實作的細節隱藏起來，只提供特定的介面讓使用者操作，使用者不需要管實作方式，只需透過介面來開發自己需要的功能，甚至自己定義抽象化的物件。

以 C 語言為例，我們不需要知道為甚麼 `printf("hello")` 可以在終端機印出 `hello` 字串，我們只管學習 `printf` 的操作方法，撰寫出符合要求的 C 程式。

C++ 的 `vector`、`stack`、`queue` 更符合資料抽象化的概念，這種型態又稱為抽象資料型態 (ADT)。

## 空間複雜度

空間複雜度$S(P)=c+S_p(I)$。其中$c$為 Fixed part，也就是程式一定要用的基本空間，通常是 `const `和 `static` 變數；$S_p$ 為 Variable part，指的是依照輸入而動代改變的空間，通常是使用 `new` 或 `malloc` 產生的變數以及遞迴函式佔用的空間。

## 時間複雜度

有三種計算方式:

- **Big Oh:**  
  存在正數 $c,n_0$ 使得 $f(n)\leq cg(n),\rm{\ for\ all\ } n>n_0$。概念上就是當 $n$ 足夠大時，$f(n)$ 小於 $g(n)$，此時時間複雜度表示為 $f(n)=O(g(n))$
- **Big Omega:**  
  存在正數 $c,n_0$ 使得 $f(n)\geq cg(n),\rm{\ for\ all\ } n>n_0$。概念上就是當 $n$ 足夠大時，$f(n)$ 大於 $g(n)$，此時時間複雜度表示為 $f(n)=\Omega(g(n))$
- **Big Theta:**  
  存在正數 $c_1,c_2,n_0$ 使得 $ c_2g(n) \leq f(n)\leq c_1g(n),\rm{\ for\ all\ } n>n_0$。概念上就是當 $n$ 足夠大時，$f(n)$ 介於 $c_1g(n)$ 與 $c_2g(n)$ 之間，此時時間複雜度表示為 $f(n)=\Theta(g(n))$

關於實際計算可以參考這篇文章:  
[Day5：[演算法]如何衡量程式的效率？——論時間複雜度（Time Complexity）](https://ithelp.ithome.com.tw/articles/10203082)