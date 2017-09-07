---
layout:     post
title:      "【DevOps】运维场景"
date:       2017-08-30 17:04:00
author:     "Yuki"
---

# nginx占用大量磁盘空间问题分析

#### 场景：Nginx占用大量磁盘空间，在删除了近100G的日志文件后，用df命令查看，发现空间并没有得到释放，为什么？

#### 解答：一个文件被进程打开后，在关闭前如果文件被删除，此时这个文件已经在它父目录的目录项中被删除掉（在父目录ls已经看不到这个文件），但该进程依然能够正常的读写文件，直到文件被该进程关闭后，这个文件的空间才会被回收。

# socket端口数量限制问题

#### 场景：linux socket使用16bit无符号整型表示端口号，最大到65535。关于端口号，有一个经典的误解就是，因为端口号有限，所以一个客户端最多建立65536个socket连接，但实际上并不是这么回事，端口是可以复用的。

#### 一个socket连接是一个[srcip, srcport, destip, destport]组成的四元组，如果再算上协议（tcp、udp、rawsocket等）就是五元组了，这个四元组中，只要有一个元素不同，就能区分两个连接；我们平时所遇到端口不够用的问题，大多出现在压测环境中，用客户端不断向某一个服务器建立连接发请求，当连接超过一定数量后，connect会出错提示Cannot assign requested address。对于这样的场景，四元组的destip，destport是确定的，对于单网卡机器srcip也是确定的（不考虑虚拟网卡情况），可变的就只有srcport，而端口的取值只能在0-65535，再加上系统的一些保留端口是不可用的，所以客户端到该特定服务间的连接数就受端口数目的限制，当连接到其他服务器时，本地的端口是可以复用的。实际上，在 不指定端口的情况下连接服务器，可用的端口号在/proc/sys/net/ipv4/ip_local_port_range中配置，默认为32768-61000。

# LVS持久链接

#### LVS的三种持久连接方式：

* PCC：每客户端持久；将来自于同一个客户端的所有请求统统定向至此前选定的RS；也就是只要IP相同，分配的服务器始终相同。

* PPC：每端口持久；将来自于同一个客户端对同一个服务(端口)的请求，始终定向至此前选定的RS。例如：来自同一个IP的用户第一次访问集群的80端口分配到A服务器，25号端口分配到B服务器。当之后这个用户继续访问80端口仍然分配到A服务器，25号端口仍然分配到B服务器。

* PFMC：持久防火墙标记连接；将来自于同一客户端对指定服务(端口)的请求，始终定向至此选定的RS；不过它可以将两个毫不相干的端口定义为一个集群服务，例如：合并http的80端口和https的443端口定义为同一个集群服务，当用户第一次访问80端口分配到A服务器，第二次访问443端口时仍然分配到A服务器。

# lvs 怎么抗SYN FLOOD

**首先来看看为什么Syn Flood会造成危害？**

这要从操作系统的TCP/IP协议栈的实现说起。当开放了一个TCP端口后，该端口就处于Listening状态，不停地监视发到该端口的SYN报文，一旦接收到客户端发来的SYN报文，就需要为该请求分配一个TCB（Transmission Control Block），通常一个TCB至少需要280个字节，在某些操作系统中TCB甚至需要1300个字节，并返回一个SYN+ACK报文，立即转为SYN_RCVD即半开连接状态，而某些操作系统在SOCK的实现上最多可开启512个半开连接（如Linux2.4.20内核）。从以上过程可以看到，如果恶意的向某个服务器端口发送大量的SYN包，则可以使服务器打开大量的半开连接，分配TCB，从而消耗大量的服务器资源，同时也使得正常的连接请求无法被响应。而攻击发起方的资源消耗相比较可忽略不计。

**SYN有哪些种类**

1．Direct Attack 攻击方使用固定的源地址发起攻击，这种方法对攻击方的消耗最小。 

2．Spoofing(欺骗) Attack 攻击方使用变化的源地址发起攻击，这种方法需要攻击方不停地修改源地址，实际上消耗也不大。 

3．Distributed Direct Attack 这种攻击主要是使用僵尸网络进行固定源地址的攻击（僵尸网络(Botnet) 是指采用一种或多种传播手段，将大量主机感染bot程序病毒，从而在控制者和被感染主机之间所形成的一个可一对多控制的网络。 攻击者通过各种途径传播僵尸程序感染互联网上的大量主机，而被感染的主机将通过一个控制信道接收攻击者的指令，组成一个僵尸网络。）

**如何防御 SYN Flood**

对于第一种攻击的防范可以使用比较简单的方法，即对SYN包进行监视，如果发现某个IP发起了较多的攻击报文，直接将这个IP列入黑名单即可。当然下述的方法也可以对其进行防范。

