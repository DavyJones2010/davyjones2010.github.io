---
title: Linux磁盘与分区的常用命令
categories: Linux常用命令
tags: [linux, disk, file-system, os]
---

# 磁盘

## 查看当前系统块设备列表
### fdisk -l
如下, `fdisk -l` 命令, 可以显示出:
1. 当前系统存在2块块设备, 分别是 `/dev/sda` 与 `/dev/sdb`, 大小分别为 21.5GB 与 31.5GB
2. `/dev/sda` 磁盘, 划为两个分区, 分别是 `/dev/sda1` `/dev/sda2`, 其中 `/dev/sda2` 为LVM文件系统类型
3. `/dev/sdb` 磁盘(个人知道是U盘), 划分为1个分区 `/dev/sdb1` , 文件系统类型是 `W95 FAT32`
4. TODO: 疑问1  `/dev/mapper/centos-root` `/dev/mapper/centos-swap` 分别都是啥? 个人已知是在 `/dev/sda2` 下继续分区出来的, 如下 `lsblk` 命令指示. 但为啥这里没有
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

```shell
[root@localhost ~]# fdisk /dev/sdb
命令(输入 m 获取帮助)：d
分区号 (1-3，默认 3)：
分区 3 已删除

命令(输入 m 获取帮助)：w
The partition table has been altered!

Calling ioctl() to re-read partition table.
正在同步磁盘。
[root@localhost ~]# lsblk -p
NAME                        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
/dev/sda                      8:0    0   20G  0 disk
├─/dev/sda1                   8:1    0    1G  0 part /boot
└─/dev/sda2                   8:2    0   19G  0 part
  ├─/dev/mapper/centos-root 253:0    0   17G  0 lvm  /
  └─/dev/mapper/centos-swap 253:1    0    2G  0 lvm  [SWAP]
/dev/sdb                      8:16   1 29.3G  0 disk
├─/dev/sdb1                   8:17   1   10G  0 part
└─/dev/sdb2                   8:18   1    4G  0 part
/dev/sr0                     11:0    1  973M  0 rom
```

### 挂载分区 mount

```shell
[root@localhost ~]# mount /dev/sdb1 /root/movie
[root@localhost ~]# mount /dev/sdb2 /root/book
[root@localhost ~]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
/dev/sdb1                 10G   33M   10G    1% /root/movie
/dev/sdb2                4.0G   33M  4.0G    1% /root/book
[root@localhost ~]# lsblk -p
NAME                        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
/dev/sdb                      8:16   1 29.3G  0 disk
├─/dev/sdb1                   8:17   1   10G  0 part /root/movie
└─/dev/sdb2                   8:18   1    4G  0 part /root/book
```


### 卸载分区 umount
> 本质上与Windows上弹出U盘(设备)是一样道理, 底层就是将该设备从文件树中删除. 
> 因此如果操作系统正在访问/占用该设备, 会卸载/弹出失败.

卸载方式: 
- 通过挂载点卸载
- 通过设备名卸载

```shell
[root@localhost ~]# umount -v /root/movie #通过挂载点卸载
umount: /root/movie (/dev/sdb1) 已卸载
[root@localhost ~]# umount -v /dev/sdb2 #通过设备名卸载
umount: /root/book (/dev/sdb2) 已卸载
[root@localhost ~]# lsblk -p
NAME                        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
/dev/sdb                      8:16   1 29.3G  0 disk
├─/dev/sdb1                   8:17   1   10G  0 part
└─/dev/sdb2                   8:18   1    4G  0 part
[root@localhost ~]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
devtmpfs                 475M     0  475M    0% /dev
/dev/mapper/centos-root   17G  1.4G   16G    9% /
/dev/sda1               1014M  138M  877M   14% /boot
```

卸载之后, 重新挂载:
1. 不要求新的挂载点与之前一直. 例如之前`/dev/sdb2`挂载到`/root/book`目录下, 卸载之后重新挂载, `/dev/sdb2`可以挂载到 `/root/movie` 目录下
2. 分区里边的内容仍然是保留的. 例如 `/dev/sdb2` 分区卸载重新挂载到其他目录, 该分区下的内容仍然是保留的, 能从新的目录下访问到该内容.


