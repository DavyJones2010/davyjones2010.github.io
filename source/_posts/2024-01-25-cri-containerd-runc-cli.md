---
title: 'CRI、Containerd、runc 与 CLI 工具链'
date: 2024-01-25 00:00:00
tags: ['cri', 'containerd', 'runc', 'tools']
category: container-orchestration
---

# cri-containerd-runc-cli

# crictl

## 配置

配置endpoint为containerd

```go
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
```

否则会去默认找 docker.sock, 从而报错: 

```go

WARN[0000] runtime connect using default endpoints: [unix:///var/run/dockershim.sock unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]. As the default settings are now deprecated, you should set the endpoint instead.
E0125 15:39:09.340202   38890 remote_runtime.go:277] "ListPodSandbox with filter from runtime service failed" err="rpc error: code = Unavailable desc = connection error: desc = \"transport: Error while dialing dial unix /var/run/dockershim.sock: connect: no such file or directory\"" filter="&PodSandboxFilter{Id:,State:nil,LabelSelector:map[string]string{},}"
FATA[0000] listing pod sandboxes: rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing dial unix /var/run/dockershim.sock: connect: no such file or directory"
```

## 命令

```go
# 查看所有pods/sandbox
$sudo crictl pods
POD ID              CREATED             STATE               NAME                       NAMESPACE           ATTEMPT             RUNTIME
b0a9d72288491       18 hours ago        Ready               qos-demo-be03              qos-example         0                   (default)
ebb390f699f40       18 hours ago        Ready               qos-demo-be02              qos-example         0                   (default)
2493312953f99       18 hours ago        Ready               qos-demo-be                qos-example         0                   (default)

# pod/sandbox操作
# 按照podconfig新建pod
$sudo crictl runp [POD_CONFIG_JSON]

# 查看pod详情
$sudo crictl inspectp b0a9d72288491

# container操作
# 在pod里新建container
$sudo crictl create ${SANDBOX_ID} container.json sandbox.json

# 启动container
$sudo crictl start ${CONTAINER_ID}

# 列出container
$sudo crictl ps

# 
#sudo crictl inspect ${CONTAINER_ID}

# 拉取镜像
$sudo crictl pull nginx

# 列出所有镜像
$sudo crictl images

```

# ctr

[How to work with container images using ctr](https://labs.iximiuz.com/courses/containerd-cli/ctr/image-management#what-is-ctr)

- ctr 命令没法查看到sandbox的信息 —> 因为 sandbox 的概念是 —> 但实际containerd版本升级之后(ctr也要相应升级), 是有sandbox命令的!
- ctr 查看容器/镜像等, 需要指定namespace, k8s默认的namespace为 [`k8s.io`](http://k8s.io)

```go

# namespaces
$sudo ctr namespaces list

# sandbox --> 要保证ctr的版本
sudo ctr -n k8s.io sandboxes ls

# container
$sudo ctr -n k8s.io c list
$sudo ctr -n k8s.io containers list

# image
$sudo ctr -n k8s.io  images list

```

# runc

- Refs: [Implementing Container Runtime Shim: runc]([https://iximiuz.com/en/posts/implementing-container-runtime-shim/#:~:text=A container runtime shim is a lightweight daemon launching runc,manager happen through the shim.)](https://iximiuz.com/en/posts/implementing-container-runtime-shim/#:~:text=A%20container%20runtime%20shim%20is%20a%20lightweight%20daemon%20launching%20runc,manager%20happen%20through%20the%20shim.))
- 发现使用containerd创建的pod/容器, 虽然底层使用了runc, 但使用runc命令是看不到的!
- runc 命令执行这个过程不涉及网络请求或复杂的数据处理，它仅仅是文件系统操作和数据格式化的过程。`runc` 会直接读取本地文件系统上的数据，将其格式化之后输出到标准输出。因此，`runc list` 命令的执行相对简单，且响应速度快。

```go

# Create a default bundle (i.e. container) config.
$ runc spec

# Run the container with ID cont1.
$ sudo runc run cont1

$ sudo runc create mycontainerid
$ sudo runc start mycontainerid

$ sudo runc list

# now delete the container
$ sudo runc delete mycontainerid

```