---
layout:     post
title:      "【数据结构】快速排序"
date:       2016-05-21 14:39:00
author:     "Yuki"
---

**背景**

排序是计算机内经常进行的一种操作，其目的是将一组“无序”的记录序列调整为“有序”的记录序列。

假设含n个记录的序列为{ R1, R2, …， Rn }

其相应的关键字序列为  { K1, K2, …，Kn }
这些关键字相互之间可以进行比较，即在它们之间存在着这样一个关系 ：

    　           Kp1≤Kp2≤…≤Kpn

按此固有关系将上式记录序列重新排列为{ Rp1, Rp2, …，Rpn }的操作称作排序。
排序算法是计算机科学中最重要的研究问题之一。对于排序的研究既有理论上的重要意义，又有实际应用价值。它在计算机图形、计算机辅助设计、机器人、模式识别、及统计学等领域具有广泛应用。 常见的排序算法有起泡排序、直接插入排序、简单选择排序、快速排序、堆排序等。

例1：有时候应用程序本身就需要对信息进行排序。为了准备客户账目，银行需要根据支票的号码对支票排序；

例2：在一个绘制互相重叠的图形对象的程序中，可能需要根据一个“在上方”关系将各对象排序，以便自下而上地绘出对象。

例3：在一个由n个数构成的集合上，求集合中第i小/大的数。

例4：对一个含有n个元数的集合，求解中位数、k分位数。

**问题描述**

在操作系统中，我们总是希望以最短的时间处理完所有的任务。但事情总是要一件件地做，任务也要操作系统一件件地处理。当操作系统处理一件任务时，其他待处理的任务就需要等待。虽然所有任务的处理时间不能降低，但我们可以安排它们的处理顺序，将耗时少的任务先处理，耗时多的任务后处理，这样就可以使所有任务等待的时间和最小。
只需要将n 件任务按用时去从小到大排序，就可以得到任务依次的处理顺序。当有 n 件任务同时来临时，每件任务需要用时ni，求让所有任务等待的时间和最小的任务处理顺序。

**基本要求**

（1）数据的输入输出格式：
	
 输入：
第一行是一个整数n，代表任务的件数。
接下来一行，有n个正整数，代表每件任务所用的时间。

 输出：
输出有n行，每行一个正整数，从第一行到最后一行依次代表着操作系统要处理的任务所用的时间。按此顺序进行，则使得所有任务等待时间最小。

（2）使用快速排序，轴值采用随机数确定。

**物理数据类型**

因为输入的汉字个数是不需要改变的，而且要进行多次的遍历，并且要利用元素位置来	进行排序，所以用顺序存储结构的线性表来储存数据。

**程序的流程**

程序由四个模块组成：

（1）输入模块：调用input()函数，完成任务的件数n的输入，把随机产生的n件任务的等待时间存放到数组中。

（2）建表模块：调用InitList()函数，建立一个空线性表，然后调用InsertList(	)函数向线性表中添加n件任务的等待时间。至此，线性表创建完毕。

（3） 排序模块：调用QuickSortt()函数，完成线性表中数据从小到大的排序。

（4）输出模块：调用output()函数，把排序前的数据和排序后的数据输出到屏幕上。

**算法的具体步骤**

1）调用输入函数input()，完成任务的件数n的输入，循环n次，调用随机函数并把随机产生的n件任务的等待时间存放到数组a[]中。

2）调用InitList()函数，调用malloc()函数为线性表开辟空间，并将线性表长度初始化为0，即完成空线性表的建立。然后利用循环语句遍历数组a[]，调用InsertList()函数把n件任务的等待时间一一添加到线性表并更新线性表长度。至此，线性表创建完毕。

3）调用QuickSortt()函数，设置两个变量i、j，排序开始的时候：i=0，j=L->length-1；以线性表第一个元素作为关键数据，赋值给e；从j开始从后向前搜索，找到第一个小于e的值L->elem[j]，将L->elem[j]和L->elem[i]互换；从i开始由前向后搜索，找到第一个大于e的值L->elem[i]，将L->elem[i]和L->elem[j]互换；如此重复，直到i>=j。然	后递归快速排序，将其他n-1个元素也调整到排序后的正确位置。至此，线性表中的数	据已从小到大排好序。

4）调用输出函数output()，遍历数组a[]和线性表，把排序前的数据和排序后的数据输出到屏幕上。

**完整代码如下：**

    #include<stdio.h>
    #include<stdlib.h>
    #include<time.h>
    #define maxs 100
    typedef int ElemType;

    /*---------------------------结构体定义---------------------------*/
    typedef struct List
    {
    	ElemType *elem;
    	int length;
    	int maxsize;
    }SqList;
    /*---------------------------函数声明---------------------------*/
    void InitList(SqList *L);
    void InsertList(ElemType e,SqList *L);
    void input(int *n,ElemType a[maxs]);
    void QuickSort(int left,int right,SqList *L);
    void output(SqList *L);
    /*---------------------------输入函数---------------------------*/
    void input(int *n,ElemType a[maxs])
    {
    	int i;
    	printf("请输入整数的个数n:\n");
    	scanf("%d",n);
    	srand(time(0));
    	for(i=0;i<*n;i++)
    	{
    		a[i]=rand()%100+1;
    	}
    }
    /*---------------------------初始化函数---------------------------*/
    void InitList(SqList *L)
    {
    	L->maxsize=maxs;
    	L->elem=(ElemType*)malloc((L->maxsize)*sizeof(ElemType));
    	L->length=0;
    	if(!(L->elem))
    	{
    		printf("线性表创建失败，程序结束\n");
    		exit(0);
    	}
    }
    /*---------------------------插入顶点函数---------------------------*/
    void InsertList(ElemType e,SqList *L)
    {
    	L->elem[L->length]=e;
    	L->length++;
    }

    /*---------------------------排序函数---------------------------*/
    void QuickSort(int left,int right,SqList *L)
    {
    	int i=left,j=right;
    	ElemType e=L->elem[left];
    	if(i>=j)
    		return;
    	while(i<j)
    	{
    	
    		while( (i<j) && (L->elem[j])>=e )
    		{
    			j--;
    		}
    		L->elem[i]=L->elem[j];
    
    		while( (i<j) && (L->elem[i])<=e )
    		{
    			i++;
    		}
    		L->elem[j]=L->elem[i];
    	}
    
    	L->elem[i]=e;
    
    	QuickSort(left,i-1,L);
    	QuickSort(i+1,right,L);
    
    }
    /*---------------------------输出函数---------------------------*/
    void output(ElemType a[maxs],SqList *L)
    {
    	int i;
    	printf("排序前的数据为：\n");
    	for(i=0;i<(L->length);i++)
    		printf("%d ",a[i]);
    	
    	printf("\n排序后的数据为：\n");
    	for(i=0;i<(L->length);i++)
    		printf("%d\n",L->elem[i]);
    }
    /*---------------------------主函数---------------------------*/
    void main()
    {
    	SqList L;
    	int i,n;
    	ElemType a[maxs];
    	input(&n,a);
    	InitList(&L);
    	for(i=0;i<n;i++)
    	{
    		InsertList(a[i],&L);
    	}
    	QuickSort(0,(L.length-1),&L);
    	output(a,&L);
    }

**运行结果**

<img src="../../../../../img/blogs/quick sort/01.png">

<img src="../../../../../img/blogs/quick sort/02.png">