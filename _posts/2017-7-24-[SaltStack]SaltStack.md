---
layout:     post
title:      "【SaltStack】SaltStack基本配置和操作"
date:       2017-07-24 17:33:00
author:     "Yuki"
---


# SaltStack架构

在SaltsStack架构中服务端叫作Master，客户端叫作Minion，都是以守护进程的模式运行，一直监听配置文件中定义的ret_port（saltstack客户端与服务端通信的端口，负责接收客户端发送过来的结果，默认4506端口）和publish_port（saltstack的消息发布系统，默认4505端口）的端口。当Minion运行时会自动连接到配置文件中定义的Master地址ret_port端口进行连接认证。