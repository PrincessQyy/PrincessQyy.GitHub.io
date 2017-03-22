---
layout:     post
title:      "【算法设计】矩阵链乘法"
date:       2016-12-16 21:00:00
author:     "Yuki"
---

**问题描述**

利用动态规划算法的思想，给定n个可乘矩阵，求其最小乘法次数及相应的加括号方式。

**算法思想**

利用动态规划算法求解矩阵链连乘问题。

设要求出矩阵连乘MiMi+1……Mj-1Mj（i<j）所需的最少乘法次数。

因共有j-i+1个矩阵，故称这个矩阵连乘的规模是j-i+1

按照做最后一次乘法的位置进行划分，该矩阵连乘一共可分为j-i种情况即有(j-i)种断开方式：Mi(Mi+1┅Mj)，(MiMi+1)(Mi+2┅Mj)，┅，(MiMi+1┅Mj-1)Mj

将上述的j-i种情况表示为通式：(Mi┅Mk) (Mk+1┅Mj)（i≤k<j）

记第t个矩阵Mt的列数为rt，并令rt-1为矩阵Mt的行数

则Mi┅Mk连乘所得是ri-1×rk维矩阵

Mk+1┅Mj连乘所得是rk×rj维矩阵

故这两个矩阵相乘需要做ri-1×rk×rj次乘法

在此之前已知

任一个规模不超过j-i的矩阵连乘所需的最少乘法次数，故(Mi┅Mk)和(Mk+1┅Mj)所需的最少乘法次数已知，将它们分别记之为mi,k和mk+1,j。

形为(Mi┅Mk)(Mk+1┅Mj)的矩阵连乘所需的最少乘法次数为：

mi,k   +   mk+1,j   +   ri-1×rk×rj。

对满足i≤k<j 的共j-i种情况逐一进行比较，可以得到矩阵连乘MiMi+1┅Mj-1Mj（i<j）所需的最少乘法次数mi,j为：

m\[i\]\[j\]=min {   m\[i\]\[k\]  +  m\[k+1\]\[j\]   +   ri-1×rk×rj   }     （i≤k<j）

**算法步骤**

1.读取文件并提取数据存入变量和列表中。

2.将pos矩阵（括号位置矩阵）和m矩阵（最小次数矩阵）初始化全部填为0。连乘矩阵个数从2开始，依次判断是否可以加括号，假如要算m[i][j],先算出m\[i+1\]\[j\]+p\[i-1\]*p\[i\]*p\[j\]赋给min_num，然后循环在k为(i+1,j)时，计算m\[i\]\[k\]+m\[k+1\]\[j\]+p\[k\]*p\[i-1\]*p\[j\]是否小于min_num，若是则赋值为该值，并且将加括号的位置标记给pos列表，若否，则min_num值不变，如此重复，算出最小次数矩阵。

3.调用find_pos()函数找到加括号的位置，传入起始位置start和终止位置end，提取pos数组中该位置的值，判断若括号左右的矩阵大于一个，则可以加括号，将该位置存入列表，若该位置左右两边的矩阵个数大于3，则有可能还有括号，所以递归find_pos()函数，直到找出所有括号并存入列表。

4.将矩阵链连乘的最小次数和加括号的位置写入文件。

**完整代码如下：**

    import re
    import time
    import random
    #找最小次数
    def dp_matrix(n,p):
        #pos矩阵存括号的位置
        pos=[[0 for i in range(n+1)]for j in range(n+1)]
        #m矩阵存乘法的最小次数
        m=[[0 for i in range(n+1)]for j in range(n+1)]

        #r代表连乘矩阵个数
        for r in range(2,n+1):
            #i代表从差值为r时，需要运算的次数为n-r+1次
            for i in range(1,n-r+2):
                index=i
                #要差值从1开始，j为i加上差值r再减去1
                j=i+r-1
                min_num=m[i+1][j]+p[i-1]*p[i]*p[j]
                
                #k表示加括号的位置
                for k in range(i+1,j):
                    tem=m[i][k]+m[k+1][j]+p[k]*p[i-1]*p[j]
                    if tem<min_num:
                        min_num=tem
                        index=k
                m[i][j]=min_num
                pos[i][j]=index
        
        find_pos(1,n,pos)
                
        return m[1][n]
        
    #找括号位置
    def find_pos(start,end,pos):
                global pos_mark
                #找括号位置
                k=pos[start][end]
                #若括号左右的矩阵大于一个，则加括号
                if k>start:
                        pos_mark.append('矩阵%d 到矩阵 %d，'% (start,k))
                if k<end-1:
                        pos_mark.append('矩阵%d 到矩阵 %d, '% (k+1,end))
                #若括号左右的矩阵大于2个，递归左右矩阵
                if(k-start>=2):
                    find_pos(start,k,pos)
                if(end-k>=2):
                    find_pos(k+1,end,pos)       

    #模块入口
    if __name__=='__main__':
	    start = time.clock()
	    n=int(input('请输入案例规模：'))
	    #产生随机数
	    with open('Matrix_input.txt','w') as f:
	        f.write('%d\n' % n)
	        for i in range(n+1):
	            column=random.randint(10,100)
	            f.write('%d ' % column)
	    #读取文件内容并存入列表
	    with open('Matrix_input.txt','r') as f:
	        n=int(f.readline())
	        s=f.read()
	    s=re.findall(r'\d+',s)
	    p=[int(item) for item in s]
	    #全局变量pos_mark用来存位置
	    pos_mark=[]   
	
	    min_num=dp_matrix(n,p)
	    #将结果写入文件
	    with open('Matrix_output.txt','w') as f:
	            f.write('最小次数是：%d\n括号的位置是：' % (min_num))
	            for each in pos_mark:
	                    f.write(each)
	    end = time.clock()
	    print ("运行时间为： %f s" % (end - start))  

**运行结果**

<img src="../../../../../img/blogs/matrix_chain/01.png">


<img src="../../../../../img/blogs/matrix_chain/02.png">


<img src="../../../../../img/blogs/matrix_chain/03.png">


<img src="../../../../../img/blogs/matrix_chain/04.png">


<img src="../../../../../img/blogs/matrix_chain/05.png">

案例规模大的时候，可能会需要等待一段时间，图太长，我就不截完了。