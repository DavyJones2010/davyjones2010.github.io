---
title: 个人书籍管理工作流总结
date: 2023-05-09 23:37:37
tags: [soft-skills, books, best-practice]
---

# 背景
个人比较喜欢看各种书(杂书居多, 😄), 手头的电子设备也巨多(数了下, 居然有9个之多🤔). 经常发现想要看某本书, 但这本书却在另外一个设备上, 没有同步过来. 
而那个设备却不在手头, 如同尿急的人找不到尿壶, 着实令人沮丧😭.
因此通过本篇总结, 将书本的管理进行系统的梳理, 形成一个惯例(convention), 牢牢遵循, 以解决上边的问题. 

{% note warning %}
整体策略就是以坚果云为核心, 以zlib为主要的下载源, 以epub和pdf为主要的格式, 统一进行管理. 
{% endnote %}

# 小说类管理方式
- 特点: 非技术类或者半技术类; 无需记录太多笔记; 基本无需进行实操; 无需在工作电脑中查看.
- 限制: **必须使用epub格式.** mobi格式只能在kindle中看, 在mac上, ipad上, 手机上, 都无法解析;
- 管理流程如下:
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202305110015750.png)

# 技术类管理方式
- 特点: 技术类; 需要记录很多笔记; 常常需要实操; 需要在工作电脑中查看.
- 限制: 通常都是pdf格式. 但由于工作电脑无法安装坚果云, 因此书籍只能通过钉钉传输, 且只能单向传输. 
- 管理流程如下:
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202305100008704.png)

# 后续事项
1. 笔记同步方案: 当前只梳理了如何把书籍同步到各个设备, 但实际上各个设备上的也记录了诸多笔记, 这些笔记如何同步管理也是令人头疼. 初步想法是通过[notion](https://www.notion.so/)来搞, 把书单也一起维护起来, 但也是个不小的工程.
2. `send to kindle`下线风险: 虽然可以通过邮件`send to kindle`, 但**[2024 年 6 月 30 日之后，用户将无法使用“发送至 Kindle”功能，也就是无法再通过邮箱等方式将电子书推送到 Kindle。](https://bookfere.com/post/985.html)** 还是要想好迁移方案. 
总之, 先这样吧, 一个问题一个问题来解决.
3. 文件格式问题: 很多半技术类的书籍, 都是pdf格式, 压根没有epub. 但pdf在手机&kindle上查看简直是灾难. 没想好咋解决.

# 其他备注
- [zlib](https://z-lib.is/) 没法用了(by 2023年05月11日), 用 [yibook](https://tool.yibook.org/) 来替代吧, 可以通过百度网盘来传输, 速度嗖嗖地.
- kindle邮箱备忘:
```shell
KindleOasis1: davyjones2010_0312@kindle.cn
KindleOasis2: davyjones2010_0311@kindle.cn
```