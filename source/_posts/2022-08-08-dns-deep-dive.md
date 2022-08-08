---
title: 深入探究DNS流程与报文
date: 2022-08-08 22:26:58
tags: [network, linux, dns]
---
# DNS服务器类型
## root nameserver
即负责.域名的, 全球只有13台(至于为啥只有13台, 自己google吧):  
```shell
davywalkerdeMBP:_assets davywalker$ dig baidu.com +trace

; <<>> DiG 9.10.6 <<>> baidu.com +trace
;; global options: +cmd
.			1	IN	NS	m.root-servers.net.
.			1	IN	NS	a.root-servers.net.
.			1	IN	NS	h.root-servers.net.
.			1	IN	NS	l.root-servers.net.
.			1	IN	NS	i.root-servers.net.
.			1	IN	NS	g.root-servers.net.
.			1	IN	NS	j.root-servers.net.
.			1	IN	NS	c.root-servers.net.
.			1	IN	NS	k.root-servers.net.
.			1	IN	NS	e.root-servers.net.
.			1	IN	NS	f.root-servers.net.
.			1	IN	NS	b.root-servers.net.
.			1	IN	NS	d.root-servers.net.
;; Received 239 bytes from 192.168.1.1#53(192.168.1.1) in 7 ms
```

## TLD(Top Level Domain) nameserver
即对应 `.com`, `.gov`, `.cn` 等的解析服务器. 


## authoritative nameserver
即对应 `.baidu.com` `.hangzhou.gov`, `gitee.cn` 等的解析服务器. 
通常 authoritative nameserver 是DNS解析的最后一步
> The authoritative nameserver is usually the resolver’s last step in the journey for an IP address.

但也不是必须的, 取决于authoritative nameserver对应的子域名配置方式: 
- 如果
- (但也可以继续往下走, 具体取决于).


# DNS查询类型
## 递归式(Recursive Query)
- DNS请求: 如下, 是个请求中的标识位, 使用dig命令, 如果`+trace`则请求自动禁用递归(即如下递归标识设置为false).
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208082351183.png)

- DNS响应: 如下, 代表当前LocalDNS服务器支持递归查询. 与此形成鲜明对比的是ROOT根DNS服务器返回不支持递归查询.
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208082352908.png)
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208082353697.png)

## 迭代式(Iterative Query)


# WireShark抓包实战

## 简单的顶级域名DNS抓包
### 命令
```shell
# 就简单地对baidu.com进行DNS抓包
davywalkerdeMBP:_assets davywalker$ dig baidu.com +trace
```

### `+dig`迭代式, WireShark分析
#### 第一步: 向LocalDNS发起请求, 请求获取`.`对应的root nameserver
##### 请求
可以看到: 
- 请求的目标IP是`/etc/resolv.conf`中对应的LocalDNS IP
- 请求是UDP协议, 目标LocalDNS的端口号是53
- 请求内容是: `<ROOT>`, 即根域名DNS; 类型是 `NS`, 即nameserver; 就是请求根域名的nameserver
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208082303614.png)
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208082259231.png)


##### 响应
可以看到: 
- 响应内容里有13个根域名服务器的域名 
- 但注意: <mark>没有返回nameserver的domain对应的IP地址!!!</mark> , 经分析与推测, 由于全球13个根域名服务器的IP是永远不会变的, 因此各个domain对应的IP地址应该是<mark>内置在操作系统中的</mark>. 

![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208082306546.png)


#### 第二步: 向root nameserver请求, 请求获取`.com.`对应的TLD's nameserver
##### 请求
可以看到: 
- 请求的目标IP是`202.12.27.33`, 经分析, 是`m.root-servers.net.`对应的IP地址. 应该是按照某种算法随机选的.
- 请求想要直接从root nameserver中获取到 baidu.com 的A记录
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208082308662.png)

