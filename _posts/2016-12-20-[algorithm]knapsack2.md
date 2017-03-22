---
layout:     post
title:      "【算法设计】贪心算法求解背包问题"
date:       2016-12-20 15:35:00
author:     "Yuki"
---

**问题描述**

给定几组数据，利用贪心算法的思想，将物品装入背包并使得其价值最大。

**算法思想**

贪心算法解背包问题的基本步骤是：首先计算每种物品单位重量的价值Vi/Wi，然后，依贪心选择策略，将尽可能多的单位重量价值最高的物品装入背包。若将这种物品全部装入背包后，背包内的物品总重量未超过C，则选择单位重量价值次高的物品并尽可能多地装入背包。依此策略一直地进行下去，直到背包装满为止。

**算法步骤**

1.读取文件并提取数据存入变量和列表中。

2.对物体的单位重量价值（Vi/wi）进行降序排序，并用mark数组标记物体位置。

3.定义x数组存储放入背包的物品，初始化为0。

4.对于按单位价值降序排好的物体，判断其重量是否大于背包剩余重量，若是，则不放入背包，若否，放入背包，并标记物体被放入（将x数组标为1），背包重量减去物体重量。

5.重复4步骤直到背包装满，若物体只有部分被装下，记录装下的比重。返回x数组

6.遍历x数组，找到被标记为1的物体写入文件，并计算物体总价值写入文件。

**完整代码如下**

    import re
    import time
    import random
    #根据单位重量价值进行排序

    def mySort(v,w,n):
	    x=[]
	    mark=[]
	    for i in range(n):
	        x.append(v[i]/w[i])
	        mark.append(i+1)
	    for i in range(len(x)-1):
	        k=i
	        max_v=x[i]
	        for j in range(i,len(x)):
	            if(x[j]>max_v):
	                k=j;
	                max_v=x[j]
	       
	        if(k!=i):
	            temp=x[i]
	            x[i]=x[k]
	            x[k]=temp
	            temp=v[i]
	            v[i]=v[k]
	            v[k]=temp
	            temp=w[i]
	            w[i]=w[k]
	            w[k]=temp
	            temp=mark[i]
	            mark[i]=mark[k]
	            mark[k]=temp
	    return v,w,mark
    #贪心算法求解背包问题
    def knapsack(n,total_weight,w,v):
	    x=[]
	    for i in range(n):
	        x.append(0)
	    for i in range(n):
	        if(w[i]>total_weight):
	            break
	        x[i]=1
	        total_weight-=w[i]
	    #如果物体只有部分被装下
	    if(i<=n):
	        x[i]=total_weight/w[i]
	    return x

    #写入文件
    def output(n,x,w,v,mark):
	    total_value=0
	    with open ('knapsack_output.txt','w') as f:
	        f.write('选择的物品为:')
	        for i in range(n):
	            if(x[i]==1):
	                f.write('第 %d 个,' % (mark[i]))
	                total_value+=v[i]
	            elif(x[i]!=0):
	                f.write('第 %d 个的一部分,' % (mark[i]))
	                total_value+=x[i]*v[i]
	        f.write('\n物体的总价值为：%d' % (total_value))

    #模块入口
    if __name__=='__main__':
	    start = time.clock()
	    #产生随机数
	    n=int(input('请输入案例规模：'))
	    with open('knapsack_input.txt','w') as f:
	        f.write('%d\n' % n)
	        total_weight=random.randint(100,1000)
	        f.write('%d\n' % total_weight)
	        for i in range(n):
	            weight=random.randint(1,50)
	            f.write('%d ' % weight)
	        f.write('\n')
	        for i in range(n):
	            value=random.randint(30,500)
	            f.write('%d ' % value)
	    #读取文件提取需要数据
	    with open('knapsack_input.txt','r') as f:
	        n=int(f.readline())
	        total_weight=int(re.split(r'[\s\n]+',f.readline())[0])
	        w=re.findall(r'[\d]+',f.readline())
	        v=re.findall(r'[\d]+',f.readline())
	        w=[int(item) for item in w]
	        v=[int(item) for item in v]
	    v,w,mark=mySort(v,w,n)
	    x=knapsack(n,total_weight,w,v)
	    output(n,x,w,v,mark)
	    end = time.clock()
	    print ("运行时间为： %f s" % (end - start))

**运行结果**

小规模数据：

<img src="../../../../../img/blogs/knapsack2/01.png">

<img src="../../../../../img/blogs/knapsack2/02.png">

中规模数据：

<img src="../../../../../img/blogs/knapsack2/03.png">

<img src="../../../../../img/blogs/knapsack2/04.png">

<img src="../../../../../img/blogs/knapsack2/05.png">

大规模数据：

<img src="../../../../../img/blogs/knapsack2/06.png">

<img src="../../../../../img/blogs/knapsack2/07.png">

<img src="../../../../../img/blogs/knapsack2/08.png">