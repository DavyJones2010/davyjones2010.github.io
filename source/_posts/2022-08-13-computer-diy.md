---
title: 瞎折腾洋垃圾电脑笔记
date: 2022-08-13 21:39:48
categories: 瞎折腾
tags: [diy]
---

垃圾佬的周末, 没事儿就是瞎折腾, 自从在B站被种草了洋垃圾, 按耐不住心中的冲动, 决定也用洋垃圾组装一台能数框框的电脑, 玩玩儿游戏, 压榨压榨性能.

# 为啥要使用洋垃圾
- 一: 穷, 
- 二: 玩, 


# 配置清单

> 几乎都是从海鲜市场淘来的
> - 优点是便宜, 性价比高; 
> - 缺点是电气特性衰减不明确, 长期稳定性没有保障. 

但对于偶尔玩一玩游戏, 不是长期高负荷跑渲染等, 整体还OK. 目标是**1000元**左右搞定.

## 详细清单

| 配件名称 | 品牌 | 数量 | 价格 | 详细说明 |
| --- | --- | --- | --- | --- |
| CPU | [Intel® Xeon® Processor E5-2630L v3](https://ark.intel.com/content/www/us/en/ark/products/83357/intel-xeon-processor-e52630l-v3-20m-cache-1-80-ghz.html) | 1 | 80 | Haswell; 8Core 16HT; 22 nm制程; Base 1.8GHz, Burst 2.90 GHz; 20MB L3 Cache; 55W TDP |
| 主板 | [X99寨板](https://item.taobao.com/item.htm?id=676047268901) | 1 | 250 |   |
| 内存 | 威刚万紫千红 | 4 |  160 + 200 | 8GB; DDR4 2133MHz; 强迫症4条必须插满, 组成4通道^_^  | 
| 闪存 | [Intel® SSD 760P](https://www.intel.com/content/www/us/en/products/sku/134583/intel-ssd-760p-series-256gb-m-2-80mm-pcie-3-1-x4-3d2-tlc/specifications.html) | 1 | 160 | 256GB; PCIe 3.1 x4 接口, NVMe  | 
| 显卡 | 蓝宝石RX460 | 1 | 350 |  4GB显存; 无需单独供电; HDMI + VGA + DP 接口 | 
| 电源 | 鑫谷全模组 | 1 | 120 |  550W, 整体绰绰有余了  | 
| CPU散热器 | 长城霄龙400 | 1 | 45 | 4铜管, 3针脚 |
| 机箱 | 待挑选 | 1 | ?? | ?? |

合计 `80 + 250 + 160 + 200 + 160 + 350 + 120 + 45 = 1,365` 
现在显卡电源都还没到, 希望组装上去之后一次点亮, 就可以说声"真香!"啦.

## 主板细节

![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208132325729.png)

{% note info %}
x99含义
所谓的x99主板, 代表的其实是`Intel X99 chipset`, 是Intel的PCH(Platform Controller Hub)即南桥芯片, 定义了主板的规范, 如下:
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208140001662.png)
{% endnote %}

其他细节: 
- CPU插槽: 单路; LGA2011, 因此是2011个针脚;
- CPU只支持DDR4内存, 但主板支持DDR3(问了店家, 主板同时也支持DDR4), 现在DDR4普遍价格比较贵, 没办法使用闲鱼上价格巨便宜的DDR3 ECC内存.
- 

## 其他

### CPU型号

买的时候没有看清楚CPU型号, 以为是 `E5-2630 V3`, 结果实际是 `E5-2630L V3`, 才知道多了的这个`L`代表<mark>低功耗(Low Power)</mark>, 基频只有`1.8GHz`不能忍. 
因此就又淘了如下两个能够适配X99主板的CPU, 准备到时候各自对比测试下:

| 配件名称 | 品牌 | 数量 | 价格 | 详细说明 |
| --- | --- | --- | --- | --- |
| CPU | [Intel® Xeon® Processor E5-2630L v3](https://ark.intel.com/content/www/us/en/ark/products/83357/intel-xeon-processor-e52630l-v3-20m-cache-1-80-ghz.html) | 1 | 80 | 8Core 16HT; 22 nm制程; Base 1.8GHz, Burst 2.90 GHz; 20MB L3 Cache; 55W TDP |
| CPU | [Intel® Xeon® Processor E5-2660 v3](https://ark.intel.com/content/www/us/en/ark/products/81706/intel-xeon-processor-e52660-v3-25m-cache-2-60-ghz.html) | 1 | 100 | 10Core 20HT; 22 nm制程; Base 2.60 GHz, Burst 3.3 GHz; 25MB L3 Cache; 105W TDP |
| CPU | [Intel® Xeon® Processor E5-2620 v3](https://ark.intel.com/content/www/us/en/ark/products/83352/intel-xeon-processor-e52620-v3-15m-cache-2-40-ghz.html) | 1 | 20 | 6Core 12HT; 22 nm制程; Base 2.40 GHz, Burst 3.20 GHz; 15MB L3 Cache; 85W TDP; 你不能对一个20块钱的CPU要求更多了! :) |

{% note info %}
> CPU盖子上的`Costa Rica`, `Malay` 代表啥意思?
> Just to clear something up, the markings on the back of the CPU refer to the packaging site - not the manufacturing site of the actual silicon chip itself.
> 也就是说<mark>代表的是封装工厂的地点</mark>, `Malay` 代表 `Malaysia`

{% endnote %}

### ECC内存
{% note warning %}
内存买成ECC却不能用 ECC内存到底是什么鬼?
{% endnote %}

- 这是因为一般的电脑为了速度，一般都是不支持ECC内存，而ECC内存由于有校验这一步骤，一般都多用在服务器领域，<mark>普通的家用主板一般都是不支持的</mark>，而且对于服务器，还要区分REG-ECC和纯ECC的区别，REG是带寄存器的ECC内存，可以支持更大的单条容量，但是由于有寄存器的存在，延迟会更高。

### CPU散热器

{% note warning %}
主板上`CPU FAN`是4针的, 买来的散热器风扇是3针的. 区别是啥? 怎么插?
{% endnote %}

> 2针, 3针与4针风扇区别?
- 2针的散热风扇（结构简单，但不能测速）: 一个用于供电、一个用于接地，不过没有测速功能。目前2针的散热风扇已经在市场上大量采用，但是2针的风扇在变速等方面上已经不能满足要求了
- 3针的散热风扇（VC电压调速，灵活性欠佳）: 第三条线, 主要是起着测速的作用，通过主板能够侦测到风扇是否在转、转速为多少，但是3针散热风扇的调速是通过调整风扇电压来实现的，此外，能不能调速还需要看下主板是否支持，显然这种调节转速并不够灵活，不能实时反映CPU状态的信息，还需要在主板BIOS中进行各种设置。
- 4针的散热风扇（更加灵活更加智能的调速）: 4针的散热风扇是在3针基础上又多了一根PWM线（ 脉宽调变），所谓的PWM线就是利用PWM机制来调节风扇转速，而风扇电压是恒定的，避免了控制电压来变速的一系列弊端。而PWM风扇无需依赖主板BIOS，更加省心方便。PWM脉冲信号可以输出不同的占空比（Duty Cycle），0-100%之间的占空比就能调节出不同的风扇转速，所以能够做到精确、灵活控制。

![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208140950640.png)

> 3针怎么插入主板4针接口中?
- 兼容, 有防呆机制.

总体来说, 风扇踩坑了, 但也够用了:
1. 应该购买6铜管
2. 应该购买4针的

## 升级潜力&计划
整体CPU还是比较强悍的, 有很大升级潜力

### 主板升级
- 升级成两路, 看有没必要吧, 这样电费吃不消, 电源可能也要升级; [X99双路主板简评](https://post.smzdm.com/p/aoo8wewm/pic_3/#bigImg) 指明了方向
### CPU升级
- X99整体还支持V4(Broadwell)的CPU, 例如 `E5-2660 V4`, 16年上市, 14nm制程. 全部列表参见: 
- 2699 v3: 650RMB左右
- 
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208141020764.png)

### 内存升级
- ECC内存: 这几个CPU都支持ECC内存, 等下批服务器淘汰, 或者DDR5主流, 就可以低价淘几个DDR4 ECC内存;
- 容量升级: CPU支持 `192GB * 4 = 768GB` 主板支持`32GB*4=128GB`; 当前是`8GB*4=32GB`, 有很大升级潜力, 等内存价格下降吧~
- 频率升级: CPU限制最大支持2133MHz, 导致最新的3200MHz都没法用(能用, 但会自动降频到2133MHz).


# 学到了啥

## 常用主板型号

- X79
  - Sandy Bridge-E, Ivy Bridge-E
  - Quad-channel DDR3, up to two DIMMs per channel
- X99
  - Haswell-E, Broadwell-E
  - Quad-channel DDR4, up to two DIMMs per channel


## 家用机箱型号

- E-ATX
- ATX
- M-ATX
- ITX

## 其他体悟
- 知道了经常说的V3, V4利旧是啥了; 对这种洋垃圾CPU市场行情有了大概的认知.
- 计算机并非只有CPU, 还有内存, 主板, 显卡甚至电源, 散热器等; 也都是极为关键的. 尤其是这种洋垃圾, CPU跟不要钱一样, 其他组件的可扩展性与稳定性就至关重要了. 很多特性不止要看CPU是否支持, 也要看主板. 例如内存的DDR代数, 最大容量, 通道数, 最大频率, 是否支持ECC; 例如CPU插槽的针脚数;
- 摩尔定律的恐怖: 上边几个CPU基本都是14年Q3上市, 到今天2022年Q2, 仅仅8年左右:  
  - 价格: 就已经跌倒了白菜价, 以`E5-2660 v3`为例, 上市价格是`$1445.00`, 现在是`100RMB=$15`. 
  - 性能: 当前主流的12代Intel, 以家用的 [i7-12700K](https://ark.intel.com/content/www/cn/zh/ark/products/134594/intel-core-i712700k-processor-25m-cache-up-to-5-00-ghz.html) 为例, 制程已经到了7nm; 更不用提ARM架构下的主流制程都是5nm了.
  
> 旧时王谢堂前燕, 飞入寻常百姓家


# Refs
- [散热风扇的2针、3针、4针都有什么区别？](https://www.expreview.com/62077.html)
