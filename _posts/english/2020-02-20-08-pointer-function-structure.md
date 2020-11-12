---
title: "Pointer, Function, Structure"
subtitle: ""
layout: post
author: "blueskyson"
header-style: text
tags:
  - en
  - c++
---

## Pointer and Structure

Pointers can also points to user-defined structures:

```cpp
#include<iostream>
#include<string>
using namespace std;

struct student {
    string name;
    int age;
    double height;
    int ID;
};

int main() {
    student *a = new student;
    (*a).name = "Alice";
    (*a).age = 14;
    (*a).height = 155.7;
    (*a).ID = 1234;
    cout << (*a).name << endl;
    cout << (*a).age  << endl;
    cout << (*a).height << endl;
    cout << (*a).ID << endl;
    return 0;
}
```

Though we can use `*` operator to access members in a structure like normal variables, sometimes we may need statement like this:

`(*(*((*a).b)).c).d`

The combination of parentheses and dot is somewhat hard to read, so there is a special operator to fix this problem. The operator `->` is called **right arrow selection**, it's used for object pointers like `struct`, `union`, `class`. We replace `(*)` with `->` in the statement above.

`a->b->c->d`

The creation of `->` is not only because of reading issue, for more information, check out [Why does the arrow (->) operator in C exist?](https://stackoverflow.com/questions/13366083/why-does-the-arrow-operator-in-c-exist) Now, we rewrite the example code above:

```cpp
#include<iostream>
#include<string>
using namespace std;

struct student {
    string name;
    int age;
    double height;
    int ID;
};

int main() {
    student *a = new student;
    a->name = "Alice";
    a->age = 14;
    a->height = 155.7;
    a->ID = 1234;
    cout << a->name << endl;
    cout << a->age  << endl;
    cout << a->height << endl;
    cout << a->ID << endl;
    return 0;
}
```

## Function

A function is a series of statements that have been grouped together and given a name. Functions have many advantages such as:

* Dividing a program to many pieces so that we can easily understand and modify the program.

* Avoiding duplicating the code that used many times.

* The same function can be reused in other program by store the code in split files.

A prototype of function is like this:

*//you can put as many arguments as you wish in parentheses*  
*return-type name(type name, ...) {*  
 . . .  
 *statements*  
 . . .  
 *return variable;  //must be return-type*  
*}*  

A function copies all arguments into itself and does statements, then finally return a value. Let's take a look at some examples:

```cpp
#include<iostream>
#include<string>
using namespace std;

struct student {
    string name;
    int age;
    double height;
    int ID;
};

//return integer 10
int ten() {
    return 10;
}

//no return value
void print_hello() {
    cout << "hello" << endl;
}

//if the first argument is greater, return true
bool is_a_greater_than_b(int a, int b) {
    if(a > b)
        return true;
    return false;
}

//a recursive function that counts numbers
int count_number(int num) {
    if(num >= 0) {
        cout << num << " ";
        return count_number(num - 1);
    }
    cout << endl;
    return 0;
}

//use student as return type
student Alice() {
    student a = {"Alice", 14, 155.7, 1234};
    return a;
}

int main() {
    int a = ten();          //assign a the return value of ten()
    cout << ten() << endl;  //directly output the return value of ten()
    print_hello();          //call a void function
    cout << is_a_greater_than_b(3, a) << endl;      //3 < a, so return false
    cout << is_a_greater_than_b(ten(), 3) << endl;  //we can also put a function into a function
    count_number(ten());

    student s = Alice();     //assign s the return value of alice()
    cout << s.name << endl;  //check if s is assigned successfully
    cout << Alice().name;    //since alice returns type student, we can use . to directly access it 
    return 0;
}
```

```non
10
hello
0
1
10 9 8 7 6 5 4 3 2 1 0
Alice
Alice
```

## Function and Pointer

Functions can also use arrays as its arguments, for example:

`int myfunction(int arr[], int n) { . . . }`  
`//arr is array name, n is array size`

As we said before, an array variable is actually a pointer. So we can rewrite the prototype above as:

`int myfunction(int *arr, int n) { . . . }`  

Write code to demonstrate how to put arrays in functions:

```cpp
#include<iostream>
#include<string>
using namespace std;

//use int a[] to represent an array argument
void list_elements(int a[], int size) {
    for(int i = 0; i < size; ++i)
        cout << a[i] << " ";
    cout << endl;
}

//use int *a to represent an array argument
void swap_arrays(int size, int *arr1, int *arr2) {
    int tmp;
    for(int i = 0; i < size; ++i) {
        tmp = arr1[i];
        arr1[i] = arr2[i];
        arr2[i] = tmp;
    }
}

int main() {
    int array1[5] = {5, 4, 3, 2, 1};
    int array2[5] = {10, 20, 30, 40, 50};
    swap_arrays(5, array1, array2);

    cout << "array1: ";
    list_elements(array1, 5);
    cout << "array2: ";
    list_elements(array2, 5);
    return 0;
}
```

```non
array1: 10 20 30 40 50
array2: 5 4 3 2 1
```

We successfully swapped two arrays in the above code, now we try two swap two variables of the same type:

```cpp
#include<iostream>
using namespace std;

void swap(double a, double b) {
    double tmp = a;
    a = b;
    b = tmp;
}
int main() {
    double a1 = 1.2, a2 = 1.4;
    swap(a1, a2);
    cout << a1 << " " << a2;
    return 0;
}
```

```non
1.2 1.4
```

Notice that the value of a and b were not really swapped. This is because the function copies the value of *a1* into *a* and copies *a2* into *b*. So what we swap were not the original variables, but two variables that created by the function. We call this situation **pass by value**.

## Pass by Reference

In order to really swap two variables, we now introduce another technique called **pass by reference**. By copying the addresses of two variables into the function, we swap the values stored in two copied pointers. Here's an example of passing by reference:

```cpp
#include<iostream>
using namespace std;

void swap(double *a, double *b) {
    double tmp = *a;
    *a = *b;
    *b = tmp;
}

int main() {
    double a1 = 1.2, a2 = 1.4;
    swap(&a1, &a2);
    cout << a1 << " " << a2;
    return 0;
}
```

```non
1.4 1.2
```

Now we really swapped the values of *a1* an *a2* ! Actually, passing an array variable into a function is equivalent of passing the reference, because an array variable is a pointer itself.

## Return a Pointer

Besides normal types, a function can also return a pointer by putting `*` in front of function name. The code below shows us how to return a pointer:

```cpp
#include<iostream>
#include<cstring>  //for strlen()
using namespace std;

char *combine(char *a, char *b) {
    int a_len = strlen(a);                  //count string length of a
    int b_len = strlen(b);                  //count string length of b
    char *c = new char[a_len + b_len + 1];  //the length of combined string, remember to allocate a space for '\0'
    int index = 0;
    for (int i = 0; i < a_len; ++i, ++index)
        c[index] = a[i];
    for (int i = 0; i < b_len; ++i, ++index)
        c[index] = b[i];
    c[index] = '\0';
    return c;
}

int main() {
    char a[6] = "hello";
    char b[6] = "world";
    char *ab = combine(a, b);  //declare *ab to catch the address returned by combine(a, b)
    cout << ab;
    delete ab;                 //remember to release the memory if not using
    ab = NULL;
    return 0;
}
```

```non
helloworld
```

This example shows how to combine two strings and return the combined string. The technique of passing by reference is usually used to process structures and classes.
