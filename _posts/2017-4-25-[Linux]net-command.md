---
layout:     post
title:      "【Linux】Linux常用网络命令"
date:       2017-04-25 19:05:00
author:     "Yuki"
---

又是京东面试，问了我怎么查看所有TCP连接，当时我只想起了netstat这个命令，面试官问还有别的吗，实在没想起来，(/▽╲)刚好最近看鸟哥的服务器篇，有这个专题，就总结一下把(′▽`〃)

### 设置网络参数

**ifconfig**

ifconfig 命令可以手动启动、查看与修改网络接口的相关参数。其语法为：

* `ifconfig {interface} {up|down}`  查看与启动接口
* `ifconfig interface options` 设置与修改接口

选项和参数：

interface：网卡接口的名称，eth0,eth1,ppp等

options：

* up、down：启动、关闭网络接口
* mtu :设置不同的mtu值
* netmask：设置子网掩码
* broadcast：设置广播地址

直接输入 `ifconfig interface` 可以查看网卡的基本信息，显示如下：

<img src="../../../../../img/blogs/net-command/01.png">

基本参数我就不提了，其中一些信息含义如下：

* txqueuelen 代表用来传输数据的缓冲区长度
* RX 一栏说的是网络启动到现在为止的收包情况，packets是收包数量，errors代表数据包发送错误的数量，dropped是造丢弃的包的数量
* TX 一栏的网络启动到现在为止的发包情况，其中collision 代表数据包冲突的情况，若发生太多次，表示网络状况不太好。

注：ifconfig 修改的配置在重启网卡后就会失效，所以要启动某个网卡，但又不让它具有IP参数时，直接使用`ifconfig ethx up`即可。



**ifup、ifdown**

ifup和ifdown这两个程序其实是script，它会直接到/etc/sysconfig/network-scripts目录下查找对应的配置文件，所以在使用前请确定ifcfg-ethx存在于正确的目录内，否则会启动失败。

要注意的一点是若是采用了ifconfig修改配置后，就无法再以ifup、ifdown的方式来启动和关闭网卡了，因为该方式会自动比对当前的网络参数与ifcfg-ethx的是否相等，若否，就会放弃本次操作，所以只能用 `ifconfig ethx up/down `来启动和关闭网卡。

**route**

一般来说，只要有网络接口，就会产生路由，可以用 route 来查看路由信息。语法如下：
    
`route [options]`  单纯的查看路由状态

options：

-n 不使用通信协议或主机名，直接使用 IP 或 port number

-ee 显示更详细的信息

下面看看用 `route -n` 显示的信息吧

<img src="../../../../../img/blogs/net-command/02.png">

其中Destination 和 Genmask 这两个参数是 network 和 netmask了，即他们组成一个完整的网络。

Gateway 指明该网络是通过哪个网关连接出去的，若是 0.0.0.0则表示该路由由主机直接传送，也就是可以通过局域网的MAC直接发送；若显示IP的话，表明该路由需要经过网关（路由器）的帮忙才能发出。



`route add/del [-net | -host] [网络或主机] netmask [mask] [gw | dev]` 添加或删除路由

选项和参数：

-net 表示后面接的路由为一个网络

-host 表示后面接的为连接到单部主机的路由

netmask 可以设定netmask决定网络大小

gw gateway的简写，后面接IP

dev 如果只是指定由哪一块网卡连接出去，则使用这个设置，后面接网卡名称

**ip**

基本上，这个命令就是综合了route和ifconfig，相当强大(⊙o⊙)。

基本语法为：`ip [option] [动作] [命令]`

option 主要有 -s ,可以显示出设备的统计信息，例如接收包的数量等。

动作和命令即是对网络参数进行的操作。主要有对设备、路由的操作等。

* 关于接口设备的相关设置： ip link

基本语法：

`ip [-s] link show` 查看该设备的相关信息

`ip link set [device] [动作与参数]` 关于接口设备的参数设置

device：eth0、eth1...

动作和参数：

up | down：启动/关闭某个接口

address：如果这个设备可以更改MAC的话，可以用这个参数更改

name：给设备命名

mtu：设置最大传输单元的值

* 关于额外ip的相关设定：ip address

基本语法：

`ip address show` 查看ip参数

`ip address [add | del] [ip参数] [dev 设备名] [相关参数]`  增加/删除ip

ip参数：主要就是网络的设置，如 192.168.100.10/24等

dev :ip参数要设置的接口，如eth0、eth1

相关参数：

broadcast：设置广播地址，若是 + 表示让系统自动计算

label：给设备设置别名，如eth0:my_eth0，将eth0命名为my_eth0

scope：这个选项的参数通常是以下几类：

1. global：允许来自所有源的连接
2. site：仅支持IPV6，仅允许本机的连接
3. link：仅允许本设备自我连接
4. host：仅允许本机内部的连接

所以基本都是使用global，默认也是global。

* 关于路由的设定：ip route

基本语法：

`ip route show` 显示路由的设置

`ip route [add | del ] [ip或网络号] [via gateway] [dev 设备]` 增加/删除路由信息

**dhclient**

使用命令` dhclient [设备] `来让设备使用DHCP协议去获得IP。

### 网络排错与查看命令

**ping**

ping命令是通过ICMP数据包来进行网络状况的报告。

基本语法：

    ping [选项与参数] IP

选项和参数：

-n：在数据输出时不进行IP和主机名的反查，直接输出IP（速度较快）

-c 数值：设置执行ping的次数

-s 数值：设置发送出去时的ICMP包大小，默认是56bytes，在某些特殊场合，比如要搜索整个网络的最大MTU时，可以设置 -s 2000 来代替

-t 数值：TTL的数值，默认是255，每经过一个节点就会减1

-W 数值：等待响应对方主机的秒数

-M [do | dont] ：主要在检测网络的MTU大小，两个常见的项目是：

1. do：代表传送一个DF标志，数据包不能分片
2. dont：代表数据包可以进行分片

举个例子，找出最大的MTU

    ping -c 2 -s 1000 -M do 192.168.122.254

若有响应，说明这个数据包可以接收，若无响应，说明数据包太大了。若是本地MTU为1500，设置-s 数值大于1500的话，会报错说本地MTU只有1500，所以此时就可以用之前的 `ip link set mtu 数值`来设置MTU吧！

这里说个题外话，MTU最好不要随便调整，因为Internet上的每台机器能支持多大的MTU我们不能都知道，乱改了MTU可能有的网站你会访问不到。那什么时候可以修改呢？在局域网里，例如集群架构的环境下，内部网络节点我们可以控制，所以可以通过修改MTU来提高网络效率。

**traceroute**

traceroute这个命令可以用来追踪两台主机间通过的节点的通信状况。

基本语法：

    traceroute [选项与参数] IP

选项与参数：

-n：在数据输出时不进行IP和主机名的反查，直接输出IP（速度较快）

-U：使用UDP的port 33434 来进行检测，这也是默认的检测协议

-T：使用TCP来进行检测

-I：使用ICMP来进行检测

-w 数值：若对方主机在几秒钟没有回复则声明不通，默认是5秒

-p 端口号：若不想使用UDP、TCP默认端口号，则在此声明

-i 设备：用在比较复杂的网络环境，如你有两条ADSL可以连接到外部，那你的主机会有两个ppp，你可以选择使用ppp0还是ppp1

**netstat**

netstat命令用于显示各种网络相关信息，如网络连接，路由表，接口状态 (Interface Statistics)，masquerade 连接，多播成员 (Multicast Memberships) 等等。

基本语法：

    netstat [选项与参数]

与路由有关的参数：

-r：列出路由表，功能如同route这个命令

-n：不使用主机名和服务名，使用IP和port number，功能如同 route -n

与网络接口有关的参数：

-a：列出所有的连接状态，包括TCP/UDP/UNIX socket等

-t：仅列出TCP数据包的连接

-u：仅列出UDP数据包的连接

-l：仅列出已在监听的服务的网络状态

-p：列出PID和program的程序名

-c：可以设置几秒自动更新一次，如-c 5为每5秒更新一次网络状态的显示

下面一张图是显示本机连接的部分截图：

<img src="../../../../../img/blogs/net-command/01.png">

关于网络连接状态的输出部分，主要有以下几项：

Proto：该链接的数据包协议，主要是TCP/UDP包

Recv-Q：由非用户程序所复制而来的总bytes数

Send-Q：由远程主机发来的但不具有ACK标志的bytes数，亦指主动连接SYN或其他的标志包所占的bytes数

Local Address：本地端的地址，可以是IP，可以是主机名

Foreign Address：远程主机IP与port number

stat：状态栏，主要的状态有：

1. ESTABLISHED：已建立连接
2. SYN_SENT：主动请求建立连接的数据包
3. SYN_RECV：接收到连接请求的数据包
4. FIN_WAIT1：主动要求断开连接的数据包，该套接字服务已中断
5. FIN_WAIT2：连接已挂断，但还在等待对方响应断开连接的确认
6. TIME_WAIT：连接已断开，但socket还在等待结束
7. Listen：通常用在服务的监听port

其实，前6个状态其实就是TCP三次握手和四次挥手的过程(*￣rǒ￣)

举个常用的小例子吧~

显示出目前已启动的网络服务：`netstat -tulnp`

查看本机上所有网络的连接状态：`netstat -atunp`

使用-p参数可以显示PID与program文件名，比如你发现有远程主机和你的主机建立连接，你想断开这条连接，就可以使用kill pid 来切断。

**tcpdump**

tupdump这个软件可以说是个黑客软件，听起来很酷吧！因为它不但可以分析数据包的流向，连数据包的内容也可以监听，如果使用明文传输数据的话，在router或hub上就可能被监听走了。

基本语法：

    tcpdump [-Aennqx] [-i 接口] [-w 存储的文件名] [-c 次数] [-r 文件名] [所要摘取的数据包的数据格式]

选项和参数：

-A：数据包的内容以ASCII显示，通常用来抓取WWW的网页数据包数据

-e：使用数据链路层的MAC数据包数据来显示

-nn：直接以IP及port number显示，而非主机名与服务名称

-q：仅列出较为简短的数据包信息，每一行内容都比较精简

-X：可列出十六进制以及ASCII的数据包内容，对于监听数据包内容很有用

-i：后接要监听的网络接口，如eth0

-w：后接文件名，将监听的数据保存到文件中

-r：后接已存在的文件，从该文件中将数据包的数据读取出来

-c：监听的数据包数，不加这个参数的话，会持续不断的监听，直到输入Ctrl+c为止

所要摘取的数据包的数据格式，常见表示方法有：

'host foo'，'host 127.0.0.1'：针对单台主机进行数据包捕获

'net 192.168'：针对某个网络进行数据包捕获

'src host 127.0.0.1'，'dst net 192.168':加上来源和目标的限制

'tcp port 21'：还可以针对通信协议检测，如tcp、udp、arp、ether等

还可以用 and 和 or 来进行数据包的整合显示

下面举个例子来看看抓到的数据是什么样的吧：

先在一个终端输入 `tcpdump -nn -i lo`，再打开另一个终端输入 `ssh localhost`，我抓到的包如下图

<img src="../../../../../img/blogs/net-command/03.png">

前两行数据是tcpdump的基本说明，第三行开始可以看出Client向Server发出SYN包请求建立连接，第四行是Server向Client发送SYN和ACK确认建立连接，第五行Client向Server发送ACK确认，至此完成TCP三次握手，后面就是数据传输了。

组合功能的使用也举个例子：

使用tcpdump监听eth0网卡且通信协议为TCP 22号端口，来源地址为192.168.1.111

`tcpdump -i eth0 -nn 'tcp port 22 and src 192.168.1.111'`

**文字接口网页浏览**

文字接口网页浏览工具就是在文字接口上浏览网页的工具，这里主要介绍links和wget。

**links**

这个命令可以用来浏览网页，但其实其最大的作用是查阅Linux本机上的以HTML语法写成的文件数据。例如，/usr/share/doc下有一些HTML格式的文件，若是以 cat 或 vim 查看的话，有碍阅读，这时候用Links就是个好方法！

基本语法：

`links [options] [URL | file]`

选项和参数：

-anouymous [0|1]：是否使用匿名登录

-dump [0|1]：是否将网页数据直接输出到standard out 而非 links软件功能

-dump_charset：后面接输出到屏幕的语系编码，简体中文用cp936

进入画面后，排版可能会有偏移（我查看了百度，偏移相当大。。。），可以使用上下键来移动光标，使用左右键来移动上下页。还有一些常用的功能按键如下：

h：history，显示曾经浏览的网页

g：Goto URL，按g后输入网页地址进入另一个网页

d：download，将该链接的数据下载到本机成为文件，具体操作按指示就行了，很简单，会用计算机的都没啥问题

o：options，进入功能参数的设置值修改，最终结果会写入/.elinks/elinks.conf中

q：Quit，退出

**wget**

如果说links重点在于网页浏览，那wget就是在于网页数据的获取了。

基本语法：

`wget [options] [URL]`

选项与参数（更多的参考man手册）：

要连接的网站有用户名和密码保护时：

--http-user=username

--http-password=password

不显示捕获数据时的显示信息：

--quiet

使用方法：

输入 wget \ 进入工作界面，然后输入要下载的文件的地址，即可完成下载了！下载的文件默认是保存在当前目录，可以使用参数-P指定保存目录，可以使用-O指定保存的目录及文件名。


### 远程连接命令和即时通信软件

**Telnet**

Telnet不但可以直接连接到服务器上，还可以用来连接BBS，不过，Telnet本身的数据在传输过程中使用的是明文，所以数据传输会有点危险，有可能会被监听！

这个软件centos默认是没有安装的，所以需要通过 `yum install telnet`来安装

基本语法：

`Telnet [host | ip [port]]`

语法比较简单，也没有多余的参数，它的一个重要的作用就是可以连接到某个port，看看这个port是否正确启动了。 

**FTP连接软件**

现在通信基本都是用E-mail软件，不过这有单封邮件容量的限制，若是想要一口气传输几百MB的邮件，还是得通过FTP协议。文字接口的协议软件有ftp和lftp两个。

* ftp

ftp这个命令主要是用来处理ftp服务器的下载数据。

基本语法：

`ftp [host | ip] [port]`

* lftp

单纯的使用ftp比较麻烦，而lftp是默认使用匿名登录FTP服务器的软件，可以用类似网址的方式取得数据。

基本语法：

`lftp [-p port] [-u user[,pass]] [host | ip]`

选项与参数：

-p：后面接远程主机FTP服务器提供的port

-u：后面接用户名和密码，若没有此参数，会默认尝试以匿名用户的方式登录

-f：可以将命令写入脚本

-c：后面加所需的命令

lftp的脚本功能真的很棒呀，比如定时去某网站下载文件的脚本可以这么写：

`vim auto_download.sh`

    lftp -c "open hostname/ip
    mget -c -d file
    bye
    "





















