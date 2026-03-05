---
title: k8s deploy arch
date: 2024-02-28 00:00:00
tags: [k8s, deployment, architecture]
---

# k8s-deploy-arch

对于k8s搭建比较感兴趣: 例如 各个组件如何部署的? 有啥先后顺序? 分别是以啥形态部署等, 以此文详细来分析下.

# 总览

如下图

![Untitled](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/source/assets/2024-02-28-k8s-deploy-arch/Untitled.png)

# 分析

1. 部署Master节点:

1. 手动通过系统服务(centos7u中的sysd service)的方式, 启动 kubelet, containerd 这两个服务, 同时在kubelet启动参数中配置好apiserver/etcd的{地址:端口}; —> 注意, 此时由于没有部署apiserver, kubelet默认需要连接apiserver, 此时肯定连不通, kubelet是设计为能够独立于apiserver正常启动并运行的组件. 

<aside>
💡 kubelet是设计为能够独立于apiserver正常启动并运行的组件。以下是kubelet启动成功的原因：

1. **独立运行**：kubelet的设计允许它在没有apiserver可用的情况下独立运行。kubelet负责节点上的容器生命周期管理，包括创建、启动、停止和删除容器。只要它能够与容器运行时（如Docker或containerd）进行通信，它就能管理容器。
2. **自我修复**：如果kubelet启动时无法连接到apiserver，它会定期重试连接。这种自我修复能力允许kubelet最终建立与apiserver的连接，一旦apiserver变得可用。
3. **静态Pod管理**：kubelet负责管理那些通过本地文件定义的静态Pod，而不依赖于apiserver。kubelet会定期扫描预配置的本地目录（如`/etc/kubernetes/manifests`）来启动和管理静态Pod。如果apiserver的manifest文件位于该目录中，kubelet会启动apiserver的Pod，即使它此时无法连接到apiserver。
4. **容错性**：kubelet的设计中包含了对网络分区和暂时性故障的容忍性。即使kubelet暂时无法连接apiserver，它也会继续保持当前的Pod运行，并按照最后的已知状态进行操作。

因此，即使在apiserver不可用的情况下，kubelet也能够启动并继续执行其容器管理的职责。一旦apiserver启动并变得可用，kubelet将与之建立连接并加入到Kubernetes集群中，开始接收新的指令和状态更新。

</aside>

1. 然后通过static pod的方式, 在master节点上启动 etcd kube-scheduler apiserver kube-controller-manager 这四种static pod

<aside>
💡 TODO: 
1. 确认下这几个static pod的pod spec, 对于资源的请求是咋样的? pod的类型是啥? BE or GU or Burstable? 在哪里可以配置? 最佳实践是咋样的?
2. 这些组件的容灾(例如kube-sched) 怎么做的? 防止这个static pod挂了, 导致业务整体不可用. 
3. 确认下这几个pod的priority是怎样的? 如何防止Node pressure的时候被逐出?

</aside>

<aside>
💡 - 其中apiserver/etcd都需要对外提供HTTP服务, 那么这些pod的端口是怎么对外暴露的呢? 
- 这些pod的spec里, 都有 hostNetwork: true 标识, 代表直接把pod的端口暴露到host上. 详细参见下文的 “hostNetwork” 部分.
例如: cat /etc/kubernetes/manifests/kube-apiserver.yaml

</aside>

1. 当etcd/apiserver启动完成后, kubelet会自动作为kube-client连接上去, 进行list/watch等操作, 同时向apiserver汇报pod信息. 

1. 至此, 基础的k8s已经启动起来. 接下来创建 kube-proxy 这个daemonset, 会在每个节点上自动启动kube-proxy-xxx的pod, 便于后续k8s的service使用. 

<aside>
💡 TODO: 但此时kube-proxy与coredns是啥关系? 有啥启动的优先级么? 需要仔细研究下

</aside>

