---
layout: articles
title: Linux安装redis
tags:  Linux
author: Warning
key:    linux-head-5
aside:
toc: true
sidebar:
nav: linux
category: [linux]
---



<!--more-->



------



# Linux 下安装 Redis

1、准备 redis 安装包，可以进入[官网](https://download.redis.io/releases/)，自行选择需要的版本下载，我下载的是 [redis-6.2.4.tar.gz](https://download.redis.io/releases/redis-6.2.4.tar.gz)

![image-20210730204801426](https://img2020.cnblogs.com/blog/2239225/202108/2239225-20210826161731178-536868431.png)



2、将本地的安装包上传到 linux 服务器上，我这里放在 /home/software 目录下



3、在 /usr/local/ 下创建 redis 文件夹

```bash
mkdir /usr/local/redis
```



4、解压安装包

```bash
tar zxvf redis-6.2.4.tar.gz -C /usr/local/redis
```

解压完之后， /usr/local/redis 目录中应该有一个相关目录

```bash
[root@xxx software]# ls /usr/local/redis
redis-6.2.4
```



5、编译并安装 redis

```bash
cd /usr/local/redis/redis-6.2.4
make && make install
```



6、将 redis 安装为系统服务并后台启动

```bash
# 进⼊ utils ⽬录，并执⾏如下脚本即可
cd utils/
./install_server.sh
```

如果出现报错，那么编辑这个文件

![image-20210731171900592](https://img2020.cnblogs.com/blog/2239225/202108/2239225-20210826161736015-1448434187.png)



```bash
vim ./install_server.sh
```

注释下面的代码

```bash
#bail if this system is managed by systemd
#_pid_1_exe="$(readlink -f /proc/1/exe)"
#if [ "${_pid_1_exe##*/}" = systemd ]
#then
#       echo "This systems seems to use systemd."
#       echo "Please take a look at the provided example service unit files in this directory, and adapt and install them. Sorry!"
#       exit 1
#fi
```


![image-20210731172018766](https://img2020.cnblogs.com/blog/2239225/202108/2239225-20210826161739575-515738799.png)

再次执行脚本即可

```bash
./install_server.sh
```


![image-20210731172126852](https://img2020.cnblogs.com/blog/2239225/202108/2239225-20210826161745526-1403723832.png)



7、查看 redis 服务启动情况

```bash
systemctl status redis_6379.service
```



8、启动自带的 redis-cli 客户端，测试 redis

```bash
[root@xxx utils]# redis-cli
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> get k1
"v1"
```



9、设置允许远程连接

编辑配置文件

```bash
vim /etc/redis/6379.conf
```

将原来的 bind 127.0.0.1 这行注释掉，改为 0.0.0.0

```bash
# bind 127.0.0.1
bind 0.0.0.0
```


![image-20210731172353517](https://img2020.cnblogs.com/blog/2239225/202108/2239225-20210826161749965-504738038.png)

重启 redis 服务

```bash
systemctl restart redis_6379.service
```



10、设置访问密码

```bash
vim /etc/redis/6379.conf
```

找到 # requirepass foobared，在这个注释下加一行，为 **requirepass 自己的密码**

```bash
# requirepass foobared
requirepass distance
```


![image-20210731172724221](https://img2020.cnblogs.com/blog/2239225/202108/2239225-20210826161755649-1030161933.png)

保存，重启 redis 服务

```bash
systemctl restart redis_6379.service
```



11、再次测试

```bash
[root@xxx ~]# redis-cli
127.0.0.1:6379> ping
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth distance
OK
127.0.0.1:6379> ping
PONG
```



12、 redis 服务管理

查看 redis 服务

```bash
ps -ef | grep redis
```

通过配置文件启动 redis

```bash
redis-server /etc/redis/6379.conf
```


![image-20210731174249027](https://img2020.cnblogs.com/blog/2239225/202108/2239225-20210826161759144-1958965316.png)

redis-cli 客户端连接

```bash
[root@xxx redis]# redis-cli -p 6379
127.0.0.1:6379> auth distance
OK
127.0.0.1:6379> ping
PONG
```


![image-20210731174337626](https://img2020.cnblogs.com/blog/2239225/202108/2239225-20210826161804555-383046213.png)

在客户端中可以关闭 redis 服务端

```bash
127.0.0.1:6379> shutdown
not connected> exit
```


![image-20210731174515927](https://img2020.cnblogs.com/blog/2239225/202108/2239225-20210826161808021-341156599.png)

13、尝试远程连接，用本地 windows 的 redis-cli 来连接 linux 服务器上的 redis server，需要本地也安装了 redis

```bash
# linux 先开启 redis 服务端
redis-server /etc/redis/6379.conf

# windows 本地打开 cmd
redis-cli.exe -h 192.168.0.102 -p 6379 -a redis
```


![image-20210731172937779](https://img2020.cnblogs.com/blog/2239225/202108/2239225-20210826161811582-773283469.png)

远程连接成功！




**END**


# 附录
## A 资源
## B 参考资料

