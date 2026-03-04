---
title: libvirt CPU 选项与超线程关闭原理剖析
date: 2024-01-08 00:00:00
tags: [virtualization, libvirt, cpu, hyperthreading, cloud-computing, iaas]
---

# 20240108-libvirt-cpuoptions-htoff

最近在, 发现可以自定义CPU选项: [通用型实例规格族取值表\_云服务器 ECS(ECS)-阿里云帮助中心](https://help.aliyun.com/zh/ecs/user-guide/cpu-options-of-general-purpose-instance-families#g6)  

那么底层是如何实现的呢? 

# Why

## 为啥需要关闭HT?

根据AWS的Spec, 主要还是为了性能. 

> You might do this for certain workloads, such as high performance computing (HPC) workloads.
> 

## 为啥需要自定义CpuOptions.Cores?

1. 节省License费用

各个云厂商的Spec, 使用 CpuOptions.Cores 关闭部分Cores, 并不会导致费用因此降低, 还是按照规格. 优点是能节省软件License的费用. —> 但这样是否划算? 为啥不干脆用一个小规格? 

> Disabling cores don't effect on instance cost. The reason why you would typically want to disable some of available cores is due to software licensing. If your application needs a lot of memory but can not benefit from extra cores, it can be cost effective to disable some cores to lower licensing cost. Even if you would still be paying for the whole instance capacity.
> 

1. 可以获取更大的 MEM/CPU 配比. 

云厂商对外, 最大的 MEM/CPU 为 8:1; 例如

- ecs.r7.2xlarge (8C64Gi) 默认 MEM/CPU=8:1;
- 如果关闭HT, 则 MEM/CPU 扩大为 16:1
- 如果再修改下 CpuOptions.Cores=2; 则 MEM/CPU 继续扩大为 32:1

> You might do this to potentially optimize the licensing costs of your software with an instance that has sufficient amounts of RAM for memory-intensive workloads but fewer CPU cores.
> 

站在个人角度理解, 关闭HT可能有. 但屏蔽掉部分Cores, 

# How(libvirt层如何实现)

## 如何关闭HT?

- 如下, 单socket, 4个物理Core, htoff的如下:

```xml
<vcpu placement="static" cpuset="0-25,52-77" current="4">4</vcpu>
<cputune>
  <vcpupin vcpu="0" cpuset="76"/>
  <vcpupin vcpu="1" cpuset="75"/>
  <vcpupin vcpu="2" cpuset="74"/>
  <vcpupin vcpu="3" cpuset="73"/>
  <emulatorpin cpuset="0-3,5-20,52-55,57-76"/>
</cputune>

<cpu mode="host-passthrough">
  <topology sockets="1" cores="4" threads="1"/>
  <numa>
    <cell cpus="0-3" memory="33554432" memAccess="shared" reserve="524288"/>
  </numa>
</cpu>
```

- 如下, 单socket, 4个物理Core, hton的如下:

```xml
<vcpu placement="static" cpuset="0-25,52-77" current="8">8</vcpu>
<cputune>
  <vcpupin vcpu="0" cpuset="77"/>
  <vcpupin vcpu="1" cpuset="25"/>
  <vcpupin vcpu="2" cpuset="76"/>
  <vcpupin vcpu="3" cpuset="24"/>
  <vcpupin vcpu="4" cpuset="75"/>
  <vcpupin vcpu="5" cpuset="23"/>
  <vcpupin vcpu="6" cpuset="74"/>
  <vcpupin vcpu="7" cpuset="22"/>
  <emulatorpin cpuset="0-3,5-16,22-25,52-55,57-68,74-77"/>
</cputune>
<cpu mode="host-passthrough">
  <topology sockets="1" cores="4" threads="2"/>
  <numa>
    <cell cpus="0-7" memory="33554432" memAccess="shared" reserve="524288"/>
  </numa>
</cpu>
```

## 如何实现自定义cpuoptions.core?

- 如下,

```xml
<vcpu placement="static" cpuset="0-25,52-77" current="2">2</vcpu>
<cputune>
  <vcpupin vcpu="0" cpuset="70"/>
  <vcpupin vcpu="1" cpuset="69"/>
  <emulatorpin cpuset="0-3,5-16,52-55,57-70"/>
</cputune>
<cpu mode="host-passthrough">
  <topology sockets="1" cores="2" threads="1"/>
  <numa>
    <cell cpus="0-1" memory="33554432" memAccess="shared" reserve="524288"/>
  </numa>
</cpu>
```

# Refs

- AWS的方式: [Optimize CPU options - Amazon Elastic Compute Cloud](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-optimize-cpu.html)