1. 部署Worker节点: 
    1. 手动通过系统服务(centos7u中的sysd service)的方式, 启动 kubelet, containerd 这两个服务, 同时在kubelet启动参数中配置好apiserver/etcd的{地址:端口};
    2. 此时由于已经部署好了apiserver, kubelet默认可以连接apiserver. 相当于该Worker节点已经加入了k8s集群中了
    3. daemonset controller watch 到新增加了node, 因此自动创建kube-proxy的pod, 指定pod.nodeName=Node1; 
    4. Node1上的kubelet watch到需要在本节点创建kube-proxy节点, 因此直接开始创建. 

# 总结

整体来说还是非常自洽的; 需要手动部署的基础组件基本只有2个, kubelet+containerd, 而这两个基本可以在操作系统镜像里做好, 这样node可以快速扩容. 

只要有kubeconfig, node就能直接连通k8s集群, 也是很方便的. 

# 其他要点

## kubeconfig详解

kubeconfig文件非常重要, 基本连接k8s集群, 一个kubeconfig文件即可. 样例如下: 

```yaml
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: xxx
    server: https://apiserverIp:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
    namespace: kube-system
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: xxx
    client-key-data: xxx
```

包含几个重要信息: 

1. clusters: 可以定义多个k8s cluster, 里边包含cluster的apiserver地址与端口; 以name作为cluster的唯一标识;
    1. `server`：Kubernetes API服务器的URL。
    2. `certificate-authority`或`certificate-authority-data`：用于验证API服务器证书的根证书。后续详细在k8s-cert中详解.
    3. `name`：集群的名称，用于引用。
2. users: 可以定义多个用户; 以name作为user的唯一标识;
    1. `client-certificate-data`  客户端证书用于TLS认证
    2. `client-key-data` 客户端私钥用于TLS认证
    3. `token`：用于承载令牌认证。
    4. `username`和`password`：基本认证。
3. contexts: 将user与cluster进行绑定

<aside>
💡 小技巧:

- 经常想要调研下kube-system下的pod, 每次  `kubectl get pod -n kube-system` 输入好麻烦, 怎么能默认使用  `kube-system` 这个namespace?
- 可以直接修改 `.kube/config` 文件, 如上, 在context里增加如下  `namespace: kube-system`  即可.
</aside>

<aside>
💡 小技巧:

- 如何快速switch context?
- 
</aside>

## hostNetwork的含义与使用场景

```go
在Kubernetes中，当Pod的定义中设置了hostNetwork: true，意味着该Pod将不会使用Kubernetes为其Pod分配的隔离网络命名空间，而是使用宿主机（Node）的网络命名空间。换句话说，这个Pod将共享Node的网络栈，并且能够直接访问Node上的网络接口。
当你把hostNetwork设置为true时，Pod中的容器将会有以下特性：
- 网络接口：容器将能够直接访问宿主机的网络接口，包括所有的网络服务和端口。
- IP地址：容器将使用宿主机的IP地址作为自己的IP地址。
- 端口冲突：由于容器共享宿主机的网络，所以任何尝试在标准端口上启动服务的容器可能会因端口冲突而失败，除非宿主机上该端口未被占用。
- 隔离性降低：使用宿主机网络意味着网络隔离性降低，因为Pod直接运行在宿主机网络环境中。
- 安全影响：由于缺乏网络隔离，使用hostNetwork: true需要考虑额外的安全措施，因为不恰当的使用可能会增加攻击面。
- DNS设置：通常，容器使用的是Kubernetes集群的DNS服务，但是在hostNetwork模式下，容器将使用宿主机的DNS设定。

Kubernetes中的某些类型的Pod（例如kube-apiserver、kube-scheduler和kube-controller-manager等核心组件）通常设置为hostNetwork: true，这样做是为了确保它们能够有效地与其他组件进行通信，并且可以方便地从集群外部访问这些服务（例如通过SSH或其他网络工具）。
```

## kube-proxy与coredns

## configmap的使用

![Untitled](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/source/assets/2024-02-28-https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/source/assets/2024-02-28-k8s-deploy-arch/Untitled.png)
