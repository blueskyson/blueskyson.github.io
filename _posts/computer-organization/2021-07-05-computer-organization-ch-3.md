---
title: "計組 Chapter 3"
subtitle: "Arithmetic for Computers"
excerpt: "computer organization 計組"
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - note
  - computer organization
---

## 基本算術邏輯單元 (Basic ALU)

### 1-bit ALU

最基本的 ALU 可以執行 AND 和 OR 運算:

| Operation(Op.) | Funct. |
|---|---|
| 0 | a AND b |
| 1 | a OR b |

### 增強版 ALU

增加加法運算，支援 (a AND b)、(a OR b)、(a + b):

| Operation(Op.) | Funct. |
|---|---|
| 0 | a AND b |
| 1 | a OR b |
| 2 | a + b |

### 減法運算

利用 **2's complement** 實現減法: $a - b = a + \bar{b} + 1$

要執行 a-b，設定 Binvert=1 和 CarryIn=1:

| Binvert | CarryIn | Op. | Function |
|---|---|---|---|
| 0 | X | 0 | a and b |
| 0 | X | 1 | a or b |
| 0 | 0 | 2 | a + b |
| 1 | 1 | 2 | a-b |

### 32-bit ALU

將多個 1-bit ALU 串接起來形成 32-bit ALU:
- 每個 ALU 的 **carry-out** 連接到下一個 ALU 的 **carry-in**
- 所有 ALU 共用相同的控制訊號 (Ainvert, Binvert, Operation)

### 增強版 ALU (支援 NOR 和 NAND)

使用 De Morgan's law:
- $\overline{ab} = \bar{a} \vee \bar{b}$ (NAND)
- $\overline{a \vee b} = \bar{a} \bar{b}$ (NOR)

| Ainvert | Binvert | CarryIn | Op. | Func. |
|---|---|---|---|---|
| 0 | 0 | X | 0 | a and b |
| 0 | 0 | X | 1 | a or b |
| 0 | 0 | 0 | 2 | a + b |
| 0 | 1 | 1 | 2 | a-b |
| 1 | 1 | X | 0 | $\bar{a} + \bar{b}$ |
| 1 | 1 | X | 1 | $\overline{ab}$ |

### Set Less Than (slt)

**slt** 指令用於比較兩數大小: `slt $t0, $t1, $t2` ⇒ 若 \$t1 < \$t2，則 \$t0 = 1，否則 \$t0 = 0

實作方式:
- 使用 a-b 來判斷
- 若 a-b < 0，signed bit = 1
- 若 a-b ≥ 0，signed bit = 0

**Less** 訊號是 LSB (最低位元) 的輸入:
- 連接到 MSB (最高位元) 的 signed bit
- 其他位元的 Less 訊號設為 0

當 $a_{31}...a_0 < b_{31}...b_0$，結果是 0...01，否則是 0...00

### 最終 32-bit ALU

改進:
- Binvert 與 CarryIn 功能相同 ⇒ 合併為 **Bnegate**
- 加入 **Zero detection** 電路: 若所有位元為 0 ⇒ Zero=1

| Bnegate | Op[1:0] | Func. |
|---|---|---|
| 0 | 00 | a and b |
| 0 | 01 | a or b |
| 0 | 10 | a + b |
| 1 | 10 | a - b |
| 1 | 11 | slt |

### 更快的加法器 - Carry Lookahead

普通的 ripple carry adder 較慢，因為 carry 需要逐位傳遞。**Carry Lookahead Adder** 平行計算 carry，加快速度:

- $g_i = a_i b_i$ (generate)
- $p_i = a_i + b_i$ (propagate)
- $c_1 = g_0 + (p_0 \cdot c_0)$
- $c_2 = g_1 + (p_1 \cdot g_0) + (p_1 \cdot p_0 \cdot c_0)$
- $c_3 = g_2 + (p_2 \cdot g_1) + (p_2 \cdot p_1 \cdot g_0) + (p_2 \cdot p_1 \cdot p_0 \cdot c_0)$
- ...

## 整數加減法

### 加法範例: 7 + 6

```
   00000111  (+7)
 + 00000110  (+6)
 -----------
   00001101  (+13)
```

### 減法範例: 7 - 6 = 7 + (-6)

```
   00000111  (+7)
 + 11111010  (-6)
   00000001  (+1) 加上進位忽略
 + 00000001
 -----------
   00000001  (+1)
```

### Overflow (溢位)

當兩個 32-bit 數的和/差**太大**無法用 32 bits 表示時發生。

範例:

