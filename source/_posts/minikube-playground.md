---
title: Minikube 实验笔记
date: 2023-11-07 00:00:00
tags: ['minikube', 'kubernetes', 'playground']
category: container-orchestration
---

1. install docker: [软件获取与安装 · Docker 文档](https://www.zhaowenyu.com/docker-doc/ops/docker-install.html)

1. install minikube: [minikube安装Kubernetes · minikube](https://www.zhaowenyu.com/minikube-doc/ops/minikube.html)

```bash
curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.18.1/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

FXXK!!!

## 内存不足

![Untitled](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/source/assets/minikube-playground/Untitled.png)

![Untitled](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/source/assets/minikube-playground/Untitled-1.png)

## checksum文件下载失败

![Untitled](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/source/assets/minikube-playground/Untitled-2.png)

修复: 

重新从官方的release中下载最新的binary Lol

```bash

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

## 下载速度太慢啦!

![Untitled](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/source/assets/minikube-playground/Untitled-3.png)

```bash
minikube start --kubernetes-version v1.17.5 \
--vm-driver=none \ 
--cni=flannel \
--image-mirror-country=cn \
--image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers \
--iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.7.3.iso  \
--registry-mirror=https://hub-mirror.c.163.com

```

## conntrack 需要提前安装

![Untitled](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/source/assets/minikube-playground/Untitled-4.png)

```bash

sudo yum install -y conntrack
```

## crictl 需要提前安装

![Untitled](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/source/assets/minikube-playground/Untitled-5.png)

参见: 手动把tgz下载下来:

[https://github.com/kubernetes-sigs/cri-tools](https://github.com/kubernetes-sigs/cri-tools)

## kubectl 下载太慢

![Untitled](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/source/assets/minikube-playground/Untitled-6.png)

1. 根据: [Install and Set Up kubectl on Linux | Kubernetes](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) 中说明, 直接配置yum源, 使用yum 安装要快很多. 
2. 安装好之后, 直接 kubectl 就可以连接到minikube集群里了. 不需要再使用 `minikube kubectl`这种命令