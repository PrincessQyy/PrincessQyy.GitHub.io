---
layout:     post
title:      "【Linux】pidstat使用总结"
date:       2018-03-23 12:28:00
author:     "Yuki"
---

很多时候我们需要查看各进行对于硬件资源的占用情况，譬如说CPU、Memory，通过pidstat命令可以查看进行对各资源的占用情况.pidstat命令用来监控被Linux内核管理的独立任务(进程)。它输出每个受内核管理的任务的相关信息。pidstat命令也可以用来监控特定进程的子进程。间隔参数用于指定每次报告间的时间间隔。它的值为0(或者没有参数)说明进程的统计数据的时间是从系统启动开始计算的。

## 基本使用

#### pidstat

`pidstat` 直接使用该命令只有正在活动的任务会被显示出来。命令执行结果如下：

<img src="../../../../../img/blogs/pidstat/01.png">

* PID - 被监控的任务的进程号
* %usr - 当在用户层执行(应用程序)时这个任务的cpu使用率，和 nice 优先级无关。注意这个字段计算的cpu时间不包括在虚拟处理器中花去的时间。
* %system - 这个任务在系统层使用时的cpu使用率。
* %guest - 任务花费在虚拟机上的cpu使用率（运行在虚拟处理器）。
* %CPU - 任务总的cpu使用率。在SMP环境(多处理器)中，如果在命令行中输入-I参数的话，cpu使用率会除以你的cpu数量。
* CPU - 正在运行这个任务的处理器编号。
* Command - 这个任务的命令名称。

#### 指定采样周期和采样次数

pidstat命令指定采样周期和采样次数，命令形式为”pidstat [option] interval [count]”，以下pidstat输出以2秒为采样周期，输出10次cpu使用统计信息：

`pidstat 2 10`

#### cpu使用情况统计

`pidstat -u` ,pidstat将显示各活动进程的cpu使用统计，命令执行结果和单独使用pid差不多，但是多了一列 uid .

#### 内存使用情况统计

`pidstat -r`

使用-r选项，pidstat将显示各活动进程的内存使用统计,结果如下：

<img src="../../../../../img/blogs/pidstat/02.png">

* minflt/s: 每秒次缺页错误次数(minor page faults)，次缺页错误次数意即虚拟内存地址映射成物理内存地址产生的page fault次数
* majflt/s: 每秒主缺页错误次数(major page faults)，当虚拟内存地址映射成物理内存地址时，相应的page在swap中，这样的page fault为major page fault，一般在内存使用紧张时产生
* VSZ:      该进程使用的虚拟内存(以kB为单位)
* RSS:      该进程使用的物理内存(以kB为单位)
* %MEM:     该进程使用内存的百分比
* Command:  拉起进程对应的命令

#### IO使用情况统计

`pidstat -d`，使用-d选项，我们可以查看进程IO的统计信息，结果如下：

<img src="../../../../../img/blogs/pidstat/03.png">

* kB_rd/s - 任务从硬盘上的读取速度（kb）
* kB_wr/s - 任务向硬盘中的写入速度（kb）
* kB_ccwr/s - 任务写入磁盘被取消的速率（kb）

#### 针对特定进程统计

使用 -p <pid> 参数可以对特定进程对资源的使用情况进行统计。

#### 排错常用

使用pidstat进行问题定位时，以下命令常被用到：

`pidstat -u 1`

`pidstat -r 1`

`pidstat -d 1`

以上命令以1秒为信息采集周期，分别获取cpu、内存和磁盘IO的统计信息。

还可以结合 sort 命令，使用 -k 参数指定按某一列（例如我们所关心的内存百分比、CPU使用率等）进行排序，加-r参数可以直接看到占用最大的那个进程。
