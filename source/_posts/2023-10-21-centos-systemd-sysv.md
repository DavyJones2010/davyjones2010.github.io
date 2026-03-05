---
title: CentOS 7 systemd 与 SysV init 对比
date: 2023-10-21 00:00:00
tags: ['linux', 'centos', 'systemd']
---

# 20231021-centos-systemd-sysv

# 前言

centos7之后, 所有的service都推荐使用systemd/systemctl来管理.

# systemctl使用方式

- 查看所有的service

```
# systemctl list-units --all
```

- 查看service的配置项

```
[root@davywalker-centos7 ~]# systemctl cat vncserver@\\:1.service
# /etc/systemd/system/vncserver@:1.service
# The vncserver service unit file

[davywalker@davywalker-centos7 ~]$ systemctl cat sshd.service
# /usr/lib/systemd/system/sshd.service
[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.service
Wants=sshd-keygen.service

```

# systemctl开发方式

// TODO: 写一个程序, 以service方式配置

# systemd控制资源

systemd除了可以控制服务的启动顺序外, 还可以控制各个服务的资源消耗. 底层也是通过cgroups来实现. 

K8s针对Host资源的控制, 有如下 2 种实现方式: [Container Runtimes | Kubernetes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cgroup-drivers)

1. 基于普通的cgroups
2. 基于systemd, 详细使用方式参见: [Chapter 14. Using systemd to manage resources used by applications Red Hat Enterprise Linux 9 | Red Hat Customer Portal](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/managing_monitoring_and_updating_the_kernel/assembly_using-systemd-to-manage-resources-used-by-applications_managing-monitoring-and-updating-the-kernel)

具体选择哪种? 

[K8s choose cgroupfs or systemd? - SoByte](https://www.sobyte.net/post/2022-07/k8s-cgroupfs-or-syste/)

如果在systemd的版本里使用cgroups驱动, 那么会导致有 2 个cgroups manager, 一个是systemd天然的,  一个是kubelet里的, 在节点压力过大时, 会导致节点不稳定. 

> there are two managers coexisting in a system, it will end up with two views of those resources. Cases have been reported in this area where some nodes are configured so that kubelet and docker use `cgroupfs` while the rest of the processes running on the node use systemd; such nodes can become unstable under resource pressure.
> 

# 与sysv/Upstart的区别

centos6 使用的是 upstart 方式
与 centos7 的 systemctl 方式有啥区别?

- systemd可以并行启动, 而sysv是串行的
- 

# Refs

- [How To Use Systemctl to Manage Systemd Services and Units | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-use-systemctl-to-manage-systemd-services-and-units)
- [How to Auto-Start Services on Boot in RHEL/CentOS 7?](https://geekflare.com/systemd-start-services-linux-7/)