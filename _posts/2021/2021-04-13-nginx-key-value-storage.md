---
title: "使用 nginx 與 php 架設 key-value storage 服務"
subtitle: ""
excerpt: "nginx php key-value storage"
layout: post
author: "blueskyson"
header-style: text
tags:
  - nginx
  - php
---

此為 2021 網路管理課程的 lab7

## 設定 php-fpm

```non
$ sudo apt install php-cli php-fpm
```

設定 php-fpm 的 `port` ，設定檔通常在 /etc/php/<版本>/fpm/pool.d

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

啟動 php-fpm 並檢查是否正常運作

```non
$ systemctl start php7.4-fpm
$ sudo netstat -anpt | grep php
tcp6       0      0 :::9000                 :::*                    LISTEN      8957/php-fpm: maste
```

## 設定 nginx

```non
$ sudo apt install nginx
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

啟動 nginx 的 `fastcgi` ，修改 /etc/nginx/sites-enabled/default.conf 即可 

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

重啟 nginx

```non
$ sudo nginx -s reload
```

在 nginx 的 `root` (預設是 /var/www/html) 中新增測試檔案 `index.php`

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

預期的功能是:

- 使用 http POST `/set/<KEY> body: value=<VALUE>` 來新增或複寫 key-value
- 使用 http GET `/get/<KEY>` 取得 value

因為只是一個小作業，所以不會用到功能較強的 MySQL，而是讓 key 作為檔名，value 作為內容儲存在目錄 /var/www/lab7/data 中，首先創建目錄

```non
$ sudo mkdir /var/www/lab7
$ sudo mkdir /var/www/lab7/data
```

### SET

以 regular expression 設定動態的網址，使用 http 的 POST `/set/<KEY> body: value=<VALUE>` 來儲存 key-value 。所以在 /etc/nginx/sites-enabled/default.conf 的 `server` 區塊新增以下:

```non
location ~ ^(/set)/[^/]+ {
        root /var/www/lab7;
        include fastcgi.conf;
        try_files /set.php $uri =404;
        fastcgi_pass 127.0.0.1:9000;
}
```

- `location ~` ^(/set)/[^/]+ 代表以 /set/ 開頭，然後由一個以上的 / 以外的字元組成的連結，用其接受 /set/<KEY> 的規則
- `root` 此區塊的的根目錄，我設定在 /var/www/lab7
- `include` 加入 fastcgi 設定檔，/etc/nginx/fastcgi.conf 中定義一些參數，如 REQUEST_URI，這些參數在 php 腳本中透過 $_SERVER[] 取得
- `try_files` 確認檔案是否存在，如果 set.php 不存在，就導向到 404
- `fastcgi_pass` 導向 php7.4-fpm 的接口

再來我們撰寫 php 腳本以實行 SET 服務，其會將 key-value 的檔案存放在 /var/www/lab7/data

```non
$ sudo vim /var/www/lab7/set.php
```

```php
<?php
if ($_SERVER['REQUEST_METHOD'] != "POST") {
        echo "Failed. Please use POST body: value=<VALUE>\n";
        return;
}

$value = $_POST['value'];
if (strlen($value) == 0 and empty($value)) {
        echo "Failed. POST body: value=empty()\n";
        return;
} elseif (strlen($value) > 256) {
        echo strlen($value);
        echo "Failed. Value too long (max length is 256).\n";
        return;
}

$key = substr($_SERVER['REQUEST_URI'], 5);
if (strlen($key) > 256) {
        echo strlen($key);
        echo "Failed. Key too long (max length is 256).\n";
        return;
}
if (strlen($key) == 256) {
        $key = hash('sha512', $key);
}

$file_name = "data/".$key;
$file = fopen($file_name, "w") or die("Unable to open file!");
fwrite($file, $value);
fclose($file);
echo "OK";
?>
```

接下來在 php 讀寫檔案方面很有可能遇到 Permission Denied，相關的紀錄可以在 /var/log/nginx/error.log 中查閱。此時我們需要修改 fpm 的設定檔

```non
$ sudo vim /etc/php/7.4/fpm/pool.d/www.conf
```

將 `listen.mode = 0660` 取消註解

```non
listen.mode = 0660
```

保險起見，將 location 的 `root` 的擁有者改成 nginx 使用者 (預設是 `www-data`) ，然後重啟 nginx 和 fpm

```non
$ sudo chown -R www-data:www-data /var/www/lab7
$ sudo service php7.4-fpm restart
$ sudo service nginx restart
```

我們可以用 curl 來測試 set.php 是否正常運作

```non
$ curl -F "value=world" localhost/set/hello
OK
```

查看檔案室否成功儲存

```non
$ tree /var/www/lab7
/var/www/lab7
├── data
│   └── hello
└── set.php
$ cat /var/www/lab7/data/hello
world
```

如果檔案沒有被成功寫入，請再去查看 nginx 的 error log

### GET

以 regular expression 設定動態的網址，使用 http 的 POST `/set/<KEY> body: value=<VALUE>` 來儲存 key-value 。所以在 `/etc/nginx/sites-enabled/default.conf` 的 `server` 區塊新增以下:

```non
location ~ ^(/get)/[^/]+ {
        root /var/www/lab7;
        include fastcgi.conf;
        try_files /get.php $uri =404;
        fastcgi_pass 127.0.0.1:9000;
}
```

撰寫 php 腳本以實行 GET 服務，其會讀取 /var/www/lab7/data 的檔案並打印出內容

```non
$ sudo vim /var/www/lab7/get.php
```

```php
<?php
$key = substr($_SERVER['REQUEST_URI'], 5);
if (strlen($key) > 256) {
        echo "key not found!";
        return;
}
if (strlen($key) == 256) {
        $key = hash('sha512', $key);
}

$file_name = "data/".$key;
if (file_exists($file_name)) {
        $file = fopen($file_name, "r");
        echo fread($file, filesize($file_name));
        fclose($file);
} else {
        echo "key not found!";
}
?>
```

將 get.php 的擁有者改成 nginx 使用者

```non
$ sudo chown www-data:www-data /var/www/lab7/get.php
```

測試 GET 是否成功

```non
$ curl localhost/get/hello
world
```