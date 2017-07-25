---
layout:     post
title:      "【Linux】DNS服务器搭建"
date:       2017-04-22 20:01:00
author:     "Yuki"
---

最近在看鸟哥的DNS服务器搭建，之前在网上看了苏勇老师的网课，也看了很多网友的博客，感觉都不系统ค(TㅅT)，所以我尽量总结得系统一些！

## DNS基础知识

**DNS如何实现解析**

实际上，DNS解析是通过NSSwitch这个框架实现的，这个框架提供了名称解析的平台，它的工作方式就是通过找寻在此平台下名称解析的工具实现名称解析的，DNS只需要在NSSwitch框架上找寻能够实现名称解析的工具，那这工具就是libnss_files.so和libnss_nss.so。NSSwitch这个框架对我们而言展示的其实就是一个配置文件:/etc/nsswitch.conf。 

当我们去访问一个主机名时，主机名是没法直接建立联系的，这时候它会调用一个库文件完成从主机名到ip的转换，这个转换机制在本机上就叫stub resolver（名称解析器），它会通过库调用去/etc/nsswitch.conf找查询次序，根据查询次序用相应机制去完成解析。

**DNS查询**

DNS是分布式数据库，上级仅知道其直接下级，默认下级只知道根的位置（得配置）。它通过分级的方式自顶向下来进行查询的。最上面的一级是根域，第二层是顶级域。

顶级域分为三种： 

* 组织域：.com , .org , .net ,.cc
* 国家域：.cn , .tw , .hk , .jp
* 反向域：IP-->FQDN(正向：FQDN-->IP,正向和反向不在同一个数据库)

若是每次查询都去根域，则大部分带宽都会被用来做DNS解析，所以就有了缓存，将查询的结果缓存下来，且缓存到一个公共位置，建立个缓存服务器，服务器专门接收每台主机的访问和请求，这样就能节省带宽。但是服务器缓存的数据是不具有权威的，由主机的直接上级告诉缓存服务器缓存的超时时间。因此，DNS服务器向缓存服务器发送回应时，不止要包含主机与IP的对应，还要包含TTL。TTL时间越长，服务器越空闲，对服务器压力越小。 

一个服务器可以管理多个域，只要维护多个数据库就可以了。所以，一个ip可以有多个主机名；一个主机也可以有多个ip，但是DNS的高级机制只会返回一个ip。

DNS的查询分为递归查询和迭代查询：

* 递归查询

递归查询是一种DNS服务器的查询模式，在该模式下DNS 服务器接收到客户机请求，必须使用一个准确的查询结果回复客户机。如果DNS 服务器本地没有存储查询DNS 信息，那么该服务器会询问其他服务器，并将返回的查询结果提交给客户机。客户机只发出一次请求。

* 迭代查询

DNS 服务器另外一种查询方式为迭代查询，DNS 服务器会向客户机提供其他能够解析查询请求的DNS服务器地址，当客户机发送查询请求时，DNS 服务器并不直接回复查询结果，而是告诉客户机另一台DNS 服务器地址，客户机再向这台DNS 服务器提交请求，依次循环直到返回查询的结果。这种情况下客户机会发出多次请求。

实际情况下，根是不参与递归的。互联网是两段式查询，对于客户端，使用递归查询，即客户端只发出一次请求，而对于服务器，使用迭代查询，即服务器发出多次请求直至得到结果，再将结果返回给客户端。对于服务器来说，一般只给其内部或兄弟主机提供递归服务。 

**DNS服务器类型**

* 主从DNS服务器

通常，DNS服务器不会只有一台，而是采取主从结构，主服务器只有一台，也只有它能进行数据修改操作，而从服务器请求数据的同步，它的数据都是从主服务器上拉取下来的。DNS服务器要定义五个属性：版本号serial number、刷新时间refresh、重试时间retry、过期时间expire、否定答案的缓存时间nagative answer TTL。

