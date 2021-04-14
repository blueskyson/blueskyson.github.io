---
title: "使用 c++ 計算行列式"
subtitle: ""
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - c++
---

## 降階公式

這是一個常用的行列式降階公式，[這裡](https://en.wikipedia.org/wiki/Laplace_expansion)有證明，雖然我是看不懂維基百科的證明啦，而且大一線代課的內容也忘得差不多了，之後有機會得再翻翻線代教科書好好複習一下了

![formula](https://wikimedia.org/api/rest_v1/media/math/render/svg/14f2f2a449d6d152ee71261e47551aa0a31c801e)

*圖片來源 [https://en.wikipedia.org/wiki/Determinant](https://en.wikipedia.org/wiki/Determinant)*

以上面的 3 × 3 行列式為例，依序取 a, b, c 為基準，並忽略 a, b, c 所在的行列，製造三個[子行列式](https://en.wikipedia.org/wiki/Minor_(linear_algebra))，也就是 $ \begin{vmatrix} e & h \\\\ f & i\end{vmatrix} $ 、 $ \begin{vmatrix} d & g \\\\ f & i\end{vmatrix} $ 、 $ \begin{vmatrix} d & g \\\\ e & h\end{vmatrix} $ 然後每個子行列式再分別乘上 a, b, c 以及乘上-1的(行加列)次方，例如 a 在第一行第一列，則 $ \begin{vmatrix} e & h \\\\ f & i\end{vmatrix} $ 就乘上 a 和 $ (-1)^{1 + 1} $ ，最後把所有降階後的行列式加起來。

推廣到 n × n 行列式:

$$ |A| = \begin{vmatrix}
A_{11} & A_{12} & A_{13} & ... & A_{1n} \\
A_{21} & A_{22} & A_{23} & ... & A_{2n} \\
A_{31} & A_{32} & A_{33} & ... & A_{3n} \\
. & . & . & & . \\
. & . & . & & . \\
. & . & . & & . \\
A_{n1} & A_{n2} & A_{n3} & ... & A_{nn}
\end{vmatrix} = \sum_{i=1}^{n} (-1)^{1 + i}*A_{1n}*M_{1,n} $$

其中 $ M_{1,n} $ 為 $ A_{1n} $ 所對應的子行列式。針對每個子行列式我們再以同樣的手法降階，直到所有行列式都被降為二階時就能直接透過公式計算結果了，很明顯看得出來透過遞迴會比較好撰寫這個程式。

## Pseudo Code

```non
function Det(A, n)
    if n == 1
        return A[0,0]
    else if n == 2
        return A[0,0] * A[1,1] - A[1,0] * A[0,1]
    end

    det <- 0                //the variable that stores result
    for a from 1 to n       //pick all elements in the first row
        M[,] <- the minor of first row and a(th) column
        if a is even
            det += A[0, a] * Det(M, n - 1)
        else
            det -= A[0, a] * Det(M, n - 1)
        end
    end

    return det
end
```

## 程式碼

```cpp
#include<iostream>
#include<iomanip>
using namespace std;

double** Matrix(int n, int m) {
    double** A = new double*[n];
    if (!A)
        return NULL;
    for (int i = 0; i < n; ++i) {
        A[i] = new double[m];
        if (!A[i])
            return NULL;
    }
    return A;
}

void DeleteMatrix(double** A, int n, int m) {
    for (int i = 0; i < n; ++i)
        delete[] A[i];
    delete[] A;
    A = NULL;
}

double Det(double** A, int n) {
    if (n == 1)
        return A[0][0];
    if (n == 2)
        return A[0][0] * A[1][1] - A[0][1] * A[1][0];

    double det = 0;
    double** M = Matrix(n - 1, n - 1);

    // for all elements in the 0th row
    for (int a = 0; a < n; ++a) {
        // create minor matrix
        for (int i = 1, minor_i = 0; i < n; ++i, ++minor_i) {
            for (int j = 0, minor_j = 0; j < n; ++j) {
                if (j == a)
                    continue;
                M[minor_i][minor_j++] = A[i][j];
            }
        }

        // add up terms
        if (a % 2 == 0)
            det += A[0][a] * Det(M, n - 1);
        else
            det -= A[0][a] * Det(M, n - 1);
    }
    DeleteMatrix(M, n - 1, n - 1);
    return det;
}

int main() {
    cout << fixed << setprecision(2);
    int n;
    cin >> n;
    double** A = Matrix(n, n);
    for (int i = 0; i < n; ++i)
        for (int j = 0; j < n; ++j)
            cin >> A[i][j];
    cout << "det(A) = "<< Det(A, n);
}
```

## 編譯執行

首先輸入行列式 A 的寬度，然後依序輸入行列式裡的元素，最後程式會印出答案

```non
> g++ det.cpp -o det
> det
5
1 2 3 4 1
0 -1 2 4 2
0 0 4 0 0
-3 -6 -9 -12 4
0 0 1 1 1
det(A) = 28.00
```
