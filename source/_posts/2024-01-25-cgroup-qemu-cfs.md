---
title: 'Cgroup、QEMU 与 CFS 调度'
date: 2024-01-25 00:00:00
tags: ['linux', 'cgroups', 'qemu', 'cfs', 'virtualization']
---

# 20240125-cgroup-qemu-cfs

针对如何实现CPU的绑核逻辑, 一直有很多疑问. 

今天跟GPT聊了一会儿, 大致勾画出如下: 

本质还是cfs-scheduler的策略, 但通过的方式不同工具/方式透出. 基本明了了.

TODO: 底层还是要研究下cfs-scheduler具体的工作机制, 才能彻底透彻.

![Untitled](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/source/assets/2024-01-25-cgroup-qemu-cfs/Untitled.png)