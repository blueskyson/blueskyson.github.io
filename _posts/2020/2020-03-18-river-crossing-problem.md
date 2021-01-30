---
title: "離散數學: 渡河問題"
header:
  image: "/assets/images/default_bg.jpg"
subtitle: "雞、狗、菜和農夫過河問題"
layout: post
author: "blueskyson"
header-style: text
tags:
  - c++
---

## 題目敘述

1. A farmer has a ship, a chicken, a bag of rice, a dog. He need to take all of them cross the river. But, this ship is too small. He only can take two of them together in his ship. But, chicken will eat rice, dog will bite chicken if they are in a ship. Please schedule this transfer plan for him. (You can try to write code to do this task.)  

2. If you want to write a program to automatically make inference, please describe how difficult it is?

## 程式解說

### 程式流程

1. 執行時可以傳入最大遞迴深度(預設為10)，或是不傳入也可以。初始化完 River 物件後進入 DFS 遞迴

2. 在遞迴函式中:

    - 若遞迴深度到達最大則 return

    - 若所有物品都在右岸，則輸出一組答案然後 return

    - 複製一份 River 物件，判斷是否有雞米或雞狗再一起的情況，若無則記錄狀態在 River 物件，進入下一層遞迴

3. 完成所有遞迴後，印出 finish 程式結束

### enum items, direct

```cpp
#include<iostream>
#include<vector>
#include<cstdlib>
using namespace std;

enum items {farmer, chicken, rice, dog, nothing};   //define items
enum direct {_left, _right};                        //define direct
int maxtime = 10;                                   //max times that farmer can move
```

items 定義為所有可以被運送的物體，包含農夫、雞、米、狗。其中 `nothing` 比較特別，因為函式設計的問題，如果運送 `nothing` ，代表當次不運送除了農夫以外的任何物品。而每次運送時，農夫必定參與運送。

direct 定義為河的兩端，包括左岸、右岸。一開始所有物體都在左岸， `nothing` 則是在兩岸都存在。

maxtime 為最大遞迴深度，預設為10，可在執行前傳入參數修改

### class River

```cpp
class River {
public:
    bool item[2][5];                       //items on both river sides
    vector< pair<int,int> > order;           //move record
    int prev_item;
```

item 代表物品在兩岸的狀態，例如: `item[_left][farmer] == true` 代表農夫在河的左岸，此時 `item[_right][farmer] == false` 一定會成立，因為農夫不可能同時在河的兩岸。`nothing` 則如前面所說，在河的兩岸都為 `true`。

order 是用來記錄搬運的過程，例如 `{rice, _right}` 代表當次米從左岸被運到右岸

prev_item 代表前一次搬運的物品，用來避免一直搬一樣的東西

### River::River()

```cpp
    River() {
        prev_item = -1;
        for (int i = 0; i < 4; ++i) {
            item[_left][i] = true;         //set all items to left side
            item[_right][i] = false;
        }
        item[_left][nothing] = true;       //nothing is an item as well
        item[_right][nothing] = true;      //nothing is always on both sides
    }
```

將所有物品先放到左岸，前一個移動的物品設為 `-1` (未定義)

### bool River::move(int, int, int)

```cpp
    // it:   what item to move
    // from: item is from left side or right side
    // to:   item will be move to which side
    bool move(int it, int from, int to) {
        if (prev_item == it)    //move the same thing again
            return false;
        if (!item[from][it])    //such item is on the other side
            return false;
        if (it != nothing) {
            item[from][it] = false;
            item[to][it] = true;
        }
        item[from][farmer] = false;
        item[to][farmer] = true;

        if (item[from][chicken]) {
            if (item[from][rice])
                return false;
            if (item[from][dog])
                return false;
        }

        prev_item = it;
        order.push_back(make_pair(it, to));
        return true;
    }
```

用來移動物品的函式，回傳 `true` 代表移動成功，反之回傳 `false`。

- 若前一次已經移動該物體，則不要再移動它一次，回傳 `false`，

- 若該物體在河對岸，則無法移動，回傳 `false`

若移動的物品不是 `nothing` 則將該物在此岸的狀態設為 `false` ，在彼岸的狀態設為 `true` 。 將農夫在此岸的狀態設為 `false` ，在彼岸的狀態設為 `true`。

若把農夫也移走後，雞米留在原地或雞狗留在原地，則移動失敗，回傳 `false` 。

如果很安全的把物體移到對岸，雞沒被咬，米沒被吃，就更新 `prev_item` 為此次移動的物品，並把移動紀錄存在 `order` 裡，回傳 `true`

### bool River::finish()

```cpp
    // determine if all items are on the right side
    bool finish() {
        for (int i = 0; i < 4; ++i)
            if (!item[_right][i])
                return false;
        return true;
    }
```

檢查所有物品是否都被移到右岸，如果是就回傳 `true` ， 否則回傳 `false`

### void River::print()

```cpp
    // print move record
    void print() {
        for (int i = 0; i < order.size(); ++i) {
            cout << '(';
            switch (order[i].first) {
                case 1:
                    cout << "chicken";
                    break;
                case 2:
                    cout << "rice";
                    break;
                case 3:
                    cout << "dog";
                    break;
                default:
                    cout << "nothing";
            }
            cout << ", ";
            if (order[i].second == 0)
                cout << "left";
            else
                cout << "right";
            cout << ") ";
        }
        cout << endl << endl;
    }
```

