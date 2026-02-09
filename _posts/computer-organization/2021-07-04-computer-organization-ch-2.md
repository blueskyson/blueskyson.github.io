---
title: "計組 Chapter 2"
subtitle: "Instructions: Language of the Computer"
excerpt: "computer organization 計組"
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - note
  - computer organization
---

## Instruction Set

**Instruction Set (指令集)** 是一台電腦所有指令的集合，不同的電腦有不同的指令集，但有很多共通的面向。指令集是硬體和軟體之間的介面 (Instruction Set Architecture, ISA)。

本章以 **MIPS** 指令集為例，MIPS 由 Stanford 開發，後由 MIPS Technologies 商業化，在嵌入式系統市場中占有很大的比例，應用在消費性電子、網路、儲存設備、相機、印表機等。各種組合語言其實非常相似，學會 MIPS 也有助於學習其他組合語言。

### 何時使用組合語言

- 當程式的**速度**或**大小**非常重要時，例如控制汽車煞車的電腦，需要快速且可預測地回應外界事件
- 利用特殊指令的能力，如 string copy 或 pattern-matching 指令

## 算術運算

MIPS 的算術運算都有三個運算元: 兩個 source 和一個 destination:

```python
add a, b, c   # a = b + c
sub a, b, c   # a = b - c
```

所有算術運算都遵循這個形式。

- *Design Principle 1:* **Simplicity favors regularity**
  - 規律性讓實作更簡單
  - 簡單性帶來更高的效能和更低的成本

範例: 將 C 程式 `f = (g + h) - (i + j);` 編譯為 MIPS:

```python
add $t0, $s1, $s2   # $t0 = g + h
add $t1, $s3, $s4   # $t1 = i + j
sub $s0, $t0, $t1   # f = (g+h) - (i+j)
```

其中 f, g, h, i, j 分別存在 \$s0, \$s1, \$s2, \$s3, \$s4。

## 暫存器運算元

算術指令使用**暫存器** (register) 運算元。MIPS 擁有 **32 個 32-bit** 的暫存器:
- 用於頻繁存取的資料
- 編號 0 到 31
- 32-bit 的資料稱為一個 **word**

- *Design Principle 2:* **Smaller is faster**
  - 較小的暫存器檔案讓運算更快
  - 暫存器比主記憶體 (有數百萬個位址) 快得多

### 暫存器命名慣例

| 名稱 | 編號 | 用途 | 呼叫時保留 |
|---|---|---|---|
| \$zero | 0 | 常數 0 | n.a. |
| \$v0-\$v1 | 2-3 | 回傳值和表達式結果 | no |
| \$a0-\$a3 | 4-7 | 引數 | no |
| \$t0-\$t7 | 8-15 | 暫時變數 | no |
| \$s0-\$s7 | 16-23 | 儲存變數 | yes |
| \$t8-\$t9 | 24-25 | 更多暫時變數 | no |
| \$gp | 28 | Global pointer | yes |
| \$sp | 29 | Stack pointer | yes |
| \$fp | 30 | Frame pointer | yes |
| \$ra | 31 | Return address | yes |

## 記憶體運算元

主記憶體用於複合資料 (陣列、結構、動態資料)。記憶體以 **byte** 為定址單位，每個位址對應一個 8-bit 的 byte。Word 在記憶體中必須**對齊**，位址必須是 4 的倍數。

### Big Endian 與 Little Endian

- **Big Endian**: Most-significant byte (MSB) 位於 word 的最小位址
- **Little Endian**: MSB 位於 word 的最大位址

例如資料 0x12FE34DC:

|  | addr 0 | addr 1 | addr 2 | addr 3 |
|---|---|---|---|---|
| Big Endian | 12 | FE | 34 | DC |
| Little Endian | DC | 34 | FE | 12 |

MIPS 是 **Big Endian**。

### Load 與 Store

將資料從記憶體載入暫存器:

```python
lw $s1, 20($s2)    # $s1 = Memory[$s2 + 20]
```

其中 \$s2 是 base register，20 是 offset。

將資料從暫存器存入記憶體:

```python
sw $s1, 20($s2)    # Memory[$s2 + 20] = $s1
```

### 記憶體運算元範例

C 程式: `g = h + A[8];`，其中 g 在 \$s1，h 在 \$s2，A 的 base address 在 \$s3:

