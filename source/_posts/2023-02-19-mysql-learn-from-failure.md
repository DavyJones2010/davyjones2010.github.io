---
title: SQL/MySQL使用过程中的踩坑合集
date: 2023-02-19 10:24:18
tags: [learn-from-failure, mysql]
---

# left join
## key在右表中重复导致结果集重复
样例参见: [SQL Fiddle](http://sqlfiddle.com/#!9/4f158d1/7)
所以left join之前, <font color='red'>一定要确认右表的key是否会重复</font>. 
尤其是在写HIVE这种比较重量级的SQL之前, 一定要注意, 否则会导致报表制作出来数据重复, 影响数据质量.

## on中写了左表的
{% note warning %}
left join 为了保证左表所有行, 因此on里的条件只对右表起作用, 控制左表的条件写到on里也没用
{% endnote %}
样例参见: [SQL Fiddle](http://sqlfiddle.com/#!9/4f158d1/10)
所以left join的时候, <font color='red'>一定要保证筛选左表的条件要放到where里, 而不要放到on里</font>. 


