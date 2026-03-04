---
title: docker deepdive
date: 2024-02-20 00:00:00
tags: [docker, container, architecture, deep-dive]
---

# docker-deepdive

# architecture

参见官方文档: 

![Untitled](docker-deepdive/Untitled.png)

- 其中 `docker daemon` 默认是通过 uds 来listen本地的client请求, 优点是权限好控制. 只允许 **root 用户**在**本机访问**

<aside>
💡 - By default, the Docker daemon listens for connections on a Unix socket to accept requests from local clients.
- By default, it will listen on `unix:///var/run/docker.sock` to allow **only local connections by the *root* user**.

</aside>

- 也可以配置按照TCP方式listen, 从而允许远程来访问, 详细参见: [dockerd | Docker Docs](https://docs.docker.com/engine/reference/commandline/dockerd/#bind-docker-to-another-hostport-or-a-unix-socket) 但需要注意安全风险

# docker.sock

之前不太理解为啥docker要监听在: `/run/docker.sock`  这个地址上? 对[uds有了完整的实践与测试](go-network-socket%20a9506a8dfda64f9aab81c62f89215379.md)之后, 对此有了透彻的认知. 

从而也理解了为啥 [如何创建Logtail配置并以DaemonSet方式采集容器文本日志\_日志服务(SLS)-阿里云帮助中心](https://help.aliyun.com/zh/sls/user-guide/use-the-log-service-console-to-collect-container-text-logs-in-daemonset-mode?spm=a2c4g.11186623.0.0.3b131a21Bt0peR) 中, 需要按照 `Logtail通过/run/docker.sock访问Docker，请确保该路径存在且具备访问权限。` 方式来收集日志. 

# image build

镜像构建流程

> Docker 镜像构建实际上是在 Docker 守护进程（Docker Daemon）上执行的。当你运行 `docker build` 命令时，构建过程中的所有操作都是由 Docker 守护进程在服务器的沙箱环境中进行的。这个沙箱环境被称作构建上下文（build context）。
> 
> 
> 这里有一些详细步骤：
> 
> 1. **构建上下文（Build Context）传递：**
> 当你执行 `docker build` 命令时，当前目录（或你指定的上下文目录）下的所有内容都会被发送给 Docker 守护进程。这就是为什么通常在最后加上一个点 `.` 来指定当前目录作为上下文。上下文中包含了 `Dockerfile` 和所有可能被 `Dockerfile` 中的指令（如 `COPY`, `ADD`）引用的文件和目录。
> 2. **构建过程：**
> Docker 守护进程接收到构建上下文后，会开始解析 `Dockerfile` 并逐步执行里面的指令。每执行一个指令，实际上 Docker 守护进程都会创建一个临时的容器来运行这些指令，并捕获其变更作为新的镜像层。当执行完成后，这个临时容器会被删除，但是新的镜像层会被保留。—> 在执行`COPY`命令时，Docker不会创建一个临时的容器来执行这条命令。 Docker在处理不同类型的Dockerfile指令时，采用不同的策略：
> - 对于`COPY`和`ADD`等涉及文件复制的操作，Docker直接在镜像层上执行，不需要创建容器。
> - 对于`RUN`指令，需要在容器环境中运行命令，因此Docker会创建一个临时容器来执行这些指令，并捕获其变更作为新的镜像层。
> 
> 1. **镜像层的创建：**
> 每个 `Dockerfile` 指令都可能创建一个新的镜像层。例如，`RUN` 指令会运行一条命令并创建一个新层，`COPY` 或 `ADD` 指令会将文件复制到镜像并创建新层。所有这些镜像层加起来就形成了最终的 Docker 镜像。
> 2. **运行于主机上：**
> 虽然 Docker 守护进程可以在远程服务器上运行，但是通常情况下，它是在你的本地机器上作为一个后台服务运行的（如果你在本地机器上使用 Docker 的话）。这意味着所有构建操作都在你的机器上发生，只要你的机器上安装了 Docker。
> 3. **隔离性：**
> Docker 使用容器技术来保证构建过程的隔离性，确保每一次构建都在干净的环境中开始，并且构建过程中的变更不会污染宿主机或其他构建。
> 
> 构建完成后，新创建的镜像会存储在本地 Docker 守护进程的镜像仓库中，你可以使用 `docker images` 查看。如果需要，还可以将其推送到远程镜像仓库，如 Docker Hub。
> 

> 是的，当你使用 `docker build` 命令构建 Docker 镜像时，Docker 守护进程会在构建过程中暂时性地创建和启动容器。这些容器被用来执行 `Dockerfile` 中的指令，并捕获这些指令执行后的文件系统变更，从而创建新的镜像层。
> 
> 
> 每个 `RUN`, `COPY`, `ADD` 指令等都会创建一个新的层，而且每个 `RUN` 指令实际上都会在一个新的临时容器内执行。例如，如果你的 `Dockerfile` 中有一个 `RUN apt-get update && apt-get install -y some-package` 指令，Docker 会：
> 
> 1. 从当前镜像创建一个新的临时容器。
> 2. 在这个临时容器中执行 `apt-get update && apt-get install -y some-package` 命令。
> 3. 捕获执行后的文件系统变更。
> 4. 提交这些变更作为新的镜像层。
> 5. 删除这个临时容器。
> 
> 整个构建过程中可能会启动多个这样的临时容器，每个 `RUN` 指令都会创建一个。这些容器在执行完相应的 `Dockerfile` 指令后就会被删除，但是它们的影响（即新的镜像层）会被保存下来以构建最终的镜像。
> 
> 这种方式保证了构建过程的每一步都是在干净且隔离的环境中进行的，避免了不同构建步骤之间的潜在冲突，并且确保了构建结果的可重复性。
> 

难道这就是为啥dockerfile的第一行必须是 FROM xxx 的原因? docker daemon会依据该镜像临时创建一个容器

> 当你执行 `docker build` 命令时，Docker 客户端（Client）会将构建上下文发送到 Docker 守护进程（Daemon）。这一步骤是通过以下方式实现的：
> 
> 1. **打包构建上下文：**
> Docker 客户端会将指定的上下文目录（通常是当前目录，除非你指定了其他目录）中的所有内容递归地打包成一个临时的 tar 归档文件。这个归档文件包括了 `Dockerfile` 以及任何可能在构建过程中被 `Dockerfile` 中的指令引用的文件和目录。
> 2. **发送归档文件：**
> Docker 客户端通过与 Docker 守护进程的 REST API 通信，将这个 tar 归档文件发送到守护进程。这个过程通常是通过 Unix 套接字（在 Linux 系统上）或者 TCP 套接字（如果配置了远程访问）完成的。
> 3. **展开归档文件：**
> Docker 守护进程接收到 tar 归档文件后，会在服务器端将其展开到一个临时的目录中。这允许守护进程访问所有文件和目录，这些文件和目录将作为构建过程的一部分。
> 4. **执行 `Dockerfile` 指令：**
> 展开构建上下文后，Docker 守护进程开始按顺序执行 `Dockerfile` 中的指令来构建最终的 Docker 镜像。守护进程会在服务器端的临时目录中操作，同时对相应的文件和目录进行必要的修改。
> 
> 以上步骤即完成了将构建上下文从 Docker 客户端发送到 Docker 守护进程的过程。这是为什么在运行 `docker build` 时，通常会看到 "Sending build context to Docker daemon" 之类的消息输出。
> 
> 为了优化这个过程，通常建议在 `.dockerignore` 文件中列出不需要包括在构建上下文中的文件和目录，类似于 `.gitignore` 文件。这样可以减小 tar 归档的大小，提高构建效率，并且防止不必要的文件被发送到 Docker 守护进程。
>