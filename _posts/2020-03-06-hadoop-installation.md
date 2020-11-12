---
title: "Hadoop 3.2.1 安裝教學"
subtitle: ""
layout: post
author: "blueskyson"
header-style: text
tags:
  - hadoop
---

最近正在學習 Hadoop，發現 Hadoop 的安裝過程繁瑣，每個人的裝置又可能產生不一樣的問題，且中文資源較少，所以想好好寫個筆記紀錄一下，未來安裝 Hadoop 時可以參考，同時也希望能幫助新手

## Step 1: 安裝 java

打開 terminal 運行以下指令

```non
$ sudo apt install openjdk-8-jre-headless
$ sudo apt install openjdk-8-jdk-headless
```

建議安裝 java 8 比較穩定，安裝太新或太舊的 java 運行 hadoop 時可能會有不預期的錯誤發生...

## Step 2: 安裝 ssh 連線工具

```non
$ sudo apt install ssh
$ sudo apt install pdsh
```

先產生一組金鑰，將公鑰複製進信任清單，然後把信任清單的權限提高

```non
$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
$ chmod 600 ~/.ssh/authorized_keys
```

嘗試不用輸入密碼，直接連到 localhost

```non
$ ssh localhost
```

如果出現

<span style="color: red">
ssh: connect to host localhost port 22: Connection refused
</span>
{: .notice--danger}

請先安裝openssh-server

```non
$ sudo apt-get install openssh-server
```

在安裝時若遇到以下錯誤訊息:

<span style="color: red">
The following packages have unmet dependencies:  
openssh-server : Depends: openssh-client (= 1:7.6p1-4)  
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&nbsp;Depends: openssh-sftp-server but it is not going to be installed  
E: Unable to correct problems, you have held broken packages.
</span>
{: .notice--danger}

請先嘗試更新 apt-get，然後再下載一次openssh-server

```non
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get autoremove
```

如果還是不行，先把 Depends 砍掉(如果有的話)，再重新安裝回去

```non
$ sudo apt-get purge openssh-client
$ sudo apt-get purge open-sftp-server
$ sudo apt-get install openssh-client
$ sudo apt-get openssh-server
```

若還是不行，請左轉 StackOverflow，或換別的版本的 Ubuntu

## Step 3: 到 Apache Hadoop 官網下載安裝包

![](https://i.imgur.com/ljoWq7t.jpg)

選 hadoop-3.2.1 下載，或是去 mirror site 選一個版本下載: [ftp.mirror.tw/pub/apache/hadoop/common/](http://ftp.mirror.tw/pub/apache/hadoop/common/)

到下載的目錄解壓縮並放入你指定的目錄，我個人習慣放到
~/Programs/

```non
$ tar -xvf hadoop-3.2.1.tar.gz
$ mkdir ~/Programs
$ mv hadoop-3.2.1/ ~/Programs/
```

其他的教學可能會把 Hadoop 放在 /opt 或 /usr/local 都無所謂，但可能遇到存取權限的問題，這個時候用 chmod 700 或 chmod 770 去放寬檔案或目錄權限就好了

## Step 4: 編輯環境變數

```non
$ vim ~/.bashrc
```

打開 bashrc，環境變數設置如下:

```non
# ~/.bashrc: executed by bash(1) for non-login shells.
# see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
# for examples

. . .

# hadoop environment variables
export JAVA_HOME=/usr/lib/jvm/"java-1.8.0-openjdk-amd64"
export CLASSPTH=$JAVA_HOME/lib
export HADOOP_INSTALL=/home/vm3/Programs/"hadoop-3.2.1"
export PATH=$PATH:$HADOOP_INSTALL/bin
export PATH=$PATH:$HADOOP_INSTALL/sbin
export HADOOP_MAPRED_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_HOME=$HADOOP_INSTALL
export YARN_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_INSTALL/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_INSTALL/lib"
export HADOOP_CLASSPATH=$(hadoop classpath)

. . .
```

請注意設置 HADOOP_INSTALL 時要填寫自己的家目錄名稱，以我的這個例子，我的家目錄是 /home/vm3/... 請勿照抄"vm3"，不然電腦會找不到路徑。

完成後執行以下命令，確認hadoop成功安裝

```non
$ source ~/.bashrc
$ hadoop version
```

## Step 5: 編輯 Hadoop 設定檔

首先指定 file system 的主機位置

```non
$ cd ~/Programs/hadoop-3.2.1/etc/hadoop/
$ vim core-site.xml
```

```xml
<!-- namenode的位置 -->
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
  </property>
</configuration>
```

指定 datanode 及 namenode 的儲存位置，這裡以 ~/Programs/hdfs/ 為例

```non
$ mkdir ~/Programs/hdfs
$ mkdir ~/Programs/hdfs/datanode
$ mkdir ~/Programs/hdfs/namenode
$ vim hdfs-site.xml
```

```xml
<configuration>
  <!-- hdfs的數據副本數量 -->
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <!-- namenode的儲存位置 -->
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/home/vm3/Programs/hdfs/namenode</value>
  </property>
  <!-- datanode的儲存位置 -->
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/home/vm3/Programs/hdfs/datanode</value>
  </property>
</configuration>
```

注意，這裡也要把 "vm3" 改成自己的 user name 呀 >"<

更多關於 hadoop-3.2.1 設定檔變數的定義可以在這裡找到

### [core-default.xml](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/core-default.xml)

### [hdfs-default.xml](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml)

### [hdfs-rbf-default.xml](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs-rbf/hdfs-rbf-default.xml)

### [mapred-default.xml](https://hadoop.apache.org/docs/stable/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml)

### [yarn-default.xml](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-common/yarn-default.xml)

## Step 6: 開始運作 Hadoop

```non
$ hadoop namenode -format
```

沒意外的話，執行完上面的指令，會如下圖所示

![](https://i.imgur.com/nd8OEpj.jpg)

倒數第五行訊息為
`INFO namenode.FSImage: FSImageSaver clean checkpoint: txid=0 when meeting shutdown.` 
按照前面步驟做的話， txid 的值應該是 0，如果不是，請檢查是否有哪一步做錯了，接下來執行

```non
$ start-all.sh
$ jps
```

如果執行時出現錯誤訊息

<div class="notice" markdown="0">
#### Headline for the Notice
localhost: rcmd: socket: Permission denied
</div>



可以參考[這裡](https://stackoverflow.com/questions/42756555/permission-denied-error-while-running-start-dfs-sh)

![](https://i.imgur.com/MhfzTui.jpg)

請注意，執行jps後，一定要有 DataNode，如果沒有的話是因為 DataNode 和 NameNode 的版本不同，請輸入以下指令把它們清空，然後重開 Hadoop

```non
$ stop-all.sh
$ rm -r /tmp/hadoop-vm3 (此目錄視使用者名稱而定)
$ rm -r ~/Programs/hdfs/datanode/*
$ rm -r ~/Programs/hdfs/namenode/*
$ hadoop namenode -format
$ start-all.sh
```

最後，打開瀏覽器，連到 localhost:9870 (2.x.x版為localhost:50070)，進入 Hadoop 的網頁 UI 就大功告成:smile:

![](https://i.imgur.com/xlVB6QJ.jpg)