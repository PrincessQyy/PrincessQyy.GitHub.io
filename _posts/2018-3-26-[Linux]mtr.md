---
layout:     post
title:      "【Linux】mtr使用总结"
date:       2018-03-26 20:19:00
author:     "Yuki"
---

在Linux中有一个很好的网络连通性判断工具，它可以结合ping nslookup tracert 来判断网络的相关特性,这个命令就是mtr。

## 基本语法

* 常用参数

-n  不用主机解释

-c   指定发送多少个数据包

--report  结果显示，并不动态显示

-a 来设置发送数据包的IP地址 这个对一个主机由多个IP地址是有用的 

-s 用来指定ping数据包的大小 