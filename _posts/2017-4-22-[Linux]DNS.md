---
layout:     post
title:      "【Linux】DNS服务器搭建"
date:       2017-04-22 20:01:00
author:     "Yuki"
---

最近在看鸟哥的DNS服务器搭建，之前在网上看了苏勇老师的网课，也看了很多网友的博客，感觉都不系统ค(TㅅT)，所以我尽量总结得系统一些！

在开始搭建服务器之前，先来看几个DNS的正、反解查询命令吧，因为后面搭好了服务器，会用这些命令进行测试，放在平时，用来解析域名/IP也是可以的。

### 查询命令

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

### 搭建DNS之前的准备

在搭建DNS服务器之前，先来看看几个需要注意的设置，避免后面搭建后的测试环节出错。

**iptables**

DNS使用的是端口号53，通常，DNS是以UDP协议来传输数据，但是以下两种情况，会用TCP来传输数据：一是DNS进行区域传输（服务器之间的传输，例如从服务器从主服务器获取数据）时；二是DNS的响应报文中TC（删减标志）被设置为1，代表响应文大小超过512字节，UDP是不允许分片的，所以此时也会通过TCP来进行数据传输。

所以，我们的防火墙要放行TCP和UDP的port 53 。

**/etc/hosts**

改配置文件下保存了Hostname和IP的对应，一般而言，Linux主机下默认的解析都以 /etc/hosts 为主，可以查看一下 /etc/nsswitch.conf，找到如下记录：

<img src="../../../../../img/blogs/DNS/01.png">

上面的 files 就是指先查 /etc/hosts ，再使用DNS服务器进行查询。所以我们保留默认配置就好，这样查询速度较快。

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


### BIND及相关配置文件简介

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

主配置文件，在里面需要添加域的定义。其模板格式如下（我只复制了一部分，因为注释和配置太长，这里就介绍下可能会用到的吧）：

    options
    {
		// Put files that named is allowed to write in the data/ directory:
		directory 		"/var/named";	// "Working" directory
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
		
	};


    zone "." IN {
		 type hint;
		 file "/var/named/named.ca";
	}；

其中options，是用来规范DNS的权限的，比如是否查询，能否转发等。其中，listen-on port 53就是来指定监听的服务器IP，默认是本机IP。allow-transfer是设置是否进行区域传输的，即是否有从服务器从主服务器上获取数据,没有从服务器就设置为none，有从服务器的话，在括号里填上从服务器的IP。

zone是设置你的domain name 、Zone类型（hint、master、slave）以及 zone file 的名称和所在。

其他我没截图的部分是设置本机管理接口及相关的密钥文件，因为这是一些高级功能，我还没学，等学到了再来更新博客吧。

