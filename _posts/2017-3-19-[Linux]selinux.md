---
layout:     post
title:      "【Linux】SELinux简单策略配置"
date:       2017-03-19 20:13:00
author:     "Yuki"
---

#### SELinux简介

SELinux称安全增强型Linux，它是NSA针对计算机基础结构安全开发的Linux安全策略机制。SELinux是集成在Linux内核中的，所以对SELinux的修改需要重启后才能生效。

我们知道，所有的安全机制都是对两种东西进行限制，一是系统进程，二是系统资源（文件、网络套接字、系统调用等）。

针对这两种类型，SELinux定义了两个基本概念：

* 域（domain）：域用来对进程进行限制
* 上下文（context）：上下文用来对系统资源进行限制

#### 策略

SELinux通过定义策略来控制哪些域可以访问哪些上下文。CentOS/RHEL使用预置的目标（target）策略。目标策略定义只有目标进程收到SELinux的限制，其他进程运行在非限制模式下。

CenTOS/RHEL中，受限制的网络服务程序在200个左右，常见的有：
dhcpd、httpd、mysqld、named、ntpd、rpcbind、squid、syslogd等。

**SELinux模式**

SELinux有三种工作模式：

* 强制（enforcing）

违反策略的行动都被禁止，并作为内核信息记录

* 允许（permissive）

违反策略的行动都不被禁止，但是会产生警告信息

* 禁用（disabled）

禁用SELinux

SELinux模式的配置文件在 /etc/sysconfig/selinux;配置文件中只有两行配置，一是配置SELinux的模式，二是其使用的策略。

**查看和设置工作状态**

* 命令 `getenforce`可以查看当前SELinux的工作状态
* 命令 `setenforce [0 | 1] `可以暂时设置当前SELinux的工作状态，注意，不是永久改变，只是相当于一个开关。其中，0代表permissive，1代表enforcing。

**查看域、上下文信息**

* 命令 `ls -Z`用以查看文件的上下文信息
* 命令`ps -Z` 用以查看进程的域信息

**修改文件上下文**

SELinux策略规定了哪些域可以访问哪些上下文。在对系统进行管理时，对文件的操作有时候会改变文件的上下文，导致一些进程无法访问某些文件，所以我们需要检查、修改文件的上下文。

* 命令 `restorecon  file_name`可以用来恢复一些预置文件的上下文。参数有：

-i 忽略不存在的文件

-f infilename 文件 infilename 中记录要处理的文件

-e directory 排除目录

-R/-r 递归处理目录

-n 不改变文件标签

-o/outfilename 保存文件列表到 outfilename，在文件不正确情况下

-v 将过程显示到屏幕上

-F 强制恢复文件安全上下文

* 命令 chcon 用以修改对象（文件）的安全上下文，比如：用户、角色、类型、安全级别。也就是将每个文件的安全环境变更至指定环境。使用--reference选项时，把指定文件的安全环境设置为与参考文件相同。语法为：`chcon --reference 参考文件 文件`



