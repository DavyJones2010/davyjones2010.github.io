---
title: 瞎折腾软路由笔记
date: 2022-08-07 20:38:43
categories: 瞎折腾
tags: [linux, open-wrt, software-based-router, router, network]
---
垃圾佬的周末, 没事儿就是瞎折腾, 前段时间被种草了软路由, 一直心痒痒. 月底下单奢侈一把, 今天给彻底搞起来了. 现在总结下.

# 为啥要使用软路由? 
有几个原因: 
1. 家里的Android(V2RayNG), Mac(ClashX), Windows, Ubuntu等设备都有相关软件能科学上网; 但iPad, iPhone等设备, 安装小火箭, 必须要登录海外AppStore, 而且软件还收费, 太麻烦, 因此没有配置科学上网. 对强迫症患者来说, 简直是灾难.
2. 想要对网络设备的工作原理能有更深入的理解. 例如路由器的底层工作原理, 实际的路由表是咋样的? 之前使用华为的路由器, 这些权限完全是没有的, 也没办法看到这些的. 对于越狱狂魔来说, 也同样是灾难.
3. 家里的路由器明显感觉力不从心, 100M的带宽, 平常根本跑不满, 就开始发热. 
4. IPTV共享: 
   1. 无法啊忍受丑陋的电信机顶盒(烽火 HG680-J), 设计得丑陋不堪, 遥控器巨难用; 
   2. 只能在客厅使用, 卧室里也想连接IPTV, 但是连接不上.

软路由本质上是一个低功耗的x86机器, 应该能很好地解决上边的问题. 

# 配置清单
| 配件名称 | 品牌 | 价格 | 详细说明 |
| --- | --- | --- | --- |
| 主机 | [倍控](https://detail.tmall.com/item.htm?id=673002078011&spm=a1z09.2.0.0.237b2e8dIxvpd0&_u=77rqrdmc3b6) | 729 | 裸机,  |
| CPU | [Intel N5105](https://www.intel.com/content/www/us/en/products/sku/212328/intel-celeron-processor-n5105-4m-cache-up-to-2-90-ghz/specifications.html) | 无, 包含在主机里 | 第11代, Jasper Lake; 4Core; 10nm; Base 2GHz, Burst 2.9GHz; 4MB L3 Cache; 10W TDP |  
| 内存 | 三星 | 380*2 | 16GB * 2(组成双通道); DDR4; 3200MHz; | 
| 闪存 | 西数 SN570 | 350 | 500GB; NVME  | 
| 网卡 | Intel I225-V | 无, 包含在主机里 | 4个物理网口/卡; 1000Mbps |
  
合计: `729 + 380*2 + 350 = 1839`, 几乎可以说是目前工控机的顶配了.

{% note warning %}
内存频率限制
CPU支持的内存频率最大为`2933 MHz`, 因此使用的 DDR4 `3200MHz` 被自动降频到了`2933 MHz`;
{% endnote %}


# 安装步骤
## OpenWrt安装
裸机上安装OpenWrt, 配置步骤直接看小电视: https://www.bilibili.com/video/BV1w541157Uo?spm_id_from=333.880.my_history.page.click&vd_source=25b2aadfc1b4b676c371c31423142e7b

## 插件配置
- OpenClash: // TODO:  

# 网络拓扑
由于家里的网络是FTTB(Fiber-To-The-Building)的, 因此没有光猫, 只有一根入户线. 
## 软路由改造前
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208072212775.png)

## 软路由改造后
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208072216377.png)

# 效果

## 效果预览
1. 安装好 [OpenClash](https://github.com/vernesong/OpenClash) 插件, 局域网内科学上网无忧.  
2. 直接SSH上去, 看到任何的网络相关信息: 
![ARP表](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208072139468.png)
![路由表](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208072140766.png)
![网桥](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208072141717.png)
3. 网速基本能跑满: 
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208072137732.png)
4. IPTV共享: 目前还未实现, 涉及到组播+vLan等, 还在研究中. 


## 其他
- CPU Load极低, 长期维持在0.1-;
- MEM 使用比例极低, 长期可用内存维持在96%+;
- 资源完全没有充分利用起来!
![](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/img/202208072130374.png)


# 总结
总体来说, 虽然达到了部分目标, 但还是大材小用了! 后边计划有几种方案来充分压榨: 
- 多跑几个docker容器, 例如搭建Jenkins, GitLab, Nginx, Redis; 方便自己平常的压测验证. 
- 改成裸机上刷esxi, 搭建几个虚拟机, 把OpenWrt放在其中一个虚拟机里, 其他的Ubuntu, Windows等作为日常休闲娱乐机.