---
title: '无状态服务多可用区多规格部署陷阱'
date: 2023-11-08 00:00:00
tags: ['stateless', 'multi-zone', 'multi-flavor', 'lessons-learned']
category: devops
---

# 20231108-stateless-multi-zone-multi-flavor-pitfall

# multi-zone pitfall

作为云计算厂商, 我们最喜欢的用户是能够在一个地域开启多个可用区. 

但实际在实践中, 发现这个看似简单的要求/最佳实践, 有点儿, 真正down to the earth之后, 发现很多场景用户是不好做/压根没办法做到跨AZ的. 

> Unlike Google, who can’t be bothered to descend the ivory tower to visit real customers, Grab’s mantra is: “Go to the ground”.
> 

个人总结有几个原因: 

## 存储绑定

存储基本都是绑定可用区的. 例如 HDFS, ZK, 云盘, RDS等. (当然RDS可以选择其他可用区来做备库)

例如用户在zoneA下创建一个HDFS集群, 或者ZK集群. 

这样应用在访问存储时希望能同可用区就近访问. 

当然你可以说: 

1. 用户应用为啥不能跨机房访问? 
2. 或者HDFS/ZK集群为啥不能跨机房部署? 

## 应用为啥不能跨机房访问?

## 集群为啥不能跨机房部署?

尤其是在ZK等分布式集群下, 节点点通信量是巨大的. 对节点之间的延时要求是很高的. 

如果机房间网络不通, 会直接导致脑裂. 

要注意: 机房间流量是有带宽限制的, 甚至可能会打满! 甚至跨机房流量会收费!

## zone/vsw数量膨胀

以 阿里云上海地域为例, 总共有12个可用区, 但ECI最多支持10个vsw

![Untitled](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/source/assets/2023-11-08-stateless-multi-zone-multi-flavor-pitfall/Untitled.png)

- 反观AWS, 最多6个AZ, 永续

# multi-flavor pitfall

[存储在多可用区部署时有哪些推荐配置\_容器服务 Kubernetes 版 ACK-阿里云帮助中心](https://help.aliyun.com/zh/ack/ack-managed-and-ack-dedicated/user-guide/recommended-storage-settings-for-cross-zone-deployment#task-2269269)

## 指令集绑定/优先

1. 指令集绑定: 视频转码AVX512指令集

## 性能无法对齐

1. 无法保证节点性能大致对齐. 尤其是在运行Job时, 会有短板效应.