# 文件系统
## 为磁盘格式化文件系统
### 不分区, 直接格式化整个设备
- 如下, 直接把U盘格式化为xfs文件系统, 之后使用 `blkid` 查看设备的文件系统类型, 使用`lsblk`只能看到该设备是裸设备: 
```shell
[root@localhost ~]# mkfs.xfs -f /dev/sdb
[root@localhost ~]# blkid /dev/sdb
/dev/sdb: UUID="b3841552-6b45-4e76-980a-911a107fbade" TYPE="xfs"
[root@localhost ~]# lsblk -p
NAME                        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
/dev/sda                      8:0    0   20G  0 disk
├─/dev/sda1                   8:1    0    1G  0 part /boot
└─/dev/sda2                   8:2    0   19G  0 part
  ├─/dev/mapper/centos-root 253:0    0   17G  0 lvm  /
  └─/dev/mapper/centos-swap 253:1    0    2G  0 lvm  [SWAP]
/dev/sdb                      8:16   1 29.3G  0 disk
/dev/sr0                     11:0    1  973M  0 rom
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

## 扩容/缩容分区

### 普通类型分区
可以无损扩容缩容么? 

### LVM类型
参照: [centos7扩容根目录](https://zhuanlan.zhihu.com/p/450057653)
自己使用的虚拟机是20G硬盘. 想要扩容下根目录, 扩容到32G, 即如下的 `MOUNTPOINT /`: 

```shell
[root@localhost ~]# lsblk -p
NAME                        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
/dev/sda                      8:0    0   20G  0 disk
├─/dev/sda1                   8:1    0    1G  0 part /boot
└─/dev/sda2                   8:2    0   19G  0 part
  ├─/dev/mapper/centos-root 253:0    0   17G  0 lvm  /
  └─/dev/mapper/centos-swap 253:1    0    2G  0 lvm  [SWAP]
```

#### VMWare Fusion中修改磁盘大小
操作方式比较简单, 直接在控制面板修改文件大小即可.
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202211222348543.png)

- 该步骤结束后, 效果:
```shell
[root@localhost ~]# lsblk -p
NAME                        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
/dev/sda                      8:0    0   32G  0 disk
├─/dev/sda1                   8:1    0    1G  0 part /boot
└─/dev/sda2                   8:2    0   19G  0 part
  ├─/dev/mapper/centos-root 253:0    0   17G  0 lvm  /
  └─/dev/mapper/centos-swap 253:1    0    2G  0 lvm  [SWAP]
```

#### 将新增的空间新建一个LVM类型分区(fdisk)

```shell
[root@localhost ~]# fdisk /dev/sda
命令(输入 m 获取帮助)：n
Partition type:
   p   primary (2 primary, 0 extended, 2 free)
   e   extended
Select (default p):
Using default response p
分区号 (3,4，默认 3)：
起始 扇区 (41943040-67108863，默认为 41943040)：
将使用默认值 41943040
Last 扇区, +扇区 or +size{K,M,G} (41943040-67108863，默认为 67108863)：
将使用默认值 67108863
分区 3 已设置为 Linux 类型，大小设为 12 GiB

命令(输入 m 获取帮助)：p

磁盘 /dev/sda：34.4 GB, 34359738368 字节，67108864 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000b7bb1

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    41943039    19921920   8e  Linux LVM
/dev/sda3        41943040    67108863    12582912   83  Linux

命令(输入 m 获取帮助)：t
分区号 (1-3，默认 3)：3
Hex 代码(输入 L 列出所有代码)：L
Hex 代码(输入 L 列出所有代码)：8e
已将分区“Linux”的类型更改为“Linux LVM”