serial number规定了若是从服务器的版本号和主服务器不同，则会更新它的数据；refresh定义从服务器到主服务器上进行版本号比对的时间；retry则定义了当一次拉取不成功时，再次拉取的间隔时间，retry是小于refresh的；expire定义了过期时间，即从服务器一直得不到主服务器的回应，超过expire则证明主服务器可能挂掉了，若是主服务器挂掉，则从服务器也跟着“自杀”了;nagative answer TTL规定了在服务器端没有客户端查询的答案时的否定回应的缓存时间，若是没有这个TTL，客户端很可能一直向服务器发起查询请求。

* 缓存DNS服务器

和上面讲过的一样，缓存服务器只负责缓存，不提供权威查询结果。

* 转发器

转发器不负责缓存，只转发请求。在客户端发出DNS请求时先到达这台服务器即使客户端不能访问互联网，这台服务器本身也是可以访问互联网的，它能将结果迭代出来返回给客户端，但由于客户端没法访问互联网，所以解析域名是能完成的，只是上不了网而已。所以这种转发器就是在客户端不能访问互联网或没法到达某个DNS服务器时，可以由转发器帮忙转发请求。

**DNS资源记录**

DNS数据库中，每一个条目都称作一个资源记录，而资源记录组合成为区域文件。

资源记录的格式（Record Resource ，RR），TTL可以统一写在最前面：

    TTL 600;
    
    NAME	[TTL] IN 	RRT		VALUE

字段含义如下：

NAME：资源名称 

IN：表明是Internet上的资源

VALUE：资源的值

RRT：资源记录类型,主要有以下几种：

* SOA（start of authority）：定义了主从服务器之间如何同步数据，即如何实现区域传送，以及起始授权对象是谁。这个记录必须出现在第一条，称为起始授权记录

举例：

    ZONE NAME		TTL		IN		SOA		FQDN	ADMINSTRATOR_MAILBOX（
    
    			serial number
    			refresh
    			retry
    			expire
    			na ttl
    
    
    ）

时间单位：M、H、D、W，默认单位是S

管理员邮箱格式：不能使用@符合，应改为.

ZONE NAME：所有区域名都可以直接写为@

可以用;来注释

具体例子：

    @ 	IN	SOA		nsl.princess7.top		admin.princess7.com(
    
    		2017051801
    		1H
    		5M
    		1W
    		1D
    
    
    )
    
* A（address）：FQDN-->IPv4，只能定义在正向区域文件中
* PTR（pointer）：IP-->FQDN，只能定义在反向区域文件中
* AAAA：FQDN-->IPv6
* MX（mail eXchanger）:Zone NAME --> FQDN，有优先级概念,优先级从0-99，数字越小，级别越高，当我们请求邮件服务器时，会优先选择优先级高的，只能定义在正向区域文件中

举例：邮件服务器也要成对出现，附加A记录

    princess7.top	IN		MX	10		mail.princess7.top
    mail.princess7.top		IN	A		1.1.1.3

* NS（name server）:Zone NAME --> FQDN，这是DNS服务器记录，主从的区分靠配置文件中的区域定义。

举例：NS记录一定要成对出现，虽然一般不会用到，因为其上级已经制定了它的地址

    magedu.com		IN		NS		ns.magedu.com
    
    ns.magedu.com		IN		A		1.1.1.2

* CNAME：别名,FQDN-->FQDN

举例:

`www2.princess7.top		IN		CNAME		www.princess7.top`

**域和区域**

域是逻辑概念，区域是物理概念。比如域叫princess7.top，正向区域和反向区域是真实存在的数据文件。域和区域没有必然的包含关系，就某个域来说，区域比域小，但是域的授权来自上级。

<img src="../../../../../img/blogs/DNS/04.png">

**区域类型**

针对传输数据时的区域类型有：

* 主区域：master

* 从区域：slave

