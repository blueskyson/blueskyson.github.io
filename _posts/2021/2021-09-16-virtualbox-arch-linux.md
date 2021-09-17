---
title: "在 VirtualBox 安裝 Arch Linux"
subtitle: ""
excerpt: "virtualBox arch linux virtual machine"
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - linux
---

## 摘要

以下安中步驟主要參考自 [youtube](https://www.youtube.com/watch?v=LaJ1yl6NckI&list=PLbu-NbyN5TS1T0380-LBFO9wyDvflonpr&index=2&t=424s)，我只是將安裝過程的指令和螢幕截圖下來，如果聽英文沒問題可以直接看影片。對分割、掛載磁碟、設定 Linux 環境很熟悉的人可以直接按照以下摘要的指令安裝：

```non
# setfont ter-224b.psf.gz
# ping google.com
^C
# ls /sys/firmware/efi/efivars
# timedatectl set-ntp true
# timedatectl status
# cfdisk

gpt
New
512M
Type
EFI System
New
<Enter>
Write
yes
Quit

# lsblk
# mkfs.fat -F32 /dev/sda1
# mkfs.ext4 /dev/sda2
# mkdir /mnt/efi
# mount /dev/sda1 /mnt/efi
# mount /dev/sda2 /mnt
# lsblk
# pacstrap /mnt base linux linux-firmware vim nano
# genfstab -U /mnt >> /mnt/etc/fstab
# cat /mnt/etc/fstab
# arch-chroot /mnt
# ln -sf /usr/share/zoneinfo/Asia/Taipei /etc/localtime
# hwclock --systohc

# vim /etc/locale.gen
取消註解 en_US.UTF-8 UTF-8
取消註解 zh_TW.UTF-8 UTF-8
<Esc>:wq

# vim /etc/locale.conf
LANG=en_US.UTF-8
<Esc>:wq

# vim /etc/hostname
鍵入 hostname
<Esc>:wq

# vim /etc/hosts
127.0.0.1   localhost
::1         localhost
127.0.1.1   hostname.localdomain    hostname
<Esc>:wq

# passwd
# useradd -G wheel,audio,video -m username
# passwd username
# pacman -Syu netctl dhcpcd sudo

# EDITOR=vim visudo
取消註解 %wheel ALL=(ALL) ALL
<Esc>:wq

# pacman -Syu grub efibootmgr
# grub-install --target=x86_64-efi --efi-directory=/efi/ --bootloader-id=Arch
# grub-mkconfig -o /boot/grub/grub.cfg
# exit
# shutdown now

移除映像檔，重開虛擬機

# cd /etc/netctl
# sudo cp examples/ethernet-dhcp .
# ip a
查看虛擬網卡

# sudo vim ethernet-dhcp
Interface=虛擬網卡
取消註解 DHCPClient=dhcpcd
<Esc>:wq

# sudo systemctl enable dhcpcd
# sudo systemctl start dhcpcd
# ping www.google.com
^C
```

## Step 1: 創建虛擬機

1. 去 Arch Linux 的鏡像站下載 iso 檔，在台灣最穩定更新的應該屬[交大資工計算機中心](https://linux.cs.nctu.edu.tw/)。

2. 安裝 VirtualBox，會點進來看的人應該都已經安裝了。

3. 創建虛擬機，儲存空間建議至少要 8 GB，會點進來看的人應該都很熟 VirtualBox 的設定選項了，但還是附一張截圖。
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/1.png)
   建議處理器可以分配 2 至 4 顆，可能會安裝比較快。

4. 接下來這一步就很重要了，請點選`系統`->`主機板`->`啟用 EFI`，這樣才能在安裝完畢後由 `grub` 引導進入系統。
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/2.png)

5. 請插入 Arch Linux iso 到虛擬機中，或是開機時 VirtualBox 也會提示要不要插入光碟鏡像。
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/3.png)

做完以上步驟就可以按下啟動，進到裝系統環節啦。