```
  0111 1111 1111...1111  (= 2,147,483,647)
+ 0000 0000 0000...0010  (= 2)
-------------------------
  1000 0000 0000...0001  (= -2,147,483,647，錯誤!)
```

檢查 **sign bit** 可以判斷是否溢位。

### Overflow 條件

Overflow 發生於:

| Operation | A | B | Result when Overflow |
|---|---|---|---|
| A+B | A≥0 | B≥0 | <0 |
| A+B | <0 | <0 | ≥0 |
| A-B | A≥0 | B<0 | <0 |
| A-B | A<0 | B≥0 | ≥0 |

### 處理 Overflow

- **Exception (異常)**: 未預期的事件，中斷程式執行
- MIPS 指令:
  - 會引發 exception: `add`, `addi`, `sub`
  - 不會引發 exception: `addu`, `addiu`, `subu`

當 overflow 發生時 (HW interrupt):
1. 將造成 overflow 的指令位址存入 **EPC** (Exception Program Counter，在 Coprocessor 0 的 \$R14)
2. 跳轉到 **service routine** 處理 exception
3. 返回程式

使用 `mfc0` (Move From CoProc 0) 將 EPC 複製到一般暫存器，再用 `jr` 返回:

```python
mfc0 $ra, EPC   # $ra = EPC
jr $ra          # 返回
```

## 乘法 (Multiplication)

### 二進位乘法

二進位乘法就是一系列的**右移**和**加法**:

```
    0010  (2)
  × 0011  (3)
  ------
    0010
   0010
  0000
 0000
 -------
 0000110  (6)
```

- Multiplicand: n bits
- Multiplier: n bits
- Product: 2n bits (雙倍精度)

### 乘法硬體 (第一版)

| 組件 | 大小 |
|---|---|
| Multiplicand | 64 bits |
| Multiplier | 32 bits |
| Product | 64 bits |

演算法:
1. 若 Multiplier 的最右位元 = 1，將 Multiplicand 加到 Product
2. 將 Multiplicand 左移 1 bit
3. 將 Multiplier 右移 1 bit
4. 重複 32 次

### 乘法範例: 0010₂ × 0011₂

| Step | Multiplicand | Product | Multiplier |
|---|---|---|---|
| Initial | 00000010 | 00000000 | 0011 |
| 1 | 00000010 | 00000010 | 0011 |
| 2 | 00000100 | 00000110 | 0001 |
| 3 | 00001000 | 00000110 | 0000 |
| 4 | 00010000 | 00000110 | 0000 |

最終結果: 00000110 = 6

### 優化的乘法器

觀察: 可以將 multiplicand **左移** 或將 product **右移**。

改進:
- Multiplicand: 32 bits (不是 64 bits)
- Multiplier 初始存在 Product 的右半部
- 每次將 Product 右移

| Step | Multiplicand | Product |
|---|---|---|
| Initial | 1000 | 0000 1011 |
| 1 | 1000 | 0000 1000<br>1000 0000<br>0100 0101 |
| 2 | 1000 | 0100 0010<br>1100 0000<br>0110 0010 |
| 3 | 1000 | 0110 0001<br>1110 0000<br>0111 0001 |
| 4 | 1000 | 0111 0000<br>1111 0000<br>0111 1000 |

### 更快的乘法器

使用多個加法器平行執行，可以大幅加速:
- **Carry lookahead adder**
- **Carry save adder**
- 可以**流水線化** (pipelining) 以減少關鍵路徑

### MIPS 乘法指令

#### mul (不考慮 overflow)

```python
mul $rd, $rs, $rt   # $rd = $rs * $rt (只保留低 32 bits)
```

#### mult (考慮 overflow)

使用 HI/LO 兩個 32-bit 暫存器:
- **HI**: 高 32 bits
- **LO**: 低 32 bits

```python
mult $rs, $rt       # HI|LO = $rs * $rt
mfhi $rd            # $rd = HI
mflo $rd            # $rd = LO
```

範例:

```python
.text
    addi $s0, $zero, 10
    addi $s1, $zero, 4
    mult $s0, $s1       # HI|LO = 10 * 4 = 40
    mflo $t0            # $t0 = 40
    li $v0, 1
    add $a0, $zero, $t0
    syscall
```

## 除法 (Division)

### 長除法

除法是一系列的**商數猜測**和**左移與減法**:

```
         1001 (quotient)
      --------
1000 ) 1001010 (dividend)
      -1000
       ------
         1010
        -1000
         ----
           10 (remainder)
```

