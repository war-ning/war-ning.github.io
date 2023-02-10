---
layout: articles
title: Linux安装mysql5.7
tags:  Linux
author: Warning
key:    linux-head-4
aside:
toc: true
sidebar:
nav: linux
category: [linux]
---



<!--more-->



------
# Linux mysql5.7安装



MySQL Linux版的安装与配置

安装步骤
1 检查工作
1.1 删除已有的相关包

在安装前需要确定现在这个系统有没有 MySQL，如果有那么必须卸载。尤其是新的 CentOS 7 系统，它自带mariaDB数据库，所以需要卸载掉。

查找已安装的MySQL软件包：

```perl
rpm -qa|grep mysql
```

- CentOS7下还需要查找是否存在mariadb包

  ```perl
  rpm -qa|grep mariadb
  ```



- 如果输入上述两个命令后都输出存在有包，则需要执行删除命令。

- 例如，前两步中终端输出了“mysql-libs-5.1.73-1.el6.x86_64”和“mariadb-libs-5.5.56-2.el7.x86_64”，则：

- ```css
  rpm -e --nodeps mysql-libs-5.1.73-1.el6.x86_64
  rpm -e --nodeps mariadb-libs-5.5.56-2.el7.x86_64
  ```



- **1.2** **提升权限**

  由于 MySQL 安装过程中，会通过 MySQL 用户在 /tmp 目录下新建 tmp_db 文件，所以需要给 /tmp 目录较大的权限：

- ```bash
  chmod -R 777 /tmp
  ```

  **1.3** **检查依赖**

  这一步需要检查系统中是否存在一些安装MySQL时需要的依赖库。因为考虑到大家有些是在虚拟机上安装的Linux系统。如果安装系统的时候用的是最小安装等原因，可能就不存在这些库。

- 执行两个查询命令看是否存在依赖库：

- ```perl
  rpm -qa|grep libaio
  rpm -qa|grep net-tools
  ```



- 如果不存在则需要安装：

- ```undefined
  yum -y install libaio net-tools
  ```

  **2** **准备安装包**

  **2.1 准备安装包方法 1**

- 首先进入到 /opt目录：

  ```bash
  cd /opt
  ```

  使用 wget下载并解压

  这里最好通过mysql.com 去查询适合当前系统的最新安装包

- ```ruby
  wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.41-1.el7.x86_64.rpm-bundle.tar
  tar -xvf mysql-5.7.41-1.el7.x86_64.rpm-bundle.tar
  ```

  下载完成后，解压，会看到很多 rpm 包，其中的 4 个是必要的：

  - mysql-community-common-5.7.16-1.el6.x86_64.rpm
  - mysql-community-libs-5.7.16-1.el6.x86_64.rpm
  - mysql-community-client-5.7.16-1.el6.x86_64.rpm
  - mysql-community-server-5.7.16-1.el6.x86_64.rpm

