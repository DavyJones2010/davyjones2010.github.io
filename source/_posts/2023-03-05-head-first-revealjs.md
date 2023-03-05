---
title: revealjs制作PPT笔记
date: 2023-03-05 17:28:22
tags: [tools, ppt, markdown]
categories: 工程提效工具
---

# 背景
最近在B站学习蒋炎岩老师的OS: 

- 内容自是不必多说, 干货满满.
- 对课件PPT更是十分欣赏. 
- [实际样例](https://jyywiki.cn/OS/2022/slides/3.slides#/)

## 特点1: 导航清晰
风格简约专业:

- 横向导航区分大类
- 纵向导航来区分小类

![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202303052225650.png)

## 特点2: 管理轻量
- 基于代码生成与管理(支持Markdown), 不使用笨重的Microsoft PPT. 
- 可以使用git进行版本管理(例如showcase中本文就会转成ppt). 


## 实现方式
> revealjs + pandoc 组合神器

- [revealjs](https://revealjs.com/markdown/)
- [pandoc](https://pandoc.org/installing.html)

刚好自己平时要做PPT, 但作为程序员, 又十分抵触内容与样式不分离的方式, 制作起来也耗时.
借鉴这种方案, 轻松写出专业且简约的PPT.

# 安装&配置

## 安装
- revealjs: [安装手册](https://revealjs.com/installation/#full-setup)
- pandoc: 把markdown转成revealjs的html, [安装手册](https://pandoc.org/installing.html)
- [tailwind css](https://tailwindcss.com/): JYY老师的, 个人还在研究中 

## revealjs个人配置
- Enable [Slide Numbers](https://revealjs.com/slide-numbers/)
- Disable [Vertical centering of slides](https://revealjs.com/config/)
- Disable [controlsTutorial](https://revealjs.com/config/)

# 实践
## 命令

- revealjs
```shell
# 默认8000端口
npm start
# 指定8001端口
npm start -- --port=8001
```

- pandoc
```shell
# md to html. 注意需要设置 --slide-level=2 参数. 把一级标题设置horizontal大类, 二级标题内容为vertical子类. 
pandoc id-token-in-wf.md -o id-token-in-wf.html -t revealjs -s -V theme=white --slide-level=2
```

## 快捷键
- `o` 进入缩略图模式.
- `b` 屏蔽当前PPT
- `f` 进入全屏模式
- `ctrl+shift+f`: 全局搜索

# showcase

## self-explained
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202303052309110.png)

# TODO

## 样式优化
与JYY老师相比, 页面还是过于简陋

- Header: 类似markdown的分隔符
- Image: 图片有些显示不全❗
- Align: 字默认居中, 需要<font color='red'>**居左**</font>
- Syntax Highlighting: 现在**太丑**
- Citation/Quote: 也**太丑**

## 参数生成
pandoc转html时, 如何支持配置revealjs init参数? 
