---
title: 关于JVM DNS Cache问题的研究
date: 2023-03-04 10:59:10
tags: [java, jvm, dns]
---
# 背景
某次jstack发现应用卡在DNS解析上, 发现应用为了做负载均衡, 故意将JVM DNS Cache失效时间设置为了0, 即永不缓存.
这才知道原来还有这个参数. 因此借此机会就查阅资料, 详细研究下.

# Java DNS流程
## 流程详解
当我们调用如下方式解析域名时, 经过了如下流程: 
```java
String hostname = "www.baidu.com";
InetAddress addr = InetAddress.getByName(hostname);
```
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202303041111414.png)

看了上图, 估计会有几个疑问: 
1. 通常域名会对应多个IP, 为啥Java中`InetAddress.getByName`解析只返回一个IP? 具体是怎么实现的?
2. DNS具体实现是怎么做的呢?
带着疑问往下看吧.

## 具体解析实现
具体解析实现, 参见上图的`getaddrinfo`. 该方法是POSIX协议的标准, 由各个操作系统来实现.
[详细点此](https://jvns.ca/blog/2022/02/23/getaddrinfo-is-kind-of-weird/)

getaddrinfo is part of a library called libc which is the standard C library. 
There are at least 3 versions of libc:
- glibc (GNU libc)
- musl libc
- the Mac OS version of libc (I don’t know if this has a name)

真正的实现就更多啦. 有些在 `getaddrinfo` 里有缓存, 有些没有, 这点要依据自己的平台注意研究.
之所以我们设置, 是因为线上使用了自研的dnsClient, 相当于重写了`getaddrinfo`, 是没有缓存的. 因此JVM设置缓存未0也就能生效.  

## 重要参数
注意上图中, "in cache?" 代表的是域名对应的IP是否已经在JVM的缓存中了. 
既然涉及到缓存, 就必然涉及到失效时间的问题. 因此JVM提供了如下2个参数:

- networkaddress.cache.ttl (default: -1): 代表DNS解析成功时, hostName->ip 的缓存失效时间. 默认-1代表永不失效. 
- networkaddress.cache.negative.ttl (default: 10): 代表DNS解析失败时, hostName->空缓存 的失效时间. 默认为10s. 就是为了做空查询保护. 

## 如何设置上述参数
### 方案1: JVM启动参数 
在应用启动时, 设置启动参数: 
`-Dsun.net.inetaddr.ttl=0 -Dsun.net.inetaddr.negative.ttl=0`

### 方案2: 应用启动后设置变量
在应用启动后, 设置参数: (注意参数名与启动参数名称不同)
```java
java.security.Security.setProperty("networkaddress.cache.ttl" , "0");
java.security.Security.setProperty("networkaddress.cache.negative.ttl" , "0");
```

### 方案3: 设置JVM配置
编辑`$JRE_HOME/lib/security/java.security` 文件, 增加如下配置: 
```java
networkaddress.cache.ttl = 0
networkaddress.cache.negative.ttl = 0
```

# 问题重现
为了模拟重现当时网络不稳定的情况, 设置某个不存在的域名, 这样解析需要花时间. 
- [测试代码](https://github.com/DavyJones2010/test-core/blob/master/src/test/java/edu/xmu/test/dns/DnsCachingTest.java)

## 不禁用negative
发现大部分线程卡在访问DNS缓存上, 达不到重现的效果:
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202303041108536.png)

## 禁用negative
这下就重现的当时的情况, 大部分线程都卡在真正解析域名上(或者叫`getaddrinfo`系统调用上): 
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202303041147157.png)

# 其他
## 关于Java DNS解析实现的吐槽
关于Java域名解析问题, 即`InetAddress.getByName()`实现, 发现了很有趣的一篇小文: 
[Java’s DNS resolution is so 90ies! Java的DNS解析实现仍然停留在90年代!](https://blog.bmarwell.de/2020/09/23/javas-dns-resolution-is-so-90ies.html)

### 存在3个问题 
1. 首先, 虽然DNS会返回多个IP, 但`InetAddress.getByName()`只会返回第一个IP. 
2. 其次, 返回的IP并且不会保证这个IP能不能连通.
3. 最后, `InetAddress` 所有实现方法都是`non-public`的, 导致扩展/修复极为困难!
> Wow, that is remarkably simple! How do we know that this IP will be reachable? Well, we do not! If there are more IPs in the answer section, they are just being ignored.

### 作者的解决方案
1. 通过JavaAgent解决了`InetAddress`难以扩展的问题
2. 通过类似python中的方案, DNS获取到IpList之后, 会尝试连接ip 3次(timeout 100ms), 直到获取可以连通的ip, 再返回.
3. 详细代码参见: [bmarwell/nameserviceagent]https://github.com/bmarwell/nameserviceagent
我就喜欢这种能喷能干的实干家:)

## 关于配置JVM DNS缓存生效情况问题
1. 实际在Mac上设置JVM DNS缓存都为0, 发现并不生效. 例如`www.baidu.com`其实返回了2个IP, 但测试代码一直返回1个.
这是由于 Mac OS has DNS caching, 即把上述 `getaddrinfo` 的系统调用也进行了缓存. 

2. 而Linux上 `getaddrinfo` 不一定会有缓存, 除非使用`systemd-resolved`等工具
> Linux doesn’t necessarily unless you use systemd-resolved or something

所以在配置JVM DNS参数前, 也请务必确认好自己的OS环境以及各自实现.

# Refs
- [Host name resolution in Java](https://maheshsenniappan.medium.com/host-name-resolution-in-java-80301fea465a)
- [How to make Java honor the DNS Caching Timeout?](https://stackoverflow.com/questions/1256556/how-to-make-java-honor-the-dns-caching-timeout)
- [Some things about getaddrinfo that surprised me](https://jvns.ca/blog/2022/02/23/getaddrinfo-is-kind-of-weird/)