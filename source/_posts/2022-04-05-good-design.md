---
layout: post
title: 记录一些有趣的产品设计
tags: [product-design, good-design]
lang: zh
---

# 前言
代码, 架构, 逻辑, 产品, 设计等, 不应该是枯燥乏味的, 不应该是痛苦复杂的, 而应该是有趣的, 简单的, 应该是geek的, 应该是美的.
这里记录下自己的点滴发现, 希望能走出CRUD与纷杂的业务逻辑, 逐渐提高一点审美与品味.
甚至觉得在软件产品的世界里，不应该由呆板无味的程序员主导，应该由产品经理，由用研，由UX等负责。
用户是知道什么好用，什么不好用的。知道哪些有趣，哪些无趣。
Mac字体最开始也是因为Steve Jobs在大学选修了这门课。
越来越意识到，软件的设计，需要的是克制，减少对用户的打扰，而不是一味增加与堆砌功能，影响体验，后边会附上自己识别到的bad design，引以为戒。
# changelog优雅的格式
- [taurus](https://gettaurus.org/docs/Changelog2020/#1-15-1sup-30-Oct-2020-sup)
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208071142374.png)


# 引导用户付费阅读
- manning的[docker-in-practice-second-edition](https://livebook.manning.com/book/docker-in-practice-second-edition/chapter-4/13), 不会强制用户点击付费才能看见付费内容, 而是用很巧妙的方法, 对内容进行模糊化
- 可以付出不同的价格, 来解锁 代码区域->小章节->大章节->整本书
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208071143420.png)
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208071143350.png)

# 输入密码
- 输入用户名时, 睁开眼睛
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208071144071.png)


- 输入密码时, 闭上眼睛
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208071144365.png)

# Github不同的页面风格
- 可以选择不同的filter, 虽然没啥卵用, 但感觉非常geek, 很有意思
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208071145971.png)

# RGB颜色的中文名称
发现Mac上截图软件 [iShot](https://apps.apple.com/us/app/ishot-screenshot-recording-ocr/id1485844094?mt=12) 会识别到截取图中鼠标当前位置的RGB值，然后给出很好的中文
中文各种颜色的翻译很有诗意, 例如:   
- 蓝色: 蔚蓝, 碧蓝, 天蓝, 鼠尾草蓝, 矢车菊蓝, 灰丁宁蓝, 土耳其蓝, 绿松石色, 灰蓝, 军服蓝, 暗岩蓝, 品蓝, 齐马蓝, 爱丽丝蓝
- 紫色: 锦葵紫, 蓝紫, 暗蓝紫, 紫水晶色, 木槿紫, 紫丁香色, 薰衣草紫, 淡紫丁香色, 蓟紫

个人找到的能对应到的RGB名称映射: [RGB颜色参考](https://www.sojson.com/rgb.html) ,遗憾的是:
1. 只有英文, 没有对应中文的名称, 诗意很难体现出来.  
2. 英文也不是很贴切, 例如 VioletRed1(#FF3E96), VioletRed2(#EE3A8C), VioletRed3(#CD3278), VioletRed4(#8B2252), 分别对应中文的 "暖粉红", "山茶红", "樱桃红", "褐色"

# Emoji的灵活运用
灵活运用Emoji, 让信息更加直观友好好玩儿. Life is short, why so serious?

- 例如 [minikube](https://minikube.sigs.k8s.io/docs/start/) 的启动console界面: 
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202211012118263.png)

- 例如 [EC2-Jenkins的ReleaseNotes](https://github.com/jenkinsci/ec2-plugin/releases): 
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202211012117440.png)

