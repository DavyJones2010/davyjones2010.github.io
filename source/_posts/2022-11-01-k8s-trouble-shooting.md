---
title: K8s部署使用错误排查
date: 2022-11-01 21:45:41
tags: [k8s, kubectl, trouble-shooting]
---


https://github.com/docker/for-mac/issues/1317

```shell
  Normal   Pulling    8s               kubelet            Pulling image "docker.io/nginx:1.23"
  Warning  Failed     8s               kubelet            Failed to pull image "docker.io/nginx:1.23": rpc error: code = Unknown desc = Error response from daemon: Get "https://registry-1.docker.io/v2/": dial tcp: lookup registry-1.docker.io on 192.168.64.1:53: server misbehaving
  Warning  Failed     8s               kubelet            Error: ErrImagePull
  Normal   BackOff    6s (x2 over 7s)  kubelet            Back-off pulling image "docker.io/nginx:1.23"
  Warning  Failed     6s (x2 over 7s)  kubelet            Error: ImagePullBackOff
```




