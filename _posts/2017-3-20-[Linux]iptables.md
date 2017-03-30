---
layout:     post
title:      "【Linux】配置iptables策略"
date:       2017-03-20 19:45:00
author:     "Yuki"
---

#### iptables简介

iptables 是与最新的 3.5 版本 Linux 内核集成的 IP 信息包过滤系统。该系统有利于在 Linux 系统上更好地控制 IP 信息包过滤和防火墙配置。

iptables 就是目前工作于较新版（内核版本高过2.4的） Linux 内核中的强大的数据包过滤的软件，它主要是由两部分组成：

* iptalbes：主要工作于用户空间，为用户提供了一个编辑规则的接口。
* netfilter：主要工作于内核空间，是内核的一部分，由一些过滤表组成。

netfilter 工作于系统的内核空间，最底层的工作，所以真正令过滤规则生效的并不是 iptables 而是 netfilter，而 iptables 工作在 netfilter之上，是一个让用户编写规则的工具。

#### netfilter 简介

netfilter可以对数据进行允许、丢弃、修改操作。它支持通过以下方式对数据包进行分类：

\- 源IP地址

\- 目标IP地址

\- 使用接口

\- 使用协议

\- 端口号

\- 连接状态（new、ESTABLISHED、RELATED、INVALID）


####　iptables结构

Netfilter 所设置的规则是存放在内存中的，而 iptables 通过Netfilter 放出的内核接口 ip_tables 来对存放在内存中的 Netfilter 配置表进行修改。这个配置表主要由 tables、chains、target 组成。


<img src="../../../../../img/blogs/iptables/01.png">

其中表主要有这四张：

* filter表
* NAT表
* mangle表
* raw表

每个表中可以用的 chains 不全相同，当然 iptables 支持新建 chains，而 chains 是 Netfilter 框架中制定的对数据包的 Hook Point，Hook Point 是一个数据包通过网卡流经系统内核相应的位置时会对数据包的流向做出一定的修改，在系统上存在5个 Hook Point 的挂载点

* PREROUTING
* INPUT
* OUTPUT
* FORWARD
* POSTROUTING

target这里只列举常用的几种规则，更多的规则可以使用 man 来查看

* ACCEPT：一旦包满足了指定的匹配条件，就会通过，并且不会再去匹配当前链中的其他规则或同一个表内的其他规则，但它还要通过其他表中的链
* DROP：一旦包满足了指定的匹配条件，将会把该包丢弃，也就是说包的生命到此结束，不会再向前走一步，效果就是包被阻塞了。不会返回任何的消息
* REJECT和DROP基本一样，一旦包满足了指定的匹配条件，将会把该包丢弃，但是它除了阻塞包之外，还向发送者返回错误信息。
* SNAT：一旦包满足了指定的匹配条件，源网络地址转换
* DNAT：一旦包满足了指定的匹配条件，目的网络地址转换
* MASQUERADE和SNAT的作用相同，区别在于它不需要指定–to-source
* REDIRECT：一旦包满足了指定的匹配条件，转发数据包一另一个端口
* RETURN：一旦包满足了指定的匹配条件，使数据包返回上一层
* MIRROR：颠倒IP头部中的源目地址，然后再转发包

#### filter表的认识

其中 filter 表的主要作用就是对数据包的过滤，访问控制。该表下有三个规则链：

* INPUT 链：INPUT 针对那些从外进入本地，也就是目的地是本地的包
* FORWARD 链：FORWARD 针对所有不是本地产生的并且目的地不是本地(即本机只是负责转发)的包
* OUTPUT 链：OUTPUT 是用来针对所有本地生成的包

在内核中使用 iptables_filter 这个模块，在代码中各模块都是相对独立的初始化，此模块的初始化在源码 net/ipv4/netfilter/iptable_filter.c –>iptable_filter_init 中，我们可以在 filter 的初始化函数 iptables_filter_init 中我们可以看到注册 Hook point 的函数接口 nf_register_hooks 中是这样写的

<img src="../../../../../img/blogs/iptables/01.png">

由此我们了解到在 filter 初始化的时候便是只注册了三个 Hook point.

#### NAT表的认识

其中 NAT（Network Address Translation） 表主要用于修改数据包的报头的 IP 地址、端口号等信息。可以实现数据包伪装、平衡负载、端口转发和透明代理。该表包含三个链：

* PREROUTING 链：作用是在包刚刚到达本机时，路由之前改变它的目的地址
* OUTPUT 链：改变本地产生的包的目的地址
* POSTROUTING 链：在包就要离开防火墙之前改变其源地址