```python
lw  $t0, 32($s3)   # 載入 A[8]，index 8 的 offset 為 8×4=32
add $s1, $s2, $t0  # g = h + A[8]
```

C 程式: `A[12] = h + A[8];`，其中 h 在 \$s2，A 的 base address 在 \$s3:

```python
lw  $t0, 32($s3)        # 載入 A[8]
add $t0, $s2, $t0       # $t0 = h + A[8]
sw  $t0, 48($s3)        # 存入 A[12]，offset 為 12×4=48
```

### 暫存器 vs. 記憶體

- 暫存器比記憶體**快** (記憶體容量大得多)
- 對記憶體資料操作需要 load 和 store 指令，雖然需要更多指令，但不需要同時處理暫存器和記憶體資料，簡化了設計
- 編譯器應盡量使用暫存器存放變數，不常用的變數才 spill 到記憶體
- 暫存器優化非常重要

## 常數或立即運算元

常數資料直接寫在指令中:

```python
addi $s3, $s3, 1    # $s3 = $s3 + 1
```

MIPS 沒有 subtract immediate 指令，可以用負常數替代:

```python
addi $s3, $s3, -1   # $s3 = $s3 - 1
```

- *Design Principle 3:* **Make the common case fast**
  - 加 1 和減 1 是非常常見的運算
  - 小常數很常見
  - 立即運算元避免了額外的 load 指令

### 常數 Zero

MIPS **register 0 (\$zero)** 的值永遠是 0，不可被修改。可用於常見的操作，例如暫存器之間的複製:

```python
add $t2, $s1, $zero   # $t2 = $s1
```

## 有號與無號數

### Unsigned Binary Integers

給定一個 n-bit 的數:

$$x = x_{n-1} \times 2^{n-1} + x_{n-2} \times 2^{n-2} + \cdots + x_1 \times 2^1 + x_0 \times 2^0$$

範圍: 0 到 $+2^n - 1$。使用 32 bits 時，範圍為 0 到 +4,294,967,295。

### 2's Complement Signed Integers

給定一個 n-bit 的數:

$$x = -x_{n-1} \times 2^{n-1} + x_{n-2} \times 2^{n-2} + \cdots + x_1 \times 2^1 + x_0 \times 2^0$$

範圍: $-2^{n-1}$ 到 $+2^{n-1} - 1$。使用 32 bits 時，範圍為 -2,147,483,648 到 +2,147,483,647。

- Bit 31 是 sign bit: 1 代表負數，0 代表非負數
- $-(- 2^{n-1})$ 無法表示
- 非負數在 unsigned 和 2's complement 中的表示方式相同

一些特殊數:
- 0: `0000 0000 ... 0000`
- -1: `1111 1111 ... 1111`
- 最小負數: `1000 0000 ... 0000`
- 最大正數: `0111 1111 ... 1111`

### Signed Negation

將 n 轉換成 -n: **取補數 (complement) 再加 1**，即 1→0, 0→1 後再 +1。

範例: 取 +2 的相反數:
- +2 = `0000 0000 ... 0010`
- 補數 = `1111 1111 ... 1101`
- +1 = `1111 1111 ... 1110` = -2

原理: 因為 $x + \bar{x} = 1111...111_2 = -1$，所以 $\bar{x} + 1 = -x$。

### Sign Extension

當需要用更多位元表示一個數但仍然保留其數值時:
- **Signed**: 將 sign bit 複製到左邊
  - +2: `0000 0010` → `0000 0000 0000 0010`
  - -2: `1111 1110` → `1111 1111 1111 1110`
- **Unsigned**: 用 0 擴展

Sign extension 在 MIPS 中用於:
- `addi`: 擴展 immediate value
- `lb`, `lh`: 擴展載入的 byte/halfword
- `beq`, `bne`: 擴展 displacement

## 指令的表示法

指令以**二進位** (binary) 編碼，稱為**機器碼** (machine code)。MIPS 指令編碼為 **32-bit** 的指令字。

常用暫存器的編號:
- \$t0–\$t7 對應 reg 8–15
- \$t8–\$t9 對應 reg 24–25
- \$s0–\$s7 對應 reg 16–23

### R-format 指令

| op | rs | rt | rd | shamt | funct |
|---|---|---|---|---|---|
| 6 bits | 5 bits | 5 bits | 5 bits | 5 bits | 6 bits |

