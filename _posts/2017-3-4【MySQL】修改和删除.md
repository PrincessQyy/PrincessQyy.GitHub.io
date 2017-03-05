---
layout:     post
title:      "【MySQL】修改和删除"
date:       2017-03-04 20:35:00
author:     "Yuki"
---
## 对表的修改

#### 重命名一张表

重命名一张表如下：
    
	`RENAME TABLE name TO new_name;

	 ALTER TABLE name RENAME new_name;

	 ALTER TABLE name RENAME TO new_name; `

#### 改变表结构

**增加一列：** 在表中增加一列的语句格式为：
    
`ALTER TABLE table_name ADD COLUMN column_name Type Constraint;` 

 其中，COLUMN可以省略不写。 

**删除一列：**删除表中某列的语句格式和添加十分相似。

    `ALTER TABLE table_name DROP COLUMN column_name;`

同样，COLUMN也可以省略不写。

**重命名一列：**这条语句准确的说是对一个列做修改

	`ALTER TABLE table_name CHANGE column_name new_column_name Type Constraint;`

注意，Type一定要有，不然修改失败！当原列名和新列名相同的时候，指定新的数据类型或约束，就可以用于修改数据类型或约束。需要注意的是，修改数据类型可能会导致数据丢失，所以要慎重使用。

**改变数据类型：**除了上一种 CHANGE 方法外，还可以用 MODIFY 来修改数据类型。

	`ALTER TABLE table_name MODIFY column_name new_type;`

#### 改变表中的数据

**修改表中的值：**有时候，我们需要对表中某一个数据进行修改。

	`UPDATE TABLE table_name set column1=value1,column2=value2...WHERE conditions;`

**删除一行记录：**删除某一行数据的语句如下：

	`DELETE FROM table_name WHERE conditions;`