印出到目前為止移動的路徑

### void recursive(River, int)

```cpp
// r:     the state of all items and record
// times: the depth of recursive
void recursive(River r, int times) {
    if (times == maxtime)
        return;
    if (r.finish()) {
        cout << "* soultion: ";
        r.print();
        return;
    }

    int t = times+1;
    River a;
    if (r.item[_left][farmer]) {            // if farmer is on left side
        a = r;
        if (a.move(nothing, _left, _right))
            recursive(a, t);
        a = r;
        if (a.move(chicken, _left, _right))
            recursive(a, t);
        a = r;
        if (a.move(rice, _left, _right))
            recursive(a, t);
        a = r;
        if (a.move(dog, _left, _right))
            recursive(a, t);
    } else {                                // if farmer is on right side
        a = r;
        if (a.move(nothing, _right, _left))
            recursive(a, t);
        a = r;
        if (a.move(chicken, _right, _left))
            recursive(a, t);
        a = r;
        if (a.move(rice, _right, _left))
            recursive(a, t);
        a = r;
        if (a.move(dog, _right, _left))
            recursive(a, t);
    }
}
```

遞迴尋找解，如果遞迴深度超過上限則 return。如果成功找到答案，則將答案印出來，然後 return。

如果沒有符合前述終止條件，則判斷農夫在河流左岸或右岸，並試著移動該岸的物品，如果移動成功則進入深度加一的遞迴。

### main function

```cpp
int main(int argc, char* argv[]) {

    // the argument to set recursive depth
    if (argc >= 2)
        maxtime = atoi(argv[1]);

    River river;
    recursive(river, 0);
    cout << "finish!" << endl;
    return 0;
}
```

如果有傳入參數的話，將最大遞迴深度 `maxtime` 改為參數。沒傳入參數的話則為預設值 10 。

宣告初始物件 River ，將其丟進遞迴，結束時印出 `finish!`
## 編譯執行

### Windows

無參數執行，預期輸出:

```non
> g++ cross_river.cpp -o cross_river
> cross_river
* soultion: (chicken, right) (nothing, left) (rice, right) (chicken, left) (dog, right) (nothing, left) (chicken, right)

* soultion: (chicken, right) (nothing, left) (dog, right) (chicken, left) (rice, right) (nothing, left) (chicken, right)

finish!
```

丟入參數 20，預期輸出:

```non
> cross_river 20
* soultion: (chicken, right) (nothing, left) (rice, right) (chicken, left) (dog, right) (nothing, left) (chicken, right)

* soultion: (chicken, right) (nothing, left) (rice, right) (chicken, left) (dog, right) (rice, left) (chicken, right) (dog, left) (rice, right) (chicken, left) (dog, right) (nothing, left) (chicken, right)

* soultion: (chicken, right) (nothing, left) (rice, right) (chicken, left) (dog, right) (rice, left) (chicken, right) (dog, left) (rice, right) (chicken, left) (dog, right) (rice, left) (chicken, right) (dog, left) (rice, right) (chicken, left) (dog, right) (nothing, left) (chicken, right)

* soultion: (chicken, right) (nothing, left) (dog, right) (chicken, left) (rice, right) (nothing, left) (chicken, right)

* soultion: (chicken, right) (nothing, left) (dog, right) (chicken, left) (rice, right) (dog, left) (chicken, right) (rice, left) (dog, right) (chicken, left) (rice, right) (nothing, left) (chicken, right)

* soultion: (chicken, right) (nothing, left) (dog, right) (chicken, left) (rice, right) (dog, left) (chicken, right) (rice, left) (dog, right) (chicken, left) (rice, right) (dog, left) (chicken, right) (rice, left) (dog, right) (chicken, left) (rice, right) (nothing, left) (chicken, right)

finish!
```

### Ubuntu

無參數執行，預期輸出:

```non
$ g++ cross_river.cpp -o cross_river
$ ./cross_river
* soultion: (chicken, right) (nothing, left) (rice, right) (chicken, left) (dog, right) (nothing, left) (chicken, right) 

* soultion: (chicken, right) (nothing, left) (dog, right) (chicken, left) (rice, right) (nothing, left) (chicken, right) 

finish!
```

丟入參數 15，預期輸出:

```non
$ ./cross_river 15
* soultion: (chicken, right) (nothing, left) (rice, right) (chicken, left) (dog, right) (nothing, left) (chicken, right) 

* soultion: (chicken, right) (nothing, left) (rice, right) (chicken, left) (dog, right) (rice, left) (chicken, right) (dog, left) (rice, right) (chicken, left) (dog, right) (nothing, left) (chicken, right) 

* soultion: (chicken, right) (nothing, left) (dog, right) (chicken, left) (rice, right) (nothing, left) (chicken, right) 

* soultion: (chicken, right) (nothing, left) (dog, right) (chicken, left) (rice, right) (dog, left) (chicken, right) (rice, left) (dog, right) (chicken, left) (rice, right) (nothing, left) (chicken, right) 

finish!
```
