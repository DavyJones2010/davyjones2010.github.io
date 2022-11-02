---
title: K8s部署使用错误排查
date: 2022-11-01 21:45:41
tags: [k8s, kubectl, trouble-shooting]
---

# ImagePullBackOff

## 错误

---
在mac上使用minikube, 创建pod出现如下 ImagePullBackOff:
```shell
  Normal   Pulling    8s               kubelet            Pulling image "docker.io/nginx:1.23"
  Warning  Failed     8s               kubelet            Failed to pull image "docker.io/nginx:1.23": rpc error: code = Unknown desc = Error response from daemon: Get "https://registry-1.docker.io/v2/": dial tcp: lookup registry-1.docker.io on 192.168.64.1:53: server misbehaving
  Warning  Failed     8s               kubelet            Error: ErrImagePull
  Normal   BackOff    6s (x2 over 7s)  kubelet            Back-off pulling image "docker.io/nginx:1.23"
  Warning  Failed     6s (x2 over 7s)  kubelet            Error: ImagePullBackOff
```
## 解决方案

---
参见: [解决方案](https://github.com/docker/for-mac/issues/1317)
1. 登录minikube的节点
```shell
$ minikube ssh
```

2. 在minikube node上手动 pull docker image, 如下, 发现问题仍然重现
```shell
$ docker pull docker.io/nginx
Using default tag: latest
Error response from daemon: Get "https://registry-1.docker.io/v2/": dial tcp: lookup registry-1.docker.io on 192.168.64.1:53: server misbehaving
```

3. 修改minikube node上的resolv.conf文件的nameserver:
```shell
$ sudo vi /etc/resolv.conf
nameserver 8.8.8.8
search .
```

4. 重新尝试手动pull image, 发现问题解决:
```shell
$ docker pull docker.io/nginx:1.23
1.23: Pulling from library/nginx
Digest: sha256:943c25b4b66b332184d5ba6bb18234273551593016c0e0ae906bab111548239f
Status: Downloaded newer image for nginx:1.23
docker.io/library/nginx:1.23
```

5. 退出node, 在Mac上重新检查pod状态, 问题解决🎉🎉🎉:
```shell
davywalkerdeMacBook-Pro:~ davywalker$ k get po
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-67b8c7bdfd-965qn   1/1     Running   0          87s
```

## 为啥叫"ImagePullBackOff"??

---
参见文章 [Kubernetes ImagePullBackOff error: what you need to know](https://www.tutorialworks.com/kubernetes-imagepullbackoff/) 说明.
> The status ImagePullBackOff means that a Pod couldn’t start, 
> because Kubernetes couldn’t pull a container image. 
> The ‘BackOff’ part means that Kubernetes will keep trying to pull the image, 
> with an increasing delay (‘back-off’).

这里的"BackOff"就是"退避"的意思, 当拉取失败时, kubelet应该有个退避算法来重试拉取. 具体退避算法是啥, 有待后续钻研. 🤔

