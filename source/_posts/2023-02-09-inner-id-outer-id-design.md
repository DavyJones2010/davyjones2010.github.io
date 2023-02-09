---
title: 工作流框架中ID与Token使用引发的思考
date: 2023-02-09 22:11:42
tags: [learn-from-failure, java, good-design]
---
# 背景

在使用工作流框架时, 发现token/id字段非常多, 尤其是

- workflow_id
- biz_id

这两个的格式都是UUID, 都是唯一的.
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/18490/1629896347576-a7614293-1ddc-4d13-9f9a-e48248cfac62.png#clientId=u1f1ec6f0-275f-4&from=paste&height=683&id=u52913c82&name=image.png&originHeight=938&originWidth=762&originalType=binary&ratio=1&rotation=0&showTitle=false&size=547248&status=done&style=none&taskId=u9488ccd6-246c-4b06-8e39-5b5ea32309a&title=&width=555)
这两个分别代表啥?

- biz_id是外部传入的业务唯一ID, 客户端排查问题时可以使用. 作为外部主键
    - 一个biz_id代表一次业务请求, 可能会对应多个workflow_id;
    - 例如 创建ECS实例的一次请求, 对应一个biz_id(request_id), 但会对应多个workflow的编排, 因此不能把biz_id作为唯一键(会导致workflow_instance唯一键冲突), 也不能直接使用workflow_id(会导致无法trace到整个请求)
- workflow_id是系统内部生成的系统唯一ID, 可以认为是唯一主键. 作为内部主键

# 几种方案

为什么不把biz_id与workflow_id合并成一个? 可以假设下述几种方案:

## 方案1: 外部不传入biz_id, 只使用workflow_id

- 存在一个先后依赖的问题: 如果在workflow_id生成前出现了问题, 根据哪个来定位这次请求, 该怎么排查?
- 无法实现biz_id对应多个workflow_id的一对多关系

## 方案2: 外部传入biz_id, 内部不生成新的workflow_id:

- 外部biz_id如果生成的不可靠, 有重复, 会导致workflow工作机制产生问题.
- 不能信任任何外部传入的ID作为内部的主键ID

所以设计两个ID也是合理的, 一个内部ID, 一个外部ID, 甚至可以认为是最佳实践.

## 方案3: 外部传入biz_id, 内部同时也生成workflow_id

- 即实现上最终选用的方案
- 当biz_id与workflow_id是一对一场景下, biz_id除了方便进行客户端定位问题, 也可以作为client_token实现幂等

# 实际应用场景

大部分都是使用clientToken用来实现幂等

- [ECS SDK调用](https://next.api.aliyun.com/document/Ecs/2014-05-26/RunInstances):
    - 支持方案1, 即客户端不传入任何requestID信息, 请求完成后会返回内部生成的requestID.
    - 也支持方案3, 即客户端传入clientToken, 请求完成后会返回内部生成的requestID.
- [AWS SDK调用](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/Run_Instance_Idempotency.html):
    - 同样支持方案1与方案3;