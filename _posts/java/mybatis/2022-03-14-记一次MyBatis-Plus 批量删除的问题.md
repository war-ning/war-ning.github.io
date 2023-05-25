---
layout: articles
title: 记一次MyBatis-Plus 批量删除的问题
tags:  Mybatis
author: Warning
key:    java-mybatis-head-01
aside:
  toc: true
sidebar:
nav: Java
category: [java, mybatis]
---

刚来新公司第一天，签完卖身契就来任务了
新项目的`mybatis-plus`批量删除报错，需要我处理
鸭梨山大啊……

<!--more-->


### 错误信息

项目用的是`JEECG`的低代码框架

在调用`removeByIds()`的时候报错

> There is no getter for property named 'id' in 'class java.lang.String'

![](https://gitee.com/war-ning/picture/raw/master/blog/Snipaste_2022-03-09_11-57-41.png)

其实挺奇怪的错误，下面是后续分析解决问题的思路，记录下



### 问题筛查



#### 一、首先看Mybatis-Plus是否好用

仔细检查了实体类字段映射，和数据库配置

写了个测试接口，试了根据ID删除，没有问题

这就很奇怪了。其他的都可以，就批量删除不行



这里仔细看报错信息，其实SQL语句是打印成功的，就是后续字段赋值出现问题报错了

然后尝试主键设置成Long类型，执行成功了



#### 二、筛查数据库问题

公司数据库用的是`PostgreSQL`，之前没接触过

怀疑是不是数据库导致的BUG

改成MySQL环境，建个测试库发现还是同样的错



#### 三、一步步筛查

因为不知道从`JEECG` 过来具体改了什么

先把`JEECG`的环境跑一下，他的批量删除没问题

这下就只能一步步比对双方配置各方面有什么区别了

内心有点崩溃中………



#### 四、最后发现是Mybatis-plus的BUG

在官方更新日志里看到, 是这个版本更新的BUG

![](https://gitee.com/war-ning/picture/raw/master/blog/20220314134652.png)





#### 五、查看源码找问题

更新的时候，这里调用的类改了

但是新的类中没有加入String类型，导致只有String的报错

![](https://gitee.com/war-ning/picture/raw/master/blog/20220314135823.png)



修复后

![](https://gitee.com/war-ning/picture/raw/master/blog/20220314135944.png)
# 附录
## A 资源
## B 参考资料

