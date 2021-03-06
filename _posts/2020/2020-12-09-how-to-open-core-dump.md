---
title: "如何使用 gdb 查看 core dump"
subtitle: ""
excerpt: "core dump"
layout: post
author: "blueskyson"
header-style: text
tags:
  - c
  - c++
---

在 Ubuntu 預設是不會產生 core dump 的，想讓系統產生 core dump ，需要輸入以下指令:
```non
$ ulimit -c unlimited
```
其中 ulimit 是限制一些 user 資源的使用量，包含 max user processes 、 open files 的上限、 virtual memory 的上限等，而 `ulimit -c` 便是設定 core 的大小， `unlimited` 代表無上限，如果想要限制 core 的大小，可以把 `unlimited` 改成其他數字。

接下來我寫了一個測試檔案，執行時會造成 segmentation fault:

```non
$ cat test.c
#include <stdio.h>
int main() {
	int *i;
	*i = 1;
	return 0;
}
```

編譯時加入 `-g` debug option ，然後執行:

```non
$ gcc -g test.c -o test
$ ./test
Segmentation fault (core dumped)
```

這時候我們想找出程式到底哪裡出錯，於是打開 gdb ，並輸入程式名稱和 core 的檔名:

```non
$ gdb test core
GNU gdb (Ubuntu 9.1-0ubuntu1) 9.1
Copyright (C) 2020 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from test...
[New LWP 3355]
Core was generated by `./test'.
Program terminated with signal SIGSEGV, Segmentation fault.
#0  0x000056139bf2f135 in main () at test.c:4
4		*i = 1;
(gdb) q
$
```

gdb 很順利的幫我找到了 `test.c` 的 `main()` 的第 4 行出錯了，開心~
如果程式比較複雜，呼叫了很多層函式，這時可以使用 `bt` (back trace) 一層一層找出一連串函式呼叫，例如以下範例:

```non
$ cat test.c
#include <stdio.h>
void func(){
	int *a;
	*a = 1;
}

int main() {
	func();
	return 0;
}
```

出錯的地點在 `func()` 中，使用 `bt` 來查看:

```non
$ rm core
$ gcc -g test.c -o test
$ ./test
Segmentation fault (core dumped)
$ gdb test core

 . . .

Program terminated with signal SIGSEGV, Segmentation fault.
#0  0x000055ab058f0135 in func () at test.c:4
(gdb) bt
#0  0x000055ab058f0135 in func () at test.c:4
#1  0x000055ab058f0150 in main () at test.c:8
```

可以看到 gdb 追朔到呼叫 `func()` 的地方，也就是 `main.c` 的第 8 行