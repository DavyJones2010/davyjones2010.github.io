---
title: Java线程BLOCKED与WAITING状态深入研究
date: 2023-02-25 17:01:22
tags: [java, javase, thread, sync, concurrent, know-why]
---

# 背景
上次机房断网的jstack分析之后, 发现其实个人并没有深入理解Java中线程的如下两个状态的区别: 
- "BLOCKED"
- "WAITING/TIMED_WAITING"
或者, 都是线程被阻塞无法运行(让出了CPU的时间片)的状态:
- 问题1: 这两个状态具体有啥区别? 
- 问题2: JVM为什么要进行上边两个状态的区分? 为什么不只用一个状态标识?

先不急着回答这个问题, 我们从一个例子出发:  

# 生产者&消费者例子
先来一道面试里时常问到的题目:
`两个线程, 分别扮演消费者&生产者的角色, 假设队列为1, 无限循环. 如何写?`

## 代码样例
- 完整代码参见: [ProducerConsumerTest.java](https://github.com/DavyJones2010/test-core/blob/master/src/test/java/edu/xmu/test/concurrent/ProducerConsumerTest.java)

```java
// consumer
while (true) {
    synchronized (lock) {
        while (isEmpty) {
            System.out.println("consumer is waiting");
            lock.wait();
        }
        System.out.println("start consuming");
        Thread.sleep((long) (Math.random() * 10000L));
        System.out.println("finished consuming");
        isEmpty = true;
        lock.notify();
    }
}

// producer
while (true) {
    synchronized (lock) {
        while (!isEmpty) {
            System.out.println("producer is waiting");
            lock.wait();
        }
        System.out.println("start producing");
        Thread.sleep((long) (Math.random() * 10000L));
        isEmpty = false;
        System.out.println("finished producing");

        lock.notify();
    }
}
```

## 变体写法-1
如果在调用`lock.notify()`之后再生产或者再消费, 会怎么样?
即代码变体如下: 

```java
// consumer
while (true) {
    synchronized (lock) {
        while (isEmpty) {
            System.out.println("consumer is waiting");
            lock.wait();
        }
        lock.notify();
        
        System.out.println("start consuming");
        Thread.sleep((long) (Math.random() * 10000L));
        System.out.println("finished consuming");
        isEmpty = true;
    }
}

// producer
while (true) {
    synchronized (lock) {
        while (!isEmpty) {
            System.out.println("producer is waiting");
            lock.wait();
        }
        lock.notify();

        System.out.println("start producing");
        Thread.sleep((long) (Math.random() * 10000L));
        isEmpty = false;
        System.out.println("finished producing");
    }
}

```

> 可以自己尝试下, 代码结果仍然是正常的, 即与正常写法完全没有区别. 

这是怎么回事儿? 
- producer调用`lock.notify()`的时候是直接把consumer唤醒开始执行了么? 但producer生产的代码在`lock.notify()`之后, 那么`isEmpty=false`是否会执行到?
- 如果`isEmpty=false`执行不到, 而是在consumer侧执行, 那么consumer也还是无法开始消费(因为无法走出`while(isEmpty)`这个判断循环), 结果应该是consumer仍然卡在`wait()`上. 
- 如果`isEmpty=true`执行到, 那么是在什么时候唤醒了consumer呢? 
好复杂, **代码的执行链路到底是怎样的??** 

## wait/notify详解
要搞清楚代码链路是怎样的, 首先我们需要清楚调用wait/notify时到底发生了什么?
> 由于wait/notify都是native的代码, 阅读不易. 因此下文主要内容/概念都是参照[Java SE Specification](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html), 进行个人理解与研究.

先给出三个重要的概念: 
- object monitor: 每个java对象都有一个object monitor
- blocked set: 每个java对象都有一个blocked set
- wait set: 每个java对象都有一个wait set

再来说明下三个概念实际是怎么使用的:
1. 如下简单代码, 实际线程执行时发生了如下事情: 
```java
Object o = new Object();
synchronized(o) {
    // do smt
}
```

- 线程执行`synchronized(o)`会导致: 
  - 线程尝试去抢占`object monitor`; 
  - 如果抢占到, 则该线程拥有了该`object monitor`, 进行临界区代码执行. 
  - 如果抢占不到, 则该线程被放入该对象的`blocked set`中. 具体 
    1. 什么时候唤醒: 可以简化认为为JVM会在底层时刻轮询`object monitor`占用情况, 一旦`object monitor`被释放, 立刻从`blocked set`中找个线程开始执行.
    2. 如果多个线程都在`block set`中, 该唤醒哪个, 由JVM来决定(TODO: 这个具体待探究, 不影响本文). 

- 线程退出`synchronized(o)`临界区会导致: 
  - 当前线程释放掉`object monitor`
  - JVM轮询到`object monitor`处于空闲, 立刻从`blocked set`中取出一个线程, 让该线程开始临界区代码执行.


2. 再加上简单的wait/notify: 
如下, 一个最简单的producer/consumer程序: 
```java
// consumer
Object o = new Object();
synchronized(o) {
    o.wait();
    // consume
}
// produce
synchronized(o) {
    // produce
    o.notify();
}
```

> 先假设consumer先执行

- consumer执行链路: 
  1. `synchronized(o)`: 获取到`object monitor`, 开始临界区代码执行.
  2. `o.wait()`: 虽然可以拆解为如下几步, 但wait本身是原子操作 
     1. 把consumer线程放入到该对象的 `wait set` 中
     2. 释放掉`object monitor`
     3. JVM将producer从`block set`中取出, (触发JVM/OS的轮询, 引发producer获取到`object monitor`从而进入临界区)
- producer执行链路:
  1. `synchronized(o)`: 获取不到`object monitor`, 被放入`block set`中 
  2. consumer退出临界区, producer被自动从`block set`中取出, 获取到`object monitor`从而进入临界区 (与consumer执行链路的2.3步骤重叠)
  3. `o.notify()`: 
    - producer并不会因为`notify()`而释放掉`object monitor`: <mark>**`notify`并不会导致当前线程释放掉`object monitor`!**</mark>, 而是继续往下执行代码.
    - JVM将consumer从`wait set`中取出, 尝试获得`object monitor`
    - 由于producer此时并没有释放掉`object monitor`, 因此JVM就把consumer放入到了`block set`中(即从`wait set`移到了`block set`中)
  4. 退出临界区: 
    - producer释放掉`object monitor`
    - JVM将consumer从`block set`中取出
    - 触发JVM内部的轮询, 引发consumer获取到`object monitor`, 从而继续`o.wait()`之后的代码片段执行
- consumer继续执行, 注意:
  1. 此时consumer是<mark>继续从`object.wait()`之后的代码开始执行. (即之前中断的地方继续).</mark> 
  2. 而<mark>不是重新通过`synchronized(o)`抢`object monitor`, 然后从头开始执行临界区代码.</mark> 因为内部JVM已经把`object monitor`给了consumer了.
  3. 退出临界区:
     - consumer释放掉`object monitor`
     - JVM尝试从`block set`中取出线程, 由于`block set`为空, nothing happens

- 终态: **producer执行完成, consumer也执行完成.**

> 再假设producer先执行

- producer执行链路:
  1. `synchronized(o)`: producer获取到`object monitor`, 开始临界区代码执行.

- consumer执行链路:
  1. `synchronized(o)`: consumer获取不到`object monitor`, 被放入`block set`中

- producer执行链路:
  1. `o.notify()`: 
     1. producer不会因为`notify()`而释放掉`object monitor`
     2. JVM从`wait set`中寻找一个线程, 移出`wait set`, 并放入到`blocked set`中. 由于此时`wait set`为空(consumer在`block set`中). 因此nothing happens 
  2. 退出临界区:
     1. producer释放掉`object monitor`
     2. JVM将consumer从`block set`中取出, 由于`object monitor`已经被producer释放, 因此consumer直接获取到`object monitor`, 开始执行临界区代码  (consumer状态 `block set` -> `RUNNABLE`)

- consumer执行链路:
  1. `o.wait()`: 
     1. 把consumer线程放入到该对象的 `wait set` 中
     2. 释放掉`object monitor`
     3. JVM尝试从`block set`中取出一个线程, 由于此时`block set`为空, 因此nothing happens

- 终态: **producer执行完成, consumer一直卡在`WAITING`状态.**

## wait/notify/synchronized总结

### 调用 synchronized(object) 时会发生: 
1. 当前线程尝试抢占 `object monitor`
2. 如果抢占到, 则进入临界区.
3. 如果抢占不到, 把当前线程放入到`blocked set`中. JVM会监控`object monitor`, 当`object monitor`归还时, 从`blocked set`中挑选一个线程继续代码执行(可能是进入临界区, 也可能是继续之前中断的代码)

### 出 synchronized(object) 时会发生:
1. 释放掉`object monitor`;
2. JVM会监控`object monitor`, 当它归还时, 从`blocked set`中挑选一个线程继续代码执行(可能是进入临界区, 也可能是继续之前中断的代码)

### 调用 object.wait() 时会发生:
1. 把当前线程放入到 `wait set` 中
2. 释放掉`object monitor`
3. JVM会监控`object monitor`, 当它归还时, 从`blocked set`中挑选一个线程继续代码执行(可能是进入临界区, 也可能是继续之前中断的代码)

### 调用 object.notify() 时会发生:
1. 不会因为`notify()`而释放掉`object monitor`, 而是继续往下执行代码.
2. JVM从`wait set`中寻找一个线程, 移出`wait set`, 并放入到`blocked set`中.(JVM会持续监控`object monitor`状态)


### 线程在不同位置的不同状态
因此可以根据线程所处的位置不同, 来区分不同状态: 

- 在`wait set`里: `WAITING`状态
- 在`blocked set`里: `BLOCKED`状态

因此也就从根本上解释了本文开头的第一个问题 `问题1: 这两个状态具体有啥区别?`

### 线程可能得状态变化
`wait set`  -> `block set`
`block set` -> `wait set`
`block set` -> `RUNNABLE`

## wait/notify/synchronized实战
此时就很容易根据三个概念的流转, 来分析下上文的 `变体写法-1` 的执行流程, 也就明白为啥也可以work了.
本文就不再赘述了.

## 变体写法


## 更加风骚的写法

## 分析线程状态

# 实际生产中

# 总结
借此机会, 也解释了自己内心以来的长久疑惑. 


# Refs
- [Java Language Specification - Chapter 17. Threads and Locks](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html)
- [SoF上一个有意思的wait-notify问题](https://stackoverflow.com/questions/39927299/in-java-if-a-thread-calls-notify-before-wait-how-does-this-not-cause-the-se)




