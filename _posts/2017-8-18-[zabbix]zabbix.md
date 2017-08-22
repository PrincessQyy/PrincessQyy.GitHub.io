---
layout:     post
title:      "【ZABBIX】Zabbix基本配置"
date:       2017-08-21 09:43:00
author:     "Yuki"
---

# 监控的意义

### 监控系统应当解决两个问题：

* 现象（什么东西出故障了？）
	
 用户可感知的现象，比如：登陆不了、支付订单变慢；

* 原因（为什么出故障？）

造成现象的潜在因素，可能只是中间因素或者相关因素，并非根本原因，	
根本原因需要运维人员介入分析并确定。

### 监控系统的告警需求

监控系统最重要的功能就是告警了，没有监控业务就相当于在线上裸奔啊(￣▽￣)~* 所以一个好的监控系统应该要能及时发现故障并且对故障能支持多种方式的报警，如短信、邮件等，此外，还应具备可定制化功能，对第三方告警介质提供可编程的接口，例如，将告警结果发送到专用的告警分析系统。这一点其实很重要，支持对告警内容的分析，可以防止漏报、误报以及防止抖动，例如：一个机房网络发生故障，按照常规告警内容，会收到无数条告警信息，内容是每个机房的设备的故障，而其实对于高级的告警信息，我们希望收到的报警是：“由于某机房网络故障，导致设备受故障，受影响的设备IP是*.*.*.* ...，受影响的业务是***”，这可以防止监控中的 “狼来了” 现象。

# zabbix架构

zabbix采用的是经典的C/S模式，分布式架构为Client/Proxy/Server，zabbix-Server将采集到的数据持久地存储到数据库中，前端UI友好的展示，下图展示的是Agent采集数据、Proxy做代理、Server处理数据的模式。

<img src="../../../../../img/blogs/zabbix/01.png">

其运行流程可以见下图

<img src="../../../../../img/blogs/zabbix/02.png">

# 安装ZABBIX

说到安装，也是有点心累，上官网找到zabbix相应版本的rpm包，然后在 /etc/yum.repos.d目录下建立zabbix.repo文件，编辑如下

<img src="../../../../../img/blogs/zabbix/03.png">

	yum clean all

	yum -y install zabbix-agent zabbix-web-mysql 

然后在装 zabbix-server-mysql 前  ，还需要解决两个依赖关系，方法和上面是一样的，我就不赘述了。安装完依赖软件 fping 和 iksemel后， `yum -y install zabbix-server-mysql` ,至此，安装完成。

在终端输入

# 安装配置MySQL服务

因为zabbix使用数据库存储数据，所以这里需要安装MySQL，因为我用的是centos7，所以安装mariadb，`yum -y install mariadb`,安装完后配置如下：

<img src="../../../../../img/blogs/zabbix/03.png">

然后启动服务 `systemctl start mariadb`

* 创建zabbix数据库

`shell # mysql -uroot -p`
`mysql> create database zabbix character set utf8;`
`mysql> grant all privileges on zabbix.* to zabbix@localhost identified by "zabbix";`
`mysql> flush privileges;`

* 导入zabbix-server的数据库

/usr/share/doc/zabbix-server-mysql-3.2.7 下执行这条命令导入数据库 

`shell # zcat create.sql.gz|mysql -uzabbix -pzabbix zabbix`

<img src="../../../../../img/blogs/zabbix/05.png">




