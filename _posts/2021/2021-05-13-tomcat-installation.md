---
title: "Apache Tomcat 簡易安裝教學"
subtitle: ""
excerpt: "tomcat"
layout: post
author: "blueskyson"
header-style: text
tags:
  - apache
---

安裝 java 8

```non
$ sudo apt install openjdk-8-jdk
```

從官網下載 tomcat 的 release 版本，我選用 [10.0.6](https://tomcat.apache.org/download-10.cgi)，下載完成後解壓縮 `apache-tomcat-10.0.6.tar.gz`，然後撰寫一個 `setenv.sh` 用來存放 tomcat 所需的環境變數。

```non
$ tar -xvf apache-tomcat-10.0.6.tar.gz
$ cd apache-tomcat-10.0.6
$ vim setenv.sh
```

- `CATALINA_HOME` 和 `CATALINA_BASE` 為 tomcat 的路徑
- `CATALINA_PID` 可任意指定位置，存放著 tomcat 的 PID，同 `sudo ps -ef | grep tomcat` 所得到的 PID

```bash
export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
export CATALINA_HOME="/home/lin/Desktop/apache-tomcat-10.0.6"
export CATALINA_BASE="/home/lin/Desktop/apache-tomcat-10.0.6"
export CATALINA_PID="$CATALINA_BASE/tomcat.pid"
```

設定好環境變數後就可以執行 tomcat 了

```non
$ chmod 755 setenv.sh
$ source setenv.sh
$ ./bin/startup.sh
```

接著開啟 localhost:8080 便可開啟預設的首頁了

![](https://raw.githubusercontent.com/blueskyson/image-host/master/tomcat.png)