- n-bit 運算元產生 n-bit 商數和 n-bit 餘數

### Restoring Division (還原式除法)

演算法:
1. 將被除數減去除數，結果放入餘數暫存器
2. 若餘數 ≥ 0:
   - 商數最右位元設為 1
3. 若餘數 < 0:
   - **還原**: 將除數加回餘數
   - 商數最右位元設為 0
4. 將除數右移 1 bit
5. 將商數左移 1 bit
6. 重複 33 次

### 除法範例: 0111₂ / 0010₂

| Iteration | Step | Quotient | Divisor | Remainder |
|---|---|---|---|---|
| 0 | Initial | 0000 | 0010 0000 | 0000 0111 |
| 1 | Rem - Div | 0000 | 0010 0000 | 1110 0111 |
|  | Rem < 0 ⇒ +Div, sll Q, Q0=0 | 0000 | 0001 0000 | 0000 0111 |
| 2 | Rem - Div | 0000 | 0001 0000 | 1111 0111 |
|  | Rem < 0 ⇒ +Div, sll Q, Q0=0 | 0000 | 0000 1000 | 0000 0111 |
| 3 | Rem - Div | 0000 | 0000 1000 | 1111 1111 |
|  | Rem < 0 ⇒ +Div, sll Q, Q0=0 | 0000 | 0000 0100 | 0000 0111 |
| 4 | Rem - Div | 0000 | 0000 0100 | 0000 0011 |
|  | Rem ≥ 0 ⇒ sll Q, Q0=1 | 0001 | 0000 0010 | 0000 0011 |
| 5 | Rem - Div | 0001 | 0000 0010 | 0000 0001 |
|  | Rem ≥ 0 ⇒ sll Q, Q0=1 | 0011 | 0000 0001 | 0000 0001 |

結果: 商數 = 0011, 餘數 = 0001

### 優化的除法硬體

類似乘法器的優化:
- 除數: 32 bits (不是 64 bits)
- 被除數初始存在 Remainder 的右半部
- 商數逐步建立

### 為什麼除法較慢?

除法比乘法慢，因為:
- 需要**餘數**來決定下一個商數位元
- 除法必須**循序**執行
- 無法像乘法一樣**平行化**

不同的除法演算法:
- Restoring
- Non-restoring
- SRT division

### MIPS 除法指令

使用 HI/LO 暫存器:
- **HI**: 32-bit 餘數
- **LO**: 32-bit 商數

```python
div $rs, $rt        # LO = $rs / $rt (商數)
                    # HI = $rs mod $rt (餘數)
mfhi $rd            # $rd = HI (餘數)
mflo $rd            # $rd = LO (商數)
```

**注意**: MIPS 除法沒有 overflow 或 divide-by-0 檢查，軟體必須自行檢查!

### 除法範例: 13 / 5

```python
.text
.globl  main
main:
    ori  $t5, $zero, 13   # $t5 = 13
    ori  $t6, $zero, 5    # $t6 = 5
    div  $t5, $t6         # LO = 13 / 5 = 2
                          # HI = 13 mod 5 = 3
    mfhi $t0              # $t0 = 餘數 = 3
    mflo $t1              # $t1 = 商數 = 2
```

## 浮點數 (Floating Point)

### 科學記號表示法

浮點數用於表示**非整數**，包括非常小和非常大的數:
- $4,600,000,000 = 4.6 \times 10^9$
- $0.0000000000000000000000000166 = 1.66 \times 10^{-27}$

在二進位中:
- $\pm 1.xxxxxx_2 \times 2^{yyyy}$

C 語言類型:
- `float`: 單精度 (32-bit)
- `double`: 雙精度 (64-bit)

### IEEE 754 標準 - 單精度 (32-bit)

| S | Exponent | Fraction |
|---|---|---|
| 1 bit | 8 bits | 23 bits |

$$x = (-1)^S \times (1 + \text{Fraction}) \times 2^{(\text{Exponent} - \text{Bias})}$$

或

$$x = (-1)^S \times \text{Significand} \times 2^{(\text{Exponent} - \text{Bias})}$$

- **S**: sign bit (0 = 非負, 1 = 負)
- **Exponent**: 使用 **excess representation** (加上 bias)
  - Single precision: Bias = 127
- **Fraction**: Significand - 1
- **Normalized number**: 永遠有 leading 1 (hidden bit)

### 單精度範例

**問題**: 以下單精度浮點數代表什麼值?

`x = 1 10000001 01000...00`

- S = 1
- Exponent = 10000001₂ = 129
- Fraction = 01000...00₂

