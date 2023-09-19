---
title: 玩一玩Ventoy
date: 2023-09-19 23:16:56
tags: [linux, tech-for-fun, bootloader]
---

核心代码: [Ventoy/INSTALL/tool/VentoyWorker.sh at master · ventoy/Ventoy](https://github.com/ventoy/Ventoy/blob/master/INSTALL/tool/VentoyWorker.sh#L314C5-L314C25)

如何查看当前磁盘分区类型(MBR or GPT)
- msdos: 代表MBR
- gpt: 代表GPT
```
[root@iZbp1d3afbt50ybdanjugiZ ~]# parted -l
Model: Virtio Block Device (virtblk)
Disk /dev/vda: 42.9GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
```

查看某个磁盘是否已经被挂载: 
```shell
grep "$DISK" /proc/mounts
```

centos下分区工具: (具体有啥区别哩?)
- `parted`命令本质是一个分区工具, 全名`GNU parted`
- `fdisk`命令也是一个分区工具

查看裸设备/磁盘大小(返回sector的数量, 每个sector通常是 512B, 因此如下值换算成GB为 83886080/1024/1024/2=40GB)
```shell
[root@iZbp1d3afbt50ybdanjugiZ ~]# cat /sys/block/vda/size
83886080
```

dd命令的使用: TODO:
```shell

```



shell 中 if 判断骚操作: 
```shell
-e file exists
-f file is a regular file (not a directory or device file)
-s file is not zero size
-d file is a directory
-b file is a block device
-c file is a character device
-p file is a pipe
-L file is a symbolic link
-S file is a socket
-t file (descriptor) is associated with a terminal device
-r file has read permission (for the user running the test)
-w file has write permission (for the user running the test)
-x file has execute permission (for the user running the test)
```