- **op**: operation code (opcode)，指明操作類型
- **rs**: first source register number
- **rt**: second source register number
- **rd**: destination register number
- **shamt**: shift amount (用於移位指令)
- **funct**: function code，擴展 opcode，指定運算的變體

範例: `add $t0, $s1, $s2`，其中 \$s1=r17, \$s2=r18, \$t0=r8:

| op | rs | rt | rd | shamt | funct |
|---|---|---|---|---|---|
| 0 | 17 | 18 | 8 | 0 | 32 |
| 000000 | 10001 | 10010 | 01000 | 00000 | 100000 |

機器碼: `00000010001100100100000000100000₂` = `02324020₁₆`

### I-format 指令

| op | rs | rt | constant or address |
|---|---|---|---|
| 6 bits | 5 bits | 5 bits | 16 bits |

用於 immediate 算術和 load/store 指令:
- **rt**: destination 或 source register number
- **constant**: $-2^{15}$ 到 $+2^{15} - 1$
- **address**: offset 加上 rs 中的 base address

範例:

```python
addi $s3, $s3, 1    # $s3 = $s3 + 1
```

| op | rs | rt | constant |
|---|---|---|---|
| 8 | 19 | 19 | 1 |

```python
lw $t0, 32($s3)     # $t0 = MEM[$s3 + 32]
```

| op | rs | rt | address |
|---|---|---|---|
| 35 | 19 | 8 | 32 |

```python
sw $t0, 32($s3)     # MEM[$s3 + 32] = $t0
```

| op | rs | rt | address |
|---|---|---|---|
| 43 | 19 | 8 | 32 |

- *Design Principle 4:* **Good design demands good compromises**
  - R-format 和 I-format 前 16 bits 的結構相同 (op, rs, rt)，保持規律性
  - R-format 將後 16 bits 分成 rd, shamt, funct
  - I-format 將後 16 bits 合成一個 address/constant 欄位

### 十六進位

十六進位 (Base 16) 是二進位字串的簡潔表示法，每 4 bits 對應一個 hex digit:

| 0 | 0000 | 4 | 0100 | 8 | 1000 | c | 1100 |
|---|---|---|---|---|---|---|---|
| 1 | 0001 | 5 | 0101 | 9 | 1001 | d | 1101 |
| 2 | 0010 | 6 | 0110 | a | 1010 | e | 1110 |
| 3 | 0011 | 7 | 0111 | b | 1011 | f | 1111 |

## 邏輯運算

| 操作 | C | Java | MIPS |
|---|---|---|---|
| Shift left | `<<` | `<<` | `sll` |
| Shift right | `>>` | `>>` | `srl` |
| Bitwise AND | `&` | `&` | `and`, `andi` |
| Bitwise OR | `\|` | `\|` | `or`, `ori` |
| Bitwise NOT | `~` | `~` | `nor` |

邏輯運算用於**擷取**和**插入**一個 word 中的特定位元群組。

### Shift 運算

**Shift left logical (sll)**: 向左移位，右邊補 0。左移 i bits 等於乘以 $2^i$。

```python
sll $t2, $s0, 2   # $t2 = $s0 << 2
```

sll 的指令格式: op=0, funct=0，shamt 表示移位量。

| op | rs | rt | rd | shamt | funct |
|---|---|---|---|---|---|
| 0 | 0 | 16 | 10 | 2 | 0 |

**Shift right logical (srl)**: 向右移位，左邊補 0。右移 i bits 等於除以 $2^i$ (僅限 unsigned)。

```python
srl $t2, $s0, 2   # $t2 = $s0 >> 2
```

srl 的指令格式: op=0, funct=2。

### AND 運算

用於 **mask** (遮罩) word 中的位元，選取特定位元，其餘清為 0:

```python
and $t0, $t1, $t2
```

AND 的指令格式: op=0, funct=100100₂。

### OR 運算

用於 **include** (納入) word 中的位元，設定特定位元為 1，其餘不變:

```python
or $t0, $t1, $t2
```

OR 的指令格式: op=0, funct=100101₂。

### NOT 運算

用於**反轉** word 中的所有位元。MIPS 使用 **NOR** 三運算元指令來實現 NOT:

```python
nor $t0, $t1, $zero   # $t0 = NOT($t1)
```

因為 `a NOR 0 = NOT(a OR 0) = NOT(a)`。

## 條件判斷

根據條件跳轉至標記的指令，否則繼續循序執行:

