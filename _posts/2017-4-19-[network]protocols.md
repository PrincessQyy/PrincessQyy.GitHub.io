---
layout:     post
title:      "【network】常见的计算机网络协议"
date:       2017-04-19 20:39:00
author:     "Yuki"
---

这几天面试京东，所以把之前学过的的协议都总结了一下。

### 网络协议栈架构

在讲协议之前，先来看看最著名的 OSI 七层模型吧（两个面试官都问到了，但这真的是很基础的东西了！），但是 TCP/IP 协议族的结构则稍有不同，它们之间的层次结构有如图对应关系：

<img src="../../../../../img/blogs/protocols/01.png">

TCP/IP 被分为 4 层，每层承担的任务不一样，各层的协议的工作方式也不一样，每层封装上层数据的方式也不一样，下面是各层常见的协议：

(1)应用层：应用程序通过这一层访问网络，常见 FTP、HTTP、DNS 和 TELNET 协议；

(2)传输层：TCP 协议和 UDP 协议；

(3)网络层：IP 协议，ARP、RARP 协议，ICMP 协议等；

(4)网络接口层：是 TCP/IP 协议的基层，负责数据帧的发送和接收。

下面就一个一个开始介绍吧，我习惯从底层开始介绍起：

### 数据链路层

网络层协议的数据单元是 IP 数据报 ，而数据链路层的工作就是把网络层交下来的 IP 数据报 封装为 帧（frame）发送到链路上，以及把接收到的帧中的数据取出并上交给网络层。

为达到这一目的，数据链路必须具备一系列相应的功能，主要有：

* 将数据封装为帧（frame），帧是数据链路层的传送单位

* 控制帧的传输，包括处理传输差错，调节发送速率与接收方相匹配

* 在两个网络实体之间提供数据链路通路的建立、维持和释放的管理

数据帧的结构是这样的：

<img src="../../../../../img/blogs/protocols/02.png">

#### 以太网

提到数据链路层，必须要说的就是以太网了，以太网(Ether-net)是指DEC 公司、Intel 公司和 Xerox 公司在 1982 年联合公布的一个标准，这个标准里面使用了一种称作 CSMA/CD 的接入方法。

<img src="../../../../../img/blogs/protocols/03.png">

### 网络层

#### ARP

关于ARP，首先要说的是它是网络层的协议，虽然他是将IP地址解析为MAC地址，看起来好像是链路层的活儿，但它是根据网络层IP数据包包头中的IP地址信息解析出目标硬件地址（MAC地址）信息的，所以这是一个网络层的协议。

工作原理如下：

1. 主机A想向主机B发包，首先，他会查看自己的ARP缓存表的信息，ARP缓存是个用来储存IP地址和MAC地址的缓冲区，其本质就是一个IP地址到MAC地址的映射，若是找到了缓存记录，主机A就可以构建以太网帧，在首部填上源目MAC地址，将包发给主机B。

2. 若是没有找到缓存记录，主机A会将ARP请求帧广播到本地网络上的所有主机。源主机A的IP地址和MAC地址都包括在ARP请求中。本地网络上的每台主机都接收到ARP请求并且检查是否与自己的IP地址匹配。如果主机发现请求的IP地址与自己的IP地址不匹配，它将丢弃ARP请求。

3. 主机B确定ARP请求中的IP地址与自己的IP地址匹配，则将主机A的IP地址和MAC地址映射添加到本地ARP缓存中。主机B将包含其MAC地址的ARP回复消息直接发送回主机A。

4. 当主机A收到从主机B发来的ARP回复消息时，会用主机B的IP和MAC地址映射更新ARP缓存。本机缓存是有生存期的，生存期结束后，将再次重复上面的过程。主机B的MAC地址一旦确定，主机A就能向主机B发送IP通信了。

#### IP
