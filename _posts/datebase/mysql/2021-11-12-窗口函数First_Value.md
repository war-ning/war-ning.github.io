---
layout: articles
title: MySQL FIRST_VALUE 函数
tags:  mysql
author: Warning
key:    datebase-mysql-head-11
aside:
  toc: true
sidebar:
nav: DateBase
category: [datebase, mysql]
---

最近项目中写的Sql需要分组取第一行，拿出来在内存里处理并且分页太蠢了，突然看到了有个First_Value，完美适用
转了个教程帖子，顺便记录一下自己的SQL

<!--more-->



## FIRST_VALUE() 函数概述

`FIRST_VALUE()`是一个[窗口函数](https://www.begtut.com/mysql/mysql-window-functions.html)，允许您选择窗口框架，分区或结果集的第一行。

以下说明了`FIRST_VALUE()`函数的语法：

```
FIRST_VALUE(expression) OVER (
        [partition_clause]
        [order_clause]
        [frame_clause]
)
```

在这个语法中：

### 表达

`FIRST_VALUE()`函数返回`expression`窗口框架第一行的值。

`OVER`条款由三个条款：`partition_clause`，`order_clause`，和`frame_clause`。

### `partition_clause`

`partition_clause`子句将结果集的行划分为函数独立应用的分区。`partition_clause`语法如下：

```
PARTITION BY expr1, expr2, ...
```

### `order_clause`

`order_clause`子句指定`FIRST_VALUE()`函数操作的每个分区中行的逻辑顺序。以下显示了以下语法`order_clause`：

```
ORDER BY expr1 [ASC|DESC], expr2 [ASC|DESC], ...
```

### `frame_clause`

`frame_clause`定义当前分区的子集（或帧）。有关frame子句语法的详细信息，请查看[窗口函数教程](https://www.begtut.com/mysql/mysql-window-functions.html)。

------

## MySQL FIRST_VALUE() 函数示例

以下语句创建一个名为`overtime`的新表，并为演示插入示例数据：

```
CREATE TABLE overtime (
    employee_name VARCHAR(50) NOT NULL,
    department VARCHAR(50) NOT NULL,
    hours INT NOT NULL,
    PRIMARY KEY (employee_name , department)
);

INSERT INTO overtime(employee_name, department, hours)
VALUES('Diane Murphy','Accounting',37),
('Mary Patterson','Accounting',74),
('Jeff Firrelli','Accounting',40),
('William Patterson','Finance',58),
('Gerard Bondur','Finance',47),
('Anthony Bow','Finance',66),
('Leslie Jennings','IT',90),
('Leslie Thompson','IT',88),
('Julie Firrelli','Sales',81),
('Steve Patterson','Sales',29),
('Foon Yue Tseng','Sales',65),
('George Vanauf','Marketing',89),
('Loui Bondur','Marketing',49),
('Gerard Hernandez','Marketing',66),
('Pamela Castillo','SCM',96),
('Larry Bott','SCM',100),
('Barry Jones','SCM',65);
```

### 1）`FIRST_VALUE()`在整个查询结果集示例中使用MySQL 函数

以下语句获取员工姓名，加班时间和加班时间最少的员工：

```
SELECT
    employee_name,
    hours,
    FIRST_VALUE(employee_name) OVER (
        ORDER BY hours
    ) least_over_time
FROM
    overtime;
```

这是输出：

```
+-------------------+-------+-----------------+
| employee_name     | hours | least_over_time |
+-------------------+-------+-----------------+
| Steve Patterson   |    29 | Steve Patterson |
| Diane Murphy      |    37 | Steve Patterson |
| Jeff Firrelli     |    40 | Steve Patterson |
| Gerard Bondur     |    47 | Steve Patterson |
| Loui Bondur       |    49 | Steve Patterson |
| William Patterson |    58 | Steve Patterson |
| Barry Jones       |    65 | Steve Patterson |
| Foon Yue Tseng    |    65 | Steve Patterson |
| Anthony Bow       |    66 | Steve Patterson |
| Gerard Hernandez  |    66 | Steve Patterson |
| Mary Patterson    |    74 | Steve Patterson |
| Julie Firrelli    |    81 | Steve Patterson |
| Leslie Thompson   |    88 | Steve Patterson |
| George Vanauf     |    89 | Steve Patterson |
| Leslie Jennings   |    90 | Steve Patterson |
| Pamela Castillo   |    96 | Steve Patterson |
| Larry Bott        |   100 | Steve Patterson |
+-------------------+-------+-----------------+
17 rows in set (0.01 sec)
```

在此示例中，`ORDER BY`子句按结果集对行中的行进行了按小时排序，并`FIRST_VALUE()`选择了第一行，表示加班时间最短的员工。

### 2）`FIRST_VALUE()`在分区示例中使用MySQL

以下语句查找每个部门加班时间最少的员工：

```
SELECT
    employee_name,
    department,
    hours,
    FIRST_VALUE(employee_name) OVER (
        PARTITION BY department
        ORDER BY hours
    ) least_over_time
FROM
    overtime;
```

输出是：

```
+-------------------+------------+-------+-----------------+
| employee_name     | department | hours | least_over_time |
+-------------------+------------+-------+-----------------+
| Diane Murphy      | Accounting |    37 | Diane Murphy    |
| Jeff Firrelli     | Accounting |    40 | Diane Murphy    |
| Mary Patterson    | Accounting |    74 | Diane Murphy    |
| Gerard Bondur     | Finance    |    47 | Gerard Bondur   |
| William Patterson | Finance    |    58 | Gerard Bondur   |
| Anthony Bow       | Finance    |    66 | Gerard Bondur   |
| Leslie Thompson   | IT         |    88 | Leslie Thompson |
| Leslie Jennings   | IT         |    90 | Leslie Thompson |
| Loui Bondur       | Marketing  |    49 | Loui Bondur     |
| Gerard Hernandez  | Marketing  |    66 | Loui Bondur     |
| George Vanauf     | Marketing  |    89 | Loui Bondur     |
| Steve Patterson   | Sales      |    29 | Steve Patterson |
| Foon Yue Tseng    | Sales      |    65 | Steve Patterson |
| Julie Firrelli    | Sales      |    81 | Steve Patterson |
| Barry Jones       | SCM        |    65 | Barry Jones     |
| Pamela Castillo   | SCM        |    96 | Barry Jones     |
| Larry Bott        | SCM        |   100 | Barry Jones     |
+-------------------+------------+-------+-----------------+
17 rows in set (0.02 sec)
```

在这个例子中：

- 首先，`PARTITION BY`条款将员工按部门划分为分区。换句话说，每个分区由属于同一部门的员工组成。
- 其次，`ORDER BY`子句指定了每个分区中的行顺序。
- 第三，`FIRST_VALUE()`按小时排序对每个分区进行操作。它返回了每个分区中的第一行，分区是部门内加班时间最少的员工。



## 当时写的SQL

```sql
        SELECT COUNT(*)
        FROM (
                SELECT
                DISTINCT RYBH,
                FIRST_VALUE (CLJG) OVER(
                    PARTITION BY RYBH
                    ORDER BY CREATE_TIME DESC) CLJG
                FROM CLJG
                WHERE DELETED = 'F') cljg
        LEFT JOIN KSSRYJBXXB ry ON cljg.RYBH = ry.RYBH
        WHERE ( <![CDATA[ZSZT & 1<<1]]> != 0 OR <![CDATA[ZSZT & 1<<2]]> != 0 OR <![CDATA[ZSZT & 1<<3]]> != 0)
            AND cljg.CLJG = '5'
            AND ry.DELETED = 'F'
```


------





# 附录
## A 资源
## B 参考资料