$$x = (-1)^1 \times (1 + 0.25) \times 2^{(129 - 127)}$$
$$= (-1) \times 1.25 \times 2^2 = -5.0$$

### 單精度範例: 表示 -0.75

- $-0.75 = (-1)^1 \times 1.1_2 \times 2^{-1}$
- S = 1
- Fraction = 1000...00 (hidden 1 不表示)
- Exponent = -1 + 127 = 126 = 01111110₂

**答案**: `1 01111110 1000...00`

### 為什麼使用 Bias?

使用 **excess representation** 讓指數比較更容易:
- 可以直接從左到右比較位元
- 不需要 2's complement 減法

| 實際指數 | 8-bit 表示 | Bias=127 |
|---|---|---|
| 127 | 01111111 | 254 = 11111110 |
| 1 | 00000001 | 128 = 10000000 |
| 0 | 00000000 | 127 = 01111111 |
| -1 | 11111111 | 126 = 01111110 |
| -127 | 10000001 | 0 = 00000000 |
| -128 | 10000000 | **保留** |

### IEEE 754 標準 - 雙精度 (64-bit)

| S | Exponent | Fraction |
|---|---|---|
| 1 bit | 11 bits | 52 bits |

$$x = (-1)^S \times (1 + \text{Fraction}) \times 2^{(\text{Exponent} - 1023)}$$

- Bias = 1023

### 雙精度範例

`x = 1 01111111101 1000...00`

- S = 1
- Exponent = 01111111101₂ = 1021
- Fraction = 1000...00₂

$$x = (-1)^1 \times (1 + 0.5) \times 2^{(1021 - 1023)}$$
$$= (-1) \times 1.5 \times 2^{-2} = -3/8$$

### 半精度 (16-bit)

| S | Exponent | Fraction |
|---|---|---|
| 1 bit | 5 bits | 10 bits |

- Bias = 15

### IEEE 754 編碼總結

| Exponent | Fraction | 物件 |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 非零 | 非正規化數 (denormalized number) |
| 1–254 | 任意 | 正規化浮點數 |
| 255 | 0 | ±∞ (infinity) |
| 255 | 非零 | NaN (Not a Number) |

### 非正規化數 (Denormalized Numbers)

最小的正規化數 (single precision):
- `0 00000001 000...00`
- Significand = 1.0
- Exponent = 1 - 127 = -126
- 值 = $1.0 \times 2^{-126}$

如何表示比這更小的數? 使用**非正規化數**:
- Exponent = 00000000
- **Hidden bit 是 0** (不是 1!)

$$x = (-1)^S \times \text{Fraction} \times 2^{-126}$$

範例:
- $0.5 \times 2^{-126}$: `0 00000000 1000...00`
- $0.25 \times 2^{-126}$: `0 00000000 0100...00`

允許**漸進式下溢 (gradual underflow)**，精度逐漸降低。

最小的非正規化數 (single precision):
- `0 00000000 000...01`
- 值 = $2^{-23} \times 2^{-126} = 2^{-149}$

### 特殊數: Infinity 和 NaN

#### Infinity (∞)

- Exponent = 111...1, Fraction = 000...0
- 可用於後續計算，避免 overflow 檢查
- 例如: $F + (+\infty) = +\infty$, $F / \infty = 0$

#### NaN (Not a Number)

- Exponent = 111...1, Fraction ≠ 000...0
- 表示非法或未定義的結果
- 例如: $0.0 / 0.0$, $\sqrt{-1}$
- 可用於後續計算

## 浮點運算

### 浮點加法

步驟:
1. **Align binary points** (對齊小數點): 移動指數較小的數
2. **Add significands** (加尾數)
3. **Normalize** result & check for over/underflow (正規化結果)
4. **Round** and renormalize if necessary (四捨五入並重新正規化)

#### 十進位範例: $9.999 \times 10^1 + 1.610 \times 10^{-1}$

1. Align: $9.999 \times 10^1 + 0.016 \times 10^1$
2. Add: $10.015 \times 10^1$
3. Normalize: $1.0015 \times 10^2$
4. Round: $1.002 \times 10^2$

#### 二進位範例: $1.000_2 \times 2^{-1} + (-1.110_2 \times 2^{-2})$

1. Align: $1.000_2 \times 2^{-1} + (-0.111_2 \times 2^{-1})$
2. Add: $0.001_2 \times 2^{-1}$
3. Normalize: $1.000_2 \times 2^{-4}$
4. Round: $1.000_2 \times 2^{-4}$ (no change) = 0.0625

