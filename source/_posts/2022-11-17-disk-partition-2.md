---
title: Linux磁盘与分区的常用命令
categories: 常用命令
tags: [linux, disk, os]
---

# 磁盘

## 查看当前系统块设备列表
### fdisk -l
如下, `fdisk -l` 命令, 可以显示出:
1. 当前系统存在2块块设备, 分别是 /dev/sda 与 /dev/sdb, 大小分别为 21.5GB 与 31.5GB
2. /dev/sda 磁盘, 划为两个分区, 分别是 /dev/sda1 /dev/sda2, 其中 /dev/sda2 为LVM文件系统类型
3. /dev/sdb 磁盘(个人知道是U盘), 划分为1个分区 /dev/sdb1 , 文件系统类型是 `W95 FAT32`
4. TODO: 疑问1  `/dev/mapper/centos-root` `/dev/mapper/centos-swap` 分别都是啥? 个人已知是在 /dev/sda2 下继续分区出来的, 如下 `lsblk` 命令指示. 但为啥这里没有
5. TODO: 疑问2 `Linux LVM` 具体是啥文件系统类型? 

> SATA device names follow the pattern /dev/sd[a-z]
> while NVMe device names have the following pattern /dev/nvme[1-9]n[1-9]

```shell
[root@localhost ~]# fdisk -l
Disk /dev/sda: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000b7bb1

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    41943039    19921920   8e  Linux LVM

Disk /dev/mapper/centos-root: 18.2 GB, 18249416704 bytes, 35643392 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/mapper/centos-swap: 2147 MB, 2147483648 bytes, 4194304 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sdb: 31.5 GB, 31457280000 bytes, 61440000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x1c9aed9e

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1   *          64    61439999    30719968    b  W95 FAT32
```

### lsblk
- 如下, 类型为"disk"的有两个, sda sdb
- sda 被划分为了2个分区, sda1 与 sda2
- sdb 被划分为了1个分区, sdb1

```shell
[root@localhost ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   20G  0 disk
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0   19G  0 part
  ├─centos-root 253:0    0   17G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
sdb               8:16   1 29.3G  0 disk
└─sdb1            8:17   1 29.3G  0 part
```


## 查看设备的文件系统类型

### 不挂载查看: fdisk -l
如上, 

### 不挂载查看: blkid

```shell
[root@localhost ~]# blkid /dev/sda2
/dev/sda2: UUID="xb72qt-XG0d-7jNA-VOEC-z3jt-y7bz-mtye35" TYPE="LVM2_member"
[root@localhost ~]# blkid /dev/mapper/centos-root
/dev/mapper/centos-root: UUID="a30443bd-8ab9-4869-917d-fe51b512b993" TYPE="xfs"
[root@localhost ~]# blkid /dev/sdb1
/dev/sdb1: LABEL="DISK_IMG" UUID="1C9A-ED9E" TYPE="vfat"
```

### 挂载后查看: df -hT
```shell
[root@localhost ~]# df -hT
Filesystem              Type      Size  Used Avail Use% Mounted on
devtmpfs                devtmpfs  475M     0  475M   0% /dev
tmpfs                   tmpfs     487M     0  487M   0% /dev/shm
tmpfs                   tmpfs     487M  7.7M  479M   2% /run
tmpfs                   tmpfs     487M     0  487M   0% /sys/fs/cgroup
/dev/mapper/centos-root xfs        17G  1.4G   16G   9% /
/dev/sda1               xfs      1014M  138M  877M  14% /boot
tmpfs                   tmpfs      98M     0   98M   0% /run/user/0
```

### 查看所有已挂载的分区与挂载点文件系统等信息 findmnt
如下: 
1. `/dev/mapper/centos-root` 分区挂载在 `/` 目录下, 文件系统类型是`xfs`
2. `/dev/sda1` 分区挂载在 `/boot/` 目录下, 文件系统类型是`xfs`

