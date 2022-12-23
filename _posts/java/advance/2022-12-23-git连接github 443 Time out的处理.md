---
layout: articles
title: git连接github 443 Time out的处理
tags:  Spring-Retry Guava-Retry 重试光加
author: Warning
key:    java-advance-head-33
aside:
  toc: true
sidebar:
nav: Java
category: [java, advance]
---

新项目是个文件同步系统, 需要做失败重试
自己手写一套终归感觉不如现有的框架
轮子还是用人家的吧

<!--more-->



------



# Failed to connect to github.com port 443 : Timed out

> **问题描述：** 如下图所示，无法git clone 来自Github上的仓库，报端口443错误

![image-20221223101150177](https://raw.githubusercontent.com/war-ning/Pic/master/img/image-20221223101150177.png)

> **问题分析：** git 所设端口与系统代理不一致，需重新设置。

> **解决方法：** 操作如下图所示

**①打开 设置>网络与Internet>代理**

![image-20221223101209409](https://raw.githubusercontent.com/war-ning/Pic/master/img/image-20221223101209409.png)

**②记录下当前系统代理的IP地址和端口号。**
如上图所示，地址与端口号为：**127.0.0.1:7890**

**③修改git的网络设置**

```bash
注意修改成自己的IP和端口号
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

```



**④重新git clone, 测试修改效果**



git clone成功！




**END**


# 附录
## A 资源
## B 参考资料