NAT(网络地址转换) 是把内部网络的 ip 地址转换为合法的公网 ip 地址，当私有网主机和公共网主机通信的IP包经过NAT网关时，将 IP 包中的源IP或目的 IP 在私有 IP 和 NAT 的公共 IP 之间进行转换。能够有效的解决公网地址不足的问题，有效地避免来自网络外部的攻击，隐藏并保护网络内部的计算机。

NAT 分为三种类型

* 静态NAT（static NAT）：是指将内部网络的私有IP地址转换为公有IP地址，IP地址对是一对一的，是一成不变的，某个私有IP地址只转换为某个公有IP地址。
*　动态NAT（dynamic NAT 或者叫 pooled NAT）：是指将内部网络的私有IP地址转换为公用IP地址时，IP地址是不确定的，是随机的，所有被授权访问上Internet的私有IP地址可随机转换为任何指定的合法IP地址。也就是说，只要指定哪些内部地址可以进行转换，以及用哪些合法地址作为外部地址时，就可以进行动态转换。动态转换可以使用多个合法外部地址集。当ISP提供的合法IP地址略少于网络内部的计算机数量时。可以采用动态转换的方式。
* NPAT（Network Address Port Translation）：网络地址端口转换在 ip 地址的层面上来说是属于一种多对一的关系。是指改变外出数据包的源端口并进行端口转换，即端口地址转换（PAT，Port Address Translation).采用端口多路复用方式。内部网络的所有主机均可共享一个合法外部IP地址实现对Internet的访问，从而可以最大限度地节约IP地址资源。同时，又可隐藏网络内部的所有主机，有效避免来自internet的攻击。

在 iptables 中我们比较常用 NPAT 的形式，而 NPAT 又细分为以下的两种：

* 源 NAT（SNAT，Source NAT）修改数据包的源地址。源NAT改变数据流的第一个数据包的来源地址，通常用于伪装内部地址，数据包伪装就是一具SNAT的例子。
* 目的 NAT（DNAT，Destination NAT）修改数据包的目的地址。它是改变第一个数据包的目的地地址，通常用于跳转，如平衡负载、端口转发和透明代理就是属于 DNAT。

若是要做 SNAT 的数据包需要添加到 POSTROUTING 链中。要做 DNAT 的数据包需要添加到 PREROUTING 链中。直接从本地出站的信息包的规则被添加到 OUTPUT 链中。

因为 SNAT 是修改的源地址，然后通过发送出去，给其他的网络设备解读，所以必须在发送出去之前进行 SNAT。所以数据包是送往 POSTROUTING 链，并且匹配了规则，则执行 DNAT 或 REDIRECT 目标。便可在路由之后修改源地址了。

因为 DNAT 是修改的目的地址，然后通过路由转发出去，为了使数据包得到正确路由，必须在路由之前进行 DNAT。所以数据包是送往 PREROUTING 链，并且匹配了规则，则执行 DNAT 或 REDIRECT 目标。便可在路由之前修改目的地址了。

附：路由是内核检查数据包的报头信息

#### mangle 表的认识

其中 mangle 表主要用于修改数据包的 TOS（Type Of Service，服务类型，根据不同的服务质量。来选择经过路由的路径）、TTL（Time To Live，生存周期，每经过一个路由器将减1，mangle 可以修改此值设定TTL要被增加的值，这个选项可以使我们的防火墙更加隐蔽，而不被 trace-routes 发现等等）以及为数据包设置 Mark 标记（特殊标记，用来做高级路由，以使不同的包能使用不同的队列要求，等等），Qos(Quality Of Service，服务质量)调整以及策略路由等应用，由于 TOS，Qos 类似的方式需要相应的路由设备支持，所以应用并不广泛。这个表中包含五个规则链：PREROUTING，POSTROUTING，INPUT，OUTPUT，FORWARD。

####　raw 表的认识

raw 表是自1.2.9以后版本的 iptables 新增的表，主要用于决定数据包是否被状态跟踪机制处理。在匹配数据包时，raw 表的规则要优先于其他表。包含两条规则链：OUTPUT、PREROUTING

#### 数据包的状态

iptables中数据包被跟踪连接有4种不同状态：

