---
layout: articles
title: Mysql的Json查询
tags:  JSON
author: Warning
key:    datebase-mysql-head-18
aside:
  toc: true
sidebar:
nav: DateBase
category: [datebase, mysql]
---

#

2023-02-10-JSON查询中 ->和->>的区别


<!--more-->




## MySQL的json查询的->、->>区别



### 表数据（member）

```
id|info                     |
--+-------------------------+
 1|{"id": 1, "name": "aaaa"}|
 2|{"id": 2, "name": "bbbb"}|
 3|{"id": 3, "name": "cccc"}|
 4|{"id": 4, "name": "dddd"}|
```





### 一、 ->在field中使用的时候结果带引号，->>的结果不带引号

`select info->"$.name" from member;`

```
info->"$.name"|
--------------+
"aaaa"        |
"bbbb"        |
"cccc"        |
"dddd"        |
```





`select info->>"$.name" from member;`

```
info->>"$.name"|
---------------+
aaaa           |
bbbb           |
cccc           |
dddd           |
```





### 二、where查询->需要注意类型的，->>不用注意类型的

`select * from member where info->"$.id" = 1;`

```
id|info                     |
--+-------------------------+
 1|{"id": 1, "name": "aaaa"}|
```





`select * from member where info->"$.id" = "1";`

```
id|info|
--+----+
```





`select * from member where info->>"$.id" = 1;`

```
id|info                     |
--+-------------------------+
 1|{"id": 1, "name": "aaaa"}|
```





`select * from member where info->>"$.id" = "1";`

```
id|info                     |
--+-------------------------+
 1|{"id": 1, "name": "aaaa"}|
```






# 附录
## A 资源
## B 参考资料


