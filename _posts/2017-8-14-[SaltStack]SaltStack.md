---
layout:     post
title:      "【SaltStack】SaltStack基本操作"
date:       2017-08-14 18:40:00
author:     "Yuki"
---

# 基础介绍
==========================================================================================
### 简介
SaltStack是一个服务器基础架构集中化管理平台，具备配置管理、远程执行、监控等功能，一般可以理解为简化版的puppet和加强版的func。SaltStack基于Python语言实现，结合轻量级消息队列（ZeroMQ）与Python第三方模块（Pyzmq、PyCrypto、Pyjinjia2、python-msgpack和PyYAML等）构建。
通过部署SaltStack环境，我们可以在成千上万台服务器上做到批量执行命令，根据不同业务特性进行配置集中化管理、分发文件、采集服务器数据、操作系统基础及软件包管理等，SaltStack是运维人员提高工作效率、规范业务配置与操作的利器。
 
2、特性
(1)、部署简单、方便；
(2)、支持大部分UNIX/Linux及Windows环境；
(3)、主从集中化管理；
(4)、配置简单、功能强大、扩展性强；
(5)、主控端（master）和被控端（minion）基于证书认证，安全可靠；
(6)、支持API及自定义模块，可通过Python轻松扩展。

# Salt安装

	 yum install python-crypto ##安装
	 yum install https://repo.saltstack.com/yum/redhat/salt-repo-latest-2.el7.noarch.rpm  ##安装一个salt的源
	 yum clean all
	 yum install salt-master  ##安装master（仅安装在master端）
	 yum install salt-minion  ##安装minion（仅安装在minion端）

# salt配置

[ 详细配置参考 ](https://wenku.baidu.com/view/9be3fe195022aaea988f0f1d.html)

#salt-ssh

Salt 在版本 0.17.0 当中，引入了新的传输系统，它支持通过 SSH 通道来实现 Salt 的通信。通过这种方式，我们可以直接通过 SSH 通道在远程主机上执行使用 SaltStack，而不需要在远程主机上运行 Salt Minion ，同时又能支持 SaltStack 的大部分功能，而且 Salt Master 也不需要运行了。这样，也就实现了免客户端方式的部署和实施。

但是由于无客户端本身的局限性 Salt SSH 并不能完全取代标准的 Salt 通信方式，只是简单地提供了一个基于 SSH 通道的可选方式，这种方式不需要 ZeroMQ 和远程 Agent 的支持；整体的工作流程和基于客户端架构大致相同。但必须意识到，通过 Salt SSH 的执行速度会远远低于 ZeroMQ 支持的标准的 Salt 通信方式。

### 安装

在master端执行 `yum -y install salt-ssh`  安装salt-ssh

### salt rosters

Roster 系统为可插拔设计，可以非常方便地加入到已有的系统中，用于 Salt SSH 获取需要连接的服务器信息。默认情况下 Roster 文件本地路径为：/etc/salt/roster。

Roster 系统编译了一个内部数据结构，称为 Targets。Targets 是一个目标系统和关于如何连接到系统属性的列表。对于一个在 Salt 中的 Roster 模块来说，唯一要求是返回 Targets 数据结构：

	<SaltID>： 	# 目标 ID
	  host： 	# 远程主机的 IP 地址或者主机名
	  user： 	# 可以登录的用户
	  passwd： 	# 可以登录用户的密码
	# 可选参数
	  port： 	# SSH 端口
	  sudo： 	# 是否运行 sudo，设置 True 或者 False
	  priv： 	# SSH 私钥的路径，默认是 salt-ssh.rsa
	  timeout： # 连接 SSH 时的超时时间
	  thin_dir：# 目标系统 Salt 的存储路径，默认是 /tmp/salt-<hash>

注意：这里定义的user一定要是远程主机上已有的user

### 初始化

第一次运行 Salt SSH 会提示进行 salt-ssh key 的部署，需要在 Rosters 中配置用户的密码，即可进行 Key 的部署，初始化代码如下：

` salt-ssh '*' test.ping -i` 一定要加上 -i 参数禁用StrictHostKeyCheck来放松接受新的和未知的主机密钥，然后根据提示操作就行了，当部署过 Key 之后，后期再执行 Salt SSH 就会通过密钥认证，不需要用户再输入密码了。

### 基本使用

salt-ssh 命令的语法和 Salt 命令类似，下面看几个典型的使用场景。

* Salt SSH Target

相对于 C/S 模式，salt-ssh 在 Target 中还有一定的局限性，目前支持如下匹配模式：

-E pcre 正则匹配
-L list 列表匹配
-G grain grains 匹配
-grain-pcre grains 加正则匹配
-N nodegroup 组匹配
-R range 范围匹配

* 命令执行

默认情况下 salt-ssh 可以直接在远程系统上运行 Salt 执行模块。通过 salt-ssh 也可以直接执行原始 shell 命令：

[root@linux-node1 ~]# salt-ssh ‘*’ -r ‘w’

该命令基本等同于 salt ‘’ cmd.run ‘w’ 
执行其他模块时也类似，如 salt-ssh ‘’ disk.usage

* 状态管理

salt-ssh 命令可以直接使用 Salt 状态管理系统。使用时将 salt 命令直接替换为 salt-ssh 即可：

salt-ssh ‘*’ state.sls init.dns

上面的命令基本等同于 salt ‘*’ state.sls init.dns