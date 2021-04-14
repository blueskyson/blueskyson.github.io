---
title: "使用 nginx 與 php 架設 key-value storage 服務"
subtitle: ""
excerpt: "nginx php key-value storage"
layout: post
author: "blueskyson"
header-style: text
tags:
  - linux
---

此為網路管理的 lab7

## 設定 `php-fpm`

```non
$ sudo apt install php-cli php-fpm
```

設定 `php-fpm` 的 port ，設定檔通常在 `/etc/php/<版本>/fpm/pool.d`

```non
$ sudo vim /etc/php/7.4/fpm/pool.d/www.conf
```

將 `listen` 設為 9000

```non
...

; The address on which to accept FastCGI requests.
; Valid syntaxes are:
;   'ip.add.re.ss:port'    - to listen on a TCP socket to a specific IPv4 address on
;                            a specific port;
;   '[ip:6:addr:ess]:port' - to listen on a TCP socket to a specific IPv6 address on
;                            a specific port;
;   'port'                 - to listen on a TCP socket to all addresses
;                            (IPv6 and IPv4-mapped) on a specific port;
;   '/path/to/unix/socket' - to listen on a unix socket.
; Note: This value is mandatory.
listen = 9000

...
```

啟動 `php-fpm` 並檢查是否正常運作

```
$ systemctl start php7.4-fpm
$ sudo netstat -anpt | grep php
tcp6       0      0 :::9000                 :::*                    LISTEN      8957/php-fpm: maste
```

## 設定 `nginx` 

```non
$ sudo apt install nginx
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

啟動 `nginx` 的 `fastcgi` ，修改 `/etc/nginx/sites-enabled/default.conf` 即可 

```non
$ sudo vim /etc/nginx/sites-enabled/default.conf
```

新增 `location ~ \.php$ { ... }` 區塊以啟動 `fastcgi`

```non
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;
        index index.html index.htm index.nginx-debian.html;
        server_name _;

        location / {
                try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass 127.0.0.1:9000;
                index index.php index.html;
        }
}
```

重啟 `nginx`

```non
$ sudo nginx -s reload
```

在 `nginx` 的 `root` (預設是 /var/www/html) 中新增測試檔案 `index.php`

```non
$ cat /var/www/html/index.php
<?php
    echo "hello";
?>
```

測試 fastcgi 是否成功啟用

```non
$ curl http://localhost/index.php
hello
```

以上便完成初步的 php 測試

## 實作 key-value storage

因為只是一個小作業，所以不會用到功能較強的 MySQL，而是讓 key 作為檔名，value 作為內容儲存在資料夾中 