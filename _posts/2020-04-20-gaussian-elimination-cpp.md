---
title: "使用 c++ 實作高斯消去"
subtitle: ""
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - c++
---

因為 COVID-19 疫情，線代的小考改為寫程式實作一些常用的矩陣運算，雖然網路上應該有很多完整又高效的函式庫可以用，但我覺得親手實作一次也算是個很好的練習。這篇文章將紀錄如何使用 c++ 做高斯消去，而之後做 PLU 分解時，就會再次利用這段 code 求上三角矩陣。

## 高斯消去

一般高斯消去時是使用增廣矩陣，但這份程式原本的用途是求上三角矩陣，所以這裡以普通的矩陣作為例子。在下面的例子中，我們會將高斯消去的過程分解:

$$
\begin{split}
A = \begin{bmatrix} 1 & 2 & 3 \\ 0 & 0 & 7 \\ 9 & 8 & 7 \\ 5 & 4 & 6 \end{bmatrix} \Rightarrow
\begin{bmatrix} 1 & 2 & 3 \\ 0 & 0 & 7 \\ 0 & -10 & -20 \\ 0 & -6 & -9 \end{bmatrix} \Rightarrow
\begin{bmatrix} 1 & 2 & 3 \\ 0 & -10 & -20 \\ 0 & 0 & 7 \\ 0 & -6 & -9 \end{bmatrix} \\
\Rightarrow \begin{bmatrix} 1 & 2 & 3 \\ 0 & -10 & -20 \\ 0 & 0 & 7 \\ 0 & 0 & 3 \end{bmatrix}
\Rightarrow \begin{bmatrix} 1 & 2 & 3 \\ 0 & -10 & -20 \\ 0 & 0 & 7 \\ 0 & 0 & 0 \end{bmatrix}
\end{split} $$

1. 以 $ A_{11} $ 作為 pivot ，第二橫列的第1個數已經為0了，所以跳過。將第一橫列乘以 -9 加上第三橫列、再將第一橫列乘以 -5 加上第四橫列。

2. 因為 $ A_{22} $ 為零，無法作為 pivot 去消除同一直行的數，所以我們選擇將第二列和第三列交換。

   <span style="color: CornflowerBlue">
   注意交換兩列後可能需要產生一個置換矩陣來紀錄交換前的橫列的順序，後續會提到置換矩陣。
   </span>
   {: .notice--info}

3. 以 $ A_{22} $ 作為 pivot ，將第二橫列乘以 $ -\frac{3}{5} $ 加上第四橫列。

4. 將第三橫列作為 pivot 將後面的橫列都消掉。

## 標準化步驟

簡單說明完筆算的規則後，接下來說明如何標準化這個流程，然後寫成程式。首先我們建立一個矩陣 A ，大小為 n * m ，矩陣中的每個數字定義為 $ A_{ij} $ ，如下:

$$ \begin{bmatrix}
A_{11} & A_{12} & A_{13} & ... & A_{1m} \\
A_{21} & A_{22} & A_{23} & ... & A_{2m} \\
A_{31} & A_{32} & A_{33} & ... & A_{3m} \\
. & . & . & & . \\
. & . & . & & . \\
. & . & . & & . \\
A_{n1} & A_{n2} & A_{n3} & ... & A_{nm}
\end{bmatrix} $$

第一個要考慮的問題是 n 和 m 的大小:

* 若 n > m, 只需做 m 次消去，第 m+1 列到第 n 列都會變成 0

* 若 n == m, 一樣只需要做 n 次消去，在沒有缺失 pivot 的狀況下是倒三角形

* 若 n < m, 便只需要做 n 次消去就好了，做完消去後，右上區域會呈現類似倒梯形

知道每個矩陣要高斯消去的次數後，再來就開始要執行消去了，現在我們以 $ A_{11} $ 作為 pivot:

由筆算的規則可以看出，我們會這樣消去第二列:

$$ \begin{bmatrix} A_{21} - A_{11}×\frac{A_{21}}{A_{11}} & A_{22} - A_{12}×\frac{A_{21}}{A_{11}} & A_{23} - A_{13}×\frac{A_{21}}{A_{11}} & ... & A_{2m} - A_{1m}×\frac{A_{21}}{A_{11}} \end{bmatrix} $$

同理，我們會這樣消去第三列:

$$ \begin{bmatrix} A_{31} - A_{11}×\frac{A_{31}}{A_{11}} & A_{32} - A_{12}×\frac{A_{31}}{A_{11}} & A_{33} - A_{13}×\frac{A_{31}}{A_{11}} & ... & A_{3m} - A_{1m}×\frac{A_{31}}{A_{11}} \end{bmatrix} $$

由上面兩個算式，可以歸納出以 $ A_{11} $ 當做 pivot 做高斯消去時，第 i 橫列第 j 直行的數字會等於以下:

$$ A_{ij} - A_{1j}×\frac{A_{i1}}{A_{11}} $$

再經過推廣，令 $ 1 \leq p \leq min\\{m, n\\} $ ， $ A_{pp} $ 為任意 pivot ，以 $ A_{pp} $ 做高斯消去時，第 i 橫列第 j 直行的數字會等於以下:

$$ A_{ij} - A_{1j}×\frac{A_{i1}}{A_{pp}} $$

