---
title: http post or get
date: 2023-11-27 00:00:00
tags: [thoughts, http, api-design]
---

# 20231127-http-post-or-get

之前记得网上有过讨论, 有公司让所有请求都用POST. 

结合最近自己的感悟, 说下. 

如果是内部系统, 那就尽量用GET吧. 关键参数都放在URL里. 不管是普通风格还是RESTFUL风格. 

优点是: 

- 便于使用Alfred等工具, 一键进入. 提效明显: 例如查询用户信息, url?uid=xxx, 这样. 而如果是POST,
-