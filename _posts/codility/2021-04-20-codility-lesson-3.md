---
title: "Codility Lesson 3 FrogJmp, PermMissingElem"
subtitle: ""
excerpt: "codility frogjmp"
layout: post
author: "blueskyson"
mathjax: false
header-style: text
tags:
  - codility
  - c++
---

## FrogJmp

A small frog wants to get to the other side of the road. The frog is currently located at position X and wants to get to a position greater than or equal to Y. The small frog always jumps a fixed distance, D.

Count the minimal number of jumps that the small frog must perform to reach its target.

Write a function:

```non
int solution(int X, int Y, int D);
```

that, given three integers X, Y and D, returns the minimal number of jumps from position X to a position equal to or greater than Y.

For example, given:

&nbsp;X = 10
&nbsp;Y = 85
&nbsp;D = 30

the function should return 3, because the frog will be positioned as follows:
- after the first jump, at position 10 + 30 = 40
- after the second jump, at position 10 + 30 + 30 = 70
- after the third jump, at position 10 + 30 + 30 + 30 = 100

Write an efficient algorithm for the following assumptions:
- X, Y and D are integers within the range [1..1,000,000,000];
- X ≤ Y.
  
### C++ Solution

```cpp
int solution(int X, int Y, int D) {
    int d = Y - X;
    int q = d / D;
    int r = d % D;
    return (r > 0) ? q + 1 : q;
}
```

## PermMissingElem

An array A consisting of N different integers is given. The array contains integers in the range [1..(N + 1)], which means that exactly one element is missing.

Your goal is to find that missing element.

Write a function:

```non
int solution(vector<int> &A);
```

that, given an array A, returns the value of the missing element.

For example, given array A such that:

&nbsp;A[0] = 2
&nbsp;A[1] = 3
&nbsp;A[2] = 1
&nbsp;A[3] = 5

the function should return 4, as it is the missing element.

Write an efficient algorithm for the following assumptions:
- N is an integer within the range [0..100,000];
- the elements of A are all distinct;
- each element of array A is an integer within the range [1..(N + 1)].

### C++ Solution

```cpp
#include <unordered_set>
int solution(vector<int> &a) {
    unordered_set<int> set;
    for (unsigned i = 0; i < a.size(); i++) {
        set.insert(a[i]);
    }
    for (unsigned i = 1; i <= a.size() + 1; i++) {
        if (set.find(i) == set.end()) {
            return i;
        }
    }
}
```

## TapeEquilibrium

A non-empty array A consisting of N integers is given. Array A represents numbers on a tape.

Any integer P, such that 0 < P < N, splits this tape into two non-empty parts: A[0], A[1], ..., A[P − 1] and A[P], A[P + 1], ..., A[N − 1].

The difference between the two parts is the value of: |(A[0] + A[1] + ... + A[P − 1]) − (A[P] + A[P + 1] + ... + A[N − 1])|

In other words, it is the absolute difference between the sum of the first part and the sum of the second part.

For example, consider array A such that:

&nbsp;A[0] = 3
&nbsp;A[1] = 1
&nbsp;A[2] = 2
&nbsp;A[3] = 4
&nbsp;A[4] = 3
We can split this tape in four places:

- P = 1, difference = |3 − 10| = 7
- P = 2, difference = |4 − 9| = 5
- P = 3, difference = |6 − 7| = 1
- P = 4, difference = |10 − 3| = 7
Write a function:

```non
int solution(vector<int> &A);
```

that, given a non-empty array A of N integers, returns the minimal difference that can be achieved.

For example, given:

&nbsp;A[0] = 3
&nbsp;A[1] = 1
&nbsp;A[2] = 2
&nbsp;A[3] = 4
&nbsp;A[4] = 3

the function should return 1, as explained above.

Write an efficient algorithm for the following assumptions:

- N is an integer within the range [2..100,000];
- each element of array A is an integer within the range [−1,000..1,000].

### C++ Solution

```cpp
#include <climits>
#include <cmath>
int solution(vector<int> &a) {
    int left = a[0], right = 0;
    int min = INT_MAX;
    for (unsigned i = 1; i < a.size(); i++) {
        right += a[i];
    }
    for (unsigned i = 1; i < a.size(); i++) {
        int val = abs(left - right);
        if (val < min) {
            min = val;
        }
        left += a[i];
        right -= a[i];
    }
    return min;
}
```