```shell
[root@localhost ~]# findmnt
TARGET                                SOURCE                  FSTYPE     OPTIONS
/                                     /dev/mapper/centos-root xfs        rw,relatime,seclabel,attr2,inode64,noquota
├─/sys                                sysfs                   sysfs      rw,nosuid,nodev,noexec,relatime,seclabel
│ ├─/sys/kernel/security              securityfs              securityfs rw,nosuid,nodev,noexec,relatime
│ ├─/sys/fs/cgroup                    tmpfs                   tmpfs      ro,nosuid,nodev,noexec,seclabel,mode=755
│ │ ├─/sys/fs/cgroup/systemd          cgroup                  cgroup     rw,nosuid,nodev,noexec,relatime,seclabel,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd
│ │ ├─/sys/fs/cgroup/memory           cgroup                  cgroup     rw,nosuid,nodev,noexec,relatime,seclabel,memory
│ │ ├─/sys/fs/cgroup/blkio            cgroup                  cgroup     rw,nosuid,nodev,noexec,relatime,seclabel,blkio
│ │ ├─/sys/fs/cgroup/cpuset           cgroup                  cgroup     rw,nosuid,nodev,noexec,relatime,seclabel,cpuset
│ │ ├─/sys/fs/cgroup/perf_event       cgroup                  cgroup     rw,nosuid,nodev,noexec,relatime,seclabel,perf_event
│ │ ├─/sys/fs/cgroup/net_cls,net_prio cgroup                  cgroup     rw,nosuid,nodev,noexec,relatime,seclabel,net_prio,net_cls
│ │ ├─/sys/fs/cgroup/devices          cgroup                  cgroup     rw,nosuid,nodev,noexec,relatime,seclabel,devices
│ │ ├─/sys/fs/cgroup/cpu,cpuacct      cgroup                  cgroup     rw,nosuid,nodev,noexec,relatime,seclabel,cpuacct,cpu
│ │ ├─/sys/fs/cgroup/hugetlb          cgroup                  cgroup     rw,nosuid,nodev,noexec,relatime,seclabel,hugetlb
│ │ ├─/sys/fs/cgroup/freezer          cgroup                  cgroup     rw,nosuid,nodev,noexec,relatime,seclabel,freezer
│ │ └─/sys/fs/cgroup/pids             cgroup                  cgroup     rw,nosuid,nodev,noexec,relatime,seclabel,pids
│ ├─/sys/fs/pstore                    pstore                  pstore     rw,nosuid,nodev,noexec,relatime
│ ├─/sys/kernel/config                configfs                configfs   rw,relatime
│ ├─/sys/fs/selinux                   selinuxfs               selinuxfs  rw,relatime
│ └─/sys/kernel/debug                 debugfs                 debugfs    rw,relatime
├─/proc                               proc                    proc       rw,nosuid,nodev,noexec,relatime
│ └─/proc/sys/fs/binfmt_misc          systemd-1               autofs     rw,relatime,fd=26,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=13709
├─/dev                                devtmpfs                devtmpfs   rw,nosuid,seclabel,size=485840k,nr_inodes=121460,mode=755
│ ├─/dev/shm                          tmpfs                   tmpfs      rw,nosuid,nodev,seclabel
│ ├─/dev/pts                          devpts                  devpts     rw,nosuid,noexec,relatime,seclabel,gid=5,mode=620,ptmxmode=000
│ ├─/dev/hugepages                    hugetlbfs               hugetlbfs  rw,relatime,seclabel
│ └─/dev/mqueue                       mqueue                  mqueue     rw,relatime,seclabel
├─/run                                tmpfs                   tmpfs      rw,nosuid,nodev,seclabel,mode=755
│ └─/run/user/0                       tmpfs                   tmpfs      rw,nosuid,nodev,relatime,seclabel,size=99568k,mode=700
└─/boot                               /dev/sda1               xfs        rw,relatime,seclabel,attr2,inode64,noquota
```

## 分区

### 划分分区: fdisk n
```shell
[root@localhost ~]# fdisk /dev/sdb
Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-61439999, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-61439999, default 61439999):
Using default value 61439999
Partition 1 of type Linux and of size 29.3 GiB is set

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

### 删除分区 fdisk d


### 挂载分区


### 卸载分区


## 文件系统

### 不分区, 直接格式化整个设备
- 如下, 直接把U盘格式化为xfs文件系统, 之后使用 `blkid` 查看设备的文件系统类型: 
```shell
[root@localhost ~]# mkfs.xfs -f /dev/sdb
[root@localhost ~]# blkid /dev/sdb
/dev/sdb: UUID="b3841552-6b45-4e76-980a-911a107fbade" TYPE="xfs"
```

### 分区, 格式化分区
- 如下, `/dev/sdb`U盘分成1个分区, 但不对分区进行格式化, blkid 查看到的信息为空
```shell
[root@localhost ~]# fdisk -l
   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048    61439999    30718976   83  Linux
[root@localhost ~]# blkid /dev/sdb
/dev/sdb: PTTYPE="dos"
[root@localhost ~]# blkid /dev/sdb1
```

- 将分区格式化之后, blkid 查看到具体分区文件系统类型信息:
```shell
[root@localhost ~]# mkfs.xfs -f /dev/sdb1
[root@localhost ~]# blkid /dev/sdb1
/dev/sdb1: UUID="6c3bcc54-b925-4e3c-a101-6edf5ca28268" TYPE="xfs"
```

# Refs
- [如何在 Linux 中查看已挂载的文件系统类型](https://linux.cn/article-10194-1.html)











