---
title: "從 0 開始學習 Docker、Dockerfile、Docker-Compose"
subtitle: ""
excerpt: "docker dockerfile docker-compose"
layout: post
author: "blueskyson"
header-style: text
tags:
  - nginx
  - docker
---

## 1. 安裝

首先使用指令安裝 Docker

```non
$ curl -fsSL https://get.docker.com | sudo bash
```

安裝 Docker Compose

```non
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```

## 2. 透過指令產生 Container

使用 nginx 當作範例，首先新增一個資料用以測試 docker，在資料夾中新增  `index.html`

```non
$ mkdir dockertest
$ vim dockertest/index.html
```

在 `index.html` 中寫入以下內容

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Hello Nginx</title>
</head>
<body>
    Hello Docker!
</body>
</html>
```

再來從 dockerhub 下載 nginx image 到本地的儲存庫

```non
$ sudo docker pull nginx
```

跟 `git pull` 不同的是，`docker image` 是集中管理的，所以不管在哪個目錄 pull 都無所謂。此外，透過 `sudo docker image` 可以查看本地有哪些 image。

接下來輸入以下指令來簡易啟動一個 Container

```non
$ sudo docker run --name nginxtest -p 8080:80 -v /home/lin/Desktop/docker:/usr/share/nginx/html -d nginx
```

- **docker run**: 創建一個新的容器 (Container) 並對其進行操作
- **--name string**: 為容器命名，我命名為 nginxtest
- **-p \<host_port\>:\<container_port\>**: 連接埠轉送，有多少 `-p h_p:c_p` 就轉送多少 port
- **-v \<host_dir\>:\<container_dir\>**: 全文為 volume，將 host 目錄掛載至 container 目錄，如此一來 container 就能在該目錄讀寫檔案，注意必須使用絕對路徑。這個參數跟 -p 一樣可以有複數個
- **-d**: 全文為 detach，代表讓 container 在背景執行
- **IMAGE**: container 中的 image，在以上範例為 nginx

其他更詳細的參數說明可以參閱 `docker run --help`

最後在終端機或瀏覽器測試是否成功執行

```non
$ curl localhost:8080
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Hello Nginx</title>
</head>
<body>
    Hello Docker!
</body>
</html>
```

以上便完成 docker 的初體驗

## 3. 刪除 Container

使用 `docker ps` 查看正在運行的服務

```non
$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                                   NAMES
cd094877361a   nginx     "/docker-entrypoint.…"   14 minutes ago   Up 14 minutes   0.0.0.0:8080->80/tcp, :::8080->80/tcp   nginxtest
```

通過 `CONTAINER ID` 或 `NAMES` 停止以及刪除 Container

```non
$ sudo docker stop cd094877361a
$ sudo docker rm cd094877361a
或
$ sudo docker stop nginxtest
$ sudo docker rm nginxtest
```

記得要開新的 Container 之前要把舊的同樣名稱的 Container 刪除

## 4. 刪除 Image

使用 `docker images` 查看本地的 image

```non
$ sudo docker images
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
nginx        latest    d1a364dc548d   7 days ago   133MB
```

透過 `REPOSITORY:TAG` 或 `IMAGE ID` 刪除 Image

```non
$ sudo docker rmi nginx:latest
或
$ sudo docker rmi d1a364dc548d
```

## 5. Dockerfile

簡而言之，Dockerfile 是一個用以製造自定義 Image 的文字檔，Dockerfile 可以指定 Base Image，並且透過 linux 指令在 Base Image 安裝環境、為其新增環境變數、決定執行時期要啟動那些程序等。

接下來我們用 python:3.7-alpine 作為 Base Image 製作一個簡單的 Image

```non
$ sudo docker pull python:3.7-alpine
```

在 dockertest 資料夾中新增一個 python 檔案

```non
$ vim dockertest/main.py
```

這個腳本每 5 秒會印出藝術文字，我要讓 Image 持續印出藝術文字

```python
from art import *
import time

while True:
    tprint("Hello Docker","random")
    time.sleep(3)
```

新增一個 Dockerfile

```non
$ vim dockertest/Dockerfile
```

寫入以下內容

```docker
FROM python:3.7-alpine
WORKDIR /code
RUN pip install art
COPY main.py .
CMD ["python", "main.py"]
```

- **FROM**: Base Image
- **WORKDIR**: 目前在 Container 中的目錄
- **RUN**: Container 正在啟動中時執行的指令
- **COPY**: 將 Host 的檔案或目錄複製到 Container 的目錄中，上面範例是將 main.py 複製進 Container 的 /code 中
- **CMD**: Container 開始執行時下的指令

更多設定可以參考 [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

接下來打包成 Image，命名為 art，tag (通常用來表示版本) 為 1

```non
$ sudo docker build -t art:1 .
```

在前台執行

```non
$ sudo docker run art:1

