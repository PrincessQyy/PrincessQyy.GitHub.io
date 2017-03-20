---
layout:     post
title:      "【数据结构】自组织线性表"
date:       2016-06-08 11:39:00
author:     "Yuki"
---
**背景**

自组织线性表根据估算的访问频率排列记录，先放置请求频率最高的记录，接下来是请求频率次高的记录，依此类推。自组织线性表根据实际的记录访问模式在线性表中修改记录顺序。自组织线性表使用启发式规则决定如何重新排列线性表。转置方法的基本原理是，在一次查找过程中，一旦找到一个记录，则将它与前一个位置的记录交换位置。这样，随着时间的推移，经常访问的记录将移动到线性表的前端，而曾经频繁使用但以后不再访问的记录将逐渐退至线性表的后面。
尽管一般情况下自组织线性表的效率可能没有查找数和已排序的线性表那么好，但它也有自身的优势。它可以不必对线性表进行排序，新记录的插入代价很小；同时也比查找树更容易实现，且无需额外的存储空间。

**问题描述**
 
用转置法实现一个自组织线性表，保存一组汉字用于查询。

**基本要求**

（1）从文件中读入一组汉字集合，用自组织线性表保存。

（2）在查询时，采用转置法调整自组织线性表的内容。

（3）从文件中依次读入需查询的汉字，把查询结果输出（如找到，返回比较的次数，如果没有找到，返回比较的次数）

**物理数据类型**

因为输入的汉字个数是不需要改变的，而且要进行多次的查找，并且返回元素位置，所	以用顺序存储结构的线性表来储存数据。

**程序的流程**

程序由四个模块组成：

（1）输入模块：调用input()函数，完成k个汉字的输入和要查找t个的汉字的输入，并分别存放到数组a\[\]\[\]，b\[\]\[\]中。

（2）建表模块：调用InitList()函数，建立一个空线性表，然后调用InsertList()函数向线性表中添加n个汉字。至此，线性表创建完毕。

（3） 查找模块：遍历b\[\]\[\]数组，调用SearchList()函数，完成汉字的查找并返回查找情况，若查找成功，则还要调用UpdateList()函数将该汉字与它在线性表中前一个位置的汉字进行交换，完成线性表的更新。

（4）输出模块：调用output()函数，把查找情况（查找成功与否和查找次数）输出到屏幕上。

完整代码如下：

    `#include<stdio.h>
    #include<stdlib.h>
    #include<string.h>
    #define maxs 100
    typedef char ElemType;
    static int out[maxs]={0},mark[maxs]={0};
    
    /*---------------------结构体定义---------------------*/
    typedef struct List
    {
    	ElemType *elem;
    	int length;
    	int maxsize;
    }SqList;
    /*---------------------函数声明---------------------*/
    void InitList(SqList *L);
    void input(int *i,int *j,char a[maxs][3],char b[maxs][3]);
    void InsertList(char e[3],SqList *L);
    void SearchList(char e[3],SqList *L,int k);
    void UpdateList(int i,SqList *L);
    void output(int j,char b[maxs][3]);
    /*---------------------初始化函数---------------------*/
    void InitList(SqList *L)
    {
	L->maxsize=maxs;
	L->elem=(ElemType*)malloc((L->maxsize)*3*sizeof(ElemType));
	L->length=0;
	if(!(L->elem))
	{
		printf("线性表创建失败\n");
		exit(0);
	}
    }
    /*---------------------插入元素函数---------------------*/
    void InsertList(char e[3],SqList *L)
    {
    	strcpy(&(L->elem[3*(L->length)]),e);
    	
    	L->length++;
    }
    /*---------------------查找函数---------------------*/
    void SearchList(char e[3],SqList *L,int k)
    {
    	int i,m=0;
    	for(i=0;i<L->length;i++)
    	{
    		if(!strcmp(e,&(L->elem[3*i])))
    		{
    
    			if(i!=0)
    				UpdateList(i,L);
    			m=1;
    			break;
    		}
    		
    	}
    	mark[k]=m;
    	if(m)
    		out[k]=i+1;
    	else
    		out[k]=L->length;
    
    }
    /*---------------------更新函数---------------------*/
    void UpdateList(int i,SqList *L)
    {
    	
    	char temp[3];
    	strcpy(temp,&(L->elem[3*i]));
    	strcpy(&(L->elem[3*i]),&(L->elem[3*(i-1)]));
    	strcpy(&(L->elem[3*(i-1)]),temp);
    }
    
    /*---------------------输入函数---------------------*/
    void input(int *i,int *j,char a[maxs][3],char b[maxs][3])
    {
    	*i=0;
    	*j=0;
    
    	printf("请输入汉字集合，每次输入一个汉字，汉字之间以空格作为分隔，输入-1时结束输入\n");
    	do
    	{
    		scanf("%s",a[(*i)++]);
    		getchar();
    
    	}while(strcmp(a[(*i)-1],"-1"));
    	
    	
    	printf("请输入要查找的汉字，每次输入一个汉字，汉字之间以空格作为分隔，输入-1时结束输入\n");
    	do
    	{
    		scanf("%s",b[(*j)++]);
    		getchar();
    	}while(strcmp(b[(*j)-1],"-1"));
    }
    /*---------------------输出函数---------------------*/
    void output(int j,char b[maxs][3])
    {
    	int i;
    	for(i=0;i<j;i++)
    	{
    		if(mark[i]==1)
    			printf("\"%s\"的查找情况：查找成功，查找了%d次\n",b[i],out[i]);
    		else
    			printf("\"%s\"的查找情况：查找失败，查找了%d次\n",b[i],out[i]);
    	}
    }
    /*---------------------主函数---------------------*/
    void main()
    {	
    	SqList L;
    	int k,t,i=0;
    	char a[maxs][3],b[maxs][3];
    
    
    	InitList(&L);
    	input(&k,&t,a,b);
    for(i=0;i<k-1;i++)
    		InsertList(a[i],&L);
    	for(i=0;i<t-1;i++)
    	{
    		SearchList(b[i],&L,i);
    	}
    	output(t-1,b);
    	
    }`