- **beq rs, rt, L1**: 若 rs == rt，跳轉到 L1
- **bne rs, rt, L1**: 若 rs != rt，跳轉到 L1
- **j L1**: 無條件跳轉到 L1

### 編譯 if 語句

C 程式:

```c
if (i == j)
    f = g + h;
else
    f = g - h;
```

f, g, h, i, j 分別在 \$s0–\$s4，編譯為:

```python
      bne $s3, $s4, Else   # if (i != j) goto Else
      add $s0, $s1, $s2    # f = g + h
      j   Exit
Else: sub $s0, $s1, $s2    # f = g - h
Exit: ...
```

### 編譯 while 迴圈

C 程式: `while (save[i] == k) i += 1;`，其中 i 在 \$s3，k 在 \$s5，save 的 base address 在 \$s6:

```python
Loop: sll  $t1, $s3, 2      # $t1 = i * 4
      add  $t1, $t1, $s6    # address of save[i]
      lw   $t0, 0($t1)      # load save[i]
      bne  $t0, $s5, Exit   # if save[i] != k, exit
      addi $s3, $s3, 1      # i += 1
      j    Loop
Exit: ...
```

### Basic Blocks

**Basic block** 是一段連續的指令序列:
- 除了結尾以外沒有分支指令
- 除了開頭以外沒有分支目標

編譯器會辨識 basic blocks 進行優化，進階處理器也可以加速 basic block 的執行 (例如將頻繁存取的資料保留在暫存器中)。

### slt 與 slti

**slt (set on less than)**: 若條件為 true，設結果為 1，否則設為 0:

```python
slt $t0, $s1, $s2    # if ($s1 < $s2) $t0 = 1; else $t0 = 0
```

**slti (set less than immediate)**:

```python
slti $t0, $s1, 10    # if ($s1 < 10) $t0 = 1; else $t0 = 0
```

配合 beq、bne 使用，可以建構出各種比較條件:

```python
# if ($s1 < $s2) branch to L
slt $t0, $s1, $s2
bne $t0, $zero, L
```

### 為什麼沒有 blt 和 bgt?

因為硬體實作 <, ≤, >, ≥ 比 = 和 ≠ 更慢，需要做 2's complement 減法或 xor。將 <, ≤, >, ≥ 與 branch 結合會使每條指令的工作量增加，導致時脈變慢，所有指令都會受到影響。beq 和 bne 是最常見的 case，因此不需要額外的 blt 和 bgt 指令，這是良好的設計折衷。

### Signed vs. Unsigned 比較

- **Signed** 整數比較使用: `slt`, `slti`
- **Unsigned** 整數比較使用: `sltu`, `sltiu`

範例:
- \$s0 = `1111 1111 1111 1111 1111 1111 1111 1111`
- \$s1 = `0000 0000 0000 0000 0000 0000 0000 0001`

```python
slt  $t0, $s0, $s1   # signed:   $t0 = 1 (因為 -1 < +1)
sltu $t0, $s0, $s1   # unsigned: $t0 = 0 (因為 4,294,967,295 > +1)
```

## 程序呼叫

執行程序 (procedure) 的步驟:
1. 將參數放入暫存器 (\$a0–\$a3)
2. 轉移控制權給程序 (使用 `jal`)
3. 取得程序所需的儲存空間 (儲存將使用的暫存器內容)
4. 執行程序的運算
5. 將結果放入暫存器供呼叫者使用 (\$v0, \$v1)
6. 還原儲存的暫存器，返回呼叫處 (使用 `jr $ra`)

### Program Counter

**Program Counter (PC)** 是一個 32-bit 暫存器，用於指向下一條即將執行的指令。

### Procedure Call 指令

**jal (jump and link)**: 呼叫程序

```python
jal Procedure_Label
```

- 跳轉到目標位址
- 將下一條指令的位址存入 \$ra (return address)

**jr (jump register)**: 從程序返回

```python
jr $ra
```

- 跳轉到 \$ra 儲存的位址

### Leaf Procedure 範例

C 程式:

```c
int leaf_example(int g, int h, int i, int j)
{
    int f;
    f = (g + h) - (i + j);
    return f;
}
```

引數 g, h, i, j 在 \$a0–\$a3，f 在 \$s0。程序中會使用 \$s0, \$t0, \$t1 三個暫存器，需要先保存:

