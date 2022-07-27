---
title: "epoll 筆記 Part 1"
subtitle: ""
excerpt: "c epoll"
layout: post
author: "blueskyson"
mathjax: true
header-style: text
tags:
  - c
---

此文章摘錄、翻譯自 [epoll() Tutorial – epoll() In 3 Easy Steps!](https://suchprogramming.com/epoll-in-3-easy-steps/)，再搭配自己的一些整理。

`epoll` 是 `poll` 的變體，允許單一個執行緒值監看多個 file descriptor 的事件，在使用者監看的事件發生時回傳以通知使用者。事件通知的時機又分為 level trigger 和 edge trigger，在 edge trigger 模式中，只有事件觸發，或是 timeout 時 `epoll_wait` 才會回傳；在 level trigger 模式，`epoll_wait` 在事件狀態未變更前都會回傳。

## Step 1:

```c
#include <stdio.h>     // for fprintf()
#include <unistd.h>    // for close()
#include <sys/epoll.h> // for epoll_create1()

int main()
{
    int epoll_fd = epoll_create1(0);

    if (epoll_fd == -1)
    {
        fprintf(stderr, "Failed to create epoll file descriptor\n");
        return 1;
    }

    if (close(epoll_fd))
    {
        fprintf(stderr, "Failed to close epoll file descriptor\n");
        return 1;
    }

    return 0;
}
```

首先使用 `epoll_create1()` 建立一個 epoll 的 file descriptor，然後透過 `close()` 關閉，如果終端機有輸出錯誤訊息，代表 kernel 版本過舊。

另外注意到 `epoll_create1()` 有個參數 `0`，原本的用途是告訴 kernel 預期會加入到 `epoll` 的 file descriptor 數量，kernel 再根據這個參數分配資源給 epoll。如今這個參數已經沒有實質意義了，kernel 會自己動態分配資源，但這個參數欄位可以確保在舊的 kernel 的程式碼可以直接移植到新的 kernel。

## Step 2:

```c
#include <stdio.h>     // for fprintf()
#include <unistd.h>    // for close()
#include <sys/epoll.h> // for epoll_create1(), epoll_ctl(), struct epoll_event

int main()
{
    struct epoll_event event;
    int epoll_fd = epoll_create1(0);

    if (epoll_fd == -1)
    {
        fprintf(stderr, "Failed to create epoll file descriptor\n");
        return 1;
    }

    event.events = EPOLLIN;
    event.data.fd = 0;

    if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, 0, &event))
    {
        fprintf(stderr, "Failed to add file descriptor to epoll\n");
        close(epoll_fd);
        return 1;
    }

    if (close(epoll_fd))
    {
        fprintf(stderr, "Failed to close epoll file descriptor\n");
        return 1;
    }
    return 0;
}
```

首先透過 `event` 變數設定要監聽的事件類型，`event.events = EPOLLIN;` 表示當監視的 file descriptor 可以被 `read` 時，就觸發事件。另外 `epoll` 預設是 level trigger，若要用 edge trigger，就要改寫為 `event.events = EPOLLIN | EPOLLET;`，此外還有 `EPOLLONESHOT` 等事件類型，詳情見 [epoll_ctl(2) — Linux manual page](https://man7.org/linux/man-pages/man2/epoll_ctl.2.html)。

`event.data.fd = 0;` 是給使用者自行設置的欄位，在此範例中我們設定為 stdin，如此一來在 Step 3 每當事件觸發時就透過 `read()` 讀取 `events[i].data.fd`。`event.data` 本身是一個 union，可以用來存一些 state、buffer 的位址、要讀取的 file descriptor 等。 

接下來要做的就是透過 `epoll_ctl(epoll_fd, EPOLL_CTL_ADD, 0, &event)` 告訴 epoll 要監視哪個 file descriptor 以及要監視哪些類型的事件，在這篇筆記中我使用 `0`，也就是 stdin 作為 epoll 監視的 file descriptor，詳情見 [epoll_ctl(2) — Linux manual page](https://man7.org/linux/man-pages/man2/epoll_ctl.2.html)。

## Step 3:

```c
#define MAX_EVENTS 5
#define READ_SIZE 10
#define TIMEOUT 10000
#include <stdio.h>     // for fprintf()
#include <unistd.h>    // for close(), read()
#include <sys/epoll.h> // for epoll_create1(), epoll_ctl(), struct epoll_event
#include <string.h>    // for strncmp

int main()
{
    int running = 1, event_count, i;
    size_t bytes_read;
    char read_buffer[READ_SIZE + 1];
    struct epoll_event event, events[MAX_EVENTS];
    int epoll_fd = epoll_create1(0);

    if (epoll_fd == -1)
    {
        fprintf(stderr, "Failed to create epoll file descriptor\n");
        return 1;
    }

    event.events = EPOLLIN;
    event.data.fd = 0;

    if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, 0, &event))
    {
        fprintf(stderr, "Failed to add file descriptor to epoll\n");
        close(epoll_fd);
        return 1;
    }

    while (running)
    {
        printf("\nPolling for input...\n");
        event_count = epoll_wait(epoll_fd, events, MAX_EVENTS, TIMEOUT);
        printf("%d ready events\n", event_count);
        for (i = 0; i < event_count; i++)
        {
            printf("Reading file descriptor '%d' -- ", events[i].data.fd);
            bytes_read = read(events[i].data.fd, read_buffer, READ_SIZE);
            printf("%zd bytes read.\n", bytes_read);
            read_buffer[bytes_read] = '\0';
            printf("Read '%s'\n", read_buffer);

            if (!strncmp(read_buffer, "stop\n", 5))
                running = 0;
        }
    }

    if (close(epoll_fd))
    {
        fprintf(stderr, "Failed to close epoll file descriptor\n");
        return 1;
    }

    return 0;
}
```

使用 `epoll_wait()` 來等待來自 `epoll_fd` 的事件發生；`events` 用來存取已經就緒的事件的資訊，比如稍早寫的 `event.data.fd = 0`；每次最多只回傳 `MAX_EVENTS` 個事件到 `events` 中；如果很長一段時間都沒有事件被觸發，至多會 block 住 `TIMEOUT` 毫秒，並回傳 0。

## 執行程式：

```non
$ ./epoll

Polling for input...
```

hello!

```non
1 ready events
Reading file descriptor '0' -- 7 bytes read.
Read 'hello!
'
```

a long long text

```non
Polling for input...
a long long text
1 ready events
Reading file descriptor '0' -- 10 bytes read.
Read 'a long lon'

Polling for input...
1 ready events
Reading file descriptor '0' -- 7 bytes read.
Read 'g text
'
```

stop

```non
Polling for input...
1 ready events
Reading file descriptor '0' -- 5 bytes read.
Read 'stop
'
```
