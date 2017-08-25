---
layout:     post
title:      "【lvs+keepalived】lvs+keepalived配置"
date:       2017-08-23 15:34:00
author:     "Yuki"
---


# LVS + keepalived的实现：

	! Configuration File for keepalived  
	  
	global_defs {  
	   notification_email {  
	         linuxedu@foxmail.com
	         mageedu@126.com  
	   }  
	   notification_email_from kanotify@magedu.com 
	   smtp_connect_timeout 3  
	   smtp_server 127.0.0.1  
	   router_id LVS_DEVEL  
	}  
	
	vrrp_script chk_schedown {
	   script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0"
	   interval 2
	   weight -2
	}
	
	vrrp_instance VI_1 {  
	    interface eth0  
	    state MASTER  
	    priority 101
	    virtual_router_id 51 
	    garp_master_delay 1 
	 
	    authentication {  
	        auth_type PASS  
	        auth_pass password  
	    }  
	
	    track_interface {  
	       eth0    
	    }  
	
	    virtual_ipaddress {  
	        172.16.100.1/16 dev eth0 label eth0:0
	    }  
	
	    track_script {  
	        chk_schedown
	    }    
	} 
	
	
	virtual_server 172.16.100.1 80 {
	    delay_loop 6
	    lb_algo rr 
	    lb_kind DR
	    persistence_timeout 50
	    protocol TCP
	
	#    sorry_server 192.168.200.200 1358
	
	    real_server 172.16.100.11 80 {
	        weight 1
	        HTTP_GET {
	            url { 
	              path /
	              status_code 200
	            }
	            connect_timeout 3
	            nb_get_retry 3
	            delay_before_retry 3
	        }
	    }
	
	    real_server 172.16.100.12 80 {
	        weight 1
	        HTTP_GET {
	            url { 
	              path /
	              status_code 200
	            }
	            connect_timeout 3
	            nb_get_retry 3
	            delay_before_retry 3
	        }
	    }
	}


### 如果要使用TCP_CHECK检测各realserver的健康状态，那么，上面关于realserver部分的定义也可以替换为如下内容：
	virtual_server 172.16.100.1 80 {
	    delay_loop 6
	    lb_algo rr 
	    lb_kind DR
	    persistence_timeout 300
	    protocol TCP
	
	    sorry_server 127.0.0.1 80
	
	    real_server 172.16.100.11 80 {
	        weight 1
	        TCP_CHECK {
		    	tcp_port 80
	            connect_timeout 3
	        }
	    }
	
	    real_server 172.16.100.12 80 {
	        weight 1
	        TCP_CHECK {
		    	connect_port 80
	            connect_timeout 3
	          }
	    }
	}

说明：其中的sorry_server是用于定义所有realserver均出现故障时所用的服务器。

