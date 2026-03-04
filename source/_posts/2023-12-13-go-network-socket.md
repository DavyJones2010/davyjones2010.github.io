---
title: go network socket
date: 2023-12-13 00:00:00
tags: [golang, network, socket, http, proxy]
---

# go-network-socket

# go proxy

- 一定要按照下边的方式, 设置好proxy, 不然依赖包下载巨慢!~

[https://github.com/goproxy/goproxy.cn](https://github.com/goproxy/goproxy.cn)

# network

## net&http&mux

[github.com/gorilla/mux](http://github.com/gorilla/mux)

```go
func ListenHttp() error {
	ln, err := net.Listen("tcp", ":8080")
	if err != nil {
		// handle error
		return errors.Wrap(err, "invoke net.Listen failed")
	}
	r := mux.NewRouter()
	r.HandleFunc("/", ProductsHandler)

	return http.Serve(ln, r)
}

func ProductsHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Print("ProductsHandler invoked")
	vars := mux.Vars(r)
	fmt.Print(vars)
	w.WriteHeader(http.StatusOK)
	fmt.Fprintf(w, "Category: %v\n", vars["category"])
}
```

1. 如何stop

## unix domain socket

[Understanding Unix Domain Sockets in Golang - DEV Community](https://dev.to/douglasmakey/understanding-unix-domain-sockets-in-golang-32n8) —> 终于搞懂了!~

- 可以基于unix domain socket来创建普通的Socket Server, 或者HTTP Server
- 想要往 unix domain socket 里发送请求:
    - 命令行方式: curl nc 等, 都需要指定相关的参数, 来说明是往Unix Domain Socket里发送.
    - Client方式:

## unix domain socket —> http server

```go
const socketPath = "/tmp/test.sock"

func ListenUnixSocket() error {
	socket, err := net.Listen("unix", socketPath)
	if err != nil {
		log.Printf("Failed to listen to socket registry")
		return err
	}

	c := make(chan os.Signal, 1)
	signal.Notify(c, os.Interrupt, syscall.SIGTERM)
	go func() {
		<-c
		os.Remove(socketPath)
		os.Exit(1)
	}()

	r := mux.NewRouter()
	r.HandleFunc("/", ProductsHandler)
	r.HandleFunc("/hello", HelloHandler)

	srv := &http.Server{Handler: r}
	err = srv.Serve(socket)
	return err
}
```

## 如何向.sock文件发送http请求?

```bash

curl -s -N --unix-socket /tmp/test.sock http://localhost/
```

- -s: slient mode, Do not show progress meter or error messages.
- -N:
- -unix-socket: (HTTP) Connect through this Unix domain socket, instead of using the network.

```bash

Example:
	curl --unix-socket socket-path https://example.com
```

## 如何向.sock文件发送socket请求?

```bash
echo "I'm a Kungfu Dev" | nc -U /tmp/test-raw.sock
```

- -U: Specifies to use Unix Domain Sockets.

## usage of uds

### docker

Docker 在 Linux 环境下使用 Unix socket 进行本地通信。Docker 客户端和 Docker 守护进程之间的通信是通过 Unix socket 进行的。Docker 守护进程会在 /var/run/docker.sock 路径下创建一个 Unix socket 文件，Docker 客户端可以通过连接到该 socket 文件来与 Docker 守护进程通信。

### why not java?

Java 16 (by 2021) 中才引入了对uds的原生支持 : [Unix-domain sockets have faster,the same system is required](https://openjdk.org/jeps/380#:~:text=Unix%2Ddomain%20sockets%20have%20faster,the%20same%20system%20is%20required.))

## vsock

<aside>
💡

如何基于 vsock 构建 socket/http server? 

</aside>

参见: [Linux VM sockets in Go · Matt Layher](https://mdlayher.com/blog/linux-vm-sockets-in-go/)

## vsock v.s. uds

- vsock 用于 vmm/host 与 guestOS 通信
- uds 用于host上各个进程间通信, 是loopback地址的一种替代, 但更高效.