* 提示区域：hint，定义根在何处

* 转发区域：forward，自己没有的记录，不用找根，找转发区域处


## 查询命令

在开始搭建服务器之前，先来看几个DNS的正、反解查询命令吧，因为后面搭好了服务器，会用这些命令进行测试，放在平时，用来解析域名/IP也是可以的。

**host**

基本语法：`host [-a] FQDN [server]`

选项和参数：

-a：代表列出该主机所有相关信息，包括ip、ttl与排错信息等

-l：若后面所接域名设置允许 allow-transfer 时，列出该域名所管理的所有主机对应的数据

server：当要利用非 /etc/resolv.conf 内的DNS主机来查询域名的IP时，可以使用这个参数

**nslookup**

基本语法：`nslookup [FQDN] [server]`

选项和参数：

* 可以直接在 nslookup后加域名或IP进行查询
* 若在nslookup后未加任何域名或IP，则默认进入nslookup的查询功能，在查询功能中，可以输入其他参数进行查询，如：

set type=any：列出所有信息正解方面的配置文件

set type=MX：列出与MX相关的信息

注意：在nslookup的查询界面中输入了set type=any 或 其他参数，就无法再进行反解的查询了，因为any或MX等标志都是记录在正解Zone中。

**dig(未来的主流，多用)**

基本语法：`dig [options] FQDN [@server]`

选项和参数：

@server：不以/etc/resolv.comf的设置来作为DNS查询时，可在此填入其他IP

+trace：从.开始追踪

-t type：查询的数据主要有MX、NS、SOA等类型

-x：查询反解信息

由于dig的输出很详细，而且是分段显示的，所以很适合作为DNS追踪回报的一个命令，可以通过这个命令了解DNS数据库是否设置正确，并进行排错。

## 搭建DNS之前的准备

在搭建DNS服务器之前，先来看看几个需要注意的设置，避免后面搭建后的测试环节出错。

**iptables**

DNS使用的是端口号53，通常，DNS是以UDP协议来传输数据，但是以下两种情况，会用TCP来传输数据：一是DNS进行区域传输为了保证数据的完整性和可靠性（服务器之间的传输，例如从服务器从主服务器获取数据，区域传送有完全区域传送axfr和增量区域传送ixfr）时；二是DNS的响应报文中TC（删减标志）被设置为1，代表响应文大小超过512字节，UDP是不允许分片的，所以此时也会通过TCP来进行数据传输。

所以，我们的防火墙要放行TCP和UDP的port 53 。

**/etc/hosts**

改配置文件下保存了Hostname和IP的对应，一般而言，Linux主机下默认的解析都以 /etc/hosts 为主，可以查看一下 /etc/nsswitch.conf，找到如下记录：

<img src="../../../../../img/blogs/DNS/01.png">

上面的 files 就是靠lib_files查 /etc/hosts 文件，通过该文件完成主机名和ip的对应；DNS指使用DNS服务进行查询，所以我们保留默认配置就好，这样查询速度较快。

**/etc/resolve.conf**

这个文件声明了要使用的DNS服务器的IP，因为我们是在本机上做测试，所以把 nameserver 改为 127.0.0.1 ，如下：

<img src="../../../../../img/blogs/DNS/02.png">

但在正常生产环境下，建议设置多台DNS服务器地址，因为当第一台宕机时，客户端能使用第二台来进行查询，不过网络正常时，永远都只有第一台提供服务。还有，尽量不要设置超过三台以上的DNS服务器地址，因为若局域网出现问题，会导致无法连接到DNS服务器，但是主机还是会向多台服务器发送连接请求，每次连接都有timeout时间的等待，会导致浪费很多的时间。

**DHCP**

我们的主机使用DHCP获得IP地址，系统会主动使用DHCP服务器传来的数据进行系统配置文件的修改，所以你可能会发现你修改了 /etc/resolve.conf ，在不久后文件又会恢复原来的样子。此时，需要在/etc/sysconfig/network-scripts/ifcfg-enp0s3 文件中把PEERDNS改成NO，如下：

