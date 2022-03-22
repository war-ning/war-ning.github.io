---
layout: articles
title: IDEA 非Maven项目打Jar包
tags:  部署, IDEA
author: Warning
key:    java-advance-head-13
aside:
  toc: true
sidebar:
nav: Java
category: [java, advance]
---



> 上周五改了个对接Demo， 这一看，不是Spring项目，连Maven项目都不是
>
> 好久没手动拷依赖包了，搞完之后打包还出了点问题
>
> 查了半天，记录一下，这代程序员真的幸福啊……
>


<!--more-->

#### 一、以Spring Boot项目为例，在未使用maven的情况下将其打成Jar包。

#### 二、将其打成散包，即项目依赖的Jar包在目录同级或子级。 好处是如果项目更新，只需要更新项目的jar，不需要更新所有。

------

### 一、检查项目是否包含META-INF文件夹。

![](https://gitee.com/war-ning/picture/raw/master/blog//20210816101234.png)

```
若包含META-INF文件夹，将其删除。等下重新生成。
```

------

### 二、构建Artifacts

2.1 选择菜单中的 File->Project Structure 。 在弹出的Project Structure中选择 Artifacts-> + -> JAR ->From modules with dependencies

![](https://gitee.com/war-ning/picture/raw/master/blog//20210816101303.png)

2.2 在弹出的窗口中选择要打包的模块和主函数，然后选择要打成散包还是一个整体。
![](https://gitee.com/war-ning/picture/raw/master/blog//20210816101323.png)

> Module： 模块，选择需要打包的模块。如果程序没有分模块，那么只有一个可以选择的。
> MainClass：选择程序的入口类。
> extract to the target JAR：抽取到目标JAR。选择该项则会将所依赖的jar包全都打到一个jar文件中，此jar解压后是所依赖的jar及本工程编译好的文件目录
> copy to the output directory and link via manifest：将依赖的jar复制到输出目录并且使用manifest链接它们。
> Direct for META-INF/MANIFEST.MF： 如果上面选择了 "copy to ... "这一项，这里需要选择生成的manifest文件在哪个目录下，META-INF/MANIFEST.MF一般放在src下面最好
> Include tests： 是否包含tests。 一般这里不选即可。

![](https://gitee.com/war-ning/picture/raw/master/blog//20210816101342.png)

选择OK，会在刚才选择的文件夹下面生成一个META-INF文件夹，下面有一个MANIFEST.MF文件

> MANIFEST.MF
> 主要一下几个：
> Manifest-Version： Manifest文件的版本，这个不用管。
> Class-Path： 描述lib包相对生成的jar的路径。
> Main-Class： 程序的入口类

### 三、将生成的MANIFEST.MF增添到JAR包

![](https://gitee.com/war-ning/picture/raw/master/blog//20210816101408.png)

在这些jar包中，黄色的是依赖的jar包，蓝色的是我们生成的要运行的jar包。在蓝色的jar包上右键，选择 “Create Directory”（创建文件夹），名称叫做META-INF。
![](https://gitee.com/war-ning/picture/raw/master/blog//20210816101423.png)

在新建的“META-INF”文件夹上右键，选择Add Copy of 下的 File，选择刚刚生成的MANIFEST.MF文件。如下图
![](https://gitee.com/war-ning/picture/raw/master/blog//20210816101442.png)

------

### 四、生成Jar包。

```
配置完上述后。选择菜单中的 build -> build artifacts...
```

![](https://gitee.com/war-ning/picture/raw/master/blog//20210816101500.png)

此时页面中间会弹出要生成的jar包，选择刚刚构建的Artifacts，选择build或者rebuild
![](https://gitee.com/war-ning/picture/raw/master/blog//20210816101515.png)

> Build：只将主Jar包重新生成，不重新生成所依赖的Jar包。
> Rebuild： 将所有jar包重新生成。
>
>
>
> 此时生成jar包已经完成



# 附录
## A 资源
## B 参考资料
https://www.cnblogs.com/lianxuan1768/p/12752482.html


