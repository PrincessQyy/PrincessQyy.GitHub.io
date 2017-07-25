---
layout:     post
title:      "【Linux】bash shell"
date:       2017-05-22 15:35:00
author:     "Yuki"
---

## 基本知识

bash shell是一种动态的，弱类型的语言。动态语言和静态语言的区别是：静态语言需要预先编译成可执行二进制文件，然后能够独立运行，比如C、C++、JAVA ；而动态语言不需要预先编译，它需要有解释器来解释每一句指令，如python、Perl、shell。强类型和弱类型语言的区别在于：强类型需要严格的在编译器检查变量的类型，而弱类型的检查很弱，仅能严格的区分指令和数据。大多数情况下，静态语言都是强类型的，而动态语言都是弱类型的。

在编写脚本时，第一行要写 `#!/bin/bash` ，这称为魔数，之前说过，shell是一种动态语言，而魔数的作用，就是告知shell调用bash解释器来解释我们的脚本。

## 变量类型

#### 本地变量

作用域为整个bash，其声明格式为：`VARNAME=VALUE`

#### 局部变量

作用域为当前代码段，其声明格式为：`local VARNAME=VALUE`

#### 环境变量

作用域为当前shell进程及其子进程，其声明格式为：`export VARNAME=VALUE`

#### 位置变量

* $1、$2...：代表引用脚本的第一个参数、第二个参数...

* shift ：变量轮换，直接举个例子吧，若在脚本中写：

	echo "$1"
	shift
	echo "$1"

以上程序会输出脚本的第一个参数和第二个参数。

shift n 同理。


#### 特殊变量

* $? ：返回上一个命令的执行状态，可能有两种：

1. 0：正确执行
2. 1-255：错误执行

* $# ：脚本的参数个数

* $*/$@ ：参数列表

* $0 ：脚本名称

#### 变量操作

* 查看shell中的环境变量

`env` `printenv` `export`都可以查看当前shell中的环境变量

* 查看shell中的变量

命令`set`可以当前shell中的所有变量

* 撤销变量

命令`unset VARNAME`用以撤销变量

**脚本只要结束，变量就会被撤销。**

## 运行和退出shell脚本

#### 启动脚本

脚本在执行时会启动一个子shell进程，命令行中启动的脚本会继承当前shell的环境变量，若是系统自启动的脚本（非命令行启动）就需要自定义环境变量。

所以启动脚本有以下几种方式：

* 使用绝对路径来执行脚本

* 进入脚本所在目录，用`./ script_ame`来执行脚本

* 用`bash/sh script_name`来执行脚本

以上几种方式，都是在当前shell的子shell里执行脚本的。下面的第三种方法是在父shell中执行脚本。

* 用`source/. script_name` 来执行脚本

#### 退出shell脚本

在脚本中可以使用 exit status_code 来退出脚本。

#### 检查脚本

* 命令 `bash -n script` 用以检查脚本的语法错误
* 命令 `bash -x script` 用以单步执行脚本


## 重定向

* > 覆盖重定向
* >> 追加重定向
* 2>>错误重定向
* &> 错误和输出重定向
* > /dev/null 重定向到bit bucket数据黑洞中

## 条件判断

#### 条件测试类型

* 整数测试
* 字符测试
* 文件测试


#### 整数比较

* -eq ：测试两个整数是否相等
* -ne ：测试两个整数是否不相等
* -gt ：测试前者是否大于后者
* -lt ：测试前者是否小于后者
* -ge ：测试前者是否大于等于后者
* -le ：测试前者是否小于等于后者

#### 文件比较

* -e file ：测试文件是否存在
* -d file ：测试所给路径是否为一个目录
* -f file ：测试文件是否为普通文件
* -r file ：测试当前用户对指定文件是否有读取权限
* -w file ：测试当前用户对指定文件是否有写权限
* -x file ：测试当前用户对指定文件是否有执行权限

#### 字符比较

* == ：判断是否相等
* != ：判断是否不等
* -n string：判断string是否为空
* -s string：判断字符是否非空

#### 条件测试的表达式

* [ expression ]
* [[ expretion ]]
* test expretion

#### 命令间的逻辑关系

* && :与
* || :或

举个例子吧：

* 若用户user1不存在，就添加用户并设置密码，若用户存在，则输出提示语

方法一：

`id user1 > /dev/null && echo 'The user existed!' || useradd user1 || echo "user1" | passwd --stdin user1 `

方法二：

`! id user1 > /dev/null &&  useradd user1 && echo "user1" | passwd --stdin user1 || echo 'The user existed!' `

#### 组合测试条件

* -a ：与
* -o ：或
* ！ ：非


#### 条件判断的控制结构

* if语法：

    if 判断条件1; then
    	statement1
    	statement2
    	...
    elif 判断条件2;then
    	statement3
    	statement4
    	...
	else
		statement5
    	statement6
    	...
    fi

条件判断中只有使用一些条件测试符，如-eq、-e等，才需要中括号。

* case语法：

	case switch in
	value1)
		statement
		...
		;;
	value2)
		statement
		...
		;;
	...
	esac
	
#### 算术运算

shell中默认所有都是字符，所以无法直接进行算术运算。要通过以下方式：

1. let 算术运算表达式
2. $[算术运算表达式]
3. $((算术运算表达式))
4.  expr 算术运算表达式，表达式中各操作数及运算符之间要有空格，而且要使用反引号

#### 循环的控制结构

**for循环**

* 格式一：

	for 变量 in 列表；do
		循环体
	done

* 格式二：

	for(( expr1 ; expr2 ; expr3 ));do
		statement
		...
	done
  
不多解释了，类似C

**while循环**

	while CONDITION;do
		statement
		...
	done

while循环的condition是条件满足才执行statement.

**until循环**

	until CONDITIOn;do
		statement
		...
	done

until循环的condition是条件不满足才执行statement，直到条件满足了就退出循环。

**如何生成列表**

* {start，end} 生成从start到end之间的列表
* seq [start [step]] end 生成从start到end,步进为step的列表，这种方式是命令，所以要用反引号``
* 直接写能产生列表的命令，用反引号``引起来，如ls命令就可以产生列表