<img src="../../../../../img/blogs/DNS/03.png">

**NetworkManager**

若开启了这个服务，据鸟哥说可能有时候会产生一些奇特的现象，所以建议还是关闭吧。

**SELinux**

SELinux最好也不要开启，可以使用`setenforce 0 `这个小开关来设置为 permissive 模式。


## BIND及相关配置文件简介

##### BIND安装及简介

BIND是DNS软件的一个开源实现，最早是由伯克利大学的一名学生编写的，不得不说，别人家的同学就是牛啊！！！现在是由ISC来编写和维护。

我们可以通过yum安装BIND及一些相关工具：

    yum -y install bind bind-chroot bind-utils

bind-utils是bind软件提供的一组DNS工具包,里面有一些DNS相关的工具.主要有:dig,host,nslookup,nsupdate.使用这些工具可以进行域名解析和DNS调试工作。

上面主要要提的是 bind-chroot，chroot代表“change to root”，早期bind默认将程序启动在/var/named中，但是该程序可以在根目录下的其他目录任意转移，所以若是程序出了问题，可能会造成整个系统的危害，所以这里将目录chroot指定为bind程序的根目录，这样即使程序被攻击，大不了也是在该目录下被破坏而已。需要注意的是，新版本的CentOS6.x已经将chroot所需要的目录通过在启动脚本中执行 `mount --bind /var/named /var/named/chroot/var/named `来进行挂载了，所以我们根本不需要切换到/var/named/chroot目录进行相关配置了，直接使用正规目录即可！！！之前我看苏勇老师的课，他还是切换到了chroot目录，然后鸟哥的书又没切换，一度让我很混乱...我在chroot里写了配置文件，reload的时候就报错说找不到配置文件，一度很迷惑，后来才发现，目录搞错了...也是醉...不知道咋踩进这个坑的...不过在踩坑过程中，也学了不少排错的方法，也算是踩坑的收获了。

##### bind配置文件

不同于其他服务，bind在安装后不会有预置的配置文件，但在其文档文件夹（/usr/share/doc/bind-9.9.4）里提供了配置文件的模板，我们可以直接拷贝过来。

    cp -rv /usr/share/doc/bind-9.9.4/sample/etc/* /var/named/chroot/etc

    cp -rv /usr/share/doc/bind-9.9.4/sample/var/* /var/named/chroot/var

**/etc/named.conf**

主配置文件，在里面需要添加BIND进程的工作属性、区域的定义。其模板格式如下（我只复制了一部分，因为注释和配置太长，这里就介绍下可能会用到的吧）：

    options
    {
		// Put files that named is allowed to write in the data/ directory:
		directory 		"/var/named";	// "Working" 
		dump-file 		"data/cache_dump.db";
	    statistics-file 	"data/named_stats.txt";
	    memstatistics-file 	"data/named_mem_stats.txt";
		//listen-on port 53	{ any; };
		listen-on port 53	{ 127.0.0.1; };
	
		//listen-on-v6 port 53	{ any; };
		listen-on-v6 port 53	{ ::1; };
		allow-query		{ localhost; };
		allow-query-cache	{ localhost; };
		allow-transfer {none；};	
		recursion yes;
		notify yes;
		
	};


    zone "." IN {
		 type hint;
		 file "/var/named/named.ca";
	}；

options是用来规范DNS的权限的。其中，listen-on port 53就是来指定监听的服务器IP，默认是本机IP。allow-transfer是设置允许谁进行区域传输的，一般只允许从服务器从主服务器上获取数据,没有从服务器就设置为none，有从服务器的话，在括号里填上从服务器的IP，这条设置可以写到区域定义里；allow-query定义允许谁来进行xafr查询，使用情况不多，一般都不允许别的主机查到你配置的所有主机；recursion定义是否递归，还有一个选项allow recursion{ ip/mask; }在括号中填上允许递归的网段，这样我们的DNS服务器就只给某个网段递归啦；notify yes;表示启用通知功能，一旦主服务器有更新，就通知从服务器去同步数据。directory是必须要存在的最重要的选项，它定义了数据文件目录。

