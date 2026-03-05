---
title: 'Node VPA 使用陷阱'
date: 2024-01-05 00:00:00
tags: ['kubernetes', 'vpa', 'lessons-learned']
category: container-orchestration
---

# 20240105-node-vpa-pitfall

节点垂直伸缩的概念, 看起来很美好, 都是大家鼓吹的. 但实际实现起来, 落地难度巨大. 本质上来说, 是需要应用层的适配. 

以下是落地不好导致的问题: 

- [K8s 节点 CPU 升级，导致 kubelet 无法启动故障一例](https://mp.weixin.qq.com/s/i0A2thWt1Ut7deDg-fmaog)
- 以Netty应用为例, 应用启动时都会getAvailableCpus; 如果直接垂直热伸缩, 会导致该信息不准确, 保不准会.

真正有落地的: 

- aws 2023 re:invent 上的 与数据库深度合作的. 相比应用层做了非常大的改造.
-