##### 响应
可以看到: 
- 响应内容里有13个`.com.`域名服务器的域名.
- 但注意: <mark>同时在Additional records部分, 把各个域名服务器对应的IP也都以A记录形式返回</mark>. 这个就是所谓的 **[Glue Record](https://blog.csdn.net/dranker/article/details/109754755)**, 试想下, 如果没有返回A记录, 那么如果`.com.`返回的nameserver是`a.com.`, 那么如何获取到这个domain对应的IP? 通过DNS么? 那就无限递归了!
- Additional records中`AAAA`记录, 代表的是各个域名服务器对应的IPV6地址.
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208082315463.png)
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208082321212.png)

#### 第三步: 向TLD's nameserver请求, 请求获取`baidu.com.`对应的authoritative nameserver
##### 请求
可以看到: 
- 请求的目标IP是`192.31.80.30`, 通过翻看上一个Glue Record, 可以知道正是`d.gtld-servers.net: type A, class IN, addr 192.31.80.30`对应的IP.
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208082323511.png)

##### 响应
可以看到: 
- 响应的内容里没有`baidu.com`的A记录信息, 而是一堆的NS记录信息, 以及NS对应的IP, 因此还需要继续往下查.
- 如果是查询顶级域名(例如github.com), 或者二级域名(例如login.github.com)等:  
  - 那么理论上这个时候就可以直接返回A记录了. 即不用再走第四步了. 此时A记录可以是个VIP, 然后根据请求具体的二级域名, 例如`login.github.com`, 通过Nginx等反向代理到对应服务即可.
  - 但实际分析了下, 大部分网站(如下图中`alibaba.com`, `zhihu.com`, `aliyun.com`等), 都是在这里返回自己的`authoritative nameserver`, 自己思考了下原因: 
    - 一是: 因为 TLD's nameserver 通常是由国家或者组织统一管理的, 各个公司如果IP变化, 不好同步到 TLD 中. 而 authoritative nameserver 一般都是各个公司自己管理, 时效性与灵活性都很高. 例如可以给某些二级/三级域名配置不同的IP.  
    - 二是: 因为如果直接A记录注册在TLD上, 那么整个网站的所有二级/三级域名等, 就只能有一个IP入口了. 整体风险就很大了. 如果VIP挂了, 整个网站都不可用了. 如果是不同的二级域名, 分配不同nameserver, 则

![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208082326335.png)

![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208082343058.png)
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208082343720.png)
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208082344089.png)

#### 第四步: 向authoritative nameserver请求, 请求获取`baidu.com.`对应的ip

##### 请求
可以看到:
- 请求的目标IP是`14.215.178.80`, 通过翻看上一个Glue Record, 可以知道正是`ns4.baidu.com: type A, class IN, addr 14.215.178.80`对应的IP.

![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208082345386.png)

##### 响应
可以看到: 
- 正确地返回了`baidu.com`的A记录
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208082347974.png)



### 不加`+dig`递归式, WireShark分析
#### 第一步: 向LocalDNS发起请求, 请求获取`aliyun.com`的IP地址(A记录)
##### 请求
可以看到:
- 请求的目标IP是`/etc/resolv.conf`中对应的LocalDNS IP

![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208090001833.png)

##### 响应
可以看到: 
- LocalDNS直接把DNS的A记录结果返回了. 所有的迭代操作都是LocalDNS执行的了.
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208090002803.png)


# 疑问
## 域名服务器本身IP解析问题
DNS协议返回了根域名服务器的域名, 例如`m.root-servers.net.`, 但实际后续向根域名服务器发起查询TLD域名服务器的请求时, 是需要知道根域名服务器的IP的! 这个IP从哪里获取? 通过WireShark抓包, 发现返回的根域名服务器域名列表里, 没有这些域名对应的IP地址. 难道也是通过DNS解析的么? 这样就涉及到循环.
<mark> Glue Record </mark>

## dig请求能否使用指定"递归式"或者"迭代式"么?
> 切换查询中的 RD（要求递归）位设置。
> 在缺省情况下设置该位，也就是说 dig 正常情形下发送递归查询。
> 当使用查询选项 +nssearch 或 +trace 时，递归自动禁用。


# Refs
- https://www.cloudflare.com/learning/dns/dns-server-types/
- https://www.jianshu.com/p/f6ef04bf6af2