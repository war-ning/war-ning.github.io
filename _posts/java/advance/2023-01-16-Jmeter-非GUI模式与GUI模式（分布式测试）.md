---
layout: articles
title: Jmeter-非GUI模式与GUI模式（分布式测试）
tags:  Jmeter
author: Warning
key:    java-advance-head-37
aside:
  toc: true
sidebar:
nav: Java
category: [java, advance]
---

咳咳, 年底工作不饱和, 开始搞了一下压力测试

学了一下用jmeter, 大概记录一下 . . .



<!--more-->


# Jmeter-非GUI模式与GUI模式（分布式测试）



一、由于Jmeter是一个纯Java的应用，用GUI模式运行压力测试时，对客户端的资源消耗是很大的，所以在进行正式的压测时一定要使用非GUI模式运行。如果并发数很高或者客户端的硬件资源比较一般的话，采取的方法是以Server模式用多个Client进行分布式测试。

二、新建 JMETER_HOME 环境变量，值为D:\Software\apache-jmeter-5.0\apache-jmeter-5.0，在Path加入：%JMETER_HOME%\bin;

三、CMD->jmeter，可以直接运行Jmeter，jmeter -v查看版本号：

![img](https://raw.githubusercontent.com/war-ning/Pic/master/img/14269037-9bed7df580841724.png)



四、官方提示：不要使用GUI模式进行负载测试，GUI模式仅用于测试创建和测试调试。对于负载测试，请使用非GUI模式：jmeter -n -t [jmx文件] -l [results文件] -e -o [Path to web report文件夹]，可以节省系统资源，能够产生更大的负载，可以通过命令行参数对测试场景进行更精细的配置。

![img](https://raw.githubusercontent.com/war-ning/Pic/master/img/14269037-506f3b2bdd179a5e.png)



五、jmeter部分参数解释：

> 1、-v：打印版本信息；
>
> 2、-p {argument}：运行时指定property文件，默认是使用JMETER_HOME/bin目录下的jmeter.properties，如果用户自定义有其它的配置，在这里加上；
>
> 3、-q {argument}：指定其它配置文件，如JVM参数等等；
>
> 4、-t {argument}：要运行的jmeter脚本，.jmx文件；
>
> 5、-l {argument}：记录采样器Log的文件，保存JTL 测试结果文件的路径；
>
> 6、-j {argument}：指定记录jmeter log的文件，默认为jmeter.log；
>
> 7、-n：以nongui模式（非GUI模式）运行jmeter；
>
> ８、-s：运行JMeter server；
>
> 9、-H {argument}：代理服务器地址；
>
> 10、-P：代理服务器端口；
>
> 11、-J {argument}={value}：定义额外的Jmeter属性properties；
>
> 12、-G, --globalproperty {argument}={value}：定义发送给server的全局属性；
>
> 13、-D, --systemproperty {argument}={value}：定义系统属性；
>
> 14、-r, --runremote (non-GUI only)：启动远程server（在jmeter properties中定义好的remote_hosts），仅在non-gui模式下此参数才生效；
>
> 15、-R, --remotestart server1,... (non-GUI only)：启动远程server（如果使用此参数，将会忽略properties中中定义的remote_hosts）；
>
> 16、-d, --homedir {argument}：Jmeter运行的主目录；
>
> 17、-X, --remoteexit：测试结束时，退出（在non-gui模式下）；
>
> 18、-g {argument}：通过csv文件来创建dashboard报告；
>
> 19、-o {argument}：运行结束后创建dashboard报告；
>
> 20、-e {argument}：在哪个目录创建dashboard报告。

六、分布式：Jmeter的集群模式可以让我们将多台机器联合起来一起产生负载，从而弥补单台机器负载生成能力不足的问题。假设我们的测试计划会产生100个threads，我们使用5台机器进行分布式测试的时候，一共会产生100 * 5 = 500的负载。

七、分布式测试的思想为：一台master主机（调度机或者称作控制机）初始化测试并控制多个slave系统（执行机或者称作负载机）。master，以GUI模式运行，同时控制测试的运行，就是client，启动脚本所在的那台机器。master也可以参与脚本的运行，master同时也是一台负载机。slave，运行客户端程序(Agent:jmeter-server.bat)，从master接收指令、向目标服务器发送请求，就是server，真正执行test plan的机器。

> slave首先启动Agent程序，待master连接；
>
> master连接上slave；
>
> master发送指令（脚本及启动命令）启动线程；
>
> slave运行脚本，回传状态（包括测试结果）；
>
> master收集结果并显示。

八、注意事项：

> 1、关闭防火墙；
>
> 2、所有的客户端都在同一个子网内；
>
> 3、如果使用192.x.x.x或者10.x.x.x这样的IP地址，server也必须在同一子网内，如果server没有使用192或者10这样的IP地址，（server同client不在同一子网内）将不会有任何问题；
>
> 4、确保Jmeter可以访问到server；
>
> 5、确保各系统的Jmeter版本保持一致，不同版本的Jmeter将不能很好的工作。

九、设置jmeter client & server

1、设置jmeter-server：用文本编辑器打开JMETER_HOME/bin目录下的jmeter.properties文件，添加运行jmeter-server的主机IP到remote-hosts，如果你不希望你的客户端也作为jmeter-server运行的话，把localhost从上面的配置中移除。

> remote_hosts=10.0.0.158, 10.0.0.140,127.0.0.1

2、将配置在remote_hosts中的机器上的jmeter-server启动(Windows以管理员身份运行JMETER_HOME/bin目录下的jmeter-server.bat）。

十、分布式测试

1、方式一：在客户端**以GUI模式**启动jmeter，然后打开或者创建一个测试脚本， 从GUI模式启动所有的远程server，点击，运行-远程全部启动，也可以单独启动某一个jmeter-server 。

![img](https://raw.githubusercontent.com/war-ning/Pic/master/img/14269037-adc2f02930e4d3f5.png)



2、方式二：**以非GUI模式**启动jmeter

2.1、新建一个文件夹，存放jmeter脚本，比如baidu.jmx；

2.2、在当前文件夹下打开cmd窗口（shift+鼠标右键选择在此处打开命令窗口），输入命令：

> jmeter -n -t baidu.jmx -l res.jtl -e -o ./report

2.3、运行结果，就是生成res.jtl文件，jmeter.log日志文件和report文件夹：

![img](https://raw.githubusercontent.com/war-ning/Pic/master/img/14269037-dda7cefc1d915f0e.png)










**END**


# 附录
## A 资源
## B 参考资料

