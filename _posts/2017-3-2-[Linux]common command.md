---
layout:     post
title:      "【Linux】Linux系统常用命令"
date:       2017-03-2 22:10:00
author:     "Yuki"
---

#### 日期时间

* 命令 date 可以查看、设置当前系统时间。

	格式化显示时间：date +%Y--%m--%d （"--"为分隔符，可随意指定）。

* 命令 hwclock（clock）可以查看硬件时钟的时间。

* 命令 cal 可以查看日历。

* 命令 uptime 可以查看系统运行的时间。

#### 基本的输出、查看命令

* 命令 echo 用以显示输入的内容。

* 命令 cat 用以显示文件的全部内容。

* 命令 head 用以显示文件的头几行内容（默认是10行）。
     
	-n 指定显示的行数

* 命令 tail 用以显示文件的后几行（默认是10行）。

	-n 指定显示的行数

	-f 追踪文件的更新（一般用于查看日志，命令不会退出，而是持续显示追加的内容）

* 命令 more 用于翻页显示文件内容（只可向下翻页）。

* 命令 less 用于翻页显示文件内容（可上下翻页）。

#### 查看硬件信息

* 命令 lspci 用来查看PCI设备（PCI不是系统和计算机外围设备。PCI是Peripheral Component Interconnect(外设部件互连标准)的缩写，是目前个人电脑中使用最为广泛的接口，几乎所有的主板产品上都带有这种插槽。它为显卡，声卡，网卡，MODEM等设备提供了连接接口）。

	-v 查看详细信息

* 命令 lsusb 用来查看USB设备。

	-v 查看详细信息

* 命令 lsmod 用来查看加载的模块（驱动）。

#### 关机、重启

* 命令 shutdown 用以关闭、重启计算机。

	-h 关闭计算机

	-r 重启计算机

	shutdown -h/r 时间 可以指定关机/重启的时间。

* 命令 poweroff 用以立即关闭计算机。

* 命令 reboot 用以立即重启计算机。

#### 查找

* 命令 locate 用来快速查找文件和文件夹。该命令需要预先建立数据库，而数据库默认每天更新一次，所以在使用此命令前可以用命令 update 更新数据库。

* 命令 find 用来高级查找文件和文件夹。

	find 查找目录 查找参数
	
	常用参数有：

	-name 按名字查找

	-perm 按执行权限查找

	-type 按类型查找

	-size 按文件大小查找

	-user 按文件所属用户查找

	-group 按组来查找

	-mtime   -n/+n 按文件更改时间来查找文件，-n指n天以内，+n指n天以前

	-atime    -n/+n 按文件访问时间来查找文件

	-exec   command   {} \;  将查到的文件执行command操作,{} 和 \;之间有空格

#### 归档、压缩

* 命令 zip 用以压缩文件

	zip 文件名.zip 要压缩的文件

* 命令 unzip 用以解压.zip文件

* 命令 tar 用以归档文件

	tar -cvf 文件名.tar 要压缩的文件

	tar -czvf 文件名.tar.gz 要压缩的文件

	常用参数有：

	-c 建立一个压缩文件

	-x 解压一个压缩文件

	-t 查看 tarfile 里的文件

	注：参数-c、-x、-t只能存在一个

	-v 压缩过程中显示文件

	-z 归档并使用gzip压缩

	-f 使用文件名，在f后要立即接压缩后的文件名，不能再跟参数了

	-p 使用原文件的属性

	-P 可以使用绝对路径压缩

	--exclude FILE 在压缩的过程中，不要将 FILE 打包！ 

	