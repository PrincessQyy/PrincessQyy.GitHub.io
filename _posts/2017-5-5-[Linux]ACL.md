---
layout:     post
title:      "【Linux】ACL权限设置"
date:       2017-05-05 19:06:00
author:     "Yuki"
---

今早刚看ACL，趁着劲头儿总结一下。ACL是Access Control List的缩写，它可以针对单一用户、单一文件或目录来进行rwx的权限设置。对于需要特殊权限的状况非常有帮助。ACL可以针对以下三个方面进行权限控制：

* 用户
* 用户组
* 默认属性：默认属性其实就相当于有效权限，用户或组所设置的权限必须要存在于mask的权限设置范围内才会有效。

#### ACL启动

可以使用命令 `mount -o remount,acl /` 即使启动ACL。但如果想要每次开机都生效，就需要修改配置文件，`vim /etc/fstab `，修改如下：

<img src="../../../../../img/blogs/ACL/01.png">

#### ACL的设置技巧

文件系统启动后，用`getfacl`和`setfacl`来查看和设置ACL。下面就这两个命令进行详细说明吧~

**getfacl**

取得某个文件/目录的ACL设置项目。

基本语法：

`getfacl filename`
因为这个命令的参数不常用，我就不列举了，想了解的话可以通过`getfacl --help`查看，没有什么生僻单词，一般都是能看懂的。

**setfacl**

设置某个文件/目录的ACL规定。

基本语法：

`setfacl [选项和参数] [{-m|-x} acl参数] 目标文件名`

选项和参数：

-b 删除所有的ACL设置参数

-k 删除默认的ACL参数

-R 递归设置ACL参数，即该目录下的子目录都会被设置

-d 设置默认的ACL参数，只对目录有效，在该目录中新建数据时会引用此值

-m 设置后续的ACL参数

-x 删除后续的ACL参数

注：参数-m和-x只能存在一个！

下面来看看几类权限的设置方式

* 针对单一用户的设置

设置规定：u:[用户账号列表]:[rwx]

例如：`setfacl -m u:princess:rw filename` 针对princess用户设置权限为rw

`setfacl -m u::rw filename` 设置该文件的所有者权限为rw

* 针对用户组的设置

设置规定：g:[用户组列表]:[rwx]

例如：`setfacl -m g:mygroup:rx filename` 针对mygroup设置权限为rx

* 针对默认权限的设置

之前有说过，默认权限其实即是有效权限，也就是说用户或组设置的权限必须要在mask范围内才会生效。我们可以通过mask来设置最大允许的权限，这样就能够避免不小心开放某些权限给其他用户或用户组了。

设置规定：m:[rwx]

例如：`setfacl -m m:r filename ` 针对file设置默认权限为r，即使其他用户或用户组设置的权限多于r，也仅有r的权限

* 让用户或组在目录下具有某权限

设置规定：d:[u|g]:用户列表:[rwx]

例如：`setfacl -m d:u:princess /etc` 让用户princess在目录/etc下具有rw的权限










