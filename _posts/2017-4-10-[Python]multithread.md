---
layout:     post
title:      "【Python】多线程"
date:       2017-04-10 10:50:00
author:     "Yuki"
---

#### 进程和线程

每个正在系统上运行的程序都是一个进程。进程也可能是整个程序或者是部分程序的动态执行。每个进程包含一到多个线程。线程是一组指令的集合，或者是程序的特殊段，它可以在程序里独立执行。也可以把它理解为代码运行的上下文。所以线程基本上是轻量级的进程，它负责在单个程序里执行多任务。通常由操作系统负责多个线程的调度和执行。

线程和进程的区别在于,子进程和父进程有不同的代码和数据空间,而多个线程则共享数据空间,每个线程有自己的执行堆栈和程序计数器为其执行上下文。多线程主要是为了节约CPU时间,发挥利用。多线程的优点有：资源利用率更好，程序设计在某些情况下更简单，程序响应更快。

#### python 多线程

尽管python支持多线程，但在某一时刻，python解释器只会执行一个线程。对python虚拟机的访问是通过全局解释器锁（GIL）完成的。这个锁就是用来保证同时只能有一个线程运行的。在多线程环境中，python虚拟机将按如下方式执行：

1. 设置GIL
2. 切换一个进程去运行
3. 执行下面操作之一：
a. 指定数量的字节指令 b.线程主动让出控制权（可使用time.sleep()来完成）
4. 将线程设置回睡眠状态（切换出线程）
5. 解锁GIL
6. 重复以上步骤

python提供了多个模块来支持多线程编程，这里主要总结一下thread和threading模块。

**thread**

除了派生线程外，thread模块还提供了基本的同步数据结构，称为锁模块。

下图是一些常用的线程函数以及LockType锁对象的方法。

<img src="../../../../../img/blogs/multithread/01.png">

我们尽量会避免使用thread模块，因为它不支持守护线程，当主线程退出时，所有的子线程都会终止。

下面给一个使用thread的实例：
    import thread
    from time import sleep

    #define sleep time
    loops=[4,2]

    def loop(nloop,loop_time,lock):
        print 'loop %d is starting sleep' % nloop
    	sleep(loop_time)
    	print 'loop %d done' % nloop
    	lock.release()
    

    def main():
	    locks=[]
	    nloops=int(len(loops))
	    
	    #create locks
	    for i in range(nloops):
	        lock=thread.allocate_lock()
	        lock.acquire()
	        locks.append(lock)
    
	    #create threads
	    for i in range(nloops):
	        thread.start_new_thread(loop,(i,loops[i],locks[i]))
	
	    #wait a lock
	    for i in range(nloops):
	        while locks[i].locked():
	            pass

    if __name__ == '__main__':
    	main()

**threading**

现在介绍更高级的threading模块。除了Thread类以外，该模块还包含了许多非常好的同步机制，下图给出了threading模块中所有可用对象的列表：

| 对象 | 描述 |
| ---------------- |:-----------------------------------:|
| Thread		   | 表示一个执行线程的对象 |
| Lock		       | 锁源语对象 |
| RLock            | 可重入锁对象（递归锁）|
| Condition        | 条件变量对象，使得一个线程可以等待另一线程满足特定条件，如改变状态 |
| Event            | 条件变量的通用版本，任意数量的线程等待某事件	发生，该事件发生后线程被激活 |
| Semaphore        |为线程间共享的有限资源提供了一个‘计数器’，如果没有可用资源时会阻塞 |
| BoundedSemaphore | 与Semaphore相似，但是不允许超过初始值 |
| Timer            | 与Thread相似，但在运行前要等待一段时间 |
| Barrier          | 创建一个‘障碍’，必须达到指定数量的线程后才可以继续 |


在threading模块中Thread类是主要的执行对象，它的属性和方法列表如下：

<img src="../../../../../img/blogs/multithread/03.png">

threading模块支持守护进程，它的工作方式是：守护线程一般是一个等待客户端请求的服务的服务器，如果没有客户端请求，该守护进程就是空闲的。如果把一个线程设置为守护线程，就表示这个线程是不重要的，进程退出时不需要等到这个线程完成。要设置一个线程为守护线程，只需要在其启动前执行如下赋值语句：`thread.daemon=True`

使用Thread类，创建线程的方式有以下三种：
1. 创建Thread的实例，传给他一个函数。
2. 创建Thread的实例，传给他一个可调用的类实例。
3. 派生Thread的子类，并创建子类的实例。

通常我们都选择第一种或第三种，第二种方案，着实有点尴尬而且代码难以阅读。

下面给出三种方案的代码实现：

**创建Thread的实例，传给他一个函数：**

    import threading
    from time import sleep

    loops=[4,2]
    def loop(nloop,loop_time):
	    print 'loop %d is sleeping' % nloop
	    sleep(loop_time)
	    print 'loop %d done' % nloop
    

    def main():
	    threads=[]
	    nloop=int(len(loops))
	    
	    #create threads
	    for i in range(nloop):
	        thread=threading.Thread(target=loop,args=(i,loops[i]))
	        threads.append(thread)
	        
	    #start 
	    for i in range(nloop):
	        threads[i].start()
	        
	    #stop
	    for i in range(nloop):
	        threads[i].join() 
        
    if __name__ == '__main__':
    	main()

**创建Thread的实例，传给他一个可调用的类实例:**

    import threading
    from time import sleep
    
    loops=[4,2]
    
    class ThreadFunc(object):
	    def __init__(self,func,args,name=''):
	        self.func=func
	        self.name=name
	        self.args=args
	        
	    def __call__(self):
	        self.func(*self.args)
	        

    def loop(nloop,loop_time):
	    print 'loop %d is sleeping' % nloop
	    sleep(loop_time)
	    print 'loop %d done' % nloop
    
    def main():
	    threads=[]
	    nloop=int(len(loops))
	
	    #create threads
	    for i in range(nloop):
	        thread=threading.Thread(target=ThreadFunc(loop,(i,loops[i]),loop.__name__))
	        threads.append(thread)
        
	    #start 
	    for i in range(nloop):
	        threads[i].start()
	           
	    #stop
	    for i in range(nloop):
	        threads[i].join() 

    if __name__ == '__main__':
    	main()


**派生Thread的子类，并创建子类的实例:**
    
    import threading
    from time import sleep

    loops=[4,2]
    
    class MyThread(threading.Thread):
	    def __init__(self,func,args,name=''):
	        threading.Thread.__init__(self)
	        self.func=func
	        self.name=name
	        self.args=args
	        
	    def run(self):
	        self.func(*self.args)
        

    def loop(nloop,loop_time):
	    print 'loop %d is sleeping' % nloop
	    sleep(loop_time)
	    print 'loop %d done' % nloop
    

    def main():
	    threads=[]
	    nloop=int(len(loops))
	    
	    #create threads
	    for i in range(nloop):
	        thread=MyThread(loop,(i,loops[i]),loop.__name__)
	        threads.append(thread)
	        
	    #start 
	    for i in range(nloop):
	        threads[i].start()
	           
	    #stop
	    for i in range(nloop):
	        threads[i].join()    
        

    if __name__ == '__main__':
    	main()










