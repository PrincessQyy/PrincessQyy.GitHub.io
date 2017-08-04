---
layout:     post
title:      "【Nginx】Nginx部署、配置及优化"
date:       2017-05-08 14:51:00
author:     "Yuki"
---
## Nginx

Nginx作为反向代理，能理解七层协议，处于七层的负载均衡，工作于用户空间。转发能力不如HAProxy，不过差别不大，但是其对系统资源的需求很小，并且有缓存功能，可以缓存opCode（PHP编译后的中间语言）执行的结果，为了避免程序反复执行，加缓存，让用户请求异步转发至Real Server。

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

## Nginx配置

Nginx的核心模块为Main和Events，此外还包括标准HTTP模块、可选HTTP模块和邮件模块，其还可以支持诸多第三方模块。Main用于配置错误日志、进程及权限等相关的参数，Events用于配置IO模型，如epoll、kqueue、select或poll等，它们是必备模块。

Nginx的主配置文件由几个段组成，这个段通常也被称为nginx的上下文，每个段的定义格式如下所示。需要注意的是，其每一个指令都必须使用分号(;)结束，否则为语法错误。

	<section> {
		<directive> <parameters>;
	}

#### 配置main模块

下面说明main模块中的几个关键参数。

**error_log**

用于配置错误日志，可用于main、http、server及location上下文中；语法格式为：

error_log file | stderr [ debug | info | notice | warn | error | crit | alert | emerg ]

如果在编译nginx时使用了--with-debug选项，还可以使用如下格式打开调试功能。

error_log LOGFILE [debug_core | debug_alloc | debug_mutex | debug_event | debug_http | debug_imap];

要禁用错误日志，不能使用“error_log off;”，而要使用类似如下选项：

error_log /dev/null crit;

**timer_resolution**

用于降低gettimeofday()系统调用的次数。默认情况下，每次从kevent()、epoll、/dev/poll、select()或poll()返回时都会执行此系统调用。语法格式为：

timer_resolution interval

例如：

timer_resolution  100ms;

**worker_cpu_affinity**

通过sched_setaffinity()将worker绑定至CPU上，只能用于main上下文。语法格式为：

worker_cpu_affinity cpumask ...

例如：
worker_processes     4;
worker_cpu_affinity 0001 0010 0100 1000;

**worker_priority**

为worker进程设定优先级(指定nice值)，此参数只能用于main上下文中，默认为0；语法格式为：

worker_priority number

**worker_processes**

worker进程是单线程进程。如果Nginx用于CPU密集型的场景中，如SSL或gzip，且主机上的CPU个数至少有2个，那么应该将此参数值设定为与CPU核心数相同；如果Nginx用于大量静态文件访问的场景中，且所有文件的总大小大于可用内存时，应该将此参数的值设定得足够大以充分利用磁盘带宽。

此参数与Events上下文中的work_connections变量一起决定了maxclient的值：
maxclients = work_processes * work_connections

2.1.6 worker_rlimit_nofile

设定worker进程所能够打开的文件描述符个数的最大值。语法格式：

worker_rlimit_nofile number

####  配置Events模块

**worker_connections**

设定每个worker所处理的最大连接数，它与来自main上下文的worker_processes一起决定了maxclients的值。

max clients = worker_processes * worker_connections

而在反向代理场景中，其计算方法与上述公式不同，因为默认情况下浏览器将打开2个连接，而nginx会为每一个连接打开2个文件描述符，因此，其maxclients的计算方法为：

max clients = worker_processes * worker_connections/4

**use**

在有着多于一个的事件模型IO的应用场景中，可以使用此指令设定nginx所使用的IO机制，默认为./configure脚本选定的各机制中最适用当前OS的版本。语法格式：

use [ kqueue | rtsig | epoll | /dev/poll | select | poll | eventport ]

#### HTTP服务的相关配置

http上下文专用于配置用于http的各模块，此类指令非常的多，每个模块都有其专用指定，具体请参数nginx官方wiki关于模块部分的说明。大体上来讲，这些模块所提供的配置指令还可以分为如下几个类别。

* 客户端类指令：如client_body_buffer_size、client_header_buffer_size、client_header_timeout和keepalive_timeout等；
* 文件IO类指令：如aio、directio、open_file_cache、open_file_cache_min_uses、open_file_cache_valid和sendfile等；
* hash类指令：用于定义Nginx为某特定的变量分配多大的内存空间，如types_hash_bucket_size、server_names_hash_bucket_size和variables_hash_bucket_size等；
*　套接字类指令：用于定义Nginx如何处理tcp套接字相关的功能，如tcp_nodelay(用于keepalive功能启用时)和tcp_nopush(用于sendfile启用时)等；

#### 虚拟服务器相关配置

	server {
		<directive> <parameters>;
	}

用于定义虚拟服务器相关的属性，常见的指令有backlog、rcvbuf、bind及sndbuf等。

#### location相关的配置

`location [modifier] uri {...} 或 location @name {…}`

通常用于server上下文中，用于设定某URI的访问属性。location可以嵌套。
