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
	for i in `cat ip_list`;do
	        ping  $i -c2 -w2  > result;
	        cat result|grep "64 bytes">&/dev/null  && echo "True" || echo "False"
	
	done