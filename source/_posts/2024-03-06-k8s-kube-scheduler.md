---
title: k8s kube scheduler
date: 2024-03-06 00:00:00
tags: [k8s, kube-scheduler, scheduler, deep-dive]
---

# k8s-kube-scheduler

# 代码结构梳理

![Untitled](k8s-kube-scheduler/Untitled.png)

# 调度流程梳理-正常

![Untitled](k8s-kube-scheduler/Untitled%201.png)

- 关键点在上图标红的部分: 实际Bind成功之后, 只watch到了1个PodUpdate事件, 但由于 FilteringResourceEventHandler 的特殊处理, 如下, 会把事件修正为
    - 1个OnAdd事件
    - 1个OnDelete事件

![Untitled](k8s-kube-scheduler/Untitled%202.png)

- 详细trace参见如下: 发现在kube-sched日志中, bind成功后, 触发了 “Delete event” + “Add event”

![Untitled](k8s-kube-scheduler/Untitled%203.png)

# SchedulingQueue处理流程

![Untitled](k8s-kube-scheduler/Untitled%204.png)

# 初始化流程梳理

# 业务逻辑梳理

# Scheduler Configuration

# PriorityClass

### 作用

1. 定义了pod在调度队列里的优先级. 优先级越高的, 越优先被从调度队列里拉出来进行调度. 
2. 定义了pod抢占行为以及被抢占行为时的优先级: 
    1. 如果没资源时, 默认高优先级的pod可以抢占低优先级的; 
    2. 如果没资源时, 默认低优先级的pod会被高优先级的pod抢占; 
    3. 某个Node上多个Pod需要被抢占, 以便给高优pod腾出空间时, 会尽量保障优先抢占低优的Pod

### 注意点

1. 一旦 Pod 被创建，它的 `spec.priorityClassName` 字段就不可更改。这是因为 Pod 的优先级是在创建时分配的，并且优先级是一个不可变的属性，不能在 Pod 生命周期中更改。

→ 为啥呢? 因为PriorityClass只在调度时生效, 一旦调度结束

# 抢占式调度(Preempt)

## 如何防止pod抢占其他pod?

> preemptionPolicy: Never
> 

该标签只能保证该pod不抢占其他pod, 但**不能保证其他更高优先级的pod不会抢占该pod**: 

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority-nonpreempting
value: 1000000
preemptionPolicy: Never
globalDefault: false
description: "This priority class will not cause other pods to be preempted."
```

默认抢占行为为, 默认抢占低优先级的pod: 

> `preemptionPolicy: PreemptLowerPriority`
> 

## 如何防止static pod被抢占?

由于static pod生命周期是由kubelet直接管理, 在抢占时kube-sched如何防止抢占掉static pod呢? 

## 与PDB的关系?

Kubernetes supports PDB when preempting Pods, but respecting PDB is best effort. The scheduler tries to find victims whose PDB are not violated by preemption, but if no such victims are found, preemption will still happen, and lower priority Pods will be removed despite their PDBs being violated.

→ 这也就是为啥看kube-sched代码里, 确实只是try best区保障PDB, 而不是严格保障. 

## 跨节点抢占

- P只能被调度到NodeN上,
- 但同时Q放在了NodeM上, P优先级比Q高
- P与Q有亲和性, 不能放在同一个可用区, 但此时NodeN&NodeM在同一个可用区.
- 如果支持跨节点抢占, 此时可以把NodeM上的Q抢占, 此时P就能顺利放在NodeN上;
- 但实际k8s当前(1.28)是不支持该特性的. 即只能同节点抢占.

# 预留(Reservation)

schedulingCycle 有啥用? 

v1.22

计算pod的cpu/mem/storage等资源消耗量算法:
pkg/scheduler/framework/plugins/noderesources/fit.go:164

// todo: 增加图例