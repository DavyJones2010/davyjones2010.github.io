---
title: Spring SPEL语法常用样例
date: 2023-06-28 22:26:04
tags: [java, spring, spring-spel, spring-cache]
---

工作中使用了SpringCache, 因此需要经常写cacheKey的表达式, 而这些表达式就是[Spring Expression Language (SpEL)](https://docs.spring.io/spring-framework/docs/3.0.x/reference/expressions.html) 
因此在这里分析了几种常见的SPEL使用方式, 废话不多说, 详细代码测试用例参见: [SpelTest.java · GitHub](https://github.com/DavyJones2010/test-core/blob/master/src/test/java/edu/xmu/test/framework/spring/spel/SpelTest.java)

上述样例, 只是满足了日常功能层面的需求, 记录下来防止遗忘. 
具体深入的性能以及能力的边界, 暂时不做讨论.