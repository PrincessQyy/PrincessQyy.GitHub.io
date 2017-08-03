---
layout:     post
title:      "【Linux】Linux软件管理-RPM、YUM"
date:       2017-03-15 10:33:00
author:     "Yuki"
---

## RPM

绝大多数的开源软件都是直接以源代码的形式发布的，这些源代码通常会被打包成 tar.gz 的格式，程序的源代码需要编译成二进制形式之后才能够运行使用，源代码的基本编译流程：

* ./configure 检查编译环境、相关库文件以及配置参数并生成Makefile
* make 对源代码进行编译，生成可执行文件
* make install 将生成的可执行文件安装到当前计算机中

源代码形式的软件操作复杂、编译时间较长，极容易出现错误，而且由于开源软件基本上会大量使用其他开源软件的功能，所以开源软件会有大量的依赖关系。但是其兼容性和可控性较好。为了方便使用，开发了RPM（Redhat Package Management）。RPM设计目标如下：

* 使用简单
* 使用单一软件包格式文件发布（.rpm）
* 可升级
* 追踪软件依赖关系
* 基本信息查询
* 软件验证功能
* 支持多平台
    
**RPM软件包常用命名规范：**

* 软件名-版本号.对应的系统和平台.操作系统位数.rpm

**RPM基础命令**

* 安装软件：rpm -i software.rpm
* 卸载软件：rpm -e software
* 升级形式安装：rpm -U software_new.rpm
* 通过http、ftp协议安装软件：rpm -ivh 链接

常用参数：

-v 显示详细信息

-h 显示进度条

**RPM查询**

* rpm -qa 列出所有安装的rpm软件
* rpm -qf file_name 查询目标文件属于哪个rpm包
* rpm -qi package_name 查询指定已安装rpm软件的信息
* rpm -ql package_name 查询指定已安装rpm软件包含的文件
* rpm -qip software.rpm 查询rpm文件信息
* rpm -qlp software.rpm 查询rpm文件所包含的文件

**RPM验证**

软件在传播过程中可能会被恶意修改，所以为了安全起见加入了对软件的验证功           能。验证一般使用非对称加密算法，所以需要一个秘钥。

* 导入秘钥：rpm --import 
* 验证rpm文件：rpm -K software.rpm
* 验证已安装软件：rpm -V software

## YUM

RPM软件包形式管理软件虽然方便，但是需要手工解决软件包依赖关系。使用YUM可以解决这个问题。YUM（YellowDog Updater,Modified）是一个RPM的前端程序，它引入了仓库概念，仓库用来存放所有现有的RPM软件包，仓库可以是本地的，也可以通过HTTP、FTP或NFS使用集中的、统一的网络仓库。YUM支持多个仓库，若存在依赖关系，会在仓库中查找依赖软件并进行安装，YUM还可以对RPM进行分组管理，并基于分组进行安装操作，其配置也很简单。

**YUM仓库**

YUM使用仓库保存管理rpm软件包，仓库的配置文件保存在 /etc/yum.respos.d/ 目录下，该目录下可以存在多个配置文件，一个配置文件可以保存多个仓库的配置信息，配置文件必须以 .repo 结尾，其格式如下：

[repo_name]

name=this is your description

baseurl=address of your repo

enabled = 1

gpgcheck=1 


ennabled 默认值是1，当某个软件仓库被配置成 enabled=0 时，yum 在安装或升级软件包时不会将该仓库做为软件包提供源。使用这个选项，可以启用或禁用软件仓库。
 
gpgcheck默认值是1，这个选项表示这个repo中下载的rpm将进行gpg的校验，已确定rpm包的来源是有效和安全的。

**YUM基本命令**

* 安装指定软件：yum install software_name
* 卸载指定软件：yum remove software_name
* 升级指定软件：yum update software_name

**YUM查询**

* yum search keyword 搜索包含关键字的软件
* yum list (all | installed | recent |updates) 列出全部、已安装的、最近的、更新的软件
* yum info packagename 显示指定软件的信息
* yum whatprovides filename 查询哪个rpm软件包含目标文件

 **创建YUM仓库**

1. 将所有rpm文件拷贝到一个文件夹中
2. 通过rpm命令手工安装createrepo软件
3. 运行命令 `createrepo -v /rpm-directory `建立索引文件repodata（-v 显示详细信息）(若有分组信息，则在运行命令时使用 -g 参数指定分组文件` createrepo -g /tmp/*comps.xml/rpm-directory`)
4. 添加配置文件指向仓库（本地文件协议为file://）
5. 用 yum clean all 清除缓存信息（每次运行安装或查询类的命令都会重建缓存）


 
    