对于源地址不停变化的攻击使用上述方法则不行，首先从某一个被伪装的IP过来的SYN报文可能不会太多，达不到被拒绝的阈值，其次从这个被伪装的IP（真实的）的请求会被拒绝掉。因此必须使用其他的方法进行处理。

1．无效连接监视释放

这种方法不停监视系统的半开连接和不活动连接，当达到一定阈值时拆除这些连接，从而释放系统资源。这种方法对于所有的连接一视同仁，而且由于SYN Flood造成的半开连接数量很大，正常连接请求也被淹没在其中被这种方式误释放掉，因此这种方法属于入门级的SYN Flood方法。

2．延缓TCB分配方法

从前面SYN Flood原理可以看到，消耗服务器资源主要是因为当SYN数据报文一到达，系统立即分配TCB，从而占用了资源。而SYN Flood由于很难建立起正常连接，因此，当正常连接建立起来后再分配TCB则可以有效地减轻服务器资源的消耗。常见的方法是使用Syn Cache和Syn Cookie技术。

* Syn Cache技术：

这种技术是在收到SYN数据报文时不急于去分配TCB，而是先回应一个SYN ACK报文，并在一个专用HASH表（Cache）中保存这种半开连接信息，直到收到正确的回应ACK报文再分配TCB。在FreeBSD系统中这种Cache每个半开连接只需使用160字节，远小于TCB所需的736个字节。在发送的SYN+ACK中需要使用一个己方的Sequence Number，这个数字不能被对方猜到，否则对于某些稍微智能一点的Syn Flood攻击软件来说，它们在发送Syn报文后会发送一个ACK报文，如果己方的Sequence Number被对方猜测到，则会被其建立起真正的连接。因此一般采用一些加密算法生成难于预测的Sequence Number。

* Syn Cookie技术：

对于SYN攻击，Syn Cache虽然不分配TCB，但是为了判断后续对方发来的ACK报文中的Sequence Number的正确性，还是需要使用一些空间去保存己方生成的Sequence Number等信息，也造成了一些资源的浪费。 
Syn Cookie技术则完全不使用任何存储资源，这种方法比较巧妙，它使用一种特殊的算法生成Sequence Number，这种算法考虑到了对方的IP、端口、己方IP、端口的固定信息，以及对方无法知道而己方比较固定的一些信息，如MSS、时间等，在收到对方的ACK报文后，重新计算一遍，看其是否与对方回应报文中的（Sequence Number-1）相同，从而决定是否分配TCB资源。

3.使用SYN Proxy防火墙

Syn Cache技术和Syn Cookie技术总的来说是一种主机保护技术，需要系统的TCP/IP协议栈的支持，而目前并非所有的操作系统支持这些技术。因此很多防火墙中都提供一种SYN代理的功能，其主要原理是对试图穿越的SYN请求进行验证后才放行，下图描述了这种过程：

<img src="../../../../../img/blogs/DevOps/01.png">

从上图（左）中可以看出，防火墙在确认了连接的有效性后，才向内部的服务器（Listener）发起SYN请求，在右图中，所有的无效连接均无法到达内部的服务器。而防火墙采用的验证连接有效性的方法则可以是Syn Cookie或Syn Flood等其他技术。 
采用这种方式进行防范需要注意的一点就是防火墙需要对整个有效连接的过程发生的数据包进行代理，如下图所示：


<img src="../../../../../img/blogs/DevOps/02.png">

因为防火墙代替发出的SYN ACK包中使用的序列号为c，而服务器真正的回应包中序列号为c’，这其中有一个差值|c-c’|，在每个相关数据报文经过防火墙的时候进行序列号的修改。

# 智能DNS的原理

#### 客户端去向local DNS发起请求的时候，这里假设查询www.jd.com，local DNS会首先查询自己的缓存，若有，则直接将记录返回，若无，则localDNS会先去根域查询，根域返回 com服务器的地址，然后local DNS去向com域查询，com域返回 jd.com域的地址，这时候因为local DNS带着自己的IP去向jd.com域查询 www主机的信息，在jd.com域的ns服务器上就必须维护一张很大的表，记录着哪个IP段属于哪个城市的哪个运营商，这个记录一定要准确，然后返回一个该城市该运营商视图下的一个A记录给local DNS。

#### DNS服务器的视图通常在配置文件中是使用view实现的。把要使用某些IP地址作单独访问的zone区域，统一放在一个命名的view段落中，并且在view中定义请求的IP地址或IP地址段，把IP地址写入match-clients选项中。如果像上面说的，区分电信和网通路线的话，那么可以使用两个acl访问控制列表写上电信或网通IP地址，定义电信网通路线，把acl名字写入view段落match-clients选项中。


# Nginx 如何拿到客户端 IP