```python
leaf_example:
    addi $sp, $sp, -12     # 調整 stack，空出 3 個 word
    sw   $t1, 8($sp)       # 保存 $t1
    sw   $t0, 4($sp)       # 保存 $t0
    sw   $s0, 0($sp)       # 保存 $s0
    add  $t0, $a0, $a1     # $t0 = g + h
    add  $t1, $a2, $a3     # $t1 = i + j
    sub  $s0, $t0, $t1     # f = (g+h) - (i+j)
    add  $v0, $s0, $zero   # 回傳值 = f
    lw   $s0, 0($sp)       # 還原 $s0
    lw   $t0, 4($sp)       # 還原 $t0
    lw   $t1, 8($sp)       # 還原 $t1
    addi $sp, $sp, 12      # 還原 stack
    jr   $ra               # 返回呼叫者
```

由於 \$t0, \$t1 是由 caller 負責維護的暫時暫存器，callee 不需要保存和還原它們，因此可以優化為:

```python
leaf_example:
    addi $sp, $sp, -4      # 調整 stack，只需保存 $s0
    sw   $s0, 0($sp)       # 保存 $s0
    add  $t0, $a0, $a1     # $t0 = g + h
    add  $t1, $a2, $a3     # $t1 = i + j
    sub  $s0, $t0, $t1     # f = (g+h) - (i+j)
    add  $v0, $s0, $zero   # 回傳值 = f
    lw   $s0, 0($sp)       # 還原 $s0
    addi $sp, $sp, 4       # 還原 stack
    jr   $ra               # 返回呼叫者
```

### Leaf 與 Nested Procedures

- **Leaf procedure**: 不呼叫其他程序的程序，只有 callee 的角色
- **Nested (non-leaf) procedure**: 會呼叫其他程序的程序，同時扮演 caller 和 callee

### 暫存器保存慣例

- **Caller** 負責保存: \$a0–\$a3, \$t0–\$t9
- **Callee** 負責保存: \$s0–\$s7, \$ra

若 callee 是 leaf procedure，不需要保存 \$ra。若 callee 是 non-leaf procedure (還會呼叫其他程序)，`jal` 會覆蓋 \$ra，所以必須保存 \$ra。

| 呼叫時保留 (Callee 負責) | 不保留 (Caller 負責) |
|---|---|
| \$s0–\$s7 | \$t0–\$t9 |
| \$sp | \$a0–\$a3 |
| \$ra | \$v0–\$v1 |
| \$fp, \$gp | stack pointer 以下的空間 |
| stack pointer 以上的空間 | |

### Non-leaf Procedure 範例

C 程式 (遞迴計算階乘):

```c
int fact(int n)
{
    if (n < 1)
        return 1;
    else
        return n * fact(n - 1);
}
```

引數 n 在 \$a0:

```python
fact:
    addi $sp, $sp, -8      # 調整 stack，空出 2 個 word
    sw   $ra, 4($sp)       # 保存 return address
    sw   $a0, 0($sp)       # 保存 argument n
    slti $t0, $a0, 1       # test for n < 1
    beq  $t0, $zero, L1    # if n >= 1, goto L1
    addi $v0, $zero, 1     # return 1
    addi $sp, $sp, 8       # pop stack
    jr   $ra               # return
L1: addi $a0, $a0, -1      # n - 1
    jal  fact              # recursive call
    lw   $a0, 0($sp)       # restore original n
    lw   $ra, 4($sp)       # restore return address
    addi $sp, $sp, 8       # pop stack
    mul  $v0, $a0, $v0     # n * fact(n-1)
    jr   $ra               # return
```

### MIPS Memory Layout

| 區段 | 位址 | 說明 |
|---|---|---|
| Stack | \$sp → 7fff fffc₁₆ | 自動儲存，向下成長 |
| Dynamic data (Heap) | | malloc (C), new (Java)，向上成長 |
| Static data | \$gp → 1000 8000₁₆ | 靜態變數、常數陣列和字串，起始於 1000 0000₁₆ |
| Text | pc → 0040 0000₁₆ | 程式碼 |
| Reserved | 0 | 保留區 |

### Stack Allocation 與 Frame Pointer

Stack 在程序呼叫時配置。由於 \$sp 在程序執行過程中可能改變，用 \$sp 的 offset 存取區域變數會比較不方便。因此使用 **frame pointer (\$fp)** 指向 procedure frame (activation record) 的第一個 word，作為穩定的 base register 來存取區域資料。
