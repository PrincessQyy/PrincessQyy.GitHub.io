---
layout:     post
title:      "【Linux】网卡收包流程"
date:       2018-04-13 15:19:00
author:     "Yuki"
---
平时对于系统内核和网卡驱动接触得比较少，一方面可能我们的系统没有需要深度调优的需求，另一方面网络编程涉及到硬件，驱动，内核，虚拟化等复杂的知识，使人望而却步。网络上网卡收包相关的资料也比较多，但是比较分散，在此梳理了网卡收包的流程。
## 整体流程
网卡收包从整体上是网线中的高低电平转换到网卡FIFO存储再拷贝到系统主内存（DDR3）的过程，其中涉及到网卡控制器，CPU，DMA，驱动程序，在OSI模型中属于物理层和链路层，如下图所示:
<img src="../../../../../img/blogs/net_packets/01.png">

## 网络收包原理
网络驱动收包大致有3种情况：

* no NAPI：mac每收到一个以太网包，都会产生一个接收中断给cpu，即完全靠中断方式来收包

缺点是当网络流量很大时，cpu大部分时间都耗在了处理mac的中断。

* netpoll：在网络和I/O子系统尚不能完整可用时，模拟了来自指定设备的中断，即轮询收包。

缺点是实时性差

* NAPI：采用中断 + 轮询的方式：mac收到一个包来后会产生接收中断，但是马上关闭。直到收够了netdev_max_backlog个包（默认300），或者收完mac上所有包后，才再打开接收中断

通过sysctl来修改 net.core.netdev_max_backlog

或者通过proc修改 /proc/sys/net/core/netdev_max_backlog

以下是非NAPI和NAPI处理流程对比：、
<img src="../../../../../img/blogs/net_packets/02.jpg">

8139CP中断处理如下：
<img src="../../../../../img/blogs/net_packets/03.jpg">

 POLL 方法的本质意义上来说就在于尽量减少中断的数目，特别在于大量的小长度的数据包的时候，减少中断，以达到不要让整个操作系统花费太多的时间在中断现场的保护和恢复上，以便把赢得的时间用来在我网络层上的处理数据的传输，例如在下面介绍的 8139CP 中断的处理过程中，目的就在于尽快把产生中断的设备挂在 poll_list，并且关闭接收中断，最后直接调用设备的POLL方法来处理数据包的接收,直到收到数据包收无可收，或者是达到一个时间片内的调度完成。


##网络收包流程

* 内核启动时的准备工作

 初始化网络相关的全局数据结构，并挂载处理网络相关软中断的钩子函数。

* 加载网络设备的驱动

这里的网络设备是指MAC层的网络设备，即TSEC和PCI网卡（bcm5461是phy）在网络设备驱动中创建net_device数据结构，并初始化其钩子函数 open(),close() 等。

* 启用网络设备

 用户调用ifconfig等程序，然后通过ioctl系统调用进入内核，在网络设备的open钩子函数里，分配接收bd，挂中断ISR(包括rx、tx、err)，对于TSEC来说gfar_enet_open

* 中断里接收以太网包 

TSEC的RX已经使能了，网络数据包进入内存的流程为：

			网线 --> Rj45网口 --> MDI 差分线

         --> bcm5461(PHY芯片进行数模转换) --> MII总线 

         --> TSEC的DMA Engine 会自动检查下一个可用的Rx bd 

         -->把网络数据包 DMA 到 Rx bd 所指向的内存，即skb->data

 

接收到一个完整的以太网数据包后，TSEC会根据event mask触发一个 Rx 外部中断。

cpu保存现场，根据中断向量，开始执行外部中断处理函数do_IRQ()


上半部处理硬中断

       查看中断源寄存器，得知是网络外设产生了外部中断

       执行网络设备的rx中断handler（设备不同，函数不同，但流程类似，TSEC是gfar_receive）

1.    mask 掉 rx event，再来数据包就不会产生rx中断

2.    给napi_struct.state加上 NAPI_STATE_SCHED 状态

3.    挂网络设备自己的napi_struct结构到cpu私有变量_get_cpu_var(softnet_data).poll_list

4.    触发网络接收软中断( __raise_softirq_irqoff(NET_RX_SOFTIRQ); ——> wakeup_softirqd() )

下半部处理软中断

		依次执行所有软中断handler，包括timer，tasklet等等

		执行网络接收的软中断handler net_rx_action

1.    遍历cpu私有变量_get_cpu_var(softnet_data).poll_list 

2.     取出poll_list上面挂的napi_struct 结构,执行钩子函数napi_struct.poll() (设备不同，钩子函数不同,流程类似，TSEC是gfar_poll）

3.    若poll钩子函数处理完所有包，则打开rx event mask，再来数据包的话会产生rx中断

4.     调用napi_complete(napi_struct *n)

5.    把napi_struct 结构从_get_cpu_var(softnet_data).poll_list 上移走，同时去掉 napi_struct.state 的 NAPI_STATE_SCHED 状态

* 在软中断中使用NAPI

执行一次网络软中断过程中，网卡本身的Rx中断已经关闭了，即不会产生新的接收中断了。local_irq_enable和local_irq_disable设置的是cpu是否接收中断。进入网络软中断net_rx_action的时候，会初始一个budget（预算），即最多处理的网络包个数，如果有多个网卡（放在poll_list里），是共享该budget，同时每个网卡也一个权重weight或者说是配额quota，一个网卡处理完输入队列里包后有两种情况，一次收到的包很多，quota用完了，则把收包的poll虚函数又挂到poll_list队尾，重新设置一下quota值，等待while轮询；另外一种情况是，收到的包不多，quota没有用完，表示网卡比较空闲，则把自己从poll_list摘除，退出轮询。整个net_rx_action退出的情况有两种：budget全部用完了或者是时间超时了。

## 总结

在NAPI机制中，对于大数据包低速率的情况，接收中断就会急剧增加，直到最后每个数据包都需要一次 POLL 的方法来进行处理，最后的结果就是每个中断都需要一次 POLL 的方法，最后造成效率的急剧下降，以至于系统的效率就会大大降低，所以 NAPI 适用于大量的数据包而且尽可能是小的数据包，但是对于大的数据包，而且低速率的，反而会造成系统速度的下降。

使用 NAPI 并不是改善网络效率的唯一途径，只能算是权益之策，根本的解决途径还是在于上层应用程序能够独占网络设备，或者是提供大量的缓冲资源。