#### 场景：在实际应用中，我们可能需要获取用户的ip地址，比如做异地登陆的判断，或者统计ip访问次数

 经过反向代理后，由于在客户端和web服务器之间增加了中间层，因此web服务器无法直接拿到客户端的ip，通过$remote_addr变量拿到的将是反向代理服务器的ip地址”。这句话的意思是说，当你使用了nginx反向服务器后，在web端使用request.getRemoteAddr()（本质上就是获取$remote_addr），取得的是nginx的地址，即$remote_addr变量中封装的是nginx的地址，当然是没法获得用户的真实ip的，但是，nginx是可以获得用户的真实ip的，也就是说nginx使用$remote_addr变量时获得的是用户的真实ip，如果我们想要在web端获得用户的真实ip，就必须在nginx这里作一个赋值操作，如下：

 `proxy_set_header            X-real-ip $remote_addr;`

 其中这个X-real-ip是一个自定义的变量名，名字可以随意取，这样做完之后，用户的真实ip就被放在X-real-ip这个变量里了，然后，在web端可以这样获取：

 `request.getAttribute("X-real-ip")`

**原理介绍**

这里我们将nginx里的相关变量解释一下，通常我们会看到有这样一些配置

	server {
	
	        listen       88;
	
	        server_name  localhost;
	
	        #charset koi8-r;
	
	        #access_log  logs/host.access.log  main;
	
	        location /{
	
	            root   html;
	
	            index  index.html index.htm;
	
	                            proxy_pass                  http://backend; 
	
	           proxy_redirect              off;
	
	           proxy_set_header            Host $host;
	
	           proxy_set_header            X-real-ip $remote_addr;
	
	           proxy_set_header            X-Forwarded-For $proxy_add_x_forwarded_for;
	
	                     # proxy_set_header            X-Forwarded-For $http_x_forwarded_for;
	
	        }

我们来一条条的看

1. `proxy_set_header    X-real-ip $remote_addr;`

 这句话之前已经解释过，有了这句就可以在web服务器端获得用户的真实ip

 但是，实际上要获得用户的真实ip，不是只有这一个方法，下面我们继续看。

2.  `proxy_set_header            X-Forwarded-For $proxy_add_x_forwarded_for;`

 我们先看看这里有个X-Forwarded-For变量，这是一个squid开发的，用于识别通过HTTP代理或负载平衡器原始IP一个连接到Web服务器的客户机地址的非rfc标准，如果有做X-Forwarded-For设置的话,每次经过proxy转发都会有记录,格式就是client1, proxy1, proxy2,以逗号隔开各个地址，由于他是非rfc标准，所以默认是没有的，需要强制添加，在默认情况下经过proxy转发的请求，在后端看来远程地址都是proxy端的ip 。也就是说在默认情况下我们使用request.getAttribute("X-Forwarded-For")获取不到用户的ip，如果我们想要通过这个变量获得用户的ip，我们需要自己在nginx添加如下配置：

 `proxy_set_header            X-Forwarded-For $proxy_add_x_forwarded_for;`

 意思是增加一个$proxy_add_x_forwarded_for到X-Forwarded-For里去，注意是增加，而不是覆盖，当然由于默认的X-Forwarded-For值是空的，所以我们总感觉X-Forwarded-For的值就等于$proxy_add_x_forwarded_for的值，实际上当你搭建两台nginx在不同的ip上，并且都使用了这段配置，那你会发现在web服务器端通过request.getAttribute("X-Forwarded-For")获得的将会是客户端ip和第一台nginx的ip。

 那么$proxy_add_x_forwarded_for又是什么？

 $proxy_add_x_forwarded_for变量包含客户端请求头中的"X-Forwarded-For"，与$remote_addr两部分，他们之间用逗号分开。

 举个例子，有一个web应用，在它之前通过了两个nginx转发，www.linuxidc.com 即用户访问该web通过两台nginx。

 在第一台nginx中,使用

 `proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;`

 现在的$proxy_add_x_forwarded_for变量的"X-Forwarded-For"部分是空的，所以只有$remote_addr，而$remote_addr的值是用户的ip，于是赋值以后，X-Forwarded-For变量的值就是用户的真实的ip地址了。

 到了第二台nginx，使用

 `proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;`

 现在的$proxy_add_x_forwarded_for变量，X-Forwarded-For部分包含的是用户的真实ip，$remote_addr部分的值是上一台nginx的ip地址，于是通过这个赋值以后现在的X-Forwarded-For的值就变成了“用户的真实ip，第一台nginx的ip”，这样就清楚了吧。

 最后我们看到还有一个$http_x_forwarded_for变量，这个变量就是X-Forwarded-For，由于之前我们说了，默认的这个X-Forwarded-For是为空的，所以当我们直接使用proxy_set_header            X-Forwarded-For $http_x_forwarded_for时会发现，web服务器端使用request.getAttribute("X-Forwarded-For")获得的值是null。如果想要通过request.getAttribute("X-Forwarded-For")获得用户ip，就必须先使用proxy_set_header            X-Forwarded-For $proxy_add_x_forwarded_for;这样就可以获得用户真实ip。