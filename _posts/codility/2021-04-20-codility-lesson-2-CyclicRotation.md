---
title: "Codility Lesson 2 CyclicRotation, OddOccurrencesInArray"
subtitle: ""
excerpt: "codility cyclicrotation"
layout: post
author: "blueskyson"
mathjax: false
header-style: text
tags:
  - codility
  - c++
---

## CyclicRotation

An array A consisting of N integers is given. Rotation of the array means that each element is shifted right by one index, and the last element of the array is moved to the first place. For example, the rotation of array A = [3, 8, 9, 7, 6] is [6, 3, 8, 9, 7] (elements are shifted right by one index and 6 is moved to the first place).

The goal is to rotate array A K times; that is, each element of A will be shifted to the right K times.

Write a function:

```non
vector<int> solution(vector<int> &A, int K);
```

that, given an array A consisting of N integers and an integer K, returns the array A rotated K times.

For example, given

```non
    A = [3, 8, 9, 7, 6]
    K = 3
```

the function should return [9, 7, 6, 3, 8]. Three rotations were made:

```non
    [3, 8, 9, 7, 6] -> [6, 3, 8, 9, 7]
    [6, 3, 8, 9, 7] -> [7, 6, 3, 8, 9]
    [7, 6, 3, 8, 9] -> [9, 7, 6, 3, 8]
```

For another example, given

```non
    A = [0, 0, 0]
    K = 1
```

the function should return [0, 0, 0]

Given

```non
    A = [1, 2, 3, 4]
    K = 4
```

the function should return [1, 2, 3, 4]

Assume that:
- N and K are integers within the range [0..100];
- each element of array A is an integer within the range [−1,000..1,000].

In your solution, focus on correctness. The performance of your solution will not be the focus of the assessment.

### C++ Solution

```cpp
vector<int> solution(vector<int> &a, int k) {
    vector<int> ret;
    if (a.size() == 0) {
        return ret;
    }
    k = k % a.size();
    for (unsigned i = a.size() - k; i < a.size(); i++) {
        ret.push_back(a[i]);
    }
    for (unsigned i = 0; i < a.size() - k; i++) {
        ret.push_back(a[i]);
    }
    return ret;
}
```

## OddOccurrencesInArray

A non-empty array A consisting of N integers is given. The array contains an odd number of elements, and each element of the array can be paired with another element that has the same value, except for one element that is left unpaired.

For example, in array A such that:

```non
  A[0] = 9  A[1] = 3  A[2] = 9
  A[3] = 3  A[4] = 9  A[5] = 7
  A[6] = 9
```

the elements at indexes 0 and 2 have value 9,
the elements at indexes 1 and 3 have value 3,
the elements at indexes 4 and 6 have value 9,
the element at index 5 has value 7 and is unpaired.
Write a function:

```non
int solution(vector<int> &A);
```

that, given an array A consisting of N integers fulfilling the above conditions, returns the value of the unpaired element.

For example, given array A such that:

```non
  A[0] = 9  A[1] = 3  A[2] = 9
  A[3] = 3  A[4] = 9  A[5] = 7
  A[6] = 9
```

the function should return 7, as explained in the example above.

Write an efficient algorithm for the following assumptions:

- N is an odd integer within the range [1..1,000,000];
- each element of array A is an integer within the range [1..1,000,000,000];
- all but one of the values in A occur an even number of times.

```cpp
int solution(vector<int> &a) {
    int buffer = 0;
    for (unsigned i = 0; i < a.size(); i++) {
        buffer = a[i] ^ buffer;
    }
    return buffer;
}
```