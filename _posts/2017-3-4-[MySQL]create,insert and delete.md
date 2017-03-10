---
layout:     post
title:      "【MySQL】表的创建、插入、删除"
date:       2017-03-04 19:06:00
author:     "Yuki"
---

#### 表的创建

* 使用语句 `CREATE DATABASE database_name;` 来创建数据库。
* 要创建表需先用语句 `USE database_name; `来选定数据库，然后用如下语句创建表：
    `CREATE TABLE table_name(Column1 Type1 Constraint1 ,Column2 Type2 Constraint2,Column3 Type3 Constraint3...);`

数据类型有：
<img src="../../../../../img/blogs/create,insert and delete/data_type.jpg">

约束我单独放到一个内容吧，这里就先不提了。

#### 表的插入

* 用语句 `INSERT INTO table_name(Colum1,Column2,Column2)...VALUES(Value1,Value2,Value3..);`

若是插入的数据和创建表的时每一列要填入的数据一致的话，可以省略Column1、Column2、Column3...

有的数据需要用单引号括起来，CHAR、VARCHAR, TEXT, DATE, TIME,ENUM 等类型的数据需要单引号修饰，而 INT,FLOAT,DOUBLE 等则不需要。

#### 数据库和表的删除

* 用语句`DROP DATABASE database_name` 来删除一个数据库。

* 用语句 `DROP TABLE table_name` 来删除一张表。