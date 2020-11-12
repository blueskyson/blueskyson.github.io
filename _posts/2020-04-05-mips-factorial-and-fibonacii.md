---
title: "計算機組織: mips 實作階乘和費氏數列"
subtitle: ""
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - mips
---

## Factorial

$ F(n) = \begin{cases}  1, & n\mbox{ = 0} \\\\ F(n - 1) * n, & n \in \mathbb{N} \end{cases} $

使用 `$a0` 做為參數，以這個程式為例，令 `$a0` 等於 5 然後跳到 `fact` 標籤就表示 F(3)，在遞迴結束時，計算結果會顯示在 `$v0`

```python
main:
    addi $a0, $zero, 5        # let the argument be 5
    jal fact                  # jump to fib label, like calling F(5)
    j exit

fact:
    addi $sp, $sp, -8         # allocate 8 bytes to this stack
    sw $ra, 0($sp)            # save return address in stack
    sw $a0, 4($sp)            # save argument value in stack

    slti $t0, $a0, 1          # if n < 1 then $t0 = 1, else $t0 = 0
    beq $t0, $zero, L1        # if $t0 == 0 then jump to branch L1
    addi $v0, $zero, 1        # let $v0 be 1
    addi $sp, $sp, 8          # let $sp point to upper stack
    jr $ra                    # jump to the next line of the line calling fib

L1:
    addi $a0, $a0, -1         # n = n - 1
    jal fact                  # jump fact again, like as returning F(n - 1)
    lw $a0, 4($sp)            # recover the value of argument
    mul $v0, $a0, $v0         # $v0 *= $a0, like as F(n) = n * F(n - 1)
    lw $ra, 0($sp)            # recover return address
    addi, $sp, $sp, 8         # let $sp point to upper stack
    jr $ra                    # jump to the next line of the line calling L1

exit:
```

在 MARS 模擬器的結果長這樣

[![mips_factorial](https://raw.githubusercontent.com/blueskyson/image-host/master/mips_factorial.JPG)](https://raw.githubusercontent.com/blueskyson/image-host/master/mips_factorial.JPG)

## Fibonacci

$ F(n) = \begin{cases}  n, & n\mbox{ = 0, 1} \\\\ F(n - 1) + F(n - 2), & n \in \mathbb{N} \end{cases} $

由 factorial 的程式修改而來，基本上就是在 L1 標籤裡把 `$a0` 減一然後跳到 `fib` 標籤，重複兩次，在 `$a0` 等於 0 或 1 時讓 `$v0` 加上 `$a0` 就完成了

```python
main:
    addi $a0, $zero, 10       # let the argument be 10
    jal fib                   # jump to fib label, like calling F(10) in c++
    j exit

fib:
    addi $sp, $sp, -8         # allocate 8 bytes to stack
    sw $ra, 0($sp)            # save return address in stack
    sw $a0, 4($sp)            # save argument value in stack

    slti $t0, $a0, 2          # if n < 2 then $t0 = 1, else $t0 = 0
    beq $t0, $zero, L1        # if $t0 == 0 then jump to branch L1
    add $v0, $v0, $a0         # let $v0 add $a0
    addi $sp, $sp, 8          # let $sp point to upper stack
    jr $ra                    # jump to the next line of the line calling fib

L1:
    addi $a0, $a0, -1         # n = n - 1
    jal fib                   # return f(n - 1)
    addi $a0, $a0, -1         # n = n - 1
    jal fib                   # return f(n - 2)
    lw $a0, 4($sp)            # recover the value of argument
    lw $ra, 0($sp)            # recover return address
    addi, $sp, $sp, 8         # let $sp point to upper stack
    jr $ra                    # jump to the next line of the line calling L1
exit:
```

[![mips_fibonacci](https://raw.githubusercontent.com/blueskyson/image-host/master/mips_fibonacci.JPG)](https://raw.githubusercontent.com/blueskyson/image-host/master/mips_fibonacci.JPG)
