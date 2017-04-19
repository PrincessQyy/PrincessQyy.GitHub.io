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

 Python中使用线程有两种方式：函数或者用类来包装线程对象。也就涉及了下面的thread和threading模块。

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

#### 简单的生产者、消费者模型

我发现在爬虫学习的资料里，不管是权威书籍，还是网络上各种视频和博客，都会讲到生产者-消费者模型，那么这个模型是什么意思呢？

在并发编程中使用生产者和消费者模式能够解决绝大多数并发问题。该模式通过平衡生产线程和消费线程的工作能力来提高程序的整体处理数据的速度。在线程世界里，生产者就是生产数据的线程，消费者就是消费数据的线程。在多线程开发当中，如果生产者处理速度很快，而消费者处理速度很慢，那么生产者就必须等待消费者处理完，才能继续生产数据。同样的道理，如果消费者的处理能力大于生产者，那么消费者就必须等待生产者。为了解决这种生产消费能力不均衡的问题，所以便有了生产者和消费者模式。

生产者消费者模式是通过一个容器来解决生产者和消费者的强耦合问题。生产者和消费者彼此之间不直接通讯，而通过阻塞队列来进行通讯，所以生产者生产完数据之后不用等待消费者处理，直接扔给阻塞队列，消费者不找生产者要数据，而是直接从阻塞队列里取，阻塞队列就相当于一个缓冲区，平衡了生产者和消费者的处理能力。

这个阻塞队列就是用来给生产者和消费者解耦的。纵观大多数设计模式，都会找一个第三者出来进行解耦，如工厂模式的第三者是工厂类，模板模式的第三者是模板类。在学习一些设计模式的过程中，如果先找到这个模式的第三者，能帮助我们快速熟悉一个设计模式。

python中是通过Queue这个模块提供线程间通信的机制，线程放入产品，消费者来解析。其实这也相当于一个线程池。那么问题又来了，什么是线程池？

诸如web服务器、数据库服务器、文件服务器和邮件服务器等许多服务器应用都面向处理来自某些远程来源的大量短小的任务。如果每当一个请求到达就创建一个新的服务对象，然后在新的服务对象中为请求服务。但有大量请求并发访问时，服务器不断的创建和销毁对象的开销很大。所以提高服务器效率的一个手段就是尽可能减少创建和销毁对象的次数，特别是一些很耗资源的对象创建和销毁，这样就引入了“池”的概念，“池”的概念使得人们可以定制一定量的资源，然后对这些资源进行复用，而不是频繁的创建和销毁。

概念性的东西说的差不多了，下面来看一个实例吧，这是爬韩寒的博客首页，只是为了后面的性能对比，所以没有输出和保存什么内容：

    import requests
    import time
    from bs4 import BeautifulSoup
    import re
    from Queue import Queue
    import threading
    
    def spider(url):
	    headers={}
	    r=requests.get(url) 
	    #print r.status_code


    class MyThread(threading.Thread):
	    def __init__(self,queue):
	        threading.Thread.__init__(self)
	        self._queue=queue
	    def run(self):
	        while not self._queue.empty():
	            page_url=self._queue.get_nowait()
	            spider(page_url)
          


    def main():
	    start=time.clock()
	    threads=[]
	    threads_count=10
	    
	    
	    queue=Queue()
	    
	    #get url
	    base_url='http://blog.sina.com.cn/s/article_sort_1191258123_10001_'
	    urls=[]
	    for i in range(1,32):
	        url=base_url+str(i)+'.html'
	        queue.put(url)
	        
	    #create threads
	    for i in range(threads_count):
	        threads.append(MyThread(queue))
	        
	    #start
	    for i in range(threads_count):
	        threads[i].start()
	        
	    #join
	    for i in range(threads_count):
	        threads[i].join()    
	        
	        
	    end=time.clock()
	    
	    print 'all done costs %f' % (end-start)
        
    
    
    if __name__ == '__main__':
		main()



#### 真正的并发

之前的多线程是一种伪的机制，当你去爬比较多的数据的时候你会发现性能不是很高，也可能是我电脑本身太次了(◐_◑)其实在python中其实还有一个强大的、几乎无人提及的模块，这个模块甚至在官方文档中也只有短短一句的描述：

<img src="../../../../../img/blogs/multithread/03.png">

简单翻译一下，就是说这个模块是复制了multiprocessing的接口只不过它是多线程模块的封装，而Multiprocessing模块是为多核或多CPU派生进程。那么这个模块就可以充分利用多核CPU的优势，实现并发的访问啊！！！这么好的东西，我居然现在才发现！话不多说，代码改进如下：

    import urllib2 
    from multiprocessing.dummy import Pool as ThreadPool 
    import time
    import requests
    
    
    def spider(urls):
	    headers={}
	    r=map(requests.get,urls)

    def main():
	    start=time.clock()
	    
	    pool=ThreadPool()
	    
	    
	    #get url
	    base_url='http://blog.sina.com.cn/s/article_sort_1191258123_10001_'
	    urls=[]
	    for i in range(1,32):
	    url=base_url+str(i)+'.html'
	    urls.append(url)
	    
	       
	    
	    spider(urls)
	    
	    pool.close()
	    pool.join()
	    
	    end=time.clock()
	    
	    print 'all done costs %f' % (end-start)

    if __name__ == '__main__':
		main()



你没有看错！！！就是这么短短的几行代码！！！聪明的小伙伴肯定还发现了，我之前爬网站的时候，是用for循环把URL一个一个传进函数的，但是这个代码用了map()函数，python的map()函数也是相当了得啊，这个函数有两个参数，第一个参数是一个函数，第二个参数是一个序列，这个函数会把序列里的每一个参数都传到函数中，然后返回一个新的序列，最重要的是，这个过程是并发执行的！！！不得不说，python就是这样，只要愿意去探索，它永远会给你新惊喜。呼~多线程的学习就暂时到这儿，继续去探索新大陆了ლ(╹◡╹ლ)






