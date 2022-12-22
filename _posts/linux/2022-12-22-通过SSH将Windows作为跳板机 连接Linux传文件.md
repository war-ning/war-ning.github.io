---
layout: articles
title: 通过SSH将Windows作为跳板机 连接Linux传文件
tags:  wget sftp
author: Warning
key:    linux-head-7
aside:
toc: true
sidebar:
nav: linux
category: [linux]
---

最近的项目是内网的, 跳板机开了个FRP透出来了个ssh端口
然后就可以搞事情了 . . .
SSH隧道连Linux服务器, 传文件咯....


<!--more-->



------



# 转化阿里网盘连接

下载油猴插件, 通过阿里网盘, 把分享链接转化成支链下载链接

### 网盘直链下载助手v6.0.2

>  https://www.youxiaohou.com/install.html





# Windwos安装wget



1. 去该网址下载二进制文件：[GNU Wget 1.21.3 for Windows](https://eternallybored.org/misc/wget/)
  - 注意：要根据你的电脑选择32位还是64位，这很重要

2. 解压，然后将wget.exe复制到C:\Windows\System32下
3. 大功告成 . . . .

## wget的使用

建议参考这位博主：[wget的使用](https://blog.csdn.net/liaowenxiong/article/details/117337527)

我盘符比较小，所以我需要把文件下载到盘符大的目录

> wget -P 目录地址 下载地址

注意：目录地址中不能有中文、空格，不然无法解析

不明白的参考这个博客：[指定下载目录](https://blog.csdn.net/weixin_43545225/article/details/106118045?ops_request_misc=%7B%22request%5Fid%22%3A%22165779128116781435487951%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fall.%22%7D&request_id=165779128116781435487951&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~pc_rank_34-2-106118045-null-null.142^v32^pc_rank_34,185^v2^control&utm_term=wget指定下载目录&spm=1018.2226.3001.4187)

> - ` wget -i filename.txt`
>
> 此命令常用于批量下载的情形，把所有需要下载文件的地址放到 filename.txt 中，然后 wget 就会自动为你下载所有文件了。





# 使用 SFTP 进行连接

因为 SFTP 是基于 SSH 协议的，所以默认的身份认证方法与 SSH 协议保持一致。通常我们使用 SSH Key 来进行连接，如果你已经可以使用 SSH 连接到远程服务器上，那么可以使用以下命令来连接 SFTP：

```text
sftp user_name@remote_server_address[:path]
```

如果远程服务器自定义了连接的端口，可以使用 `-P` 参数：

```text
sftp -P remote_port user_name@remote_server_address[:path]
```

连接成功后将进入一个 SFTP 的解释器，可以发现命令行提示符变成了 `sftp>`，使用 `exit` 命令可以退出连接。



## sftp的基础命令



SFTP 解释器中预置了常用的命令，但是没有自带的 Bash 来得丰富。

1）显示当前的工作目录：

```text
sftp> pwd
Remote working directory: /
```

2）查看当前目录的内容：

```text
sftp> ls
Summary.txt     info.html       temp.txt        testDirectory
```

3）使用 `-la` 参数可以以列表形式查看，并显示隐藏文件：

```text
sftp> ls -la
drwxr-xr-x    5 demouser   demouser       4096 Aug 13 15:11 .
drwxr-xr-x    3 root       root           4096 Aug 13 15:02 ..
-rw-------    1 demouser   demouser          5 Aug 13 15:04 .bash_history
-rw-r--r--    1 demouser   demouser        220 Aug 13 15:02 .bash_logout
-rw-r--r--    1 demouser   demouser       3486 Aug 13 15:02 .bashrc
drwx------    2 demouser   demouser       4096 Aug 13 15:04 .cache
-rw-r--r--    1 demouser   demouser        675 Aug 13 15:02 .profile
. . .
```

4）切换目录：

```text
sftp> cd testDirectory
```

5）建立文件夹：

```text
sftp> mkdir anotherDirectory
```

以上的命令都是用来操作远程服务器的，如果想要操作本地目录呢？只需要在每个命令前添加 `l`即可，例如显示本地操作目录下的文件：

```text
sftp> lls
localFiles
```

使用 `!` 可以直接运行 Shell 中的指令：

```text
sftp> !df -h
Filesystem      Size   Used  Avail Capacity iused               ifree %iused  Mounted on
/dev/disk1s1   466Gi  360Gi  101Gi    79% 3642919 9223372036851132888    0%   /
devfs          336Ki  336Ki    0Bi   100%    1162                   0  100%   /dev
/dev/disk1s4   466Gi  4.0Gi  101Gi     4%       5 9223372036854775802    0%   /private/var/vm
map -hosts       0Bi    0Bi    0Bi   100%       0                   0  100%   /net
map auto_home    0Bi    0Bi    0Bi   100%       0                   0  100%   /home
```

## 传输文件

### 从远程服务器拉取文件

使用 `get` 命令可以从远程服务器拉取文件到本地：

```text
sftp> get remoteFile [newName]
```

如果不指定 `newName`，将使用和远程服务器相同的文件名。

使用 `-r` 参数可以拉取整个目录：

```text
sftp> get -r remoteDirectory
```

### 从本地上传文件到服务器

使用 `put` 命令可以从本地上传文件到服务器：

```text
sftp> put localFile
```

同样的，可以使用 `-r` 参数来上传整个目录，但是有一点要注意，**如果服务器上不存在这个目录需要首先新建**：

```text
sftp> mkdir folderName
sftp> put -r folderName
```



**END**


# 附录
## A 资源
## B 参考资料