## Step 2: 設定字體、時間

1. 開機畫面**必須**長這樣，如果你發現開機畫面有一張漂亮的淺藍色底圖，代表沒有進到 EFI 模式，請回去設定`系統`->`主機板`->`啟用 EFI`。
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/4.png)

2. 首先進到終端機介面，最令人頭痛的就是預設字體實在小到不行，我們可以透過以下指令列出字體設定檔。
   ```non
   # ls /usr/share/kbd/consolefonts/
   ```
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/5.png)
   從列表中的字體包一個一個試自己看得舒服的字體，我是按照影片使用 ter-224b.psf.gz 
   ```non
   # setfont ter-224b.psf.gz
   ```
   如果終端機畫面太小，印不出目錄中所有檔案，可以用 `less` 查看：
   ```non
   # ls /usr/share/kbd/consolefonts/ | less
   ```

3. 調整完字體後，測試是否連上網路，照理來說是可以連上的，連不上請去其他網站找解決方法。
   ```non
   # ping google.com
   ```
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/6.png)

4. 再次檢查 EFI 是否已啟用，如果列出來是空的或是 no such directory，也請移駕他處尋求解答。
   ```non
   # ls /sys/firmware/efi/efivars
   ```
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/7.png)

5. 透過 ntp 進行網路校時，並確認時間是否同步。
   ```non
   # timedatectl set-ntp true
   # timedatectl status
   ```
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/8.png)

接下來開始分割磁區。

## Step 3: 分割磁碟

1. 有若干個分割磁碟的工具如 fdisk、gdisk、cfdisk、cgdisk，我這裡使用 cfdisk。
   ```non
   # cfdisk
   ```
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/9.png)
   選擇 `gpt`
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/10.png)
   選擇 `New`，我們首先要分割 EFI 分區，影片中是分割 1 G，我這次則是給它 512 MB
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/11.png)
   接下來選取 `Type`
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/12.png)
   選擇最上面的 `EFI System` 然後按下 \<Enter\>
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/13.png)
   選取剩餘的磁碟按下 `New` 
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/14.png)
   把剩下空間全部劃為檔案系統。有些教學會再多劃分一個 swap 分區，但是我的硬碟空間很小，而起記憶體夠大，所以就不劃分 swap 了。
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/15.png)
   選擇 `Write` 把目前配置寫入磁碟。
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/16.png)
   選擇 `Quit`
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/17.png)

2. 接下來用 `mkfs` 格式化磁碟區，EFI 用的是類似 USB 的 FAT，而 Linux 系統通常用 ext4。建議可以先用 `lsblk` 查看分區大小及編號是否正確，通常命名為 sda1、sda2。
   ```non
   # lsblk
   # mkfs.fat -F32 /dev/sda1
   # mkfs.ext4 /dev/sda2
   ```
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/18.png)

3. 掛載磁區，首先建立 EFI 掛載點 /mnt/efi，然後依序掛載 EFI 和系統磁區。
   ```non
   # mkdir /mnt/efi
   # mount /dev/sda1 /mnt/efi
   # mount /dev/sda2 /mnt
   # lsblk
   ```
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/19.png)

4. 接下來終於要透過 `pacstrap` 將 Linux 系統安裝到掛載點了，必須要安裝 base、linux、linuxfirmware，以及你常用的編輯器 vim 或 nano 等。需要花一些時間等它下載安裝。
   ```non
   # pacstrap /mnt base linux linux-firmware vim nano
   ```
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/20.png)
5. 用 `genfstab` 產生開機自動掛載的配置檔案。注意 /dev/sda1 和 /dev/sda2 都要有，如果不見了，請先刪除 /mnt/etc/fstab，回到 3. 重新 `mount` 一次，然後再試一次 `genfstab`。
   ```non
   # genfstab -U /mnt >> /mnt/etc/fstab
   # cat /mnt/etc/fstab
   ```
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/21.png)

