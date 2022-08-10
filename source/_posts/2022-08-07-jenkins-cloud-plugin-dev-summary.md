---
title: Jenkins Alibaba Cloud 插件开发总结
date: 2022-08-07 22:31:55
categories: 技术 
tags: [jenkins, ci, ci/cd]
---

# Cloud框架的架构与概念

--- 
## 架构
### ER图
```mermaid
erDiagram
    Jenkins ||--o{ hudson_slaves_Cloud : contains
    hudson_slaves_Cloud {
        id name
    }
    hudson_slaves_Cloud ||--|{ SlaveTemplate : contains
    SlaveTemplate {
        id templateName
    }
    SlaveTemplate ||--o{ AlibabaEcsSpotFollower : "provision"
    AlibabaEcsSpotFollower {
        id ecsInstanceId
    }
    AlibabaEcsSpotFollower ||--|| SlaveComputer : createComputer
    SlaveComputer {
        id 
    }
```

### 各Entity关联关系

// TODO: 待补充完善.
```java
Jenkins.get();

```

## 概念
## noDelayProvisioning
- 当任务队列里有任务时, 自动会在 instanceFloor与instanceCap 之间进行弹性创建node.

// TODO: 待补充完善.


# 其他重要信息

对象序列化后XML文件路径:
```shell
cat ${proj_path}/alibabacloud-ecs-plugin/work/config.xml
```

# 插件开发的经验

---

## 规格选择
在规格选择栏, 设计的时候为了简便用户选择:
1. 下拉框方式, 方便不熟悉ECS的用户选择
2. 默认使用2C8G, 即独享型最小规格. 从而减少下拉框规格量, 便于入门.

但在实际企业级场景下, 用户
1. 需要有大规格, 基本都是48C以上, 即 12xlarge, 16xlarge, 24xlarge 的规格
2. 本身使用ECS, 对ECS规格熟悉, 有目标规格.

## SKU选择
1. 单个CloudProvider, 只支持一个SKU, 数量可以选择多个.
但在实际场景下:
1. 用户希望在多可用区, 多规格, 这样需要重复逐一配置CloudProvider, 非常麻烦.

## 系统盘类型选择
在系统盘选择项上, 设计的时候为了简便用户上手:
1. 不需要用户填入系统盘类型, 创建ECS(RunInstances)时不传入系统盘类型&大小, 从而让ECS使用的默认值, 即cloud_efficiency
2. 不需要用户填入系统盘大小, 即使用默认的20GB

但在实际场景下, 用户:

1. 选择ecs.c7.xxx规格, 7代规格不支持cloud_efficiency, 只支持cloud_essd, 即RunInstances接口必须传入cloud_essd系统盘类型

## 数据盘类型
在数据盘选项上, 为了用户上手方便:
1. 默认不创建&挂载系统盘, 因此不需要用户选择数据盘类型&数据盘大小
但在实际场景下, 用户:
1. 会有全镜像, 即镜像中既包含系统盘, 又包含数据盘

## Master与Slave通信方式
JenkinsMaster与Slave的联通方式, 设计的时候为了方便应对用户Master在云上/其他云上/云下等场景, 默认为Slave创建公网IP, 从而Master通过公网IP与Slave联通.
但在实际场景下:
1. 创建公网IP会导致频繁的攻击.
```java
【XXX】尊敬的xxxx：
云安全中心检测到恶意XX.XX.xxx.xxx正在尝试攻击您的服务器：XX.XX.xxx.xxx（XX-XX...），已为您创建IP拦截策略，并成功拦截该恶意IP，建议您登录云安全中心控制台安全告警页中查看IP拦截策略。若您需要放行该 IP ，您可以在IP 拦截策略中禁用安全策略。
```
2. 用户本身VPC就是在阿里云上, 通过公网连接, 会产生额外的费用.

