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

#### SELinux的启动、关闭和查看

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

#### SELinux的安全性环境

SELinux通过定义策略来控制哪些域可以访问哪些上下文。CentOS/RHEL使用预置的目标（target）策略。目标策略定义只有目标进程收到SELinux的限制，其他进程运行在非限制模式下。

CenTOS/RHEL中，受限制的网络服务程序在200个左右，常见的有：
dhcpd、httpd、mysqld、named、ntpd、rpcbind、squid、syslogd等。

SELinux的安全性环境存在于主体程序和目标文件资源中，程序在内存中，因此安全性环境存入是没问题的，那文件的安全性环境记录在哪里呢？事实上，文件的安全性环境是记录在文件的inode内的。所以当主体程序需要读取目标资源时，同样需要读取文件的inode就可以比对安全性环境以及rwx等权限值是否正确，从而给予适当的读取权限。

**查看域、上下文信息**

* 命令 `ls -Z`用以查看文件的上下文信息
* 命令`ps -Z` 用以查看进程的域信息

如下图查看文件的上下文信息：

<img src="../../../../../img/blogs/selinux/01.png">

安全性环境主要由冒号分为三个字段：

Indentify:role:type

这三个字段的含义如下：

Indentify：身份识别，相当于账号的身份识别，主要的身份识别有以下几种：

1. root：表示root账户身份
2. system_u：表示系统程序
3. user_u：表示一般用户身份

role：角色，通过这个字段可以知道这个数据是代表程序、文件资源还是用户。主要类型如下：

1. object_r：代表是文件或目录等资源，这是最常见的
2. system_r：代表是程序，不过，一般用户也会被指定为这个类型

type：类型，在默认的targeted策略中，Indentify和role字段基本是不重要的，一个主体程序是否能读取到某个文件资源，与类型字段有关。类型字段在文件和程序中的定义不同。如下：

1. Type：在文件资源中称为类型
2. Domain：在主体程序中称为域

通过Indentify和role搭配，可以大概知道某个程序代表的意义：

1. root：system_r 代表供root账户登录时所取得的权限
2. system_u:system_r 由于是系统账号，因此是非交互式的系统运行程序
3. user_u:system_r 一般可登录用户的程序

主体程序所取得的Domain和目标文件资源的Type之间的关系如下：

<img src="../../../../../img/blogs/selinux/02.png">


**修改文件上下文**

SELinux策略规定了哪些域可以访问哪些上下文。在对系统进行管理时，对文件的操作有时候会改变文件的上下文，导致一些进程无法访问某些文件，比如，复制文件操作，SELinux的Type字段会继承自目标目录，而移动操作，连SELinux的类型也会被移动过去，所以我们需要检查、修改文件的上下文。


* 命令 `restorecon  file_name`可以用来恢复一些预置文件的上下文。参数有：

-i 忽略不存在的文件

-f infilename 文件 infilename 中记录要处理的文件

-e directory 排除目录

-R/-r 递归处理目录

-n 不改变文件标签

-o/outfilename 保存文件列表到 outfilename，在文件不正确情况下

-v 将过程显示到屏幕上

-F 强制恢复文件安全上下文

* 命令 chcon 用以修改对象（文件）的安全上下文，比如：用户、角色、类型、安全级别。也就是将每个文件的安全环境变更至指定环境。

基本语法为：

`chcon [-R] [-t type] [-u user] [-r role] 文件`

选项和参数：

-R 连同下面的子目录也一起修改

-t 后面接安全性环境的字段，例如 httpd_sys_content_t

-u 后面接身份识别，例如 system_u

-r 后面接角色，例如 system_r


--reference=范例文件 把指定文件的安全环境设置为与参考文件相同。语法为：`chcon --reference=范例文件 要修改文件`

#### semanage查询和修改

SELinux Type会在移动/复制的过程中产生一些变化，所以需要restorecon和chcon命令来修改。那这里就有一个疑问了，怎么restorecon会知道本来的SELinux Type呢？这是因为系统有记录嘛，记录在/etc/selinux/targeted/contexts这个目录下。但是该目录下有很多数据，要通过文本查看命令查看的话会很麻烦，所以就有了semanage这个功能来查询和修改。

基本语法：

`semanage {login|user|port|interface|fcontext|translation} -l `

`semanage fcontext -{a|d|m} {-first} file_spec`

选项与参数：

fcontext 主要用在安全性方面，-l用于查询

-a 增加，你可以增加一些目录的默认安全性环境类型设置

-m 修改

-d 删除

举例：

* 查询某个目录的默认安全性环境类型设置

`semanage fcontext -l | grep directory`

directory可以使用正则表达式去指定范围

* 设置目录的默认安全环境为 public_content_t

`semanage fcontext -a -t public_content_t  "directory(/.*)?"` 

#### SELinux策略

**查看规则**

centos6.X默认使用target策略，可以使用`seinfo`来查询策略提供的规则。

基本语法：

`yum install setools-console` 安装相关工具

`seinfo [-Atrub]` 

选项和参数：

-A 列出SELinux的状态、规则布尔值、身份识别、角色、类别等所有信息

-t 列出SELinux的所有type种类

-r 列出SELinux的所有role种类

-u 列出SELinux的所有user种类

-b 列出所有的规则种类（布尔值）

**查看详细规则**

使用seinfo查询到相关的类别或是布尔值后，想要知道详细规则，可以使用`sesearch`命令。

基本语法：

sesearch [--all] [-s 主体类别] [-t 目标类别] [-b 布尔值]

选项和参数：

--all 列出该类别或布尔值的所有相关信息

-t 后面接类别

-b 后面接布尔值

举例：

* 找出目标资源类别为httpd_sys_content_t的有关信息

`sesearch --all -t httpd_sys_content_t `

出现如下信息：

<img src="../../../../../img/blogs/selinux/02.png">

格式为：
`allow 主体程序安全性环境类别 目标文件安全性环境类别`

说明这个类别可以被那个主题程序的类别所读取，以及目标文件资源的格式。

* 查看布尔值httpd_enable_homedirs规范了什么

`sesearch --all -b httpd_enable_homedirs`

出现如下信息：

<img src="../../../../../img/blogs/selinux/02.png">

布尔值规范了非常多的主体程序和目标文件资源的放行与否，你的主体程序能否对目标文件资源进行访问，与布尔值非常有关系。因为布尔值可以设置为0（关闭）和1（启动）。

**布尔值的查询**

既然主体程序和目标文件资源能否有访问权限是由布尔值控制的，系统有多少可以通过`seinfo -b` 来查询，那每个布朗值是启动还是关闭的就可以通过getsebool来查看。

基本语法：

`getsebool [-a] [布尔值条款]`

选项与参数：

-a 列出系统上所有布尔值的开启与否

后面直接跟要查询的布尔值条款就可以查询具体某个布尔值的启动与否

**布尔值的修改**

布尔值的修改通过setsebool来完成。

基本语法：

`setsebool [-P] 布尔值条款=[0|1]`

选项与参数：

-P 将设置写入配置文件

#### SELinux日志文件记录所需的服务

上述的命令都是为了当你的某些网络服务无法提供相关功能时才需要修改的，但是，我们怎么知道哪个时候才需要进行这些命令的修改呢？我们怎么知道系统是因为SELinux的问题而导致网络服务出问题的呢？所以，检测在登录SELinux时产生的错误就很有用啦~这是通过auditd和setroubleshoot服务来完成的。

**setroubleshoot**

启动这个服务之前需要安装 setroubleshoot 和 setroubleshoot-server。





















