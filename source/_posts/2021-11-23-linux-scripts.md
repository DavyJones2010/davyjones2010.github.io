---
layout: post 
title: 常用的Linux命令
subtitle: 常用的Linux命令
tags: [code-snippets, linux]
---

# 背景
# 常用Linux命令

## 文本处理
* 文件按固定行数切分

```shell
split -l 20 cn-hangzhou.csv cn-hangzhou-
```

问题是: 生成的文件都是没有后缀名的, 如下可以使用xargs, 统一增加后缀名, 如下：
- mac:

```shell
split -l 20 cn-hangzhou.csv cn-hangzhou- && ls | grep cn-hangzhou- | xargs -n1 -I{} mv {} {}.txt
```

- linux:

```shell
split -l 20 cn-hangzhou.csv cn-hangzhou- && ls | grep cn-hangzhou- | xargs -n1 -i{} mv {} {}.txt
```

* 文件按照某列进行排序

```shell
sort -n -k 1 -t , sample.csv
```

* 文件查看某列去重数据

```shell  
awk -F',' '{print $1}' sample.csv | sort -n | uniq
```

* 计算文件中相同的行数量, 并按照数量倒序排列:

todo: 

* grep正则表达式
  grep正则表达式元字符集（基本集）
  ^锚定行的开始 如：'^grep'匹配所有以grep开头的行。
  $锚定行的结束 如：'grep$'匹配所有以grep结尾的行。

* JSON处理

```shell
jq
```

* 创建空的大文件(全被0占据的文件, 而非打洞)

```shell
fallocate -l 50M /data/web/www/html/b.zip
## -l: 指定文件的长度
```

## 目录管理

* 查看目录大小

```shell
sudo du -h --max-depth=1 /home/admin/
```

* 查看目录下文件大小

```shell
du -sh * | sort -nr | head
du -sh * | sort -nr | fgrep "G" | head
```

## 系统管理
### top
#### 进程排序
- 以 CPU 占用率大小的顺序排列进程列表 
  - top进入之后， 按P 
  - 或者 `top -o %CPU`
- 以内存占用率大小的顺序排列进程列表
  - top进入之后，按M 
  - 或者 `top -o %MEM`
- 以总共消耗的CPU时间片排序
  - top进入之后，按T 
  - `top -o +TIME`

#### 线程排序
- 查看某个进程下各个线程CPU占用情况，倒序排列
  - `top -Hp ${pid} -o %CPU`
- 查看某个进程下各个线程内存占用情况，倒序排列
  - `top -Hp ${pid} -o %MEM`
- 查看某个进程下各个线程CPU时间片占用情况，倒序排列
  - `top -Hp ${pid} -o +TIME`

### centOS yum源路径

```shell
/etc/yum.repos.d/
```

## 进程&网络管理

* 查看端口被哪个进程占用了

```shell
lsof -i:${port}
netstat -natp | fgrep ${port}
```

* 根据进程号, 查看进程占用了哪些tcp端口

```shell
netstat -natp | fgrep ${pid}
sudo lsof -i -P | fgrep ${pid}
```

* 查看进程信息:

```shell
ps aux | fgrep ${pid}
```

## 用户管理
* 锁定用户:

```shell
sudo passwd -l username
```

* 查看用户是否被锁定:

```shell
sudo passwd -S username
sudo passwd --status interlive
```

- 或者看下密码前边是否有 "!!" 标记, 如果有, 则证明被标记了

```shell
sudo fgrep "interlive" /etc/shadow
```

## 绑核
- 查看进程绑核情况

```shell
taskset -p $PID
taskset -cp $PID
```
```shell
davywalker@davywalker-ThinkPad-X1-Carbon-4th:~/Downloads/Clash-Linux$ taskset -p 20697
pid 20697's current affinity mask: f
davywalker@davywalker-ThinkPad-X1-Carbon-4th:~/Downloads/Clash-Linux$ taskset -cp 20697
pid 20697's current affinity list: 0-3
```

- 执行进程绑核操作

```shell
taskset -p COREMASK PID
```


# 磁盘操作
## 查看块设备(包括已分区与未分区的)
```shell
root@OpenWrt:~# lsblk
NAME          MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
nvme0n1       259:0    0 465.8G  0 disk
├─nvme0n1p1   259:1    0    16M  0 part /boot
│                                       /boot
├─nvme0n1p2   259:2    0   500M  0 part /rom
├─nvme0n1p3   259:3    0   128G  0 part /overlay
├─nvme0n1p4   259:4    0   128G  0 part /mnt/nvme0n1p4
├─nvme0n1p5   259:5    0   128G  0 part /mnt/nvme0n1p5
├─nvme0n1p6   259:6    0  81.3G  0 part /mnt/nvme0n1p6
└─nvme0n1p128 259:7    0   239K  0 part
```

## 查看分区的文件系统类型
```shell
root@OpenWrt:~# parted -l
Model: WD Blue SN570 500GB SSD (nvme)
Disk /dev/nvme0n1: 500GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name  Flags
128     17.4kB  262kB   245kB                      bios_grub
 1      262kB   17.0MB  16.8MB  fat16              legacy_boot
 2      17.0MB  541MB   524MB
 3      542MB   138GB   137GB   ext4
 4      138GB   275GB   137GB   ext4
 5      275GB   413GB   137GB   ext4
 6      413GB   500GB   87.2GB  ext4
```

## 为块设备分区


## 为分区格式化文件系统类型

# 其他
## 查看centos版本
```shell
[root@localhost ~]# rpm --query centos-release
centos-release-7-9.2009.0.el7.centos.x86_64
```

## 查看Linux内核版本
```shell
[root@localhost ~]# uname -a
Linux localhost.localdomain 3.10.0-1160.el7.x86_64 #1 SMP Mon Oct 19 16:18:59 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```