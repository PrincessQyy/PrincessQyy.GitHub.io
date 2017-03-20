---
layout:     post
title:      "【Linux】管道和重定向"
date:       2017-03-14 16:28:00
author:     "Yuki"
---

在Linux中，绝大多数的命令都是以纯文本格式输入的，而且大多数命令的输出也是纯文本格式，这就为命令的组合提供了条件，让多命令的协作成为可能。我们也知道，大多数的命令都很简单，极少出现复杂的功能，所以Linux为我们提供了管道和重定向机制支持多命令协作。

首先，我们来看看shell的数据流的定义：

<img src="../../../../../img/blogs/pipes and redirection/flow.png">

命令通过 STDIN 接收参数或数据，通过 STDOUT输出结果或通过STDERR输出错误。通过管道和重定向我们可以控制CLI的数据流。

<img src="../../../../../img/blogs/pipes and redirection/pipe.png">

* 管道通常用来组合不同的命令，以实现一个复杂的功能。
* 重定向通常用来保存某命令的输出信息或错误信息，可以用来记录执行结果或保存错误信息到一个指定的文件。



