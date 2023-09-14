---
layout: post
title: 无用的知识又增加了
tags: [tech-for-fun]
lang: zh
---

CPU芯片中vendor_id:
- `intel --> Genuine Intel`
- `amd --> Authentic AMD`

哈哈, 为啥都加个"真实的"呢? 生怕别人伪造么?

---
- 为啥 Java 的 HeapDump 文件后缀是`hprof`, 例如`20230914-tmp.hprof`?
- 代表`Heap Profiling`, 即堆内存分析. 也是很早以前JDK提供的工具: [HPROF: 一个Heap/CPU Profiling工具](https://www.cnblogs.com/linhaohong/archive/2012/07/12/2588657.html)

---
- K8s中各种shim组件, 具体为啥叫 shim ?
- 直译过来是`垫片`, 这是什么鬼? 很难理解. 其实本质就是Adaptor. 当然也有人也有类似的吐槽, 好好地叫adaptor很难么? [Design Patterns: WTF is a Shim?](https://hackernoon.com/design-patterns-wtf-is-a-shim-la1h338v)

---
- 到底在什么场景下该用`java.util.LinkedList`? 具体谁用了?
- 巧了, LinkedList的作者`Joshua Bloch`也有同样的困惑. 基本结论, 就是学习思路可以, 但在实际生产环境中就尽量不要用啦.
  ![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202309142358549.png)



