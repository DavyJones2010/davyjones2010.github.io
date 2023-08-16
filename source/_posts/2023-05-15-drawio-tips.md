---
title: draw.io使用技巧总结
date: 2023-05-15 22:24:12
tags: [soft-skills, diagrams, software-engineering, best-practice]
---
# 前言
自从上次发奋要好好学习做图, 做PPT, 数据可视化之后. 近两个月对draw.io进行了深度的使用.
学习到了很多技巧, 使用起来更加得心应手, 也逐渐开始享受画图的过程, 做到了`Happy diagramming!~`
这里把使用的一些心得总结下来, 防止以后遗忘.

{% note info %}
draw.io博大精深, 本文只列出当前自己常用的, 后续逐步完善.
{% endnote %}

# 快捷键
- [删除 Shapes+Connections](https://www.drawio.com/doc/faq/shapes-delete-connections): `ctrl/cmd + Delete`
- 选择所有Edges: `cmd+shift+e`
- 选择所有Shapes/Vertices: `cmd+shift+i`
- Group: `cmd+g`; Ungroup: `cmd+shift+u`
- Bring to Front: `ctrl+shift+f`; Send to Back: `ctrl+shfit+b`
- CopySize: `option+shift+x`  pasteSize: `option+shift+v`
- Duplicate: `cmd+enter` or `cmd+d` --> 比 cmd+c ++ cmd+v 好用多啦!
- Reset view: `enter`  --> 之前画布拖动太多, 导致图形丢失了. 还得拖动bar来找, 很麻烦, 用这个简直提效神器!
- Lock Shape: `cmd+l` --> 在多个图形overlap, 需要勾选其中某些的时候, 可以先把不希望选到的图形lock住
- AutoReset Size: `cmd+shift+y` --> 会把shape的size自动设置为刚好包下里边的Text

# 技巧
## LineJump
{% note info %}
线段交叉时, 用`linejump`, 清晰易读
{% endnote %}

![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202305152320352.png)


## WayPoint
{% note info %}
箭头分叉时, 用`waypoint`
{% endnote %}
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202305152325340.png)


## Container
{% note info %}
多个Shape组合时, 用`container`
{% endnote %}


## Floating & Fixed connectors
参见: [Floating & Fixed connectors](https://www.drawio.com/blog/connectors)
https://www.youtube.com/watch?v=xM04I-WVXlE&ab_channel=draw.io
- Floating connectors: 连接线前后用`o`标记, 默认的都是Floating connectors.
- Fixed connectors: 连接线前后用`x`标记

![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202305162239974.png)

## 调整单个对象的大小
{% note warning %}
调整大小时, 保持中心不动, 是非常重要的, 防止结构被破坏. 
{% endnote %}

- Hold `CMD`, 然后拖动对象大小, 可以保持对象**中心不动**. 
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202305182305893.gif)

- Hode `CMD+Shift`, 然后拖动对象大小, 可以保持对象中心不动, 且对象比例等比缩放. --> 非常好用!
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202305182306535.gif)

- Copy&Paste Size, 快捷键 copySize: `option+shift+x`; pasteSize: `option+shift+v`
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202305190001415.png)

## 同时选择多个对象
- 按住 `Option` 然后拖选, 只要触碰到的, 都会被选到(不用把整个对象都包含进去) --> 解救人类!
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202305190008608.gif)

- 但貌似有的时候行为不一致, 按下`Option`然后拖选的行为是防止点选到外部的容器/Shape


## 同时调整多个对象的大小
1. 使用分组
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202305172248220.gif)

2. 多选, Arrange里调整
[Scaling a group of shapes in Diagramly](https://webapps.stackexchange.com/questions/52230/scaling-a-group-of-shapes-in-diagramly)

3. 使用容器
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202305172256303.gif)

{% note warning %}
善用 **分组** 与 **容器**
{% endnote %}

## 移动对象
- 按住 `Shift+CMD` 移动, 把对象限制在水平&垂直方向移动(强约束)
- 按住 `Shift` 移动, 可以将对象尽量在水平/垂直方向移动(弱约束, 粘性)
- 防止移动的对象被放入到container里: 拖动对象, 然后按住option
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202308162219643.gif)

## 旋转Shape里的Text
- 经常会遇到子图形覆盖掉Parent的Text, 导致可读性很差. 此时可以旋转Parent的 Text 位置到左侧/右侧, 如下样例: 
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202308162232737.png)
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202308162231340.gif)

## 先决定布局, 再决定风格
先把布局决定好, 之后统一调整风格(颜色, 大小, 样式)会容易一些.

# 风格
## 泳道图
最好每个泳道设定背景色(LaneColor), 然后里边的各个Task背景白色, 能突出重点, 美观. 对比下: 
- 未设定Lane Color
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202305190018059.png)

- 设定了Lane Color
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202305190017534.png)


# Refs
{% note info %}
draw.io[官网文档](https://drawio-app.com/blog/)清晰易懂, 是个真正的好产品!
{% endnote %}