## Step 4: 設定系統環境

1. 透過 `arch-chroot` 進入新系統的根目錄。
   ```non
   # arch-chroot /mnt
   ```

2. 設定 timezone 以及系統時間。
   ```non
   # ln -sf /usr/share/zoneinfo/Asia/Taipei /etc/localtime
   # hwclock --systohc
   ```
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/22.png)

3. 設定系統語言，取消註解 en_US.UTF-8 以及 zh_TW.UTF-8，保存退出。
   ```non
   # vim /etc/locale.gen
   ```
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/23.png)
   影片中沒有執行以下兩個動作。動作一：
   ```non
   # locale-gen
   ```
   動作二：
   ```non
   # vim /etc/locale.conf
   LANG=en_US.UTF-8
   <Esc>:wq
   ```
   這兩個動作其實也可以等到裝完系統後再處理。

4. 設定 etc/hostname 與 /etc/hosts，注意以 tab 分隔 IP 和 hostname，不是用空白鍵分隔。
   ```non
   # vim /etc/hostname
   ```
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/24.png)
   ```non
   # vim /etc/hosts
   ```
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/25.png)

5. 設定 root 密碼、新增 user (我新增的 user 叫　lin)。
   ```non
   # passwd
   # useradd -G wheel,audio,video -m lin
   # passwd lin
   ```
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/26.png)

6. 下載網路和套件和 sudo。
   ```non
   # pacman -Syu netctl
   # pacman -Syu dhcpcd
   # pacman -Syu sudo
   ```

7. 給予 wheel 群組系統權限，即給予剛剛創建的使用者 (lin) 系統權限。
   ```non
   # EDITOR=vim visudo
   ```
   取消註解 `%wheel ALL=(ALL) ALL`
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/27.png)

8. 安裝 grub 和 efi 套件。
   ```non
   # pacman -Syu grub efibootmgr
   # grub-install --target=x86_64-efi --efi-directory=/efi/ --bootloader-id=Arch
   # grub-mkconfig -o /boot/grub/grub.cfg
   ```
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/28.png)

至此終於安裝完 Arch Linux 了，可以退出 arch-chroot，關閉電腦。

```non
# exit
# shutdown now
```

## Step 5: 重開虛擬機、啟動 dhcpcd 服務

1. 回到 VirtualBox 的 `存放裝置`，退出開機光碟。
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/29.png)

2. 重新啟動虛擬機，用剛剛註冊的 user (lin) 登入。此時我們會發現無法連上網路，這是因為 dhcpcd 尚未啟用，透過以下指令啟用 dhcpcd。
   ```non
   # cd /etc/netctl
   # sudo cp examples/ethernet-dhcp .
   # ip a
   ```
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/30.png)
   記住虛擬網卡的名子叫 enp0s3，要把網卡寫進設定檔中，並取消註解 `DHCPClient=dhcpcd`。
   ```non
   # sudo vim ethernet-dhcp
   ```
   ![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/31.png)
   啟用並開始執行 dhcpcd 服務。
   ```non
   # sudo systemctl enable dhcpcd
   # sudo systemctl start dhcpcd
   ```

3. 測試網路是否成功連接，如果這裡出問題也請自行搜尋解法。
   ```non
   # ping www.google.com
   ```

# Step 6: 大功告成

下載 neofetch 並截圖跟親朋好友分享你的喜悅吧！

```non
# sudo pacman -Syu neofetch
# neofetch
```

![](https://raw.githubusercontent.com/blueskyson/image-host/master/archlinux-vm/32.png)

如果這篇網誌成功幫助到您，不妨動動小手幫這個網站點顆星星 [https://github.com/blueskyson/blueskyson.github.io](https://github.com/blueskyson/blueskyson.github.io)。

安裝完 Arch Linux 可以參考這一篇設定系統：[Arch Linux 基本設定](/2021/09/16/archlinux-setup/)