* NEW：该包想要开始一个连接（重新连接或将连接重定向）
* RELATED：该包是属于某个已经建立的连接所建立的新连接。例如：--icmp-type 0 ( ping 应答) 就是--icmp-type 8 (ping 请求)所RELATED出来的。
* ESTABLISHED ：只要发送并接到应答，一个数据连接从NEW变为 * ESTABLISHED ,而且该状态会继续匹配这个连接的后续数据包。
* INVALID：数据包不能被识别属于哪个连接或没有任何状态比如内存溢出，收到不知属于哪个连接的 ICMP 错误信息，一般应该 DROP 这个状态的任何数据。

我们可以通过这样一张表了解到数据包在内核中是怎样的一个前进的过程，到底要经过多少次省核，多少个关卡

<img src="../../../../../img/blogs/iptables/03.png">

图中可以看出每个数据包的进与出都会经历这样的一个流程：

1. 当有数据包进入网卡时，数据包首先到 PREROUTING 链中，若是有表对应的表匹配，首先应该是到 raw 中，然后到 mangle 中最后到 NAT 的 PREROUTING 链中，上文我们也提过PREROUTING 链中我们有机会在到内核的路由模块之前修改数据包的目的 IP ，然后内核的"路由模块"根据数据包目的 IP 以及内核中的路由表判断往哪里转发(注意，这个时候数据包的目的地址有可能已经被我们修改过了)
2. 若是该数据包的目的地址就是本机的地址，也就是该数据包就是发送给本地的，那么就会进入 INPUT 链，而进入 INPUT 链首先到 mangle 表中看看，然后到 filter 表中看看。通过之后便会发给本地的相应的程序
本地相应的程序若是做出相应，产生新的数据包往外发送，数据包将进入到 OUTPUT 链，而在 OUTPUT 链中与 INPUT 链匹配表的顺序相同，依旧是首先查看 raw 表，然后查看 mangle 表，查看 NAT表 ，最后查看 filter 表，若是该数据包还能继续前进将会被发送到 POSTROUTING 链中。
3. 若是之前该数据包的目的地址并不是本机，只是把这里当中转站的话，就会将该数据包发给 FORWARD 链，在 FORWARD 链中，依旧先查看 mangle 表，然后查看 filter 表，若是该包还能进去前进则将进入 POSTROUTING 链中
4. 在 POSTROUTING 链中首先查看 mangle 表，然后查看 NAT 表，因为他们可以在最后发送出去之前修改数据包中的源地址。
6. 通过这些的层层把关，最后数据包便可以从网卡发送出去了

从数据包在内核中的走向我们可以得出以下两点：

* iptables 中匹配规则的表示有顺序的，我们可以得出优先级的顺序是 raw 表> mangle 表> NAT 表> filter 表
*　iptables 中的表里面的链的规则也是有匹配顺序的，优先级的顺序是 PREROUTING > INPUT FORWARD OUTPUT POSTROUTING

#### 配置iptables的一些规则

因为netfilter是内核级的机制，所以默认是启动的，可以通过命令 `iptables service status` 来查看其状态。

* 命令 `sudo iptables -nL` 用以查看当前iptables中已经写下的规则

* 若我们还想到跟详细的结果可以使用这样的一个命令 `sudo iptables -nvL --line`。其中：

第一列 num 显示了该规则在该链中的顺序位置

第二列 pkts 显示了该规则处理的数据包数

第三列 bytes 显示了该规则处理的字节数，

第四列 target 显示了该规则所做的行为，

第五列 port 匹配的端口

第六列 opt 是 TCP 协议头部中 options 的一部分，并不是重点，我们
可以不必关注

第七列、第八列 in、out 分别表示对从网卡进入与出去的限制 ip 的匹配条件

第九列、第十列 source、destination 表示对包中分析得出的数据源地址与数据的目的地址的匹配


**规则的相关知识** 

我们使用规则来配置iptables。规则有如下特点：

* 一个规则使用一行配置
* 规则按顺序排列，规则的顺序非常重要
* 当收到、发出、转发数据包时，使用规则对数据包进行匹配，数据包按照第一个匹配上的规则执行相关动作：丢弃、放行、修改
* 没有匹配规则时，会使用默认动作（每条chain拥有各自的默认动作）

一条规则包含以下几个部分：

iptables -t table -A chain [选项] properties -j action

\- table：规定使用的表（filter、nat、mangle）

\- chain：规定过滤点

\- properties：匹配属性，规定匹配数据包的特征，可以写多个属性（源/目IP、端口、协议、TCP状态、接口）

\- action：匹配后的动作，放行、丢弃、修改

**基本操作**

