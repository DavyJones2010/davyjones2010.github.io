---
layout: post
title: 一个通用数据巡检框架的设计
tags: [java, software-engineering, best-practice, cruiser, software-design, arch]
---


# 背景
在实际项目中, 有许多业务场景, 会存在多张表数据不一致或者脏数据残留的问题, 
需要定期执行一些SQL来进行数据一致性的巡检与订正, 以实现数据的最终一致性.
例如下表, 需要定期查询&统计出'deleted'状态的用户数量, 报警, 并清理这些数据.
```sql
-- user表
user_id| age| status|
1| 18| deleted|
```


# 初步方案: 
方案1: 可以针对每个需求, 单独写SQL->DAO->Service等, 但带来的问题是: 
  1. 开发量大, 每次新的需求, 都需要撸一遍, 重复工作量大.
  2. SQL->Service整体功能分散, 不便于维护.
方案2: 可以基于此, 抽象出一个面向SQL的通用巡检框架, 便于统一接入与维护.

# 架构设计思考
从用户角度, 可以:
定时任务框架: 
1. 创建巡检任务, 与输入自定义的SQL绑定
2. 定义巡检周期
3. 手动触发运行(便于首次配置之后调试)
4. 查看运行记录(便于)

报警与订正框架: 
1. 定义任务结果筛选与报警条件
2. 定义任务结果执行的订正Action


## 第一部分: 定时任务框架
核心就是一个面向SQL的分布式定时任务框架, 调研现有的任务框架实现(例如Quartz): 

- 任务核心: 
  - 任务描述与定义(即数据源支持哪些? SQL支持哪些?)
  - 定时任务的调度与执行
  - 历史任务运行情况
    - 耗时
    - WorkerIp
    - 执行状态
- 运维后台: 
  - 手工触发
    - 指定
  - 任务

## 第二部分: 数据订正报警框架



# 详细设计
任务分为 "全量巡检" 与 "增量巡检" 两类. 
全量巡检: 即每次执行SQL, 对报警结果全量输出. 
增量巡检: 即每次执行SQL, 只输出与上次执行有差异的(新增的)数据报警.