zone是设置你的domain name 、Zone类型（hint、master、slave，forward）以及 zone file 的名称和所在，区域定义的语法中：每一个地址后都必须加分号;，大括号{}前后一定要有空格。

其他我没截图的部分是设置本机管理接口及相关的密钥文件，因为这是一些高级功能，我还没学，等学到了再来更新博客吧。

**区域定义格式**


    zone "zone name" IN {

		type master|slave|hint|forward;
	
		；若为主区域（分号用来注释）
		file “区域数据文件”
	
		;若为从区域
		file “区域数据文件”
		masters { master1_ip;master2_ip; };
	

    };



**/etc/rndc.key**

rndc：remote name domain controller，这是密钥文件，配置信息在/etc/rndc.conf。

**/var/named/**

在/var/named/目录下自己定义区域数据文件。

**/etc/rc.d/init.d/named**

这是named的启动脚本，支持参数 start、stop、restart、status、reload。

## 搭建本地DNS服务器

搭建本地DNS服务器其实主要就分两步：

1. 在主配置文件中添加域的定义
2. 添加所定义的域的数据文件

下面列出我的具体操作吧~

* 编辑主配置文件，`vim /etc/named.conf`，编辑内容如下：

<img src="../../../../../img/blogs/DNS/05.png">

因为只有directory是必须要的，我这里就只添加了它的配置，下面的zone分别是根域、本地域、反解域和自己定义的域。因为它们都是第一个定义的域，没有任何它们的主服务器存在，所以类型都是master。

* 添加域的数据文件

在/var/named/目录下添加上面四个域的数据文件。在该目录下，默认是存在根域、本地域、反解域的数据文件的，分别是named.ca、named.localhost和named.loopback，所以实际上我们需要编辑的只有我们自己添加的域princess.top。使用`vim /var/named/princess.top.zone`,编辑如下：

<img src="../../../../../img/blogs/DNS/06.png">

这里要注意，第一条资源记录一定要是SOA记录，然后NS记录代表你的DNS服务器，所以下面DNS服务器的A记录一定要是一个可以上网的IP！！！之前我没注意，随意配了个IP，结果根本上不了网。。。这里我用的是我局域网分配的ip。还有就是给ns1配A记录的时候，一定要用全称域名，不然后面这个用dig查询的时候你会发现ns1的A记录并不是你配的ip！！！

* 其他配置及测试

配置完以上两个文件后，可以用以下两个命令来检查主配置文件和数据文件的语法错误。

`named-checkconf`

`named-checkzone "princess.top" /var/named/princess.top.zone`

若没有消息则就是好消息~

然后更改你的 /etc/resolve.conf 文件，将其中nameserver更改为你的DNS服务器的ip，如下：

<img src="../../../../../img/blogs/DNS/07.png">

然后用`setenforce 0 `将SELinux改为permissive。

然后防火墙放行53号端口：

`iptables -I INPUT 1 -p tcp --sport 53 -j ACCEPT`

`iptables -I INPUT 1 -p udp --sport 53 -j ACCEPT`

准备工作做完了，可以使用dig命令进行测试噜~我的测试结果如下~能够成功进行解析啦~

<img src="../../../../../img/blogs/DNS/08.png">

<img src="../../../../../img/blogs/DNS/09.png">

##搭建主从DNS服务器

其实之前搭好了DNS，要搭主从结构就比较容易了，我的设置是从服务器和主服务器在一个网段。主要有以下几步：