### 浮點乘法

步驟:
1. **Add exponents** (加指數): 對於 biased exponents，需減去一個 bias
2. **Multiply significands** (乘尾數)
3. **Normalize** result & check for over/underflow
4. **Round** and renormalize if necessary
5. **Determine sign** (決定正負號)

#### 十進位範例: $1.110 \times 10^{10} \times 9.200 \times 10^{-5}$

1. Add exponents: $10 + (-5) = 5$
2. Multiply: $1.110 \times 9.200 = 10.212$ ⇒ $10.212 \times 10^5$
3. Normalize: $1.0212 \times 10^6$
4. Round: $1.021 \times 10^6$
5. Sign: positive

#### 二進位範例: $1.000_2 \times 2^{-1} \times (-1.110_2 \times 2^{-2})$

1. Add exponents:
   - Unbiased: $(-1) + (-2) = -3$
   - Biased: $(-1 + 127) + (-2 + 127) - 127 = -3 + 127 = 124$
2. Multiply: $1.000_2 \times 1.110_2 = 1.110_2$ ⇒ $1.110_2 \times 2^{-3}$
3. Normalize: $1.110_2 \times 2^{-3}$ (no change)
4. Round: $1.110_2 \times 2^{-3}$ (no change)
5. Sign: $+ve \times -ve = -ve$ ⇒ $-1.110_2 \times 2^{-3} = -0.21875$

### FP 運算硬體

- FP 加法器比整數加法器**複雜得多**
  - 需要移位指數和尾數、加法、正規化等步驟
- 若要在一個 clock cycle 內完成會太慢
  - 會拖慢所有指令的時脈
- FP 運算通常需要**多個 cycles**
  - 可以**流水線化** (pipelining)

FP 運算硬體通常支援:
- 加法、減法、乘法、除法、倒數、平方根
- FP ↔ 整數轉換

### 提高精度: Guard, Round, Sticky Bits

IEEE 754 指定額外的 rounding control 位元:
- **Guard bit**: 右邊第 1 個額外位元
- **Round bit**: 右邊第 2 個額外位元
- **Sticky bit**: 若 round bit 右邊有任何非零位元則設為 1

這些位元改善中間運算的精度。

#### 範例: $2.56 \times 10^0 + 2.34 \times 10^2 = 236.56$

**Without guard and round bit**:
- $0.02 \times 10^2 + 2.34 \times 10^2 = 2.36 \times 10^2$

**With guard and round bit**:
- $0.0256 \times 10^2 + 2.3400 \times 10^2 = 2.3656 \times 10^2 = 2.37 \times 10^2$

更接近正確答案!

### 四捨五入: Round to Nearest Even

**GRS (Guard, Round, Sticky) bits** 決定如何 round:
- **0xx**: <0.5, round down (不變)
- **101**: >0.5, round up
- **110**: >0.5, round up
- **111**: >0.5, round up
- **100**: 正好 0.5 ⇒ **round to nearest even** (若 Fraction 最右位元是 1 則 round up，否則 round down)

範例:
- `01 100` (GRS) ⇒ `10 000` (round up)
- `00 100` (GRS) ⇒ `00 000` (round down, to even)

## Fallacies and Pitfalls

### Fallacy: 右移 = 除以 2

**左移** i 位 = 乘以 $2^i$ ⇒ 右移 i 位 = 除以 $2^i$?

- 對於 **unsigned** 數，正確
- 對於 **signed** 數，錯誤!

範例: $-5 / 4 = -1$ 餘 $-1$

```
11111011₂ >> 2 = 00111110₂ = 62 (錯誤!)
```

正確作法需要考慮 sign extension。

### Pitfall: FP 加法不滿足結合律

$(x + y) + z$ 不一定等於 $x + (y + z)$!

範例:

| | (x+y)+z | x+(y+z) |
|---|---|---|
| x | -1.50E+38 | -1.50E+38 |
| y | 1.50E+38 | 1.50E+38 |
| z | 1.0 | 1.0 |
| Result | 1.00E+00 | 0.00E+00 |

- 平行程式可能以不同順序交錯運算
- 不能假設結合律成立
- 需要在不同平行度下驗證程式

## 總結

### 重要觀念

- **Bits 本身沒有意義** - 解釋取決於指令
- 電腦的數字表示有**有限範圍和精度**
- ISA 支援算術運算:
  - Signed 和 unsigned 整數
  - 浮點數 (對實數的近似)
- 運算可能 **overflow** 和 **underflow**
- FP 運算不完全遵循數學定律 (如結合律)