|_| _|| _   |~\ _  _|  _._
| |}_||(_)  |_/(_)(_|<}_| 

#  #        ##    ##                ###               #                 
#  #         #     #                #  #              #                 
####   ##    #     #     ##         #  #   ##    ##   # #    ##   ###   
#  #  # ##   #     #    #  #        #  #  #  #  #     ##    # ##  #  #  
#  #  ##     #     #    #  #        #  #  #  #  #     # #   ##    #     
#  #   ##   ###   ###    ##         ###    ##    ##   #  #   ##   #    

. . .

```

我自己實作時不會三秒顯示一個藝術文字，而是過 30 秒突然爆出 10 個藝術文字，不知道是不是因為 `time.sleep()` 在 docker 中會跟 stdout 互相干擾

刪除 Container 和 Image 就和前述步驟差不多，不再演示

## 6. Docker-Compose

前面實際操作如何創建 Container 和 Image 後不難體會 Docker 容器隔離和輕量化的方便性，docker 可以透過 image 快速複製出許多環境相同的 container，同時幾乎不會影響到 host 的運作，又節省空間。

然而要提供一個完整的服務，僅僅只用一個 image 和一個 container 顯然是不夠的，雖然仍然有人製作包山包海的 docker image，但是考慮效能、彈性、維護性，這不是個好的設計模式；另一方面，同時執行多個 Container 也很讓人頭大，沒有人的腦容量可以記得各個容器繁雜的指令、以及每個容器之間的交互關係。

這時就是 Docker-Compose 出馬的時候了，其用於處理多個容器一起執行的情景，透過一個 yml 檔規範所有容器的 dockerfile、掛載目錄、虛擬網路等，使用**單個指令**便能啟動所有容器。

以下的操作取自官網的 [Get started with Docker Compose](https://docs.docker.com/compose/gettingstarted/)，其為一個 flask 結合 redis 的網頁伺服器

首先創建 app.py

```non
$ vim dockertest/app.py
```

```python
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```

requirements.txt

```non
$ vim dockertest/requirements.txt
```

```
flask
redis
```

建立 Dockerfile

```non
$ vim dockertest/Dockerfile
```

```docker
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```

建立 Compose File

```non
$ vim dockertest/docker-compose.yml
```

```yml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```

- **version**: 規範 yml 該如何撰寫，每個版本大同小異，細節見官網
- **services**: 基本上每個 service 都是一個 container
- **web**、**redis**: 自定義的 service 名稱
- **build**: 該服務的 dockerfile 所在目錄，有些 image 不需要過度客製化，所以沒有 dockerfile
- **ports**: 連接埠轉送
- **image**: 若無 build 標籤，則由 image 指定 base image

更多資訊詳見 [Compose file version 3 reference](https://docs.docker.com/compose/compose-file/compose-file-v3/) 

最後，只要在 dockertest 目錄執行一個指令便可啟動服務，-d 代表背景執行

```non
$ sudo docker-compose up -d
```

使用終端機或瀏覽器測試

```non
$ curl localhost:5000
Hello World! I have been seen 1 times.
$ curl localhost:5000
Hello World! I have been seen 2 times.
...
```

輸入以下指令終止服務

```non
$ sudo docker-compose down
```

如果想要**修改**這份 flask 範例，要記得在 web 服務掛載 dockertest 目錄，這樣每次 docker-compose up 時才會更新 app.py 的程式碼，修改後的 docker-compose.yml 如下:

```yml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
    environment:
      FLASK_ENV: development
  redis:
    image: "redis:alpine"
```

## 心得

Docker 太神啦!

## 參考資料

[Docker 建立 Nginx 基礎分享](https://medium.com/@xroms123/docker-%E5%BB%BA%E7%AB%8B-nginx-%E5%9F%BA%E7%A4%8E%E5%88%86%E4%BA%AB-68c0771457fb) <=大推這份筆記，本篇內容很多是參考他的，感謝這位網友!!

[Get started with Docker Compose](https://docs.docker.com/compose/gettingstarted/) <=官網其實資訊很足夠，又不會多到讓人看不完!!

[Docker Tip #10: Project Structure with Multiple Dockerfiles and Docker Compose](https://nickjanetakis.com/blog/docker-tip-10-project-structure-with-multiple-dockerfiles-and-docker-compose) <= 這篇討論串在談如何處理多個 Dockerfile，算延伸閱讀吧