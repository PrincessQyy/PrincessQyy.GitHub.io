---
layout:     post
title:      "【Linux】网站监控-SNMP"
date:       2017-03-30 11:18:00
author:     "Yuki"
---

简单网络管理协议（SNMP）是TCP/IP协议簇的一个应用层协议。基本说到网络监控，就会涉及到SNMP。它可以用来获取或设置某个系统上的数据、参数。现在基本各种网络设备上都可以看到默认启用的SNMP服务，从交换机到路由器，从防火墙到网络打印机，无一例外。

#### 简单介绍

**工作原理**

SNMP使用UDP协议，端口号为161、162.

<img src="../../../../../img/blogs/SNMP/01.png">

在典型的SNMP用法中，有许多系统被管理，而且有一或多个系统在管理它们，管理系统称为NMS（network management station）。每一个被管理的系统上又运行一个叫做代理者（agent）的软件元件，且通过SNMP对管理系统报告资讯。每个SNMP设备（Agent）都有自己的MIB（management information base）。MIB也可以看作是NMS和Agent之间的沟通桥梁。

MIB数据库使用树状结构管理数据，并且使用OID(Object Identifier)定位数据,MIB树如下：

<img src="../../../../../img/blogs/SNMP/01.png">

如：ipInReceives的实例数字表示为：1.3.6.1.2.1.4.3.0.

**SNMP方法**

监控设备通过 GET、GET NEXT、GET BULK、SET获取、设置被监控设备的数据，这要求监控设备主动发出请求才能获取和设置数据。

被监控设备也可以通过TRAPS、INFORM被触发在遇到紧急情况时主动向监控设备发送数据。

**SNMP版本**

SNMP一共有三个版本，SNMPv1，SNMPv2c(SNMPv2u)、SNMPv3。SNMPv1和SNMPv2是使用最多的两个版本，因为很多有SNMP服务的设备都有一定年头了，比如路由器、交换机啥的，不支持SNMPv3,就算支持，老版本用着也不错，升级也麻烦，所以几乎不会去升级。

三个版本的区别就是数据的安全性不同，SNMPv1是最没安全性的，数据在网络上的传输是明文的，SNMPv2提高了安全性，SNMPv3安全性最高。

#### net-snmp

SNMP最常用的开源实现就是 net-snmp。

* 用命令 `service snmpd start` 启动net-snmp服务
* net-snmp 配置文件在 /etc/snmp/snmpd.conf





