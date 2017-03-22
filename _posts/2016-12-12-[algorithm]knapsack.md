---
layout:     post
title:      "【算法设计】0-1背包问题"
date:       2016-12-12 10:12:00
author:     "Yuki"
---

**问题描述**

利用动态规划算法的思想，给定n种物品（每种一个）和一个背包，尽可能将背包装满并使得其价值最大。（注：每个物品要么不放，要么放1个。）

**算法描述**

利用动态规划算法的思想，给定n种物品（每种一个）和一个背包，尽可能将背包装满并使得其价值最大。

假定我们定义一个函数c[i, w]表示到第i个元素为止，在限制总重量为w的情况下我们所能选择到的最优解。那么这个最优解要么包含有i这个物品，要么不包含，肯定是这两种情况中的一种。如果我们选择了第i个物品，那么实际上这个最优解是c[i - 1, w-wi] + vi。而如果我们没有选择第i个物品，这个最优解是c[i-1, w]。这样，实际上对于到底要不要取第i个物品，我们只要比较这两种情况，哪个的结果值更大就是最优的，并且最优解是基于选择物品i时总重量是在w范围内的，基于前面讨论，可以得到如下的递推公式：

<img src="../../../../../img/blogs/knapsack/01.png">

**算法步骤**

1.读取文件并提取数据存入变量和列表中。

2.生成n行，total_weight列的数组（n为物品个数，total_weight为背包能承受的总重量），初始化全部元素为0。

3.对于这n个元素a1, a2, ...ak来说，依次在重量(j)大于等于它自身的重量时将其价值填入c数组，并且判断放入该物品和不放该物品时背包价值的大小，取大的值填入c数组，同时满足组成的物品组合必然满足总重量<=背包重量限制。

4.重复2步骤直到遍历完c数组，这样c数组最后一个元素就是背包能放的物品最大价值。返回c数组。

5.将c数组最后一个元素也就是背包能放的物品最大价值写入文件，然后倒着遍历c数组，若是某一行最后一个数值大于其前一行该位置的数值，证明一定选择了该物品，标记选择的物品并写入文件。

**完整代码如下：**

    import re
    import time
    import random
    #背包算法

    def knapsack(n,total_weight,w,v):
	    #初始化
	    c=[[0 for j in range(total_weight+1)] for i in range(n+1)]
	    #动态规划
	    for i in range(1,n+1):
	        for j in range(1,total_weight+1):
	            c[i][j]=c[i-1][j]
	            if j>=w[i-1] and c[i][j]<c[i-1][j-w[i-1]]+v[i-1]:
	                c[i][j]=c[i-1][j-w[i-1]]+v[i-1]
	    return c

    #写入文件
    def output(n,total_weight,w,c):
	    with open ('knapsack_output.txt','w') as f:
	        f.write('最大价值为：%d\n选择的物品为:' % c[n][total_weight])
	        x=[False for i in range(n)]  
	        j=total_weight  
	        for i in range(n,0,-1):  
	            if c[i][j]>c[i-1][j]:  
	                x[i-1]=True  
	                j-=w[i-1]    
	        for i in range(n):  
	            if x[i]:  
	                f.write('第 %d 个,' % (i+1))  
      
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
	    c=knapsack(n,total_weight,w,v)
	    output(n,total_weight,w,c)
	    end = time.clock()
	    print ("运行时间为： %f s" % (end - start))

**运行结果**

小规模数据：

<img src="../../../../../img/blogs/knapsack/02.png">

<img src="../../../../../img/blogs/knapsack/03.png">

中规模数据：

<img src="../../../../../img/blogs/knapsack/04.png">

<img src="../../../../../img/blogs/knapsack/05.png">

大规模数据：

<img src="../../../../../img/blogs/knapsack/06.png">

<img src="../../../../../img/blogs/knapsack/07.png">

<img src="../../../../../img/blogs/knapsack/08.png">