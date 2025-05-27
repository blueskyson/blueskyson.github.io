---
title: "在 Ubuntu 24 使用 QEMU KVM 安裝虛擬機"
subtitle: ""
excerpt: "qemu kvm libvirt virt manager"
layout: post
author: "blueskyson"
header-style: text
tags:
  - linux
---

QEMU/KVM，簡稱 Quick Emulator，是一套開源的虛擬化解決方案，結合了 QEMU 的模擬能力與 KVM（Kernel-based Virtual Machine）的硬體加速優勢。QEMU 本身屬於 Type 2 Hypervisor，透過軟體層模擬虛擬機，而當與 Type 1 Hypervisor 的 KVM 搭配時，則能實現接近原生的執行效能。在 Ubuntu 24 上部署 QEMU/KVM 不僅能有效建立與管理虛擬機，還能為開發、測試與學習提供強大而穩定的平台。

## 安裝 QEMU/KVM

開啟 libvirtd 虛擬化服務

```
$ sudo systemctl enable libvirtd.service
$ sudo systemctl start libvirtd.service
```

安裝 QEMU/KVM 相關套件，重新開機

```
$ sudo apt update
$ sudo apt install qemu-kvm virt-manager bridge-utils
$ sudo reboot now
```

為了讓你的使用者帳號可以不使用 root 權限就操作虛擬機，你必須將使用者加入這些授權群組：
- libvirt：讓使用者能與 libvirt daemon 互動，控制虛擬機的建立、啟動、暫停等操作。
- libvirt-qemu 或 libvirt-kvm：讓使用者能夠與 QEMU/KVM 後端直接互動，包括存取虛擬機映像檔、虛擬網路等。

```
$ groups 
lin adm cdrom sudo dip plugdev users lpadmin libvirt
$ sudo useradd -g $USER libvirt
$ sudo useradd -g $USER libvirt-kvm
```

## 新增虛擬機

到你所需的 Linux 發行版官網下載 ISO 映像檔。我以 Ubuntu 24 Server 為例。

在終端機中輸入 `virt-manager`，或是 Apps 選單中啟動虛擬機管理器，透過 GUI 介面來建立新的虛擬機。

![](https://raw.githubusercontent.com/blueskyson/image-host/refs/heads/master/2025/qemu-kvm-1.png)

左上角點選 File -> New Virtual Machine。

在新視窗中，選擇 Local install media (ISO image or CDROM)，然後點選 Forward。

選擇你剛才下載的 ISO 映像檔，然後點選 Forward。

選擇虛擬機的名稱、記憶體大小、CPU 核心數量等設定。

完成設定後，虛擬機會自動啟動，並進入 Linux 的安裝流程。

## 設定 NAT DHCP 為虛擬機分配的 IP 位址

QEMU/KVM 預設使用 NAT 模式來提供虛擬機的網路連線，並會隨機分配一個 IP 位址給虛擬機，例如 192.168.XXX.XXX。如果你想要讓虛擬機有固定的 IP 位址，可以透過修改 `/etc/libvirt/qemu/networks/default.xml` 檔案來設定。

首先查看各個虛擬機的 MAC 位址，以下圖我有四台虛擬機為例：

![](https://raw.githubusercontent.com/blueskyson/image-host/refs/heads/master/2025/qemu-kvm-2.png)

```
$ virsh dumpxml vm1 | grep 'mac address'
     <mac address='52:54:00:e4:de:fb'/>
$ virsh dumpxml vm2 | grep 'mac address'
     <mac address='52:54:00:14:67:fa'/>
$ virsh dumpxml vm3 | grep 'mac address'
     <mac address='52:54:00:39:aa:1d'/>
$ virsh dumpxml vm4 | grep 'mac address'
     <mac address='52:54:00:95:aa:41'/>
```

查看你欲修改的 NAT 網路（一般就是 `default`）：

```
$ virsh  net-list
 Name      State    Autostart   Persistent
--------------------------------------------
 default   active   yes         yes
```

編輯 `/etc/libvirt/qemu/networks/default.xml` 檔案：

```
$ virsh  net-edit default
```

或是在 GUI 介面中，點選 Edit -> Preferences -> Enable XML editing。

再用 Edit -> Connection Details -> Virtual Networks -> XML 來修改。

![](https://raw.githubusercontent.com/blueskyson/image-host/refs/heads/master/2025/qemu-kvm-3.png)

原本的 `<dhcp>` 區塊應該為：

```xml
<dhcp>
  <range start="192.168.122.100" end="192.168.122.254"/>
</dhcp>
```

在 `<range>` 區塊下方新增 `<host>` 區塊，並填入虛擬機的 MAC 位址、虛擬機名稱、與你想要分配的 IP 位址：

```xml
<dhcp>
    <range start="192.168.122.2" end="192.168.122.254"/>
    <host mac="52:54:00:e4:de:fb" name="vm1" ip="192.168.122.11"/>
    <host mac="52:54:00:14:67:fa" name="vm2" ip="192.168.122.12"/>
    <host mac="52:54:00:39:aa:1d" name="vm3" ip="192.168.122.13"/>
    <host mac="52:54:00:95:aa:41" name="vm4" ip="192.168.122.14"/>
</dhcp>
```

儲存並關閉檔案後，重新啟動 NAT 網路：

```
virsh  net-destroy default
virsh  net-start default
```

重新啟動虛擬機即可。如果虛擬機有啟用 OpenSSH Server，則可以透過 SSH 連線到虛擬機：

```
$ ssh user@192.168.122.11
```

## 參考資料

- [How To install QEMU KVM & VirtManager on Ubuntu Run Virtual Machines On Ubuntu](https://www.youtube.com/watch?v=4m6eHhPypWI)
- [KVM/libvirt: How to configure static guest IP addresses on the virtualisation host](https://serverfault.com/questions/627238/kvm-libvirt-how-to-configure-static-guest-ip-addresses-on-the-virtualisation-ho)