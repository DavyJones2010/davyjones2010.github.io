---
layout: post
title: 记一次分布式场景下状态机设计缺陷导致的问题
tags: [learn-from-failure, java, state-machine, lock, good-design, bad-design]
lang: zh
---

# 背景
- 为某个对象item建模, 有如下几种状态变迁, 在数据库中state字段记录:
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202207072308671.png)

- 其中processing过程可能会持续时间较久, 10min左右
- 在服务端某个线程响应请求, 该对象处于processing过程中时, 服务器发布重启, 导致item状态一直卡在processing 
- 而processing状态, **本身是中间状态, 无法进行任何人肉干预/操作**, 从而导致只能临时提交数据订正, 将状态字段修改回"init", 然后再执行一次process.   


# 方案
仔细思考了下, 发现设计的时候, 根本原因是对于中间状态没有做好处理, 如背景中介绍的服务重启的处理. 
而<mark>在分布式场景下, 服务重启是by design需要被接受的.</mark> 
这里思考了下可能的几种处理方式:

## 原则
- 如何区分状态机的中间状态与终态? 
- 一条原则: <mark>终态->终态之间, 必须是可以人肉有入口触发的(而不是系统自动触发的); 必须是可重入的.</mark>

## 方案1: 状态机设计修改: 把processing作为纯粹的中间状态
1. 将processing从状态机中删除掉, 如下: <br/>
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202207072351350.png)

4. 使用 分布式锁/db字段锁 来实现排他功能(即item同时只能被一个线程处理, 防止多个线程同时处理一个item导致死锁/重复计算等问题).
   1. 在item执行前, 加上锁+锁超时时间(如例子中的10min); 其他线程要执行时, 无法抢到该item的锁. 
   2. item执行完成之后, 状态修改为finished之后, 再释放掉item锁.
   3. item执行异常中断(例如服务器重启, 线程crash等): 等待锁超时. 由于仍然是init状态(终态), 因此可以重新人肉触发, 新的线程抢到锁, 重新执行.
   4. item执行失败: 线程里catch住异常, 主动释放掉该item锁. 由于仍然是init状态(终态), 因此可以重新人肉触发, 新的线程抢到锁, 重新执行.
5. 或者使用事务:(不过本例子中不适合, 因为10min太久了, 其他执行耗时较短30s以内的可以使用该方案)  
   1. item执行前开启事务;
   2. 执行后修改状态为finished, commit事务.
   3. item执行异常中断(例如服务器重启, 线程crash等): 事务自动回滚. 由于仍然是init状态(终态), 因此可以重新人肉触发, 新的线程抢到锁, 重新执行.
   4. item执行失败: 主动回滚事务. 由于仍然是init状态(终态), 因此可以重新人肉触发, 新的线程抢到锁, 重新执行.

## 方案2: 状态机设计修改: 把processing作为纯粹的终态
1. 需要设计从processing->finished/init的人肉触发入口.
   ![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202207072354738.png)

2. 如何防止多个线程同时触发该item从processing->finished/init的变迁? 参见方案1中锁/事务的方式

## 方案3: 优雅停机
1. 在shutdown-hook里注册事件:
   1. 将状态改回init. --> required.
   2. 将worker线程interrupt掉. --> optional, 因为即使不interrupt, 进程停止线程也会被回收.
2. 但该方案有很大的缺陷, 如果直接`kill -9`, 则shutdown-hook根本不会执行.

## 最终方案
最终采用了方案2, 因为从状态机中删除掉一个终态, 对现有代码改造量太大了.

# 其他思考 
1. 状态机设计的时候, 一定要慎重考虑哪些是终态, 哪些是中间状态. 不是说因为在某个状态持续时间较长(如例子中的processing), 就要作为终态. 
2. 状态机中每个终态之间, 必须考虑可重入性; 必须要保证能人肉触发终态之间的转化. 本文的例子就是反面教材, item卡在processing这个终态, 无法进行程序上的任何操作.


