---
layout:     post
title:      "【MySQL】数据的查找"
date:       2017-03-04 21:05:00
author:     "Yuki"
---

#### 基本的 select 语句

我们通过 select 语句来查找所需的内容。其基本格式为：

	`SELECT column,column2,column3...FROM table_name WHERE conditions`

如果要查询表的所有内容，则把 要查询的列名 用一个星号 * 号表示，代表要查询表中所有的列。

#### 数学符号条件

SELECT 语句通常需要 WHERE 限制条件，来达到更加精确的查询。

在操作条件中可以使用：>、>=、=、<=等计算符。

不等于符号：在SQL中如果要想使用不等于符号，可以有两种形式：“<>”、“!=”

#### AND 和 OR

WHERE 后面可以有不止一条限制，而根据条件之间的逻辑关系，可以用 OR(或) 和 AND(且) 连接多个条件。AND 代表“且”，表示两个条件必须同时满足，OR 则代表“或”，表示两个条件只要满足其一即可。

#### IN 和 NOT IN

关键词IN和NOT IN的作用和它们的名字一样明显，用于筛选“在”或“不在”某个范围内的结果。

#### 通配符

关键字 LIKE 在SQL语句中和通配符一起使用，通配符代表未知字符。SQL中的通配符是 _ 和 % 。其中 _ 代表一个未指定字符，% 代表不定个未指定字符。

比如，要只记得电话号码前四位数为1101，而后两位忘记了，则可以用两个 _ 通配符代替：


    SELECT name,age FROM my_contacts WHERE phone LIKE '1101__';

#### 对结果排序

为了使查询结果看起来更顺眼，我们可能需要对结果按某一列来排序，这就要用到 ORDER BY 排序关键词。默认情况下，ORDER BY的结果是升序排列，而使用关键词ASC和DESC可指定升序或降序排序。 

#### SQL 内置函数和计算

SQL 允许对表中的数据进行计算。对此，SQL 有 5 个内置函数，这些函数都对 SELECT 的结果做操作：

<img src="../../../../../img/blogs/data searching/data_searching.jpg">

其中 COUNT 函数可用于任何数据类型(因为它只是计数)，而另4个函数都只能对数字类数据类型做计算。

具体举例，比如计算出salary的最大、最小值，用这样的一条语句：
    
    SELECT MAX(salary) AS max_salary,MIN(salary) FROM employee;

附：使用AS关键词可以给值重命名，比如最大值被命名为了max_salary：