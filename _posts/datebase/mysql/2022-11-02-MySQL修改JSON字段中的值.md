---
layout: articles
title: MySQL修改JSON字段都值
tags:  JSON
author: Warning
key:    datebase-mysql-head-17
aside:
  toc: true
sidebar:
nav: DateBase
category: [datebase, mysql]
---

#

因为某种很傻逼的原因,项目中的字段是存到了一个JSON中, 现在有个功能要批量更新JSON中的某个字段都值

虽然可以拿出来在内存里改完再塞回去, 可这尼玛也太沙雕了. . .

而且这个表每个月要进十多万条数据, 这么搞明显不妥

既然Mysql能取JSON的值, 就一定能改


<!--more-->



# JSON_REPLACE()函数修改JSON属性值

#


下面就来简单说说MySQL中用于替换JSON属性值的这个函数：**[`JSON_REPLACE(`json_doc`, `path`, `val`[, `path`, `val`] ...)`](https://dev.mysql.com/doc/refman/5.7/en/json-modification-functions.html#function_json-replace)**。

# 官方描述

JSON_REPLACE()：

官方的第一句解释是：Replaces existing values in a JSON document and returns the result。就冲这句话，基本可以锁定它就是我要找的解决方案。我们来看一下官方的示例：

```sql
mysql> SET @j = '{ "a": 1, "b": [2, 3]}';
mysql> SELECT JSON_REPLACE(@j, '$.a', 10, '$.c', '[true, false]');
+-----------------------------------------------------+
| JSON_REPLACE(@j, '$.a', 10, '$.c', '[true, false]') |
+-----------------------------------------------------+
| {"a": 10, "b": [2, 3]}                              |
+-----------------------------------------------------+
```

上面的例子很好理解，**首先，它定义了一个JSON字符串变量，然后通过JSON_REPLACE函数将这个变量中的属性a的值由1 改为了10，然后将属性c的值改为了‘[true, false]’，但由于json字符串中并没有属性 c 因此只有属性 a 修改成功了**。（这里需要注意，mysql函数仅仅是做数据转换，并不涉及到真正的增删改查，因此还需要配合具体的**UPDATE**才能够真正更新数据）

看过这个例子之后，我思考了一下我的应用场景，只需要在更新的sql语句中调用**JSON_REPLACE**函数，将json对象所对应的字段比作上面的JSON字符串变量，然后通过 ‘$xxx’ 匹配到我所希望修改的值，然后就可以成功修改JSON对象中的属性了。

# 验证测试

现在我有一张**employee表**，它的**last_name**字段是一个简单的JSON字符串：

![img](https://raw.githubusercontent.com/war-ning/Pic/master/img/20190118111500209.png)

现在，我希望修改**emp_id = 1**的记录，将这条记录中last_name中的**name**改成“**Harry**”，**gender**改成 “**女**”。于是我执行了下面的SQL：

```sql
UPDATE employee
SET last_name = json_replace(last_name, '$.name', "Harry", "$.gender", "女")
WHERE emp_id = 1;
```

这里注意，我特意将 **"$.gender"** 写为双引号，目的就是为了测试这样的通配是否可以生效，执行成功后，查询结果如下：

![img](https://raw.githubusercontent.com/war-ning/Pic/master/img/20190118111629414.png)

可以看到，last_name字段中的name属性已经修改成Harry，gender也被修改成了“女”，因此通配符 用双引、单引都是可以的。

除了JSON_REPLACE()函数，还有很多其他的json函数，如JSON_REMOVE()等，具体函数用法可以查看官方文档：

https://dev.mysql.com/doc/refman/5.7/en/json-functions.html

里面还贴心的将用法进行了归类：

![img](https://raw.githubusercontent.com/war-ning/Pic/master/img/20190118112136618.png)

但是，这些函数应该都对mysql数据库的版本比较敏感，一般对**MySQL5.7**以上的版本支持比较理想，个别函数可能需要5.7.8以上，我的数据库是5.7.23，因此JSON_REPLACE()用着还可以，如果你的版本在5.7以下，就得好好看看是否支持这些函数了。

简单查看MySQL数据库的版本：SELECT VERSION();

![img](https://raw.githubusercontent.com/war-ning/Pic/master/img/20190118112554115.png)






# 附录
## A 资源
## B 参考资料


