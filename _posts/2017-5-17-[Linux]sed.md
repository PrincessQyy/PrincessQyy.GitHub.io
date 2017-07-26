---
layout:     post
title:      "【Linux】sed使用总结"
date:       2017-05-17 14:03:00
author:     "Yuki"
---

## sed基本介绍

sed，流编辑器，它是文本处理器，用于处理纯ASCII文本，在处理文本时是逐行读取文本到内存中，在内存中完成模式匹配，若匹配到了则在模式空间中完成编辑命令，处理结束后，将模式空间中的内容输出到屏幕。完成模式匹配的内存空间称为模式空间。 为什么叫模式空间呢？因为被处理的行可以做模式过滤。 

默认情况下，sed不编辑原文件，仅对模式空间中的数据做处理。

#### sed用法总结

#### 基本语法：

`sed [options] 'AddressCommand' file1 file2 ...`

**options:**

-n 静默模式，只跟命令相关，不显示模式空间中的内容

-i 直接修改原文件

-e script -e script ... ：可同时执行多个脚本， 'AddressCommand'称为脚本

-f /path/to/sed_script file：将路径/path/to/sed_script中的脚本用在文件file上

-r 使用扩展正则表达式 


**Address:** 

1. startLine,endLine ：指定起始行和结束行，中间以逗号作为分隔
2. /Regex/ ：以正则表达式来指定地址，例如/^root/表示指定以root为开头的行
3. /pattern1/,/pattern2/ ：表示从第一次被模式1匹配到的行到第一次被模式2匹配到的行为止，中间的所有行
4. LineNumber ：指定某行
5. $ ：最后一行； $-1 ：倒数第二行
6. StartLine,+N ：从StartLine指定的行开始，向后的N行（共N+1行）
7. 不写表示全文

**Command：**

d ：删除指定行 

p ：显示符合条件的行，匹配的行会显示两次，未匹配的显示一次 

a \string ：在指定行后面追加新行string,可以使用\n来换行

i \string ：在指定行前面增加新行string,可以使用\n来换行
 
r filename：将指定文件的内容添加至符合条件的行处，这可以用于合并文件，通常都用于合并在文件中间处

w filename：将地址指定的范围内的行另存至指定文件中 

s/pattern/string/修饰符 ：将地址指定的范围内的行中符合模式匹配的内容替换为指定的string ，默认只替换行中第一次被模式匹配到的string，加修饰符g表示全局替换，加修饰符i表示查找时忽略字符大小写 

s###、s@@@和s///含义一样，只是分隔符不同了，如果你要匹配的内容中有/，就可以使用其他分隔符啦，这样就可以不用转义喔~

()\1,\2... ：在查找替换中,后向引用

& ：在查找替换中，将引用模式匹配到的整个串

**举例**


1.将文本中以l开头e结尾的单词后面加上r

`sed 's/l..e/&r/g' filename`

2.将文本中以l开头e结尾的单词的l变成大写L

`sed 's#l(..e)#L\1#g' filename`

3.使用sed将行首空白字符删除

`sed -r 's/^[[:space:]]+//g' filename`

4.删除空白行

`sed '/^$/d' filename`

5.删除以#号开头的#和#后面的空白符，要求#后面必须有空白符

`sed -r 's/^#[[:space:]]+//g filename'`

4.取出某路径的父目录

`sed -r 's/(.*\/)+.*/\1/' filename'      '`





