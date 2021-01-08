---
title: "Exponential Golomb coding"
subtitle: ""
excerpt: "Exponential Golomb coding"
layout: post
author: "blueskyson"
mathjax: true
header-style: text
tags:
  - note
---

### Exponential Golomb coding

摘錄自維基百科，[Exponential-Golomb coding](https://en.wikipedia.org/wiki/Exponential-Golomb_coding) (或稱 Exp-Golomb code) 是一種 [Universal code](https://en.wikipedia.org/wiki/Universal_code_(data_compression))，也就是 Exp-Golomb code 能夠映射到所有正整數。

假設輸入為 `x` ，其編碼步驟為:

1. 以二進位寫下 `x + 1` 
2. 數 `x + 1` 的二進位數字總共有多少位數，並在 `x + 1` 前面補上等同其位數減 1 的數量的 `0`

**Example 1:**
```
x = 5
1. Write down 5 + 1 = 6 as binary "110".
2. "110" has 3 digits, we write 2 "0"s preceding "110".
   The Exp-Golomb code of 5 is "00110".
```
**Example 2:**
```
x = 24
1. Write down 24 + 1 = 25 as binary "11001".
2. "11001" has 5 digits, we write 4 "0"s preceding "11001".
   The Exp-Golomb code of 24 is "000011001".
```

按照上述規則計算 0 ~ 8 的 Exp-Golomb code:
```
 x   step 1   step 2
 0 ⇒  1     ⇒ 1
 1 ⇒  10    ⇒ 010
 2 ⇒  11    ⇒ 011
 3 ⇒  100   ⇒ 00100
 4 ⇒  101   ⇒ 00101
 5 ⇒  110   ⇒ 00110
 6 ⇒  111   ⇒ 00111
 7 ⇒  1000  ⇒ 0001000
 8 ⇒  1001  ⇒ 0001001
```

如果要讓 Exp-Golomb code 編碼**有號**整數，只需要讓有號數一一對應無號數即能達到目的:
```
 x   map    step 1  step 2
 0 ⇒  0  ⇒  1     ⇒ 1
 1 ⇒  1  ⇒  10    ⇒ 010
−1 ⇒  2  ⇒  11    ⇒ 011
 2 ⇒  3  ⇒  100   ⇒ 00100
−2 ⇒  4  ⇒  101   ⇒ 00101
 3 ⇒  5  ⇒  110   ⇒ 00110
−3 ⇒  6  ⇒  111   ⇒ 00111
 4 ⇒  7  ⇒  1000  ⇒ 0001000
−4 ⇒  8  ⇒  1001  ⇒ 0001001
```

我們可以按照順序，每 $2^i$ 個 Exp-Golomb code 分為一組。對於每個編碼，前面有 $i$ 個 `0` 就代表該編碼屬於 $S_i$ ，每個編碼正中央紅色的 `1` 作為分界點，後綴的編碼以二進位表示為第 $S_i$ 組的第 $j$ 個元素，記為 $G_{i,j}$ :

$ \\ S_0 \quad \quad \quad S_1 \quad \quad \quad \quad \quad \quad \quad \quad \\ \\ S_2 \\\\ \\{\color{red}1\\},\\ \\{0\color{red}10,\\ 0\color{red}11\\},\\ \\{00\color{red}100,\\ 00\color{red}101,\\ 00\color{red}110,\\ 00\color{red}111\\},\\  ... \\\\ G_{0,0} \\ \\ \\ G_{1,0} \\ \\ \\ G_{1,1} \quad \quad G_{2,0} \quad \\ \\ G_{2,1} \quad \\ \\ G_{2,2} \quad \\ G_{2,3}$

### Order-k Exponential Golomb coding

接下來，對 Exp-Golomb code 的編碼方式一般化，前綴的 `0` 的數量一樣代表該編碼在第幾組，但是每一組的元素數量變為 $2^{i+k}$ 個，後綴的編碼也加長 $k$ 位，我們將這種編碼方式稱為 $k$ 階 Exp-Golomb code。以下為範例:

**order-0**

$ \\ S_0 \quad \quad \quad S_1 \quad \quad \quad \quad \quad \quad \quad \quad \\ \\ S_2 \\\\ \\{\color{red}1\\},\\ \\{0\color{red}10,\\ 0\color{red}11\\},\\ \\{00\color{red}100,\\ 00\color{red}101,\\ 00\color{red}110,\\ 00\color{red}111\\},\\  ... \\\\ G_{0,0} \\ \\ \\ G_{1,0} \\ \\ \\ G_{1,1} \quad \quad G_{2,0} \quad \\ \\ G_{2,1} \quad \\ \\ G_{2,2} \quad \\ G_{2,3}$

**order-1**

$ \\ \quad S_0 \quad \quad \quad \quad \quad \quad \quad S_1 \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad S_2 \\\\ \\{\color{red}10,\\ \color{red}11\\},\\ \\{0\color{red}100,\\ 0\color{red}101,\\ 0\color{red}110,\\ 0\color{red}111\\},\\ \\{00\color{red}1000,\\ 00\color{red}1001,\\ 00\color{red}1010,\\  ... \\\\ G_{0,0} \\ \\ G_{0,1} \quad \\ G_{1,0} \quad G_{1,1} \quad G_{1,2} \quad G_{1,3} \quad \quad G_{3,0} \quad \quad \\ G_{3,1} \quad \quad G_{3,2}$

**order-2**
$ \\ \quad S_0 \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad S_1 \\\\ \\{\color{red}100,\\ \color{red}101,\\ \color{red}110,\\ \color{red}111\\},\\ \\{0\color{red}1000,\\ 0\color{red}1001,\\ 0\color{red}1010,\\ 0\color{red}1011,\\ 0\color{red}1100,\\ 0\color{red}1101,\\ 0\color{red}1110,\\ 0\color{red}1111\\},\\ ...\\\\ G_{0,0} \\ \\ \\ G_{0,1} \\ \\ G_{0,2} \\ \\ G_{0,3} \quad \\ \\ \\  G_{1,0} \quad \\ \\ G_{1,1} \quad \\ \\ G_{1,2} \quad \\ \\ G_{1,3} \quad \\ \\ G_{1, 4} \quad \\ \\ G_{1, 5} \quad \\ \\ G_{1, 6} \quad \\ G_{1, 7}$

產生 $k$ 階 Exp-Golomb code 的步驟為:
1. 以二進位寫下 $x+2^k$
2. 數 $x+2^k$ 的二進位數字總共有多少位數，並在 `x + 1` 前面補上等同其位數減 $k$ 的數量的 `0`

|x |order-0|order-1|order-2|order-3|order-4|
|--|-------|-------|-------|-------|-------|
|0 |1      |10     |100    |1000   |10000  |
|1 |010    |11     |101    |1001   |10001  |
|2 |011    |0100   |110    |1010   |10010  |
|3 |00100  |0101   |111    |1011   |10011  |
|4 |00101  |0110   |01000  |1100   |10100  |
|5 |00110  |0111   |01001  |1101   |10101  |
|6 |00111  |001000 |01010  |1110   |10110  |
|7 |0001000|001001 |01011  |1111   |10111  |
|8 |0001001|001010 |01100  |010000 |11000  |
|9 |0001010|001011 |01101  |010001 |11001  |
|10|0001011|001100 |01110  |010010 |11010  |
|11|0001100|001101 |01111  |010011 |11011  |
|12|0001101|001110 |0010000|010100 |11100  |