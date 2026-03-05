---
title: '瘦客户端使用陷阱'
date: 2023-11-07 00:00:00
tags: ['thin-client', 'lessons-learned', 'architecture']
category: tech-notes
---

# 20231107-thin-client-pitfall

发现在开发很多问题排查&提效工具时, 服务端同学仍然陷入定势思维. 强调服务端优先, 瘦客户端. 从而导致整体实现过为臃肿, 开发&改造周期长, 性能不太好等问题. 

但个人思考, 要跳出定势思维与舒适区, 定义好服务端&客户端的边界. 把很多功能放到 前端 或者 与Alfred整合, 可以很好地解决上边的问题. 

插件&工具 经常会处于鄙视链底端, 大家都想要做平台, 把饼画得大大的. 

但实际上好的插件&工具, . 例如, 支付宝是不是个工具? 微信其实本质也是个工具. 为啥会觉得做工具Low呢?  

举几个例子: 

1. Tracing 系统, 关键词过滤

N多个上下游系统, 通过traceId(UUID)把请求关联. 建设了一个Tracing系统. 具体实现是: 

想要输入某个traceId之后, 会把traceId拼接成SLS SQL, 请求服务端→请求SLS获取结果.

如果想要基于traceId再添加其他关键词进行进一步筛选: 

- 目前实现: 输入其他关键词后, 继续拼接SLS SQL, 请求服务端→请求SLS获取结果. 太耗时了, 很容易把使用者的耐心消耗掉. 尤其是在需要频繁修改附加关键词的时候.
- 个人建议: 把关键词筛选直接放在前端, 前端有很多很好用的搜索框架. 例如 [Fuse.js | Fuse.js](https://www.fusejs.io/) https://github.com/mattyork/fuzzy 等

1. 多账号登录Chrome插件
2. Alfred插件

---

个人也愈发不满Java的笨重, 上来就是一套JVM+Spring, 还得编译打包. 要开发个web应用就更重了, 还得引入servelet容器. 

运行时内存overhead也不小. 

虽然SpringBoot让开发相对简单了. 

但总体来说还是过于臃肿.印象中Java就是一个严肃古板的老学究. 

尤其是跟Go/Python比对. 所以多尝试写写Go/Python吧.

Small is beautiful, small is powerful~