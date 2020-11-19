---
title: "在 VirtualBox 虛擬集群上安裝 Zookeeper 3.6.2"
subtitle: "系統為 Ubuntu 20.04.1 LTS"
excerpt: "install zookeeper on vitualbox"
layout: post
author: "blueskyson"
header-style: text
tags:
  - apache
---

因為目前打算自幹一個分散式檔案系統作為專題，所以就想操作看看 zookeeper ，看其對分散式叢集的管理有多強大。最前面如何安裝 Ubuntu 就不演示了。

## 1st Step:

下載 zookeeper 3.6.2 ，我是在這裡下載的 [https://www.apache.org/dyn/closer.lua/zookeeper/zookeeper-3.6.2/apache-zookeeper-3.6.2-bin.tar.gz](https://www.apache.org/dyn/closer.lua/zookeeper/zookeeper-3.6.2/apache-zookeeper-3.6.2-bin.tar.gz)

下載 jvm 和 jdk ，我是自己是安裝 java 8

```non
$ sudo apt install openjdk-8-jre-headless
$ sudo apt install openjdk-8-jdk-headless
```

## 2nd Step:

解壓縮 zookeeper 放在你想放的路徑 (很多教學是放在 /opt )

```non
$ sudo tar -zxvf apache-zookeeper-3.6.2-bin.tar.gz
$ sudo mv zookeeper-3.6.2-bin /opt
```

## 3rd Step:

這時我們先關閉虛擬機，回到 VirtualBox 的介面右鍵選擇 `再製虛擬機`

![](https://raw.githubusercontent.com/blueskyson/image-host/master/zk-install/vm-clone.png)

下一步在 `MAC 位址原則` 選擇 `為所有網卡產生新的 MAC 位址`

![](https://raw.githubusercontent.com/blueskyson/image-host/master/zk-install/vm-clone-2.png)

然後選擇完整再製就能複製出資料一模一樣但虛擬網卡不同的機器，我總共再製了兩個，分別為 vm2 和 vm3

## 4th Step:

再製完後，幫每個機器新增 "僅限主機介面卡" ，它可以讓這部電腦上的虛擬機器互相連線，當然主體 OS 也能跟虛擬機連線

`設定` (橘色齒輪) -> `網路` -> `介面卡2` -> 勾選 `啟用網路卡`

- 選擇 附加到: `僅限主機介面卡`
- 介面卡類型我不懂， intel 機器就選 intel 的介面卡吧，其他平台的可能要試了才知道哪些是能用的
- 點開 "進階" 選擇 `全部允許`

![](https://raw.githubusercontent.com/blueskyson/image-host/master/zk-install/vm-clone-3.png)

每胎虛擬機都必須照上面步驟設定一次

## 5th Step:

修改虛擬機的 `hostname` ，將所有再製虛擬機重新命名，我自己是將新機器的 hostname 改名為 slave1 和 slave2

```non
$ sudo vim /etc/hostname
$ cat /etc/hostname
slave1
```

修改虛擬機的 `hosts` ，首先檢查每台虛擬機的 ip ，在 enp0s8 應該會有一組 192.168.x.x 的 ip ，以下面為例就是 192.168.56.107

```non
$ ip addr

. . .

3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:5a:89:a1 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.107/24 brd 192.168.56.255 scope global dynamic noprefixroute enp0s8
       valid_lft 399sec preferred_lft 399sec
    inet6 fe80::5b56:c2e0:6710:9fac/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

記住每個 hostname 對應的 ip ，修改所有虛擬機的 `hosts` ，以我的三個機器為例:

- master
  ```non
  $ sudo vim /etc/hosts
  $ cat /etc/hosts
  127.0.0.1	localhost
  127.0.1.1	master
  192.168.56.106  master
  192.168.56.107  slave1
  192.168.56.108  slave2

  # The following lines are desirable for IPv6 capable hosts
  ::1     ip6-localhost ip6-loopback
  fe00::0 ip6-localnet
  ff00::0 ip6-mcastprefix
  ff02::1 ip6-allnodes
  ff02::2 ip6-allrouters
  ```
- slave1
  ```non
  $ sudo vim /etc/hosts
  $ cat /etc/hosts
  127.0.0.1	localhost
  127.0.1.1	slave1
  192.168.56.106  master
  192.168.56.107  slave1
  192.168.56.108  slave2

  # The following lines are desirable for IPv6 capable hosts
  ::1     ip6-localhost ip6-loopback
  fe00::0 ip6-localnet
  ff00::0 ip6-mcastprefix
  ff02::1 ip6-allnodes
  ff02::2 ip6-allrouters
  ```
- slave2
  ```non
  $ sudo vim /etc/hosts
  $ cat /etc/hosts
  127.0.0.1	localhost
  127.0.1.1	slave2
  192.168.56.106  master
  192.168.56.107  slave1
  192.168.56.108  slave2

  # The following lines are desirable for IPv6 capable hosts
  ::1     ip6-localhost ip6-loopback
  fe00::0 ip6-localnet
  ff00::0 ip6-mcastprefix
  ff02::1 ip6-allnodes
  ff02::2 ip6-allrouters
  ```

修改完設定後請將所有虛擬機重開

## 6th Step:

接下來進到 zookeeper 目錄中的 conf/，新增一個 `zoo.cfg`

```non
$ sudo vim /opt/zookeeper-3.6.2-bin/conf/zoo.cfg
```

zoo.cfg 的設定如下:

```non
$ cat /opt/zookeeper/conf/zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
clientPort=2181
dataDir=/var/zookeeper
quorumListenOnAllIPs=true
server.1=master:2888:3888
server.2=slave1:2888:3888
server.3=slave2:2888:3888
```

`dataDir` 可以隨意指定，但大部分教學是設在 /var/zookeeper ，創建 dataDir

```non
$ sudo mkdir /var/zookeeper
```

`quorumListenOnAllIPs` 在虛擬機中最好要設定為 `true` ，因為 zookeeper 在設定 socket 時可能會綁到虛擬的 port ，導致其他機器連不上，在執行 zkServer.sh status 時會出錯

`server.x` 的 `x` 是叢集中的機器編號，在 zookeeper 根目錄和 dataDir 要新增 `myid` 檔案儲存對應的編號

- master
  ```non
  $ echo "1" > /opt/zookeeper-3.6.2-bin/myid
  $ echo "1" > /var/zookeeper/myid
  ```
- slave1
  ```non
  $ echo "2" > /opt/zookeeper-3.6.2-bin/myid
  $ echo "2" > /var/zookeeper/myid
  ```
- slave2
  ```non
  $ echo "3" > /opt/zookeeper-3.6.2-bin/myid
  $ echo "3" > /var/zookeeper/myid
  ```

## 7th Step:

走完以上流程後，在每台機器執行 `/opt/zookeeper-3.6.2-bin/bin/zkServer.sh start` 以開啟 zookeeper

再用 `/opt/zookeeper-3.6.2-bin/bin/zkServer.sh status` 查看哪一台機器為 leader

若這兩個指令都沒問題就代表安裝成功了，如果有指令失敗，可以查看 `/opt/zookeeper-3.6.2-bin/logs/` 中的內容想辦法處理

感謝收看!