---
layout:     post
title:      "【Nginx】Nginx部署、配置及优化"
date:       2017-05-08 14:51:00
author:     "Yuki"
---
## Nginx

能理解七层协议，处于七层的负载均衡，工作于用户空间。转发能力不如HAProxy，不过差别不大，但是其对系统资源的需求很小，并且有缓存功能，缓存opCode（PHP编译后的中间语言）执行的结果，为了避免程序反复执行，加缓存，让用户请求异步转发至Real Server。

**OPCode：**PHP执行代码会经过如下4个步骤(确切的来说，应该是PHP的语言引擎Zend)

1. Scanning(Lexing) ,将PHP代码转换为语言片段(Tokens)
2. Parsing, 将Tokens转换成简单而有意义的表达式
3. Compilation, 将表达式编译成Opocdes
4. Execution, 顺次执行Opcodes，每次一条，从而实现PHP脚本的功能。

## Nginx 安装

说到安装，因为我用的是centos系统，而Nginx位于第三方yum源里，而不在centos的官方源里，所以要先安装EPEL（Extra Packages for Enterprise Linux。所以安装Nginx流程如下：

1. 先去网站 [](http://fedoraproject.org/wiki/EPEL)把你系统对应的EPEL包下载下来，我是用`wget`的方式。 
2. 安装你下载完的EPEL包，可以用`rpm -vhi`的方式。
3. 用`yum install nginx`安装Nginx。