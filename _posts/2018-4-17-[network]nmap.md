---
layout:     post
title:      "【Net】nmap原理分析"
date:       2018-04-17 11:45:00
author:     "Yuki"
---
在主机监控中，我们通常会用ping的方式来检测主机是否存活，然而有的交换机是比较坑的，对ICMP数量有限制，一旦达到上限，就会产生丢包现象，造成误判，所以，用nmap的方式来探测主机的存活性更为合适。
##  Nmap基本扫描方法
Nmap主要包括四个方面的扫描功能，主机发现、端口扫描、应用与版本侦测、操作系统侦测。
### 主机发现
主机发现（Host Discovery），即用于发现目标主机是否在线（Alive，处于开启状态）。
#### 主机发现原理
默认情况下，Nmap会发送四种不同类型的数据包来探测目标主机是否在线。

1.      ICMP echo request

2.      a TCP SYN packet to port 443

3.      a TCP ACK packet to port 80

4.      an ICMP timestamp request

依次发送四个报文探测目标机是否开启。只要收到其中一个包的回复，那就证明目标机开启。使用四种不同类型的数据包可以避免因防火墙或丢包造成的判断错误。
### 端口扫描
端口扫描是Nmap最基本最核心的功能，用于确定目标主机的TCP/UDP端口的开放情况。

默认情况下，Nmap会扫描1000个最有可能开放的TCP端口。

Nmap通过探测将端口划分为6个状态：

* open：端口是开放的。
* closed：端口是关闭的。
* filtered：端口被防火墙IDS/IPS屏蔽，无法确定其状态。
* unfiltered：端口没有被屏蔽，但是否开放需要进一步确定。
* open|filtered：端口是开放的或被屏蔽。
* closed|filtered ：端口是关闭的或被屏蔽。

#### 端口扫描原理
Nmap在端口扫描方面非常强大，提供了十多种探测方式。

* TCP SYN scanning

这是Nmap默认的扫描方式，通常被称作半开放扫描（Half-open scanning）。该方式发送SYN到目标端口，如果收到SYN/ACK回复，那么判断端口是开放的；如果收到RST包，说明该端口是关闭的。如果没有收到回复，那么判断该端口被屏蔽（Filtered）。因为该方式仅发送SYN包对目标主机的特定端口，但不建立的完整的TCP连接，所以相对比较隐蔽，而且效率比较高，适用范围广。

TCP SYN 检测到端口关闭：
<img src="../../../../../img/blogs/nmap/01.jpg">

TCP SYN 检测到端口开放：
<img src="../../../../../img/blogs/nmap/02.jpg">

* TCP connect scanning

TCP connect方式使用系统网络API connect向目标主机的端口发起连接，如果无法连接，说明该端口关闭。该方式扫描速度比较慢，而且由于建立完整的TCP连接会在目标机上留下记录信息，不够隐蔽。所以，TCP connect是TCP SYN无法使用才考虑选择的方式。

TCP connect 检测到端口关闭：
<img src="../../../../../img/blogs/nmap/03.jpg">

TCP connect 检测到端口开放：
<img src="../../../../../img/blogs/nmap/04.jpg">

* TCP ACK scanning

向目标主机的端口发送ACK包，如果收到RST包，说明该端口没有被防火墙屏蔽；没有收到RST包，说明被屏蔽。该方式只能用于确定防火墙是否屏蔽某个端口，可以辅助TCP SYN的方式来判断目标主机防火墙的状况。

TCP ACK 探测到端口被屏蔽：
<img src="../../../../../img/blogs/nmap/05.jpg">

TCP ACK 探测到端口未被屏蔽：
<img src="../../../../../img/blogs/nmap/06.jpg">

* TCP FIN/Xmas/NULL scanning

这三种扫描方式被称为秘密扫描（Stealthy Scan），因为相对比较隐蔽。FIN扫描向目标主机的端口发送的TCP FIN包或Xmas tree包/Null包，如果收到对方RST回复包，那么说明该端口是关闭的；没有收到RST包说明端口可能是开放的或被屏蔽的（open|filtered）。

其中Xmas tree包是指flags中FIN URG PUSH被置为1的TCP包；NULL包是指所有flags都为0的TCP包。

TCP FIN 探测到端口关闭：
<img src="../../../../../img/blogs/nmap/07.jpg">

TCP FIN探测到端口开放/被屏蔽
<img src="../../../../../img/blogs/nmap/08.jpg">

* UDP scanning

