---
layout:     post
title:      "【Linux】底层平台常见安全问题"
date:       2017-03-20 16:15:00
author:     "Yuki"
---

这里，主要总结一些底层平台安全。主要有以下几类：

* DDoS
* 扫描
* 暴力破解
* 恶意篡改
* 针对某种特定服务的攻击


#### DDoS

**基本概念**

DoS,Denial of Service，即拒绝服务攻击。这是一种利用正常/非正常流量使服务器资源耗尽的手段。单一的DoS攻击一般是采用一对一方式的，当攻击目标CPU速度低、内存小或者网络带宽小等等各项指标不高的性能，它的效果是明显的。随着计算机与网络技术的发展，计算机的处理能力迅速增长，内存大大增加，同时也出现了千兆级别的网络，这使得DoS攻击的困难程度加大了-目标对恶意攻击包的"消化能力"加强了不少。这时候分布式的拒绝服务攻击手段（DDoS）就应运而生了。DDoS就是利用更多的傀儡机（肉鸡）来发起进攻。

**攻击分类**

这里，按TCP/IP协议的层次可将DDOS攻击分为基于ARP的攻击、基于ICMP的攻击、基于IP的攻击、基于UDP的攻击、基于TCP的攻击和基于应用层的攻击。

* 基于ARP欺骗（ARP重定向）的攻击

由于ARP本身的特点，攻击者很容易实施ARP欺骗。所谓ARP欺骗，又叫ARP重定向，就是向目标主机发送ARP应答报文，其中含有攻击者伪造的IP地址和硬件地址的对应信息。目标主机收到该报文后，会用报文中伪造的信息刷新ARP高速缓存，如果攻击者定时向目标主机发送该报文，而且时间间隔比ARP缓存的超时间隔要小的话，含有错误源地址信息的ARP请求和含有错误目标地址信息的ARP应答均会使上层应用忙于处理这种异常而无法响应外来请求，使得目标主机丧失网络通信能力。

* 基于ICMP

攻击者向一个子网的广播地址发送多个ICMP Echo请求数据包。并将源地址伪装成想要攻击的目标主机的地址。这样，该子网上的所有主机均对此ICMP Echo请求包作出答复，向被攻击的目标主机发送数据包，使该主机受到攻击，导致网络阻塞。这种攻击方式要求攻击主机处理能力和带宽要大于被攻击主机，否则自身被DoS了。

* 基于IP

TCP/IP中的IP数据包在网络传递时，数据包可以分成更小的片段。到达目的地后再进行合并重装。在实现分段重新组装的进程中存在漏洞，缺乏必要的检查。利用IP报文分片后重组的重叠现象攻击服务器，进而引起服务器内核崩溃。如Teardrop是基于UDP的病态分片数据包（由于UDP数据报不会自己进行分段，因此当长度超过了MTU时，会在网络层进行IP分片）的攻击方法，其工作原理是向被攻击者发送多个分片的IP包（IP分片数据包中包括该分片数据包属于哪个数据包以及在数据包中的位置等信息），某些操作系统收到含有重叠偏移的伪造分片数据包时将会出现系统崩溃、重启等现象。（假设数据包中第二片IP包的偏移量小于第一片结束的位移，而且算上第二片IP包的Data，也未超过第一片的尾部，这就是重叠现象）利用UDP包重组时重叠偏移的漏洞对系统主机发动拒绝服务攻击，最终导致主机宕机。

* 基于UDP

利用UDP无连接特性，发送大量UDP大包，消耗攻击目标的网络带宽资源，造成DDoS攻击效果。

* 基于TCP

SYN洪水攻击就属于DDoS攻击的一种，它利用TCP协议缺陷，恶意攻击方通过发送大量的半连接请求，服务器可用的TCP连接队列将很快被阻塞，系统可用资源急剧减少，网络可用带宽迅速缩小，服务器将无法向用户提供正常的合法服务。

* 基于应用层

应用层包括SMTP，HTTP，DNS等各种应用协议。

其中SMTP定义了如何在两个主机间传输邮件的过程，基于标准SMTP的邮件服务器，在客户端请求发送邮件时，是不对其身份进行验证的。另外，许多邮件服务器都允许邮件中继。攻击者利用邮件服务器持续不断地向攻击目标发送垃圾邮件，大量侵占服务器资源。

HTTP上的DOS这种针对HTTP服务器的攻击很简单，就是不断发送没有完成的HTTP头，直到你的服务器耗尽所有的资源。Apache的情况是，每接受一个连接新开一个进程或线程，这样新接受一个连接的资源消耗很大。

UDP DNS Query Flood攻击采用的方法是向被攻击的服务器发送大量的域名解析请求，通常请求解析的域名是随机生成或者是网络世界上根本不存在的域名，被攻击的DNS 服务器在接收到域名解析请求的时候首先会在服务器上查找是否有对应的缓存，如果查找不到并且该域名无法直接由服务器解析的时候，DNS 服务器会向其上层DNS服务器递归查询域名信息。域名解析的过程给服务器带来了很大的负载，每秒钟域名解析请求超过一定的数量就会造成DNS服务器解析域名超时。

#### 扫描

扫描不是一种攻击手段，而是攻击前的准备，通过扫描，可以获取被攻击主机开启的服务、端口、软件、IP等。

#### 暴力破解

穷举法是一种针对于密码的破译方法。简单来说就是将密码进行逐个推算直到找出真正的密码为止。也就是说用我们破解任何一个密码也都只是一个时间问题。

#### 恶意篡改

这是在服务器已经被攻破的情况下会发生的，通常攻击者不会直接大量删除数据，因为这样太明显了，很容易被察觉，通常，攻击者是利用服务器资源，比如搭一个广告网站或者**网站(◐_◑)
