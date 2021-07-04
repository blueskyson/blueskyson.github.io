---
title: "線代 Chapter 1.2"
subtitle: "Row Echelon Form"
excerpt: "linear algebra 線代 Row Echlelon Form"
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - note
  - linear algebra
---

若矩陣無法符合 strict triangular form，而是呈現倒梯形，並且每一列的首個非 0 項為 1，則稱 ***row echelon form*** (**階梯形矩陣**)，如以下矩陣:

$$
\left[
\begin{array}{ccccc|c}
1 & 1 & 1 & 1 & 1 & 1 \\
0 & 0 & 1 & 1 & 2 & 0 \\
0 & 0 & 0 & 0 & 1 & 3 \\
0 & 0 & 0 & 0 & 0 & -4 \\
0 & 0 & 0 & 0 & 0 & 3
\end{array}
\right]
$$

必須滿足以下三個條件才能稱矩陣為 row echelon form:

1. 每一列的首個非 0 項，其值為 1
2. 第 k + 1 列的 0 項 必須比第 k 列多，除非第 k 列所有項皆為 0
3. 所有項次都為 0 的列必須排在矩陣最下面

使用 row operation 將矩陣化為 row echelon form 的過程就稱為 ***gaussian elimination*** (**高斯消去**)

若 row echelon form 包含 $[0 \\ 0 \\ ...\\ 0 \\ \|\\ 1]$ 則此系統為 inconsistent，否則為 consistent

若一個系統有多於未知數的線性方程式，則稱其為 **overdetermined system**；若一個系統有少於未知數的線性方程式，則稱其為 **underdetermined system**

接下來再介紹 ***reduced row echelon form***，在 row echelon form 的基礎下，讓每一列的第一個非 0 項是該行的唯一非 0 項。用 row operation 求取 reduced row echelon form 的過程則稱為 ***Gauss-Jordan reduction***，例如:

$$
\left[
\begin{array}{cccc|c}
-1 & 1 & -1 & 3 & 0 \\
3 & 1 & -1 & -1 & 0 \\
2 & -1 & -2 & -1 & 0
\end{array}
\right] \Rightarrow
\left[
\begin{array}{cccc|c}
-1 & 1 & -1 & 3 & 0 \\
0 & 4 & -4 & 8 & 0 \\
0 & 1 & -4 & 5 & 0
\end{array}
\right] \Rightarrow \\
\left[
\begin{array}{cccc|c}
-1 & 1 & -1 & 3 & 0 \\
0 & 4 & -4 & 8 & 0 \\
0 & 0 & -3 & 3 & 0
\end{array}
\right] \Rightarrow
\left[
\begin{array}{cccc|c}
1 & -1 & 1 & -3 & 0 \\
0 & 1 & -1 & 2 & 0 \\
0 & 0 & 1 & -1 & 0
\end{array}
\right]
\rm{(Row\ Echelon\ Form)} \\
\Rightarrow \left[
\begin{array}{cccc|c}
1 & -1 & 0 & -2 & 0 \\
0 & 1 & 0 & 1 & 0 \\
0 & 0 & 1 & -1 & 0
\end{array}
\right] \Rightarrow
\left[
\begin{array}{cccc|c}
1 & 0 & 0 & -1 & 0 \\
0 & 1 & 0 & 1 & 0 \\
0 & 0 & 1 & -1 & 0
\end{array}
\right]
\rm{(Reduced\ Row\ Echelon\ Form)}
$$

若一個系統的所有方程式常數項都為 0，即增廣矩陣右邊全為 0，則稱此系統為 ***homogeneous system*** (**齊次**)。若一個系統為齊次，則此系統必為 consistent。

homogeneous system 必定包含一解 $(0,0,...,0)$，這種全部都為 0 的解也稱為 ***trivial solution***；此外 homogeneous system 也可能是無限多解 (包含 trivial solution)