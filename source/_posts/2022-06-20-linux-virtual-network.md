---
layout: post
title: Linux虚拟网络与网络设备
subtitle: Linux虚拟网络与网络设备
tags: [linux, network, cloud-computing, iaas, nat, pat]
---

# 背景
使用VMWare或者KVM设置VM网络时, 通常会有几种网络模式: 
- 网桥模式(Bridge模式)
- NAT模式

之前一直比较迷惑, 不是特别清楚区别, 最近研究终于搞懂了, 总结如下: 

# 网桥模式
## 本质
1. 本质上把Linux网桥看做一个二层的交换机.
2. VM连接到该网桥, 获取访问外网能力.

![linux-bridge](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202206202216042.png)

## 特点
因此
1. VM的IP与Host的IP是在同一个网段上的
2. VM在网络中的位置与Host是并列的

![VM](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202206202214199.png)
![Host](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202206202215507.png)
![VM Route Table](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202206202216645.png)

## 实操



## 常用命令

- 网桥操作
```shell
brctl show
```

- 安装工具
```shell
yum install net-tools -y
yum install bridge-utils -y
```

- 配置


# NAT模式
## 本质
1. 本质上是Host看做一个NAT设备
2. VM连接到该NAT设备上, 获取访问外网能力.

![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202206202212554.png)

## 特点
因此
1. VM的IP与Host的IP不在同一个网段
2. VM在网络中的位置是从属于Host

![VM在192.168.230.0/24网段](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202206202159960.png)
![Host在192.168.3.0/24网段](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202206202200319.png)

## 实现方式
// TODO:



## 其他注意事项
这里 VM的网桥模式 与 docker的网桥/Bridge模式 是有很大区别的。
- VM 网桥/桥接模式: VM与HOST在同一个网段。
- Docker Bridge模式： Docker容器与HOST不在同一个网段。
> Bridge 是 docker 默认的网络模式。
> 原理跟 vmware 的 NAT 模式相同。
> 安装 docker 时，会给宿主机创建一个 docker0 网卡，该网卡会与一个虚拟交换机相连，
> 当容器以 Bridge 模式创建启动时，会给容器创建一个虚拟网卡，该网卡分配的 IP 与宿主机的 docker0 所在同一个局域网内 (一般是 172.16.0.0)。
> 然后过程就和 vmware 的 NAT 模式完全相同。

# Refs
- [ Vmware和Docker的网络模式讲解](https://learnku.com/docs/go-micro-build/1.0/explain-the-network-mode-of-vmware-and-docker/8879)