* 用setup命令修改从服务器的网络配置，如下图：在static IP处填上从服务器IP，在主服务器上用route命令查询网关地址，并填在Default Gateway IP处。在DNS选项中在Primary DNS处填上从服务器的IP，在DNSsearch path处填上搜索域，即我这儿填的就是 princess.top

<img src="../../../../../img/blogs/DNS/10.png">

<img src="../../../../../img/blogs/DNS/11.png">

* 在主服务器的配置文件中添加允许区域传送的从服务器ip，如下图：只需要在进行区域传送的zone中添加就行啦~

<img src="../../../../../img/blogs/DNS/12.png">

* 在主服务器的数据文件中添加从服务器ns2的记录，这里我就不截图了，和其他资源记录格式一样就行。数据文件在/var/named/princess.top.zone

* 在从服务器上添加主配置文件，配置文件和主服务器的配置文件不同点在于princess.top域的配置，并且在从服务器中，数据文件是保存在/var/named/slaves/目录下的。如下：

<img src="../../../../../img/blogs/DNS/13.png">

所有配置完成后，记得关闭SELinux，放行tcp、udp的53号端口，然后从服务器ssh到主服务器上，重启named服务，可以用 `tail /var/log/messages` 命令看到区域传送完成了~在从服务器的 /var/named/slaves/目录下也可以找到数据文件。

## rndc用法

#### rndc简介

rndc（Remote Name Domain Controllerr）是一个远程管理bind的工具，通过这个工具可以在本地或者远程了解当前服务器的运行状况，也可以对服务器进行关闭、重载、刷新缓存、增加删除zone等操作。  

使用rndc可以在不停止DNS服务器工作的情况进行数据的更新，使修改后的配置文件生效。在实际情况下，DNS服务器是非常繁忙的，任何短时间的停顿都会给用户的使用带来影响。因此，使用rndc工具可以使DNS服务器更好地为用户提供服务。在使用rndc管理bind前需要使用rndc生成一对密钥文件，一半保存于rndc的配置文件中，另一半保存于bind主配置文件中。rndc的配置文件为/etc/rndc.conf，在CentOS或者RHEL中，rndc的密钥保存在/etc/rndc.key文件中。rndc默认监听在953号端口（TCP），其实在bind9中rndc默认就是可以使用，不需要配置密钥文件。

#### rndc配置

rndc通常我们不会设置来让远程主机进行控制，这样做相当危险，所以下面的配置只是在本机生成配置文件，然后可以用rndc命令来进行刷新缓存操作（这个操作比较重要）。

* 用rndc-confgen >> /etc/rndc.conf生成配置文件,配置文件内容如下：

<img src="../../../../../img/blogs/DNS/14.png">

从文件14行可以看出，我们需要把14行以下的内容拷贝到 /etc/named.conf 中。

* 将以上内容保存到/etc/named.conf中后，删除默认存在的/etc/rndc.key文件，重启named服务，就可以使用rndc来控制本机DNS服务器啦~

下面截张图证明确实可以使用rndc了~

<img src="../../../../../img/blogs/DNS/15.png">

## DNS子域授权

子域授权，也就是DNS的分布式。

通过在原有的域上划出一个小的区域，并给新DNS服务器管理。
如果有客户端请求解析在这个划分区域中的域名，则只要找新的子DNS服务器。
这样的做的好处可以减轻主DNS的压力，也便于管理。

试想一下，假如在princess.top这个域内，我们想进一步划分子域，即划分为市场部 market.princess.top 和 fin.princess.top，要如何实现呢？其实也很简单，因为所有我们想要加入的资源，在DNS看来就是增加记录而已，所以大致步骤如下：

* 在主服务器的数据文件中增加你要添加的域的NS记录和A记录，如下：（因为我每次DHCP上网的ip不同，所以当我进行到这一步的时候，我的ip和之前图片所示的ip已经不同了，了解思想就行）



## 视图




