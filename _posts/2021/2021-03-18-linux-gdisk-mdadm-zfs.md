---
title: "gdisk 新增磁區、mdadm 磁碟陣列、ZFS 設定"
subtitle: ""
excerpt: "gdisk mdadm RAID ZFS"
layout: post
author: "blueskyson"
header-style: text
tags:
  - linux
---

此文為`計算機系統與網路管理`課程 lab4 的實作

## gdisk

首先助教分配了 `/dev/vdb` 和 `/dev/vdc` 這兩顆磁碟，各自都有 8 GiB 的容量。首先要將它們各自切成 4 GiB 的分割，總共會產生 4 個磁區。

因為我是先做完一次了，所以 `gdisk` 進入時會偵測到 GPT table 已經存在。如果是完全空的磁碟， `gdisk` 會在記憶體中先創建一個 GPT table ，在分割完存檔之後會自動把 GPT table 寫到硬碟中。記得設定完成一定要輸入 `w` 儲存。

```non
$ sudo gdisk /dev/vdc
GPT fdisk (gdisk) version 1.0.5

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with corrupt MBR; using GPT and will write new
protective MBR on save.

Command (? for help): p
Disk vdc: 16777216 sectors, 8.0 GiB
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 2D642148-4BF0-4563-BB9A-9272A52AB41E
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 16777182
Partitions will be aligned on 2048-sector boundaries
Total free space is 16777149 sectors (8.0 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name

Command (? for help): n
Partition number (1-128, default 1):
First sector (34-16777182, default = 2048) or {+-}size{KMGTP}:
Last sector (2048-16777182, default = 16777182) or {+-}size{KMGTP}: +4096M
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300):
Changed type of partition to 'Linux filesystem'

Command (? for help): n
Partition number (2-128, default 2):
First sector (34-16777182, default = 8390656) or {+-}size{KMGTP}:
Last sector (8390656-16777182, default = 16777182) or {+-}size{KMGTP}:
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300):
Changed type of partition to 'Linux filesystem'

Command (? for help): p
Disk vdc: 16777216 sectors, 8.0 GiB
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 2D642148-4BF0-4563-BB9A-9272A52AB41E
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 16777182
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         8390655   4.0 GiB     8300  Linux filesystem
   2         8390656        16777182   4.0 GiB     8300  Linux filesystem

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): Y
OK; writing new GUID partition table (GPT) to vdc.
The operation has completed successfully.
```

這樣便分割好 `\dev\vdc` ，對於 `\dev\vdb` 也是相同的操作

## mdadm 建立 RAID1 陣列

我第一次做時沒看清楚題目，把 `vdb1` 和 `vdb2` 做成 RAID0 ，指令如下:

`sudo mdadm -Cv -l 0 /dev/md0 -n 2 /dev/vdb1 /dev/vdb2`

正確來說 `-l` 後面要打 `1` 才是 RAID1，所以我又輸入以下指令還原:

```non
$ cat /proc/mdstat
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid0 vdb2[1] vdb1[0]
      8376832 blocks super 1.2 512k chunks

unused devices: <none>
$ sudo mdadm --stop /dev/md0
mdadm: stopped /dev/md0
$ sudo mdadm --zero-superblock /dev/vdb1 /dev/vdb2
$ sudo cat /proc/mdstat
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
unused devices: <none>
```

現在用 `vdb1` 和 `vdc1` 重新造一個 RAID1

```non
$ sudo mdadm -Cv -l 1 /dev/md0 -n 2 /dev/vdb1 /dev/vdc1
mdadm: /dev/vdb1 appears to contain an ext2fs file system
       size=4194304K  mtime=Thu Jan  1 00:00:00 1970
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: /dev/vdc1 appears to contain an ext2fs file system
       size=4194304K  mtime=Thu Jan  1 00:00:00 1970
mdadm: size set to 4189184K
Continue creating array?
```

會遇到上面這樣的提示，直接按 Y 就好。

之後需要等 mdadm 重新同步:

```non
$ sudo cat /proc/mdstat
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid1 vdc1[1] vdb1[0]
      4189184 blocks super 1.2 [2/2] [UU]
      [=>...................]  resync =  7.0% (295872/4189184) finish=12.7min speed=5069K/sec

unused devices: <none>
```

同步完成後，將其格式化成 ext4

```non
$ sudo mkfs.ext4 /dev/md0
mke2fs 1.45.5 (07-Jan-2020)
Discarding device blocks: done
Creating filesystem with 1047296 4k blocks and 262144 inodes
Filesystem UUID: 2a43811b-197a-4139-893f-825dabd463d7
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

mount 到 `/mnt/raid1`

```non
$ sudo mkdir /mnt/raid1
$ sudo mount /dev/md0 /mnt/raid1
```

## ZFS

```non
$ sudo zpool create ncku-nasa /dev/vdb2 /dev/vdc2
$ zpool status
  pool: ncku-nasa
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        ncku-nasa   ONLINE       0     0     0
          vdb2      ONLINE       0     0     0
          vdc2      ONLINE       0     0     0

errors: No known data errors
$ zfs get all ncku-nasa | grep compress
ncku-nasa  compressratio         1.00x                  -
ncku-nasa  compression           off                    default
ncku-nasa  refcompressratio      1.00x                  -
$ sudo zfs set compression=lz4 ncku-nasa
$ zfs get all ncku-nasa | grep compress
ncku-nasa  compressratio         1.00x                  -
ncku-nasa  compression           lz4                    local
ncku-nasa  refcompressratio      1.00x                  -
```