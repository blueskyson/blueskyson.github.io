---
title: "用 Docker 執行 FluentD"
subtitle: ""
excerpt: "fluent fluentd docker"
layout: post
author: "blueskyson"
header-style: text
tags:
  - docker
---

作業系統: Ubutu 20.04
安裝 curl、docker、postman

## 下載範例檔案

新增一個工作目錄:

```non
$ mkdir custom-fluentd
$ cd custom-fluentd
```

從 Github 下載 fluent.conf 和 entrypoint.sh，此時最新版為 v1.15:

```non
$ curl https://raw.githubusercontent.com/fluent/fluentd-docker-image/master/v1.15/alpine/fluent.conf > fluent.conf
$ curl https://raw.githubusercontent.com/fluent/fluentd-docker-image/master/v1.15/alpine/entrypoint.sh > entrypoint.sh
$ chmod +x entrypoint.sh
```

新增插件目錄，目錄中的 plugins scripts 會被複製進這個 Docker Image 中:

```non
$ mkdir plugins
```

下載 Dockerfile 範例:

```non
curl https://raw.githubusercontent.com/fluent/fluentd-docker-image/master/Dockerfile.sample > Dockerfile
```

## 製作 Dockerfile

在 Dockerfile 中填入 `MAINTAINER` 和下載 Fluentd plugins，預設範例已經有 fluent-plugin-elasticsearch。

```dockerfile
FROM fluent/fluentd:v1.15-1
MAINTAINER Jack_Lin <jacklin@cybersoft4u.com>

USER root

RUN apk add --no-cache --update --virtual .build-deps \
        sudo build-base ruby-dev \
 # cutomize following instruction as you wish
 && sudo gem install fluent-plugin-elasticsearch \
 && sudo gem sources --clear-all \
 && apk del .build-deps \
 && rm -rf /home/fluent/.gem/ruby/2.5.0/cache/*.gem

COPY fluent.conf /fluentd/etc/
COPY entrypoint.sh /bin/

USER fluent
```

改寫 fluent.conf:

```xml
# Hello World configuration will take events received on port 18080 using
# HTTP as a protocol

# set Fluentd's configuration parameters
<system>
    Log_Level info
</system>

# define the HTTP source which will provide log events
<source>
    @type http
    port 18080
</source>

# accept all log events regardless of tag and write them to the console
<match *>
    @type stdout
</match>

```

建置 Docker Image:

```non
$ docker build -t custom-fluentd:latest ./
```

## 測試

用剛剛建立的 Docker Image 啟動一個 Container:
- 命名為 custom-docker-fluent-logger
- 將 logs 目錄掛載到 Container
- 把連接埠 18080 對應到主機的 18080。

```non
$ mkdir -p logs
$ docker run -it --rm --name custom-docker-fluent-logger -v $(pwd)/logs:/fluentd/logs -p 18080:18080 custom-fluentd:latest
```

接著打開 Postman，傳送一個 json 到 localhost:18080。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/fluentd-1.png)

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/fluentd-2.png)

每傳送一次，在 custom-docker-fluent-logger 就會輸出一行 `{"Hello": "World"}`:

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/fluentd-3.png)
