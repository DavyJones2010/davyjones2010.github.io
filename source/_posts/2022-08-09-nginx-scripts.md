---
title: 常用的Nginx命令与配置
date: 2022-08-09 21:42:45
tags: [nginx, network, linux, mac]
---

# MAC环境
## 安装
```shell
brew install nginx
```

## 内容
```shell
/usr/local/var/www
/usr/local/Cellar/nginx/1.21.3/html
```

## 配置
### 配置文件路径
```shell
/usr/local/etc/nginx
/usr/local/etc/nginx/nginx.conf
```

### 禁止通过IP访问
- 绑定了hosts
```shell
127.0.0.1 example.org www.example.org
```
- 只允许通过`localhost:8080`或者`example.org:8080`或者`www.example.org:8080`访问, 
- 禁止通过`127.0.0.1:8080`访问, 返回502错误.
```shell
    server {
        listen  8080;
        return 502;
    }

    server {
        listen       8080;
        server_name  localhost example.org www.example.org;
    }
```

![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208092149192.png)

### 单Host多域名, 不同域名访问不同服务
- 绑定了hosts
```shell
127.0.0.1 example.org www.example.org
```
- 通过`localhost:8080`访问, 则访问到 `$NGINX_HOME/html/` 目录
- 通过`example.org:8080`或者`www.example.org:8080`访问, 则访问到 `$NGINX_HOME/html2/` 目录
- 禁止通过`127.0.0.1:8080`访问, 返回502错误.
```shell
    server {
        listen  8080;
        return 502;
    }
    server {
        listen       8080;
        server_name  example.org www.example.org;
        location / {
            root html2;
            index index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html2;
        }
    }
    server {
        listen       8080;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
```
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208092152127.png)
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208092152948.png)

## 环境变量
NGINX_HOME:
```shell
/usr/local/Cellar/nginx/1.21.3
location / {
root   html;
index  index.html index.htm;
}
```
上边的html就是 NGINX_HOME/html/

## 服务
```shell
brew services start nginx
brew services restart nginx
brew services stop nginx
```

## 其他
### Nginx是如何知道本次请求, 请求的是IP还是域名? 如果是域名的话, 多个域名如何区分?
- 如下图, 本质上还是Nginx解析HTTP协议, 根据请求行里的`RequestURL`来进行判断. 
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208092154665.png)
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208092154173.png)