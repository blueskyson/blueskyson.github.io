---
title: "LDAP 安裝和設定"
subtitle: ""
excerpt: "LDAP"
layout: post
author: "blueskyson"
header-style: text
tags:
  - linux
---

此為計算機網路管理課程的 lab11

## 安裝

參考這篇文章 [https://magiclen.org/ubuntu-server-ldap/](https://magiclen.org/ubuntu-server-ldap/)

安裝時，照著 ldap 的類 GUI 提示設定 admin 密碼、將 nasa.imslab.org 設為 root dn

```non
$ sudo apt install slapd
$ sudo dpkg-reconfigure slapd
$ sudo apt install ldap-utils
```

透過以下指令確認 LDAP 是否成功執行

```non
$ ldapsearch -x -b '' -s base '(objectclass=*)' namingContexts
```

輸出

```non
# extended LDIF
#
# LDAPv3
# base <> with scope baseObject
# filter: (objectclass=*)
# requesting: namingContexts
#

#
dn:
namingContexts: dc=nasa,dc=imslab,dc=org

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

透過以下指令確認 LDAP 監聽的 port (通常為 389)

```non
$ sudo netstat -tulnp | grep slapd
```

輸出

```non
tcp   0   0 0.0.0.0:389   0.0.0.0:*   LISTEN      60490/slapd
tcp6  0   0 :::389        :::*        LISTEN      60490/slapd
```

## 編輯 ldif 檔

假設需求為

- Base DN: dc=nasa, dc=imslab, dc=org
- OU:
  + user
    - F74076027

### Step 1. 將 nasa.imslab.org 設定為 organization

在任意目錄下創建 ldif 檔案，

```non
$ vim organization.ldif
```

```non
dn: dc=nasa,dc=imslab,dc=org
objectclass: dcObject
objectclass: organization
o: NASA
dc: nasa

dn: cn=admin,dc=nasa,dc=imslab,dc=org
objectclass: organizationalRole
cn: admin
```

```non
$ ldapadd -x -D "cn=admin,dc=nasa,dc=imslab,dc=org" -W -f organization.ldif
```

### Step 2. 新增 admin 為 nasa.imslab.org 的管理者

這一步似乎不需要做，但是我看到大部分教學都有設定一個 admin 所以就跟著做了。我不確定這個動作是覆寫預設的 admin，還是創建一個毫不相干的 admin，詳情需要再爬文

```non
$ vim admin.ldif
```

```non
dn: cn=admin,dc=nasa,dc=imslab,dc=org
objectclass: organizationalRole
cn: admin
```

```non
$ ldapadd -x -D "cn=admin,dc=nasa,dc=imslab,dc=org" -W -f organization.ldif
```

### Step 3. 新增 user group

```non
$ vim user.ldif
```

```non
dn: ou=user,dc=nasa,dc=imslab,dc=org
ou: user
objectClass: organizationalUnit
```

```non
$ ldapadd -x -D "cn=admin,dc=nasa,dc=imslab,dc=org" -W -f user.ldif
```

### Step 4. 新增學號 unit

```non
$ vim f74076027.ldif
```

```non
dn: cn=F74076027,ou=User,dc=nasa,dc=imslab,dc=org
objectClass: organizationalPerson
objectClass: person
objectClass: posixAccount
cn: F74076027
gidNumber: 10001
homeDirectory: /home/F74076027
sn: F74076027
uid: F74076027
uidNumber: 10001
```

```non
$ ldapadd -x -D "cn=admin,dc=nasa,dc=imslab,dc=org" -W -f f74076027.ldif
```

### 查看是否成功

```non
$ ldapsearch -x -b "dc=nasa,dc=imslab,dc=org"
```

```non
# extended LDIF
#
# LDAPv3
# base <dc=nasa,dc=imslab,dc=org> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# nasa.imslab.org
dn: dc=nasa,dc=imslab,dc=org
objectClass: top
objectClass: dcObject
objectClass: organization
o: nasa
dc: nasa

# user, nasa.imslab.org
dn: ou=user,dc=nasa,dc=imslab,dc=org
ou: user
objectClass: organizationalUnit

# admin, nasa.imslab.org
dn: cn=admin,dc=nasa,dc=imslab,dc=org
objectClass: simpleSecurityObject
objectClass: organizationalRole
cn: admin
description: LDAP administrator

# F74076027, user, nasa.imslab.org
dn: cn=F74076027,ou=user,dc=nasa,dc=imslab,dc=org
objectClass: organizationalPerson
objectClass: person
objectClass: posixAccount
cn: F74076027
gidNumber: 10001
homeDirectory: /home/F74076027
sn: F74076027
uid: F74076027
uidNumber: 10001

# search result
search: 2
result: 0 Success

# numResponses: 5
# numEntries: 4
```

接下來會裝設 ldap 的 GUI 介面，以上 step. 1 ~ 4 的操作其實可以在 GUI 介面點一點就完成

## 透過 ldapadmin 連線至 LDAP

在這裡下載 ldapadmin 執行檔 [http://www.ldapadmin.org/download/ldapadmin.html](http://www.ldapadmin.org/download/ldapadmin.html)

首先用 ssh port fowarding 將本機的 8080 轉送至實驗室虛擬機的 389，在 windows 可以用 openssh 或 putty 達成目的，以下是在本地端的 windows 系統使用 openssh:

```non
ssh F74076027@140.116.246.189 -p 22034 -L 8080:localhost:389
```

打開 ldapadmin，點擊左上 `start` -> `Connect` -> `New Connection` 輸入 ldap 的資訊:

![](https://raw.githubusercontent.com/blueskyson/image-host/master/ldap.png)

完成後即可連線至 ldap

![](https://raw.githubusercontent.com/blueskyson/image-host/master/ldap2.png)

