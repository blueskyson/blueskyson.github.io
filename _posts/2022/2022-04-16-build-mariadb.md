---
title: "在 Ubuntu 20.04 編譯 MariaDB 原始碼"
subtitle: ""
excerpt: "build MariaDB source code 原始碼"
layout: post
author: "blueskyson"
mathjax: false
header-style: text
tags:
  - database
  - sql
---

此篇筆記參考官方文件與個人實測所撰寫，安裝當下的 MariaDB 版本為 10.9。

## 從原始碼安裝 MariaDB

```non
$ sudo apt install -y build-essential bison
$ sudo apt build-dep mariadb-server
```

此時你可能會遇到以下錯誤：

```non
E: You must put some 'source' URIs in your sources.list
```

解決方法為開啟 `software-properties-gtk`：

```non
$ sudo software-properties-gtk
```

勾選 `Source code` 選項。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/mariadb1.png)

接下來再次執行 `apt build-dep mariadb-server` 即可。

安裝完套件後，到 github 下載原始碼編譯：

```non
git clone https://github.com/MariaDB/server.git mariadb
cd mariadb
cmake . -DBUILD_CONFIG=mysql_release && make -j8
```

安裝 mariaDB：

```
$ sudo make install
```

在終端機查看安裝訊息，應該會發現必要檔案都被複製到 `/usr/local/mysql`（在其他 distro 可能有所不同），至此我們透過原始碼安裝完 MariaDB server。

## 執行 MariaDB

找到 `mysqld` 執行檔的位置（我的在 `/usr/local/mysql/bin/mysqld`），執行時手動指定 `datadir`：

```non
$ mkdir data
$ /usr/local/mysql/bin/mysqld --datadir=./data
```

另一種啟動方法是透過 `systemctl` 以 `root` 或指定 user 啟動，但我也還在摸索中，以後弄清楚怎麼用再分享。

成功執行之後，第一行和最後一行訊息如下：

```non
2022-04-16  1:29:18 0 [Note] /usr/local/mysql/bin/mysqld (server 10.9.0-MariaDB) starting as process 110416 ...

... 

Version: '10.9.0-MariaDB'  socket: '/tmp/mysql.sock'  port: 3306  MariaDB Server
```

這兩個訊息特別重要，第一個訊息是當要停止伺服器時，透過 `kill 110416` 可以安全的關閉伺服器，mariadb。最後一個訊息的 `/tmp/mysql.sock` 是 client 連到 MariaDB 時需要用到。

## 測試 MariaDB

首先安裝 `mysql`：

```non
$ sudo apt install mariadb-client-core-10.3
```

連上 MariaDB：

```non
$ mysql --socket=/tmp/mysql.sock
```

參考 [MariaDB Basics](https://mariadb.com/kb/en/mariadb-basics/) 建個簡單的資料庫：

```sql
CREATE DATABASE bookstore;
USE bookstore;
```

```sql
CREATE TABLE books (
isbn CHAR(20) PRIMARY KEY, 
title VARCHAR(50),
author_id INT,
publisher_id INT,
year_pub CHAR(4),
description TEXT );
```

```sql
CREATE TABLE authors
(author_id INT AUTO_INCREMENT PRIMARY KEY,
name_last VARCHAR(50),
name_first VARCHAR(50),
country VARCHAR(50) );
```

```sql
INSERT INTO authors
(name_last, name_first, country)
VALUES('Kafka', 'Franz', 'Czech Republic');
```

```sql
INSERT INTO books
(title, author_id, isbn, year_pub)
VALUES('The Castle', '1', '0805211063', '1998'),
('The Trial', '1', '0805210407', '1995'),
('The Metamorphosis', '1', '0553213695', '1995'),
('America', '1', '0805210644', '1995');
```

接下來讀取資料庫：

```sql
SELECT title FROM books;
```

```sql
SELECT title, name_last 
FROM books 
JOIN authors USING (author_id);
```

## 參考資料

[Get, Build and Test Latest MariaDB the Lazy Way](https://mariadb.com/kb/en/get-build-and-test-latest-mariadb-the-lazy-way/)

[Error :: You must put some 'source' URIs in your sources.list](https://askubuntu.com/questions/496549/error-you-must-put-some-source-uris-in-your-sources-list)

[MariaDB Basics](https://mariadb.com/kb/en/mariadb-basics/)
