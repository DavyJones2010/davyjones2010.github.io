---
title: K8s containerd 等研究
date: 2023-05-24 23:31:29
tags:
---
容器运行时: 运行和管理容器进程、镜像的工具
## 低层运行时:
### 功能
- 负责与宿主机操作系统打交道，根据指定的容器镜像在宿主机上运行容器的进程，并对容器的整个生命周期进行管理。
- 而这个低层运行时，正是负责执行我们前面讲解过的设置容器 Namespace、Cgroups等基础操作的组件。

### 实现分类
- runc: 传统的运行时，基于Linux Namespace和Cgroups技术实现，代表实现Docker. libcontainer(Docker公司) ---开源--> 改名为runc
- runv: 基于虚拟机管理程序的运行时，通过虚拟化 guest kernel，将容器和主机隔离开来，使得其边界更加清晰，代表实现是Kata Container和Firecracker. 目前已经废弃, 推荐使用kata container. 
- runsc：runc + safety ，通过拦截应用程序的所有系统调用，提供安全隔离的轻量级容器运行时沙箱，代表实现是谷歌的gVisor

## 高层运行时:
### 功能
负责镜像的管理、转化等工作, 为容器的运行做前提准备

### 实现分类
主流的高层运行时
- containerd
- CRI-O

高层运行时与低层运行时各司其职，容器运行时一般
1. 先由高层运行时将容器镜像下载下来，并解压转换为容器运行需要的操作系统文件
2. 再由低层运行时启动和管理容器。
![](https://pic3.zhimg.com/v2-388832b9ff6ded6f9e04e30c02078a72_r.jpg)


## CRI
Kubernetes早期是利用Docker作为容器运行时管理工具, 后来增加了rkt等. 随着运行时种类的增加, 
Kubernetes将对容器的操作抽象为一个接口，将接口作为kubelet与运行时工具之间的桥梁，kubelet通过发送接口请求对容器进行启动和管理，各个容器工具通过实现这个接口即可接入Kubernetes。
这个统一的容器操作接口，就是容器运行时接口(Container Runtime Interface, CRI)。
![](https://pic4.zhimg.com/v2-e8c76976f12a9b6552381a2dd4402887_r.jpg)

- kublet: 接收拉起/销毁容器的请求, 把请求通过grpc方式调用CRI接口, 请求路由到CRI shim上. (而不会直接调用docker的API)
- CRI shim: 作为gRPC服务端来响应CRI请求，负责将CRI请求的内容转换为具体的容器运行时API，在kubelet和运行时之间充当翻译的角色
- 
任何容器运行时如果想接入Kubernetes，都需要实现一个自己的CRI shim，来实现CRI接口规范。

CRI接口, 包含如下2个服务: 
- RuntimeService
  - PodSandbox 的管理接口：
    - CRI 设计的一个重要原则，就是确保这个接口本身，只关注容器，不关注 Pod。
    - PodSandbox 是对 Kubernete Pod 的抽象，用来给容器提供一个隔离的环境（比如挂载到相同的 CGroup 下面），并提供网络等共享的命名空间。PodSandbox 通常对应到一个 Pause 容器或者一台虚拟机；
  - Container 的管理接口：在指定的 PodSandbox 中创建、启动、停止和删除容器；
  - Streaming API接口: 包括 Exec、Attach 和 PortForward 等三个和容器进行数据交互的接口，这三个接口返回的是运行时 Streaming Server 的 URL，而不是直接跟容器交互。kubelet 需要跟容器项目维护一个长连接来传输数据。这种 API，我们就称之为 Streaming API。
  - 状态接口：包括查询 API 版本和查询运行时状态。

- ImageService
  - 查询镜像列表
  - 拉去镜像到本地
  - 查询镜像状态
  - 删除本地镜像
  - 查询镜像占用空间

# Refs
- [容器运行时](https://zhuanlan.zhihu.com/p/577765547)
- [CRI and ShimV2: A New Idea for Kubernetes Integrating Container Runtime](https://www.alibabacloud.com/blog/cri-and-shimv2-a-new-idea-for-kubernetes-integrating-container-runtime_594783)
- [CRI shim：kubelet怎么与runtime交互（一）](https://zhuanlan.zhihu.com/p/438351320)