---
layout:     post
title:      "【Linux】awk使用总结"
date:       2017-01-01 20:44:00
author:     "Yuki"
---

## awk基本介绍

awk 与其说是文本处理器，不如说是个报告生成器，它的主要功能是从文本文件中抽取符合条件的信息并以特定的格式显示。在Linux上我们使用的是gawk,在有些使用风格上，并不与POSIX完全兼容。

#### awk处理机制

awk在执行时一次从文件中抽取一行文本，自动将这行文本按字符串的分隔符进行分隔，这些分隔后的片在其内部会使用变量来引用，这和脚本类似，用$1、$2来标识切片，$0标识整行文本。

## awk使用总结

#### 基本语法

`awk [options] 'script' file1 file2 ...`

`awk [options] 'pattern {action}'file1 file2 ...`

第二句就是第一句脚本的具体表现形式，也就是说 script 由 pattern 和 action 组成，action一定要用大括号括起来，pattern 和 action要用单引号引起来，action 中若有多个语句，语句之间以分号分隔（类似于编程语言~ ）。

#### awk的输出

**print关键字**

print的使用格式：`print item1,item2...` 各项目之间以逗号分开，但输出时默认分隔符是空格，可以用OFS指定分隔符。

输出的item可以为字符串或数值、当前记录的字段（如$1）、变量或awk的表达式，数值会先转换成字符串再输出。

item可以省略，此时功能相当于 `print $0`,因此，要输出空白行，可以使用 `print ""`

* 举例

`awk -F: '{print $1,$2}' /etc/passwd` 输出/etc/passwd的以冒号分隔的第一个字段和第二个字段，如下图：

<img src="../../../../../img/blogs/awk/02.png">

`awk -F: '{OFS="#"} {print $1,$2}' /etc/passwd` 输出/etc/passwd的以冒号分隔的第一个字段和第二个字段,输出时以#分隔，如下图：
<img src="../../../../../img/blogs/awk/01.png">

`awk 'BEGIN {print "Line one\nLine two\n"}'` BEGIN是一个特殊的模式，此模式后面可以不跟文件，这个例子的输出如下：

<img src="../../../../../img/blogs/awk/03.png">

**printf关键字**

printf关键字用以格式化字符串输出。 他不会自动换行。

printf使用语法为：`printf format,item1,item2...`

format格式的指示符都以%开头，后跟一个字符，整体要用双引号引起来，如下：

* %c：显示字符
* %d、%i：显示十进制整数
* %e、%E：用科学计数法显示数值
* %g、%G：以科学计数法或浮点数的格式显示数值
* %f：显示浮点数
* %s：显示字符串
* %u：显示无符号整数
* %%：显示%自身，哈哈

每个format格式符都可以用修饰符修饰，修饰符有下：

* N：指定显示宽度
* -：左对齐
* +：显示数值符号

* 举例

<img src="../../../../../img/blogs/awk/04.png"> 

<img src="../../../../../img/blogs/awk/05.png">

<img src="../../../../../img/blogs/awk/06.png">

#### awk内置变量

**记录变量**

* FS：field separator，指定读取文本时所使用的字段分隔符，功能相当于 -F，默认是空格
* RS：record separator，指定读取文本时所使用的的换行符，默认就是换行符\n
* OFS：output field separator，指定输出文本时所使用的字段分隔符。
* ORS：output record separator，指定输出文本时所使用的换行分隔符。

**数据变量**

* NR：多个文件中处理的行总数
* FNR：每个文件中正在处理的行数
* NF：当前记录的字段数

**自定义变量** 

* `awk -v var_name="value" 'pattern {action}' file1 file2...`
* `awk 'BEGIN {var_name="value";action}'`

#### awk操作符

#### awk的模式

**常见模式类型**

* regex：正则表达式，格式为`/regular exception/`
* exception：表达式，其值非0或字符非空时满足条件，如：`$1　~ /foo/` 或` $1 == "foo"` ，用运算符~(匹配)和！~(不匹配)。
* ranges：指定范围，使用格式为part1，part2。
* BEGIN/END：特殊模式，仅在awk命令执行前/执行后运行一次 
* EMPTY：空模式，即对文件的每一行都做处理。

**常见的Action**




 