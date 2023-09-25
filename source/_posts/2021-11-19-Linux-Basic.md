---
layout: post
title: 鸟哥的Linux私房菜笔记
tags: [2021, books, GFS, Google, linux]
lang: zh
---

# Linux 中各种文件类型
经常在`ls -ltrh`的时候, 出现的文件类型不太认识/熟悉, 这里整理下, 详细参见: 
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202309252337411.png)

典型样例: 
- `-`: 普通文件
- `d`: 目录
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202309252339985.png)

- `b`: 块设备, 都是在`/dev/`目录下
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202309252342096.png)

- `c`: 字符设备, 典型的例如tty, 如 urandom, random
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202309252352764.png)

- `s`: socketfile, 用于进程间socket通信. 详细使用方式参见: [What are socket files?](https://askubuntu.com/questions/372725/what-are-socket-files). 另外还可以了解下vsock, 作为host与guestOS通信的常用方案.
  ![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202309252349265.png)
