---
title: "C++ Beginning"
subtitle: ""
layout: post
author: "blueskyson"
header-style: text
tags:
  - en
  - c++
---

## main() Function

```cpp
int main()    //function heading
{
  statements
  return 0;
}
```

A function has two parts. One is called **function heading**. The first word `int` is call **return type**, indicating what information will be returned after executing the function. The content in parentheses is called **argument list**.  

The other part is **function body**, which is enclosed in braces. Function body contains some **statements** tell what the program to do. Every statement must end with a semicolon.  

Main function is a special function that it's never called by any other part of the program. Sometimes, main function is written as

```cpp
int main(int argc, char* argv[])
```

Arguments `argc` stands for **argument count** and `argv` stands for **argument value**. `argc` is the number that arguments input in command line, and `argv` contains every char array of arguments. for example:

```cpp
#include <iostream>
using namespace std;
int main(int argc, char *argv[])
{
  cout << "There are " << argc << " arguments" << endl;
  for (int i = 0; i < argc; ++i) {
    cout << "argument " << i << ": " << argv[i] << endl;
  }
  return 0;
}
```

Name it as test.cpp, compile and execute it in command line:

```bash
> g++ test.cpp -o test
> test.exe hello world 1 2 3
There are 6 arguments
argument 0: test.exe
argument 1: hello
argument 2: world
argument 3: 1
argument 4: 2
argument 5: 3
```
