---
title: "Array and String"
subtitle: ""
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - en
  - c++
---

## What is an Array

An array is a data format that store several value of the same type. For example, an array can store 12 integers indicating day number of every month.

[![array](https://raw.githubusercontent.com/blueskyson/image-host/master/array.jpg)](https://raw.githubusercontent.com/blueskyson/image-host/master/array.jpg)

>Compiler won't check if the index is valid. If we write a statement `month[100] = 0`, the code may be compiled successfully. However this statement will break other data in the program or even stop the program when it's running.

## Array Initialization

The most common form of array initializer is a list of constant expressions enclosed in braces and separated by commas:

`int arr[6] = {1, 2, 3, 4, 5, 6};`  
`// initialize value of arr is {1, 2, 3, 4, 5, 6}`

`int arr[6] = {1, 2}`  
`// initialize value of arr is {1, 2, 0, 0, 0, 0}`S

Using this feature, we can use {0} to initialize an array to all zeros:

`int arr[6] = {0}`  
`// initialize value of arr is {0, 0, 0, 0, 0, 0}`

`int arr[6] = {1}`  
`// initialize value of arr is {1, 0, 0, 0, 0, 0}`

For `char` array, it's better to put null character ('\0') into the last element. So that IO functions like `cout`, `printf` can process it normally. Characters after the first null character will be ignored by `cout` and `printf`.

`char str1[6] = {'h', 'e', 'l', 'l', 'o', '!'};`  
`// cout << str1; may cause unpredictable behavior since it has no '\0' at the end`

`char str2[7] = {'h', 'e', 'l', 'l', 'o', '!', '\0'};`
`// a string with null character at the end`

`char str3[10] = "hello"`  
`// initialize value of arr is {'h', 'e', 'l', 'l', 'o', '!', '\0', '\0', '\0', '\0'}`

## Use sizeof() with Arrays

```cpp
#include<iostream>
using namespace std;
int main() {
  int arr[21];
  cout << sizeof(arr) << endl;
  /* get the size of entire array*/
  cout << sizeof(arr[0]) << endl;
  /* get the size of one element in the array*/
  return 0;
}
```

```non
84
4
```

## Multidimensional Arrays

C++ stores arrays in row-major order shown as below:

[![array-2](https://raw.githubusercontent.com/blueskyson/image-host/master/array-2.jpg)](https://raw.githubusercontent.com/blueskyson/image-host/master/array-2.jpg)

## Multidimensional Array Initialization

`int arr[3][4] = { {1, 2, 3, 4}, {4, 3, 2, 1}, {5, 6, 7, 8} };`

Higher dimensional arrays are constructed similarly.

## String Object

In ISO/ANSI C++98, `string` is added in C++ library. Though `string` is quite simple to manipulate, it is not a basic type but an object wrapped in `class`, the later will be explained in another post. Here are some comparisons of string and char array.

* String use tha same initialize statement as char array  
`string str1 = "string";`  
`char ch1[20] = "char";`  

* String can be assigned to another string, but char array not  
`string str2 = str1;`  
`//valid assignment, the value of str2 becomes "string"`  
`char ch2[20];`  
`ch2 = ch1; //invalid, no array assignment`

* Two strings can be connected by operator `+` and `+=`  
`string str3 = str1 + str2;`  
`//valid operation, the value of str3 becomes "stringstring"`  
`char ch3[40] = ch1 + ch2 //invalid`  
`str1 += "hello"`  
`//valid operation, the value of str1 becomes "stringhello"`

* String use both `[]` operator and `at()` method to get char in specific index  
`cout << str1[3]; //this statement shows 'i'`  
`cout << str1.at(3) //this also shows 'i'`  

> Notice that `[]` operator checks if the index is out of the range of string, `at()` method not. So we should use `at()` carefully.  

## Some Useful String Methods

`str1.size();`  
`//return the length of string (no '\0' at the end)`  
`//in this case it returns 11`  

`str1.erase(5,3);`  
`//erase some words from some index`  
`//in this case, it erases 3 chars from index str1[5]`  
`//so str1 becomes "strinllo"`  

`string str4 = str3.substr(6,6);`  
`//return a new string storing some words from some index`  
`//in this case, it copies 6 chars from index str3[6]`  
`//so str4 becomes "string"`  

There are still many more methods. [cplusplus.com](http://www.cplusplus.com/reference/string/string/) has more explanations about string. Make good use of it : )
