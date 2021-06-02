---
title: "使用 nginx 與 flask 架設 key-value storage 服務運行於 docker 容器"
subtitle: ""
excerpt: "nginx flask proxy_pass docker"
layout: post
author: "blueskyson"
header-style: text
tags:
  - nginx
  - flask
  - python
  - docker
---

此為 2021 網路管理課程的 lab14，花了一整晚搞不定 php-fpm 在 docker 寫檔的問題 (已經調整過權限、目錄也 mount 了還是不行) 耗費五到六個小時，後來砍掉重練，改成用 python + redis 的記憶體儲存，兩個小時就幹完了，真是累死人。

以下只有 code，沒力氣打詳細解釋了...，共五個檔案

**app.py**

```python
import time

import redis
from flask import Flask, request

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

@app.route('/set/<key>', methods=['POST'])
def set(key):
    retries = 5
    while True:
        try:
            value = request.form.get('value')
            if value != None:
                cache.set(key, value)
                return 'OK'
            else:
                return ''
            
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/get/<key>', methods=['POST', 'GET'])
def get(key):
    if cache.exists(key):
        return cache[key]
    else:
        return ''
```

**requirements.txt**

```
flask
redis
```

**site.conf**

```
server {
    listen 80;
    index index.php index.html;
    server_name localhost;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /lab14;

    location / {
        proxy_pass http://flask:5000;
    }

    location ~ ^(/set)/[^/]+ {
        proxy_pass http://flask:5000;
    }
    
    location ~ ^(/get)/[^/]+ {
        proxy_pass http://flask:5000;
    }
}
```

**Dockerfile**

```docker
# syntax=docker/dockerfile:1
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

**docker-compose.yml**

```docker
version: "3.9"
services:
  nginx:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./site.conf:/etc/nginx/conf.d/default.conf
  flask:
    build: .
    volumes:
      - .:/code
  redis:
    image: "redis:alpine"
```

將這些檔案放到讀一目錄，然後執行 `sudo docker-compose up` 即可

用 localhost:8080 測，測試方式跟 [lab7](/2021/04/13/nginx-key-value-storage/) 一樣