---
layout:     post
title:      "【Linux】shell scripts编写上要注意的点"
date:       2017-04-30 19:43:00
author:     "Yuki"
---

这篇博客，持续更新，主要是记录我作为新人踩过的坑...

* if 后面一定要有空格，若是if后面加[]判断的话，[]括号里开头和结束都要有括号

* $引用变量的值，要用{}，例如，引用变量site的值，${site}

* 使用sed 如果要在特定位置插入字符串，而特定位置是先用变量得到得到的，比如pos里存着行号，那么，在用sed 的时候只能用双引号来引用变量的值 ，举个例子  在 pos 后插入字符串“hahaha” , `sed -i "${pos}a\hahaha" file_name`

* 使用echo如果想要显示制表符啥的，需要加上-e选项才能进行格式控制，例如：`echo -e "hello\tworld"` 

