---
layout:     post
title:      "【DNS】DNS之前没了解但实际生产中会用到的配置"
date:       2017-07-25 16:46:00
author:     "Yuki"
---

# options定义

**dump-file**
当执行rndc dumpdb 命令时，服务器存放数据库文件的路径名。如果没有指定，缺省名字是named_dump.db。

**Zone-statistics**
如果是yes，缺省情况下，服务器将会收集在服务器所有域的统计数据。这些统计数据
可以通过使用rndc stats 来访问，rndc stats 命令可以将这些信息转储到statistics-file 中的文件中去。

**memstatistics-file**
服务器输出的内存使用统计文件的路径名。如果没有指定，默认值为named.memstats。
注意：还没有在BIND9 中实现！


# logging定义

**例子**

    channel "an_example_channel" {
    file "example.log" versions 3 size 20m;
    print-time yes;
    print-category yes;
    }; 

文件的size 选项用来限制日志的增长。如果文件超过了限制，又没有version 选项，则named 就会停止写入文件

如果对日志文件设置了size 选项，那么仅当此文件超过了设定的大小时，系统就会进行更名

    channel "an_example_channel" {
    file "example.log" versions 3 size 20m;
    print-time yes;
    print-category yes;
    };

print-time 也可以针对syslog的通道进行设置，但因为syslog 也打印日期和时间，所以一般来讲，这没有什么意义。如果设置了print-category 参数，则信息的分类也会记录下来。如果设置了print-severity 参数，则信息的严重级别也会记录下来。

**category 短语**

这里存在许多分类，用户可根据需要定义想看到或不想看的日志。如果你不将某个分类指定到某些通道的话，那么在这个分类的日志信息就会被发送到default 分类通道中。

 

| default        | 没有配置的分类会使用default 的分类的日志配置
| ------------- |:-------------:
|security     | 接受和拒绝的请求 
| lame-servers  |  未知服务器。这些都是有于其它DNS 服务器中的错误配置引起的     
| queries |  请求，使用queries 分类将会产生日志用户请求

**controls语句定义**

    controls {
    inet ( ip_addr | * ) [ port ip_port ] allow { address_match_list }
    keys { key_list };
    [ inet ...; ]
    };
    
controls 语句定义了系统管理员使用的，有关本地域名服务器操作的控制通道。这些控制通道被 rndc 用来发送命令，并从域名服务器中检索非DNS 的结果。
         
**ACL访问控制列表**

访问控制列表（ACLs）是地址的匹配列表，你可以建立一个访问地址列表并指定名称，
在以后的allow-notify, allow-query, allow-recursion, blackhole, allow-transfer 等配置里面使用。
使用访问控制列表可以使你对于访问域名服务器的人进行良好的控制，而不使你的配
置文件因为大量的IP 地址而变得混乱。
使用ACLs 来控制对你服务器的访问是一个很好的选择。由外部的用户来限制对你服务
器的访问可以防止对域名服务器的欺骗和DoS攻击。
