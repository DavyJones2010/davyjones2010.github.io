---
title: sys stat page
date: 2023-11-01 00:00:00
tags: [thoughts, system-design, dashboard]
---

# 20231101-sys-stat-page

无意间发现: [37signals Status](https://www.37status.com/) 页面非常有趣, 把所有服务的状态都非常清晰地列出来, 如下图:

![Untitled](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/source/assets/2023-11-01-sys-stat-page/Untitled.png)

另外顺便搜索了下, 发现这个页面还是Atlassian 提供的服务: [Statuspage | Atlassian](https://www.atlassian.com/software/statuspage?utm_campaign=www.37status.com&utm_content=SP-notifications&utm_medium=powered-by&utm_source=inapp)

不得不说这种方式真的非常直观美观, 且把故障发生&结束&持续时长都一目了然地给出. 

以此为对比,  [阿里云健康状态](https://status.aliyun.com/?spm=5176.8789780.J_9220772140.26.348c39fbxOAGXq#/) 就显得差了很多: 

1. 一是不够清晰美观. 
2. 二是数据准确度存疑. 一直都是状态正常, 让人觉得不可思议…