UDP扫描方式用于判断UDP端口的情况。向目标主机的UDP端口发送探测包，如果收到回复“ICMP port unreachable”就说明该端口是关闭的；如果没有收到回复，那说明UDP端口可能是开放的或屏蔽的。因此，通过反向排除法的方式来断定哪些UDP端口是可能出于开放状态。

UDP端口关闭：
<img src="../../../../../img/blogs/nmap/09.jpg">

UDP端口开放/被屏蔽：
<img src="../../../../../img/blogs/nmap/10.jpg">

* 其他方式

除上述几种常用的方式之外，Nmap还支持多种其他探测方式。例如使用SCTP INIT/COOKIE-ECHO方式来探测SCTP的端口开放情况；使用IP protocol方式来探测目标主机支持的协议类型（TCP/UDP/ICMP/SCTP等等）；使用idle scan方式借助僵尸主机（zombie host，也被称为idle host，该主机处于空闲状态并且它的IPID方式为递增。详细实现原理参见：![](http://nmap.org/book/idlescan.html)来扫描目标在主机，达到隐蔽自己的目的；或者使用FTP bounce scan，借助FTP允许的代理服务扫描其他的主机，同样达到隐藏自己的身份的目的。

### 版本侦测
版本侦测，用于确定目标主机开放端口上运行的具体的应用程序及版本信息。

Nmap提供的版本侦测具有如下的优点：

* 高速。并行地进行套接字操作，实现一组高效的探测匹配定义语法。
尽可能地确定应用名字与版本名字。
* 支持TCP/UDP协议，支持文本格式与二进制格式。
* 支持多种平台服务的侦测，包括Linux/Windows/Mac OS/FreeBSD等系统。
* 如果检测到SSL，会调用openSSL继续侦测运行在SSL上的具体协议（如HTTPS/POP3S/IMAPS）。
* 如果检测到SunRPC服务，那么会调用brute-force RPC grinder进一步确定RPC程序编号、名字、版本号。
* 支持完整的IPv6功能，包括TCP/UDP，基于TCP的SSL。
* 通用平台枚举功能（CPE）
* 广泛的应用程序数据库（nmap-services-probes）。目前Nmap可以识别几千种服务的签名，包含了180多种不同的协议。

#### 版本侦测原理
简要的介绍版本的侦测原理。
版本侦测主要分为以下几个步骤：

1. 首先检查open与open|filtered状态的端口是否在排除端口列表内。如果在排除列表，将该端口剔除。
2. 如果是TCP端口，尝试建立TCP连接。尝试等待片刻（通常6秒或更多，具体时间可以查询文件nmap-services-probes中Probe TCP NULL q||对应的totalwaitms）。通常在等待时间内，会接收到目标机发送的“WelcomeBanner”信息。nmap将接收到的Banner与nmap-services-probes中NULL probe中的签名进行对比。查找对应应用程序的名字与版本信息。
3. 如果通过“Welcome Banner”无法确定应用程序版本，那么nmap再尝试发送其他的探测包（即从nmap-services-probes中挑选合适的probe），将probe得到回复包与数据库中的签名进行对比。如果反复探测都无法得出具体应用，那么打印出应用返回报文，让用户自行进一步判定。
4. 如果是UDP端口，那么直接使用nmap-services-probes中探测包进行探测匹配。根据结果对比分析出UDP应用服务类型。
5. 如果探测到应用程序是SSL，那么调用openSSL进一步的侦查运行在SSL之上的具体的应用类型。
6. 如果探测到应用程序是SunRPC，那么调用brute-force RPC grinder进一步探测具体服务。

### OS侦测
操作系统侦测用于检测目标主机运行的操作系统类型及设备类型等信息。

Nmap拥有丰富的系统数据库nmap-os-db，目前可以识别2600多种操作系统与设备类型。

#### OS侦测原理
Nmap使用TCP/IP协议栈指纹来识别不同的操作系统和设备。在RFC规范中，有些地方对TCP/IP的实现并没有强制规定，由此不同的TCP/IP方案中可能都有自己的特定方式。Nmap主要是根据这些细节上的差异来判断操作系统的类型的。

具体实现方式如下：

1. Nmap内部包含了2600多已知系统的指纹特征（在文件nmap-os-db文件中）。将此指纹数据库作为进行指纹对比的样本库。
2. 分别挑选一个open和closed的端口，向其发送经过精心设计的TCP/UDP/ICMP数据包，根据返回的数据包生成一份系统指纹。
3. 将探测生成的指纹与nmap-os-db中指纹进行对比，查找匹配的系统。如果无法匹配，以概率形式列举出可能的系统。