在配置规则的时候我们会用到各种参数，下面列出一些常用的参数：


<img src="../../../../../img/blog s/iptables/04.png">
附：在参数后加"!"表示取反

下面举几个例子

* 插入规则：`iptables -I INPUT 3 -p tcp --dport 22 -j ACCEPT `，插入一条规则在input链的第三个位置，对所有tcp类型并且目标端口号是22号的数据包采取接收
* 删除规则：`iptables -D INPUT 3`
* 删除所有规则：`iptables -F`


**常用功能**

* Linux主机作为服务器使用：

\- 过滤到本机的流量： input链，filter表

\- 过滤本机发出的流量：output链、filter表

例：控制到本机的网络流量

> iptables -A INPUT -s 192.168.1.100 -j DROP 

> iptables -A INPUT -s 192.168.1.100 -p tcp --dport 80 -j DROP

>iptables -A INPUT -i eth0 -j ACCEPT 

* Linux主机作为路由器使用：

\- 过滤转发的流量：forward链、filter表

\- 对转发的数据的源、目IP进行修改：prerouting/postrouting链、NAT表


例：禁止所有192.168.1.0/24 到 10.1.1.0/24的流量

> iptables -A FORWARD -s 192.168.1.0/24 -d 10.1.1.0/24 -j DROP

例：通过NAT进行跳转（指将数据包跳转到另一个服务器，实现伪装保护或负载均衡 ）目标地址转换在PREROUTING。

> iptables -t nat -A PREROUTING -p tcp --dport -j DNAT --to-dest 192.168.1.0

例：通过NAT对出向数据进行跳转

> iptables -t nat -A OUTPUT -p tcp --dport 80 -j DNAT --to-dest 192.268.1.100:8080

例：通过NAT对数据流进行伪装（将内部地址全部伪装为一个外部公网IP地址）源地址转换在POSTROUTING。

> iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

例：通过NAT隐藏源IP地址

> iptables -t nat -A POSTROUTING -j SNAT --to-source 1.2.3.4

注意：以上规则只是暂时生效，并非永久保存

**配置文件**

通过iptables添加的规则并不会永久保存，若需永久保存规则，则要将规则保存在/etc/sysconfig/iptables配置文件中

* 可通过命令 `service iptables save` 将所有的规则写入配置文件

注意：保存自定义规则会覆盖默认规则

#### 防护网站安全-iptables策略


**只允许特定流量通过，禁止其他流量**

允许SSH流量（很重要！！！若是远程管理Linux主机并修改iptables规则，则必须允许来自客户端主机的SSH流量并确保这是第一条iptables规则，否则很可能由于配置失误而将自己锁在外面(◐_◑)）

* `iptables -A INPUT -p tcp --dport 22 -j ACCEPT`

允许DNS流量（没有这个功能服务也能正常进行，只是域名解析会慢一些）

* `iptables -I INPUT 1 tcp --sport 53 -j ACCPET`
* `iptables -I INPUT 1 udp --sport 53 -j ACCPET`

允许业务流量（这里假设我们是一个网站服务器 ）

* `iptables -I INPUT 1 -p tcp --dport 80 -j ACCEPT`

所有策略的最后一条，一定要禁用其他所有流量

* `iptables -A INPUT -j DROP`

**阻拦攻击流量**

方法一：通过netstat信息来发现攻击源

Netstat 命令用于显示各种网络相关信息，如网络连接，路由表，接口状态 (Interface Statistics)，masquerade 连接，多播成员 (Multicast Memberships) 等等。参数有：

-a (all)显示所有选项，默认不显示LISTEN相关

-t (tcp)仅显示tcp相关选项

-u (udp)仅显示udp相关选项

-n 拒绝显示别名，能显示数字的全部转化成数字。

-l 仅列出有在 Listen (监听) 的服務状态

-p 显示建立相关链接的程序名

-r 显示路由信息，路由表

-e 显示扩展信息，例如uid等

-s 按各个协议进行统计

-c 每隔一个固定时间，执行该netstat命令，直到用户中断它

提示：LISTEN和LISTENING的状态只有用-a或者-l才能看到

这里假设是一台网站服务器，可以先用命令 `netstat -tn | grep s_ip | awk '{print $5}' | awk -F ":" '{print $1}' |  sort |uniq -c` 统计出连接到本服务器的 ip 地址和 其对应的TCP 连接数。

然后使用命令 `iptables -I INPUT 1 -s ip -j DROP` 来禁止掉攻击源。

方法二：

