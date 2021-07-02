---
title: "線代 Chapter 1.1"
subtitle: "Systems of Linear Equations"
excerpt: "linear algebra 線代"
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - note
  - linear algebra
---

## linear system

擁有 $n$ 個未知數的 ***linear equation*** (**線性方程式**) 表示為

$$a_1x_1+a_2x_2+ \cdots +a_nx_n=b$$

一個 $m \times n$ 的 ***linear system*** (**線性系統**)，即擁有 $m$ 個方程式與 $n$ 個未知數的聯立方程組表示為

$$\begin{align*}
& a_{11}x_1+a_{12}x_2+ \cdots +a_{1n}x_n=b_1 \\\\
& a_{21}x_1+a_{22}x_2+ \cdots +a_{2n}x_n=b_2 \\\\
& \vdots \\\\
& a_{m1}x_1+a_{m2}x_2+ \cdots +a_{mn}x_n=b_2
\end{align*}$$

舉個例子，以下為一個 $3\times 2$ 的線性系統

$$\begin{align*}
& x_1+x_2=2 \\\\
& x_1-x_2=1 \\\\
& x_1=4
\end{align*}$$

一個線性系統的解可以用 $n$ 個元素的 tuple (元組) 來表達，如 $(x_1,x_2)=(1,2)$。若一個線性系統有解，則稱其為 ***consistent system*** (**相容方程組**)；若一個線性系統無解，則稱其為 ***inconsistent system*** (**矛盾方程組**)

## equivalent

如果兩個線性系統有相同數目的未知數，並且有相同的解(必須是一樣的集合)，則稱這兩個系統為 ***equivalent systems***(**等效系統**)。對一個線性系統座以下三種操作可以得到等效系統

1. 調換方程式順序
2. 對方程式等號左右同乘一個常數
3. 將同一系統的兩個方程式相加，替換掉原本的方程式

這三種操作在矩陣中也稱作 ***row operation***

## strict triangular form

若一個系統的第 k 個方程式的前 k - 1 個係數皆為 0，則稱其為 ***strict triangular form***，如下

$$\begin{align*}
3x_1+2x_2+x_3&=1 \\
x_2-x_3&=2 \\
2x_3&=4
\end{align*}$$

用 strict triangular form 可以直接求得最末端的變數，並且透過 ***back substitution*** 求得線性系統的解。透過 ***augmented matrix*** (**增廣矩陣**) 可以很容易求得 strict triangular form

$$
\left[
\begin{array}{ccc|c}
1 & 2 & 1  & 3 \\
2 & 3 & 1  & 4 \\
3 & -1 & -3  & -1
\end{array}
\right] \Rightarrow
\left[
\begin{array}{ccc|c}
1 & 2 & 1  & 3 \\
0 & -1 & -1  & -2 \\
0 & -7 & -6  & -10
\end{array}
\right] \Rightarrow
\left[
\begin{array}{ccc|c}
1 & 2 & 1  & 3 \\
0 & -1 & -1  & -2 \\
0 & 0 & 1  & 4
\end{array}
\right]
$$