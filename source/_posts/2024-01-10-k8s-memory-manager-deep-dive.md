---
title: k8s memory manager deep dive
date: 2024-01-10 00:00:00
tags: [k8s, memory-manager, deep-dive]
---

# k8s-memory-manager-deep-dive

- 只能针对GU, 不保证burstable的pod是否跨numa访问.
- kube-scheduler无法感知numa: The default Kubernetes scheduler is not aware of the node's NUMA topology, and it can be a reason for many admission errors during the pod start.

# 疑问汇总

1. mm怎么跟cadvisor交互, 来获取内存信息? 详细代码实现. ❌

1. 如下图, 如果到操作系统来给pod发送OOM, 如何保证 “Low-priority containers are targeted before the high-priority containers”? 在进程优先级上有设定? 底层是怎么实现的? ✅

答案: 参见 [Node-pressure Eviction | Kubernetes](https://kubernetes.io/docs/concepts/scheduling-eviction/node-pressure-eviction/#node-out-of-memory-behavior) 底层是给不同QoS的Pod设定不同的oom_score_adj, 然后通过Linux的 oom_killer 识别&执行.

![Untitled](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/source/assets/2024-01-10-k8s-memory-manager-deep-dive/Untitled.png)

1. 为啥下边的场景, Pod2创建会失败? ❌

![Untitled](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/source/assets/2024-01-10-k8s-memory-manager-deep-dive/Untitled-1.png)

1. 怎么跟cpumanager进行交互? 

# Refs

- [Kubernetes Memory Manager moves to beta | Kubernetes]([https://kubernetes.io/blog/2021/08/11/kubernetes-1-22-feature-memory-manager-moves-to-beta/#How-does-it-work?)](https://kubernetes.io/blog/2021/08/11/kubernetes-1-22-feature-memory-manager-moves-to-beta/#How-does-it-work?))
- [enhancements/keps/sig-node/1769-memory-manager/mm\_tm\_diagram.png at master · kubernetes/enhancements](https://github.com/kubernetes/enhancements/tree/master/keps/sig-node/1769-memory-manager)