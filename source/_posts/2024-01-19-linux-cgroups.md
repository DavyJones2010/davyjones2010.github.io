---
title: linux cgroups
date: 2024-01-19 00:00:00
tags: [linux, cgroups, resource-management]
---

# linux-cgroups

# 背景

nrm里通过cgroups来对各个pod资源进行限制(例如是否绑核), 但势必需要知道有哪些操作, 具体怎么操作?

同时学习下CFS相关的

# 理论

- **0.0 st** ：%st和虚拟机有关，当系统运行在虚拟机中时，当前虚拟机就会和宿主机以及其它的虚拟机共享CPU，%st就表示当前虚拟机在等待CPU为它服务的时间。该值越大，表示物理CPU被宿主机和其它虚拟机占用的时间越长，导致当前虚拟机得不到充足的CPU资源。如果%st长时间大于0，说明CPU资源得不到满足，这时可以考虑将虚拟机移到其它机器上，或者减少当前机器运行的虚拟机数量。
- 在cgroup里面，跟CPU相关的子系统有[cpusets](https://link.segmentfault.com/?enc=biuRLGgeMDPQb6dvVaiApA%3D%3D.bKQXU9h%2Btb1qjQmY%2FrNvNKWgu74VP39WpZwyHHjwij5i28Lfl7i01u4%2Bkc%2BK69SehRGaVBrULROg9wSxHGPS5Q%3D%3D)、[cpuacct](https://link.segmentfault.com/?enc=frktheHBHdM87%2FMsyytQIw%3D%3D.4gHFVDAsinkl3bI2nD9Zi5ZHwWoMjjdSOyKinMAJzXFKPwUeSfjHj1v7tFDsMULvU0%2BVYiueVK6ntLwV8Ge57w%3D%3D)和[cpu](https://link.segmentfault.com/?enc=eeWWT6pVbWUHMTU1ArS74g%3D%3D.jsnSkVkRwP6V0iZGlmbOjMJv%2F%2BLPAi1UEhPJPJfLbTcYFwVkIHtswL4TfBFxw2nYxAWe5GzgsV%2BGbMDNf%2FMVxhT06BNTHkbf2UBBuHp4iwE%3D)。
- cpuset: cpuset主要用于设置CPU的亲和性，可以限制cgroup中的进程只能在指定的CPU上运行，或者不能在指定的CPU上运行，同时cpuset还能设置内存的亲和性。

# cgroups/cpu

- cpu下包含如下:

```go
-r--r--r-- 1 root root 0 9月  14 16:36 cpu.stat
-rw-r--r-- 1 root root 0 9月  14 16:36 cpu.shares
-rw-r--r-- 1 root root 0 9月  14 16:36 cpu.rt_runtime_us
-rw-r--r-- 1 root root 0 9月  14 16:36 cpu.rt_period_us
-rw-r--r-- 1 root root 0 9月  14 16:36 cpu.cfs_quota_us
-rw-r--r-- 1 root root 0 9月  14 16:36 cpu.cfs_period_us
```

- cfs_period_us: 配置时间周期长度; 取值范围为1毫秒（ms）(即1000) 到1秒（s）即(1000000); 默认值是100000us(100ms)，一般cpu.cfs_period_us作为系统默认值我们不会去修改它。
- cfs_quota_us: 配置当前cgroup在设置的周期长度内所能使用的CPU时间数; 取值大于1ms即可; -1（默认值），表示不受cpu时间的限制

# cgroups/cpuset

```go
-rw-r--r-- 1 root root 0 9月  14 19:25 cpuset.sched_relax_domain_level
-rw-r--r-- 1 root root 0 9月  14 19:25 cpuset.sched_load_balance
-rw-r--r-- 1 root root 0 9月  14 19:25 cpuset.memory_spread_slab
-rw-r--r-- 1 root root 0 9月  14 19:25 cpuset.memory_spread_page
-r--r--r-- 1 root root 0 9月  14 19:25 cpuset.memory_pressure
-rw-r--r-- 1 root root 0 9月  14 19:25 cpuset.memory_migrate
-rw-r--r-- 1 root root 0 9月  14 19:25 cpuset.mem_hardwall
-rw-r--r-- 1 root root 0 9月  14 19:25 cpuset.mem_exclusive
-r--r--r-- 1 root root 0 9月  14 19:25 cpuset.effective_mems
-r--r--r-- 1 root root 0 9月  14 19:25 cpuset.effective_cpus
--w--w--w- 1 root root 0 9月  14 19:25 cgroup.event_control
-rw-r--r-- 1 root root 0 9月  14 19:25 cgroup.clone_children
-rw-r--r-- 1 root root 0 9月  14 19:28 cgroup.procs
-rw-r--r-- 1 root root 0 9月  14 19:31 cpuset.cpu_exclusive
-rw-r--r-- 1 root root 0 9月  14 19:32 cpuset.mems
-rw-r--r-- 1 root root 0 9月  14 19:32 tasks
-rw-r--r-- 1 root root 0 9月  14 19:37 cpuset.cpus
```

# 操作

# cpuset

- 一定要同时配置 cpuset.mems, 来决定mem的node是哪个. 否则会报错如下:

![](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/18490/1694691235488-42588ca3-b4ef-4b31-b0b9-413975aa32fd.png)

- 设定不绑核: 则随便在飘

![](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/18490/1694691356771-f3ea9977-e8d8-4da5-8560-56a9e2e71ba8.png)

# cpu

- 限制某个PID只能使用10%的CPU:

```go
# 把PID加入cgroup里
echo 30342 > /sys/fs/cgroup/cpu/foo/tasks

# 查看cfs的period, 是默认值100ms
cat /sys/fs/cgroup/cpu/foo/cpu.cfs_period_us
100000

# 设置该cgroup最大能使用10ms, 即10%的cpu利用率
echo 10000 > /sys/fs/cgroup/cpu/foo/cpu.cfs_quota_us
```

- 在多核场景下会是啥样的? 想要限制某个PID只能使用某个CPU的10%? 能使用2个CPU, 各10% 分别如何配置?
- 如下, 不设置cpuset, 默认会绑核到cpu0上, 从而使用了单核10%的资源: --> TODO: 为啥会默认绑定到cpu0上呢?

![](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/18490/1694690608748-05cbd7d7-9a4b-4411-a655-52d7567488e2.png)

- 如下, 可以通过cpuset把进程绑定到其他单个核上, 利用率仍然是10%

![](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/18490/1694691191868-934d2f00-d138-45bc-8870-a35ddbb625f4.png)

- 绑定到0-3核上(即可以随便飘), 每个Core利用率叠加起来刚好是10%
- 把进程绑定在cpu0上, quota设置为 `200ms`, 实际会怎样?
- 仍然是单个cpu0利用率为100%, 等价于 quota设置为`100ms`
- 把进程绑定在cpu0-1上, quota设置为 `200ms`, 实际会怎样?
- 仍然是单个cpu0利用率为100%, (因为测试python脚本是单线程死循环). 偶尔飘到cpu1上, 就把cpu1打到100%
- 初步观测, cpu漂移并不频繁

> 4C8G
> 

| 场景编号 | 测试脚本 | cgroup内进程数 | cpuset | cpu.cfs_quota_us | 观测结果 |
| --- | --- | --- | --- | --- | --- |
| s-1 | 单线程, 死循环 | 1 | NULL | 未加入cgroup | ?? |
|  |  |  | NULL | 10ms | cpu0 10%; 其他空闲 |
|  |  |  | NULL | 100ms | cpu0 100%; 其他空闲 |
|  |  |  | cpu0 | 100ms | cpu0 100%; 其他空闲 |
|  |  |  | cpu0 | 200ms | cpu0 100%; 其他空闲 |
|  |  |  | cpu1 | 100ms | cpu1 100%; 其他空闲 |
|  |  |  | cpu0-1 | 100ms | • 随机挑选一个cpu 100%
• 会飘到其他cpu, 把其他cpu 打满
• 但漂移频率不高 |
|  |  |  | cpu0-3 | 50ms | • 随机挑选一个cpu 50%
• 会飘到其他cpu, 把其他cpu打到50%
• 漂移频率不高, 但比 s-12 高 |
| s-12 |  |  | cpu0-3 | 100ms | • 随机挑选一个cpu 100%
• 会飘到其他cpu, 把其他cpu打满
• 但漂移频率不高 |
|  | 2线程, 死循环 | 1 | NULL | 未加入cgroup | • 4个Core都有一定的利用率
• 加起来基本是200% |
|  |  |  | NULL | 100ms | • 4个Core都有一定的利用率
• 加起来基本是200% |
|  |  |  |  | 50ms |  |
|  |  |  | cpu0 | 100ms |  |
|  |  |  | cpu0 | 200ms |  |
|  | 单线程, 死循环 | 2 |  |  |  |
- 注意, 进程 13121 下边创建了2个线程: 13122, 13123; 把 13121 加入tasks里, 并不能对下边2个线程生效.
- TODO: 如何自动对子线程生效?

![](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/18490/1694694132369-7241a9a1-93fe-4d01-9467-5f70d4446bb2.png)

# Q&A

1. vmm如何使用cgroup来限制把某个虚拟机限制为2C4Gi? 使用的是cgroup么? CPU/MEM分别都是怎么限制的? 还是其他技术?
2. 如何模拟 steal>0 的场景?
3. cgroup针对cpu, 限制的是用户态 or 内核态 的usage?
4. cgroup下cpuset可以设定绑核, 同时 numactl 里有 taskset 也可以绑核, 两者区别是啥?

# 常用

## 命令

```go
# 查看绑核情况
cat /sys/fs/cgroup/cpuset/test/cpuset.cpus
cat /sys/fs/cgroup/cpuset/test/tasks

# 查看cfs情况
cat /sys/fs/cgroup/cpu,cpuacct/test/cpu.cfs_period_us
cat /sys/fs/cgroup/cpu,cpuacct/test/cpu.cfs_quota_us
cat /sys/fs/cgroup/cpu,cpuacct/test/tasks
```

## 脚本

- 单线程死循环

```python
while True:
    pass
```

- 2个线程死循环

```python
import threading

def death_loop():
    while True:
        pass

if __name__ =="__main__":
    t1 = threading.Thread(target=death_loop)
    t2 = threading.Thread(target=death_loop)

    # starting thread 1
    t1.start()
    # starting thread 2
    t2.start()

    # wait until thread 1 is completely executed
    t1.join()
    # wait until thread 2 is completely executed
    t2.join()
```

# 操作方式

## 直接操作cgroups

```bash
# 查看绑核情况
cat /sys/fs/cgroup/cpuset/test/cpuset.cpus
cat /sys/fs/cgroup/cpuset/test/tasks

# 查看cfs情况
cat /sys/fs/cgroup/cpu,cpuacct/test/cpu.cfs_period_us
cat /sys/fs/cgroup/cpu,cpuacct/test/cpu.cfs_quota_us
cat /sys/fs/cgroup/cpu,cpuacct/test/tasks
```

## 通过libcgroup-tools操作cgroups

```bash
# sudo yum install libcgroup-tools

$ UUID=$(uuidgen)
$ cgcreate -g cpu,memory:$UUID
$ cgset -r memory.limit_in_bytes=100000000 $UUID
$ cgset -r cpu.shares=512 $UUID

$ cgset -r cpu.cfs_period_us=1000000 $UUID
$ cgset -r cpu.cfs_quota_us=2000000 $UUID

$ cgexec -g cpu,memory:$UUID \
>     unshare -uinpUrf --mount-proc \
>     sh -c "/bin/hostname $UUID && chroot $ROOTFS /bin/sh"
/ # echo "Hello from in a container"
Hello from in a container
/ # exit

$ cgdelete -r -g cpu,memory:$UUID
$ rm -r $ROOTFS
```

## 通过systemd操作cgroups

# Refs

- [Linux Cgroup系列（02）：创建并管理cgroup - Linux程序员 - SegmentFault 思否](https://segmentfault.com/a/1190000007241437)
- [Linux CPU使用率 - Linux程序员 - SegmentFault 思否](https://segmentfault.com/a/1190000008322093)
- [CFS Bandwidth Control — The Linux Kernel documentation](https://www.kernel.org/doc/html/v5.6/scheduler/sched-bwc.html)
- [重学容器29: 容器资源限制之限制容器的CPU | 青蛙小白](https://blog.frognew.com/2021/07/relearning-container-29.html)
- [重学容器06: 容器资源限制背后的技术cgroups | 青蛙小白](https://blog.frognew.com/2021/05/relearning-container-06.html)
- [CPUSETS — The Linux Kernel documentation](https://docs.kernel.org/admin-guide/cgroup-v1/cpusets.html#:~:text=New%20cpusets%20are%20created%20using,cpusets%20directory%2C%20as%20listed%20above.)
- [Container Runtimes Part 2: Anatomy of a Low-Level Container Runtime](https://www.ianlewis.org/en/container-runtimes-part-2-anatomy-low-level-contai)