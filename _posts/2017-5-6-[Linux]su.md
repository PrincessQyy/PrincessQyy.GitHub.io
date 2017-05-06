---
layout:     post
title:      "【Linux】su和sudo"
date:       2017-05-06 14:43:00
author:     "Yuki"
---

之前一直以为，这两个命令没什么特别的地方，直到最近重新看了鸟哥的Linux，才发现里面也是有不少学问的啊，设置好了确实能省不少事儿！那下面就来说说这两个命令的用法吧~

#### su

su是最简单的用户切换命令了，可以进行任意身份的切换。

基本语法;

`su [-lm] [-c 命令] username`

选项与参数：

\-  单纯使用-表示用login-shell 的变量文件读取方式来登录shell，若后面不加用户名，则表示切换为root用户

-l 与-类似，但后面必须加上用户名

-m -m与-p是一样的，表示使用目前的环境设置，而不读取新用户的配置文件

-c 后面接命令，表示仅进行一次命令

tips：

* 若只想执行一次root命令，用 `su - -c command`的方式来处理
* 使用root切换为任何用户时，都不需要输入密码

#### sudo

虽然使用su很方便，但是当多人管理主机时，如果大家都要使用su来切换为root身份，那么每个人都得知道root密码，这样相当不妥，此时就可以使用sudo，sudo的执行仅需要自己的密码即可，但是并非所有人都能执行sudo命令，而是仅有/etc/sudoers内的用户才能执行sudo命令。

基本语法：

`sudo [-b] [-u username] command`

选项与参数：

-b 将后序命令让系统自动执行，而不与当前shell产生影响

-u 后面可以接预切换的用户，若无此项则代表切换为root用户

sh -c "commands" 要执行一串命令可以使用此项，命令之间用;分隔

举例：

`sudo -u sshd touch filename` 以sshd的身份新建一个文件，该例子中我们无法使用`su -l sshd`去切换为sshd用户，因为sshd的系统账号的shell是/sbin/nologin，所以此时sudo命令就很方便！

**visudo与/etc/sudoers**

之前有提到，若是想要使用sudo命令，该用户必须存在于/etc/sudoers内。所以若是希望某用户能够使用sudo命令，我们需要去修改该文件。那为什么不用vim而用visudo呢？这是因为/etc/sudoers是有语法的，如果设置错误可能会造成无法使用sudo的不良后果，而使用visudo去修改时，在结束离开界面时，系统会去检验/etc/sudoers的语法，实在是很人性化呀~

一般来说，/etc/sudoers的设置有几种简单的方法，下面就直接举几个例子来说明吧~

* 让princess这个账号可以使用root的任何命令

先用命令 visudo打开设置文件，跳转到98行，我的文件是在98行，如下：

<img src="../../../../../img/blogs/su/01.png">

上面这一行的四个参数的含义是：
1. 用户账号：系统的哪个账号可以使用sudo命令
2. 登录者的来源主机名：这个账号由哪台主机连接到本Linux主机，这个设置值可以指定信任用户，默认值ALL代表可以来自任何一台网络主机
3. 可切换的身份：这个账号可以切换成什么身份来执行后续命令，默认root可以切换为任何人
4. 可执行的命令：这个命令请务必使用绝对路径编写，默认root可以执行任何命令

在该行下面添加：

`princess	 ALL=(ALL)		ALL`

* 让用户组可以使用免密码的方式执行sudo

上面的方法，一次只能设置一个用户，实在是有点麻烦，所以接下来的方法就很重要啦~

先用visudo打开配置文件，我的文件是在105行

<img src="../../../../../img/blogs/su/01.png">

改行最左边有一个%，这个符号代表后面接的是一个用户组。所以只要使用命令 `usermod -G wheel username`把用户加入wheel组,就可以使用sudo来执行命令了，当然，wheel可以替换为任何你想要的组名。那么如何提供不带密码就能使用sudo的方法呢？就通过如下的方式：

<img src="../../../../../img/blogs/su/01.png">

注意到，这一行中出现了NOPASSWD的关键字，这个关键字就是免除密码的意思啦~是不是很方便~这样sudoers使用sudo时，就可以不输入密码直接操作啦~

* 有限的命令操作

上面两个例子都能让用户以root的身份执行任何命令，这样总是不太好的，若想要用户仅能执行某些命令，要如何编写呢？

用命令 visudo打开设置文件，跳转到98行，如下：

<img src="../../../../../img/blogs/su/01.png">

在改行下面添加：

`user1 ALL=(root) /usr/bin/passwd`

注意，命令必须使用绝对路径哦，这样user1就能使用passwd命令来修改密码啦，但是，发现什么问题没有？user1可以用该命令修改任何人的密码啊，甚至包括root的密码，这实在是太危险了，所以我们必须要现在用户命令的参数。将上面那行修改为：

`user1 ALL=(root) ！/usr/bin/passwd,/user/bin/passwd [a-zA-Z],!/usr/bin/passwd root`

和大多数编程语言一样，!的意思是非，也就是说，user1仅可以使用passwd来修改他用户的密码，但是不能修改root的密码。

* 通过别名设置visudo

如上述案例，若是我有很多个用户想要同时加入管理员行列，一行一行设置岂不是很麻烦，这时就可以通过别名来简化设置啦~

假设user1、user2、user3、user4、user5要加入上述管理员行列中，那可以新建一个账户别名为USERS,命令别名为USERSCNMD然后处理如下：

用命令 visudo打开设置文件。

`User_Alias USERS = user1,user2,user3,user4,user5`
`Cnmd_Alias USERSCNMD = ！/usr/bin/passwd,/user/bin/passwd [a-zA-Z],!/usr/bin/passwd root`
`USERS ALL=(root) USERSCNMD`

这样的设置就比较有弹性了，将来若是我要修改，就只需要修改前两行啦~

* sudo搭配su的使用方式

很多时候我们需要大量的执行root的工作，所以一直使用sudo会觉得很烦，那有办法解决这个难题吗？有的！

用命令 visudo打开设置文件。

我们新建一个ADMINS账户别名，然后更改设置如下：

`User_Alias ADMINS = user1,user2,user3,user4,user5`
`ADMINS ALL=(root) /bin/su -`

接下来，user1,user2,user3,user4,user5只要输入命令`sudo su -`，然后输入自己的密码，就会立即变成root身份，不但root的密码不外泄，用户的管理也变得很方便了~

**sudo命令的时效性**

有的小伙伴可能会发现，如果使用同一账号在短时间重复操作sudo来运行命令时，只有第一次会需要输入密码，这是因为sudo命令是有时效性的，若两次执行sudo的间隔时间在5分钟内，再次执行sudo时就不需要重复输入密码啦~


 