命令(输入 m 获取帮助)：w
The partition table has been altered!
正在同步磁盘。
[root@localhost ~]# partprobe
Warning: 无法以读写方式打开 /dev/sr0 (只读文件系统)。/dev/sr0 已按照只读方式打开。
```

- 该步骤结束后, 效果如下, 看到多了一块分区`/dev/sda3`, TODO: 为啥这里看到的类型还是`part`??

```shell
[root@localhost ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   32G  0 disk
├─sda1            8:1    0    1G  0 part /boot
├─sda2            8:2    0   19G  0 part
│ ├─centos-root 253:0    0   17G  0 lvm  /
│ └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
└─sda3            8:3    0   12G  0 part
sr0              11:0    1  973M  0 rom
```
#### 将该LVM分区添加到pv group下, 并扩展原来的Logical volume, 实现逻辑卷扩容
1. 将该LVM分区添加到pv group下, `vgextend`:
```shell
[root@localhost ~]# lvm
lvm> pvcreate /dev/sda3
  Physical volume "/dev/sda3" successfully created.
lvm> pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda2
  VG Name               centos
  PV Size               <19.00 GiB / not usable 3.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              4863
  Free PE               0
  Allocated PE          4863
  PV UUID               xb72qt-XG0d-7jNA-VOEC-z3jt-y7bz-mtye35

  "/dev/sda3" is a new physical volume of "12.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sda3
  VG Name
  PV Size               12.00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               f71O6u-tlAf-F2Cr-rTyy-zGcq-YqE2-PvO5Ri

lvm> vgdisplay 
  --- Volume group ---
  VG Name               centos
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <19.00 GiB
  PE Size               4.00 MiB
  Total PE              4863
  Alloc PE / Size       4863 / <19.00 GiB
  Free  PE / Size       0 / 0
  VG UUID               seZW4N-tjt9-C5Qg-UFYp-z0uY-TZgq-2F94Hd

lvm> vgextend centos /dev/sda3 # 将该LVM分区添加到pv group下
  Volume group "centos" successfully extended
```

2. 扩展原始的Logical Volume, `lvextend`:
```shell
lvm> lvdisplay
  --- Logical volume ---
  LV Path                /dev/centos/swap
  LV Name                swap
  VG Name                centos
  LV UUID                wltk2Z-WIJW-vB6N-en6c-z5kg-U1IV-MWF7iX
  LV Write Access        read/write
  LV Creation host, time localhost, 2022-10-22 07:51:02 -0400
  LV Status              available
  # open                 2
  LV Size                2.00 GiB
  Current LE             512
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:1
lvm> lvextend -l +100%FREE /dev/centos/root
  Size of logical volume centos/root changed from <17.00 GiB (4351 extents) to 28.99 GiB (7422 extents).
  Logical volume centos/root successfully resized.
```

3. 该步骤结束后, 效果如下: 

```shell
[root@localhost ~]# lsblk -p
NAME                        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
/dev/sda                      8:0    0   32G  0 disk
├─/dev/sda1                   8:1    0    1G  0 part /boot
├─/dev/sda2                   8:2    0   19G  0 part
│ ├─/dev/mapper/centos-root 253:0    0   29G  0 lvm  /
│ └─/dev/mapper/centos-swap 253:1    0    2G  0 lvm  [SWAP]
└─/dev/sda3                   8:3    0   12G  0 part
  └─/dev/mapper/centos-root 253:0    0   29G  0 lvm  /
```

#### 同步到文件系统, 实现根目录扩容
注意, 这里的 `/dev/centos/root` 是`Logical volume`的`LV Path`, 即使用 `lvdisplay` 之后显示的`LV Path` 
而不是 `/dev/mapper/centos-root` 这个路径.

```shell
[root@localhost ~]# xfs_growfs /dev/centos/root
meta-data=/dev/mapper/centos-root isize=512    agcount=4, agsize=1113856 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=4455424, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 4455424 to 7600128
```

- 该步骤结束后, 效果如下: 
```
[root@localhost ~]# lsblk -p
NAME                        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
/dev/sda                      8:0    0   32G  0 disk
├─/dev/sda1                   8:1    0    1G  0 part /boot
├─/dev/sda2                   8:2    0   19G  0 part
│ ├─/dev/mapper/centos-root 253:0    0   29G  0 lvm  /
│ └─/dev/mapper/centos-swap 253:1    0    2G  0 lvm  [SWAP]
└─/dev/sda3                   8:3    0   12G  0 part
  └─/dev/mapper/centos-root 253:0    0   29G  0 lvm  /

[root@localhost ~]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root   29G  1.4G   28G    5% /
/dev/sda1               1014M  138M  877M   14% /boot
```


# 总结&思考
深刻体会到了磁盘与文件系统的关系, 如知乎某位大神所言, **根本没有关系!** 
- 一个磁盘(块设备)可以分为多个分区, 每个分区都可以分别格式化为不同的文件系统, 然后分别挂载在不同的目录下.
- 裸设备也可以不分区, 直接格式化为文件系统.
- 文件系统里一个文件可以作为(虚拟化为)一个块设备使用 
- 多个磁盘/多个分区也可以共同组成一个设备, 该设备使用一个分区/文件系统类型.

TODO: 
1. 研究下LVM吧, 不求源码级别的深入, 但原理与实践需要跟上.
2. 分区与文件系统格式方案的最佳实践是啥?

# Refs
- [如何在 Linux 中查看已挂载的文件系统类型](https://linux.cn/article-10194-1.html)
- [centos7扩容根目录（/dev/mapper/centos-root）](https://zhuanlan.zhihu.com/p/450057653)