- 3 开始安装
  使用 rpm 命令按顺序依次安装 4 个包：

  ```vbscript
  rpm -ivh mysql-community-common-5.7.16-1.el7.x86_64.rpm
  rpm -ivh mysql-community-libs-5.7.16-1.el7.x86_64.rpm
  rpm -ivh mysql-community-client-5.7.16-1.el7.x86_64.rpm
  rpm -ivh mysql-community-server-5.7.16-1.el7.x86_64.rpm
  ```

  ![image-20210412151223700](https://img-blog.csdnimg.cn/img_convert/03d4c19b466928d4c459968be72e940c.png)



注意：1. 安装 server 会比较慢；
2. 如果前面第 1.3 步没检查好，在安装 mysql-community-server 会报错。

3. 安装报错:

**/usr/bin/perl 被 mysql-community-server-8.0.18-1.el7.x86_64 需要
perl(Getopt::Long) 被 mysql-community-server-8.0.18-1.el7.x86_64 需要
perl(strict) 被 mysql-community-server-8.0.18-1.el7.x86_64 需要**

![image-20221013110745728](https://raw.githubusercontent.com/war-ning/Pic/master/img/image-20221013110745728.png)

缺少依赖, 安装执行:

> **yum install -y perl-Module-Install.noarch**


4 查看MySQL 的安装版本
执行 " mysqladmin –version " 命令，类似 " java -version " 如果输出版本消息，即为安装成功。
![image-20210412151230355](https://img-blog.csdnimg.cn/img_convert/b525a1109e7477dbb8b0ff63ea0dc059.png)

### （三）相应配置

**5** **初始化** MySQL

- MySQL 5.7下载完后需要手动初始化：

- ```sql
  mysqld --initialize --user=mysql
  ```



- 查看并记住初始密码，“root@localhost:” 后面的就是初始化密码，要记下来，后面连接数据库会用到。

- ```bash
  cat /var/log/mysqld.log | tail -n 10
  ```

  ![image-20210412151241435](https://img-blog.csdnimg.cn/img_convert/9264dec6d1e2939a542e70d85f9aabbe.png)

  **6** **启动 MySQL 服务**

- 启动服务

-

```sql
systemctl start mysqld.service
```



- 关闭服务

- ```vbnet
  systemctl stop mysqld.service
  ```



- 查看服务状态

- ```lua
  systemctl status mysqld
  ```

  ![image-20210412151246509](https://img-blog.csdnimg.cn/img_convert/46225880d9ec5a863f93709085beaff8.png)



- 查看是否自启动

  ```perl
  systemctl list-unit-files|grep mysqld.service
  ```

  ![image-20210412151251842](https://img-blog.csdnimg.cn/img_convert/0b7b62d1b6d00f90b0394da4a8c68403.png)



- 设置自启动

- ```bash
  systemctl enable mysqld.sercice
  ```

  **7** **首次登陆**

- 首次登陆通过 “mysql -uroot -p” 进行登录，在 “Enter password：” 后输入初始化密码。正确输入密码后输出如下所示则表明成功连接数据库：

- ![image-20210412151258243](https://img-blog.csdnimg.cn/img_convert/9d4f44d5f5dd51f61a28efde52feafc2.png)



- 因为初始化密码默认是过期的，必须修改新密码后才能正常使用数据库：

- ```sql
  ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
  ```

  注意：密码太简单可能会报以下错：

  ERROR 1819 (HY000): Your password does not satisfy the current policy requirements

  如果想设置简单的密码，可以根据自己需要来设置以下参数：

- ```csharp
  set global validate_password_policy=LOW; // 设置密码的验证强度等级为低(LOW)
  set global validate_password_length=6; // 设置密码长度为6，最小为4
  ```

  **8** **修改字符集**

- 输入以下语句可以发现数据库和服务端的默认字符都是latin1，如果不修改容易出现乱码：

  ```sql
  show variables like 'character%';
  ```

  ![image-20210412151305503](https://img-blog.csdnimg.cn/img_convert/1bc64ba1337bc703a2dca57ad31884c7.png)



- 输入 "vim /etc/my.cnf " 或用Xftp打开 "/etc/my.cnf " 文件进行编辑，在最后加上

- ```sql
  character_set_server=utf8
  init_connect=’SET NAMES utf8’
  ```



- ![image-20210412151538784](https://img-blog.csdnimg.cn/img_convert/5cb2f7e8da17b981380b1c0d98f63fde.png)



- 重启MySQL服务

- ```undefined
  systemctl restart mysqld
  ```



- 重新连入数据库后修改数据库的字符集（其中mydb为数据库名）

- ```sql
  alter database mydb character set 'utf8';
  ```



- 修改数据库表的字符集（其中mytbl为表名）

- ```sql
  alter table mytbl convert to character set 'utf8';
  ```

  ![image-20210412151545657](https://img-blog.csdnimg.cn/img_convert/c2ce2b9789f1d08a3d7711a70aa40237.png)



**9** **远程访问**

- 输入以下语句查看MySQL的用户信息：

- ![image-20210412151551524](https://img-blog.csdnimg.cn/img_convert/91406878f752fd41a53884bb8533fda5.png)



host：表示连接类型

%：表示所有远程通过 TCP方式的连接
IP地址：如 (192.168.1.2,127.0.0.1) 通过制定ip地址进行的TCP方式的连接
机器名：通过制定i网络中的机器名进行的TCP方式的连接
::1：IPv6的本地ip地址 等同于IPv4的 127.0.0.1
localhost：本地方式通过命令行方式的连接 ，比如mysql -u xxx -p 123xxx 方式的连接。
user：表示用户名

同一用户通过不同方式链接的权限是不一样的。

authentication_string：密码

所有密码串通过password (明文字符串)生成的密文字符串。加密算法为MYSQLSHA1，不可逆。MySQL 5.7的密码保存到authentication_string字段中不再使用password字段(在5.5中使用)。

可见都是localhost，想要用Navicat等管理工具连过来需要增加远程用户，否则远程连接会报以下错误：
![image-20210412151559514](https://img-blog.csdnimg.cn/img_convert/d67da4bc83a235850d23909a811d076a.png)

因此下面给出了用户授权、创建用户、删除用户和修改密码的命令。

- 用户授权命令(该命令执行授权时如果发现没有指定的用户，则会直接创建一个新用户来完成授权)：

- ```vbnet
  grant 权限1,…权限n on 数据库名.表名 to 用户名@用户地址 identified by ‘密码’;
  ```

  例如，授予通过网络方式登录的root用户，有对所有库、所有表的全部权限，密码设为”newpwd123”：

- ```csharp
  grant all privileges on *.* to root@'%' identified by 'newpwd123';
  ```



- 创建用户，例如名为root的用户，密码设为123abc：



```sql
create user root identified by '123abc';
```

- 修改用户名

- ```sql
  update mysql.user **set** user='li4' where user='wang5';
  flush privileges;  # 所有通过user表的修改，必须用该命令才能生效。
  ```



- 删除用户

- ```sql
  drop user user@host;
  ```



- 修改当前用户的密码

- ```python
  set password = password('new_password')
  ```



- 修改某个用户的密码

- ```sql
  update mysql.user set password=password('new_password') where user='name';
  flush privileges; # 所有通过 user表的修改，必须用该命令才能生效。
  ```

  10 相关目录介绍
  文件   位置
  客户端程序及脚本   /usr/bin
  mysqld服务   /usr/sbin
  配置文件   /etc/my.cnf
  数据目录   /var/lib/mysql
  错误日志   /var/log/mysqld.log
  私有安全文件   /var/lib/mysql-files
  系统初始化脚本   /etc/init.d/mysqld
  系统服务   mysqld
  Pid文件   /var/run/mysql/mysqld.pid
  Socket   /var/lib/mysql.mysqld.pid
  字典目录   /var/lib/mysql-keyring
  操作手册   /usr/share/man
  头文件   /usr/include/mysql
  LIB库   /usr/lib/mysql
  共享文件   /usr/share/mysql
  11 可能出现的其它错误
  错误一：在第 6 步时，如果出现以下报错信息：
  ————————————————
  ![image-20210412150522732](https://img-blog.csdnimg.cn/img_convert/246ec3c375fb3c6539b9570c2d27cb7b.png)

  看报错信息里说 " --initialize specified but the data directory has files in it. Aborting. "，说明mysql的数据目录已经存在了。可能是之前安装过mysql，卸载的时候没卸干净，没有连着数据目录一起删除。

  \1. 最简单的解决方法

  已知MySQL的数据目录默认是 /var/lib/mysql，则我们可以对它进行修改：

  终端输入 " vim /etc/my.cnf " 对配置文件进行编辑，修改datadir为自己想存放数据的路径。![image-20210412150528323](https://img-blog.csdnimg.cn/img_convert/944d9b35bdea737d7f81561b2a1fe071.png)

- **2. 最彻底的解决方法**

- 搜索出带 " mysql " 的所有目录：

-  find / -name mysql

-

![image-20210412150631456](https://img-blog.csdnimg.cn/img_convert/4784ecdf9457c859daa43ff7fe1286c5.png)



- 将这些目录递归删除：

- ```bash
  rm -rf /etc/logrotate.d/mysql
  rm -rf /var/lib/mysql
  rm -rf /var/lib/mysql/mysql
  rm -rf /usr/bin/mysql
  rm -rf /usr/lib64/mysql
  rm -rf /usr/share/mysql
  ```



- 重新开始安装。

- 错误二：依然无法远程连接MySQL服务

  检查自己的服务器是否开放了MySQL服务的访问端口，开放了3306端口



### 开启远程访问

```sql
# 登录到SQL中

use mysql;

# 如果所有的都是localhost,则为不允许远程访问
select host from user;

# 开启root用户远程访问
update user set host ='%' where user ='root';

# 权限刷新
flush privileges;



```




**END**


# 附录
## A 资源
## B 参考资料