基本上推到這裡，就可以開始寫程式了，然而有個特例，那就是當 $ A_{pp} = 0 $ 時，需要往下找其他列交換，再做高斯消去。若此時強行做高斯消去，會發生除以 0 的錯誤，例如下面這種狀況:

$$ \begin{bmatrix}
a & b & c & d \\
0 & 0 & f & g \\
0 & h & i & 0 \\
0 & j & 0 & k \\
\end{bmatrix} $$

此時 pivot 為 $ A_{22} = 0 $ ，無法以 A_{22} 消除 h 、 j ，所以要將第三列跟第二列對調再進行高斯消去:

$$ \Rightarrow \begin{bmatrix}
a & b & c & d \\
0 & h & i & 0 \\
0 & 0 & f & g \\
0 & j & 0 & k \\
\end{bmatrix} $$

若很不巧， h 和 j 也恰巧是 0 ，則跳過此輪高斯消去。

## 置換矩陣

置換矩陣主要是用來表達對一個矩陣翻轉、交換、對調等行為，通常以 P 表示，置換矩陣裡的所有元素都是 1 或 0 ，下面的例子是對 $ A $ 交換一二列的置換矩陣:

$$ P×A = \begin{bmatrix}
0 & 1 & 0 & 0 \\
1 & 0 & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1 \\
\end{bmatrix} × \begin{bmatrix}
a & b & c \\
d & e & f \\
g & h & i \\
j & k & l \\
\end{bmatrix} = \begin{bmatrix}
d & e & f \\
a & b & c \\
g & h & i \\
j & k & l \\
\end{bmatrix} $$

用程式算出置換矩陣就很容易了，首先初始化一個 n × n 的單位矩陣，在做高斯消去時，矩陣 $ A $ 交換哪兩列，矩陣 $ P $ 就跟著交換那兩列。

## Pseudo Code

In GE (gaussian elimination) function:

```non
P[,] <- n × n identity matrix
A[,] <- n × m matrix

// determine how many time to perform GE
if n > m
    ge_level <- m
else
    ge_level <- n
end

// perform GE
for p from 1 to ge_level

    // pick a pivot that unequals to 0
    pivot <- A[p,p]
    count <- p
    while pivot == 0 and count <= n
        count <- count + 1
        pivot <- A[count, p]
    end

    // if all pivot equals 0, skip elimination of this column
    if count > n
        continue
    end

    // perform row exchange
    if count != p
        temp[] <- A[p, 1 to m]
        A[p, 1 to m] <- A[count, 1 to m]
        A[count, 1 to m] <- temp[1 to m]

        temp[] <- P[p, 1 to m]
        P[p, 1 to m] <- temp[count, 1 to m]
        P[count, 1 to m] <- temp[1 to m]

    // start GE of this column
    for i (p to m)
        elim <- A[i, p];
        for j (p to m)
            A[i, j] <- A[i][j] - (A[p, j] * elim) / pivot
        end
    end
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

void PrintMatrix(double** A, int n, int m) {
    for (int i = 0; i < n; ++i) {
        for (int j = 0; j < m; ++j)
            cout << setw(6) << A[i][j];
        cout << endl;
    }
    cout << endl;
}

void GE(double** P, double** A, int n, int m) {
    int ge_level;
    if (n < m)
        ge_level = n;
    else
        ge_level = m;

    for (int p = 0; p < ge_level; p++) {
        double pivot = A[p][p];
        int count = p, tmp;
        while (pivot == 0 && count < n)
            pivot = A[++count][p];

        if (count == n)
            continue;

        if (count != p) {
            double* temp = A[p];
            A[p] = A[count];
            A[count] = temp;

            temp = P[p];
            P[p] = P[count];
            P[count] = temp;
        }

        for (int i = p + 1; i < n; ++i) {
            double elim = A[i][p];
            for (int j = p; j < m; ++j)
                A[i][j] = A[i][j] - (A[p][j] * elim) / pivot;
        }
    }
}

int main() {
    cout << fixed << setprecision(2);
    int n, m;
    cin >> n >> m;

    // initialize P as a identity matrix
    double** P = Matrix(n, n);
    for (int i = 0; i < n; ++i) {
        for (int j = 0; j < n; ++j)
            P[i][j] = 0;
        P[i][i] = 1;
    }

    // read in A
    double** A = Matrix(n, m);
    for (int i = 0; i < n; ++i)
        for (int j = 0; j < m; ++j)
            cin >> A[i][j];

    GE(P, A, n, m);
    cout << endl << "P =" << endl;
    PrintMatrix(P, n, n);
    cout << "A =>" << endl;
    PrintMatrix(A, n, m);

    DeleteMatrix(A, n, m);
    DeleteMatrix(P, n, n);
    return 0;
}
```

## 編譯執行

這是在 windows 上執行的，所以我沒有打 "./"。執行後先輸入 m 和 n 的值，然後輸入矩陣 $ A $ ，最後會印出轉置矩陣和執行高斯消去的結果。

```non
> g++ ge.cpp -o ge
> ge
3 3
1 2 0
1 2 1
0 2 1
P =
  1.00  0.00  0.00
  0.00  0.00  1.00
  0.00  1.00  0.00

A =>
  1.00  2.00  0.00
  0.00  2.00  1.00
  0.00  0.00  1.00
```
