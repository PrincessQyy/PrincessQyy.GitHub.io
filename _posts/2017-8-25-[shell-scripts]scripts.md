---
layout:     post
title:      "【shell-scripts】常用shell小脚本"
date:       2017-08-25 15:01:00
author:     "Yuki"
---

# 移动指定修改时间的文件

### 需求：某些目录下的文件可能过一段时间就需要清理，这个脚本的功能就是将指定年份的文件移动到某个指定目录下。

    #!/bin/bash
    
    #author:
    #	Qiyuyue
    #program:
    #   This script is used to back up files that specify the modification time
    
    if [ ! -d /tmp/etc ];then
    mkdir /tmp/etc
    fi
    
    ls -l /etc | grep 2016 | awk '{print $9}' >> file_dir
    
    for each in `cat file_dir`;do
    cp -r /etc/${each}  /tmp/etc/
    done

# 测试指定IP是否能ping通

### 需求：在服务器批量装系统前，需要测试是否具备装系统条件，即其lo口要能ping通，而其eth0 ip应该是ping不通的，此脚本就是检测ip列表的ip是否能ping通

	#!/bin/bash

	#author:
    #	Qiyuyue
    #program:
    #   This script is used to ping a machine

	for i in `cat ip_list`;do
	        ping  $i -c2 -w2  > result;
	        cat result|grep "64 bytes">&/dev/null  && echo "True" || echo "False"
	
	done

# 统计网络数据包

### 需求：某些场景下，可能会需要统计某个端口的进出数据包数，以便于分析流量等，监听时间可以指定
	
	#!/bin/bash

	#author:
    #	Qiyuyue
    #program:
    #   This script is used to count the number of packages
	#usage:
	#	bash $0 port time

	tcpdump -i eth0 "port $1" &>tmp.txt &
	
	sleep $2
	#kill tcpdump process ,remenber to filter command "grep"
	kill `ps aux | grep "tcpdump" |grep -v "grep" | awk '{print $2}'`

	a=`cat tmp.txt | grep "received" | awk '{print $1}'`
	
	echo "Packages: ${a}"

# 提取文本中的链接信息

### 需求：文本中可能包含有用的链接信息，这里把链接和链接的文字描述提取出来（md文本格式的，所以链接形式是 ![链接描述](链接)）

	#!/bin/bash
	#author:
    #	Qiyuyue
    #program:
    #   This script is used to get links
	
	#创建临时文件，仅做缓冲
	if [ -f tmp ];then
	        rm -f tmp &> /dev/null
	fi
	#将含有链接的行输出到文件
	for line in `cat $1`;do
	        echo "$line" | grep "https://" | sed -n '/[.+]/,$p' >> tmp
	done
	#有的一行里含有多个链接，所以先按逗号把每个包含链接的语句单独成行，然后只显示我们想要的[]()部分，再进行操作
	cat tmp | sed 's/,/\n/g' |grep -o "\[.*\](.*)" |sed -r 's/\[(.*)\]\((https:\/\/.*)\)/\1 \2/g' | grep -v "jpg" | grep -v "png"

