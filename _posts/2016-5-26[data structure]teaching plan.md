---
layout:     post
title:      "【数据结构】教学编制计划"
date:       2016-05-26 19:00:00
author:     "Yuki"
---


**背景**

大学的每个专业都要制定教学计划。假设任何专业都有固定的学习年限，每学年含两学期，每学期的时间长度和学分上限值均相等。每个专业开设的课程都是确定的，而且课程在开设时间的安排必须满足先修关系。每门课程有哪些先修课程是确定的，可以有任意多门，也可以没有。每门课恰好占一个学期。试在这样的前提下设计一个教学计划编制程序。

**问题描述**

若用有向网表示教学计划，其中顶点表示某门课程，有向边表示课程之间的先修关系（如果A课程是B课程的先修课程，那么A到B之间有一条有向边从A指向B）。试设计一个教学计划编制程序，获取一个不冲突的线性的课程教学流程。（课程线性排列，每门课上课时其先修课程已经被安排）。

**基本要求**

（1）输入参数：课程总数，每门课的课程号（固定占3位的字母数字串）和直接先修课的课程号。

（2）若根据输入条件问题无解，则报告适当的信息；否则将教学计划输出到用户指定的文件中。

**物理数据类型**

题目首先要求输入整数n，因为n为正整数，用C语言中的整型数据来定义变量。用三	个二维字符数组来存储输入的课程编号和课程之间的先修关系。
因为输入的课程之间存在一对多、多对一的关系，且我们只关系课程之间的先修关系（即	只考虑结点的出度），所以用邻接表来存储有向图。

**程序的流程**

程序由四个模块组成：

（1）输入模块：调用input()函数，完成课程总数n、n个课程编号以及课程之间的先修关系的输入。

（2）建图模块：调用IntGraph()函数，建立一个空图，然后调用InsertVerList()函数把n个课程编号插入到顶点表中，并根据输入的课程关系调用InsertEdge()函数将边的关系插入边表中，至此，邻接表的建立完成。

（3）排序模块：调用TopologicalSort()函数，对邻接表中的课程进行排序，并把排序结果存储到数组中。

（4）输出模块：调用output()函数，把排序结果（排序成功与否，即课程的线性排列）输出到屏幕上。

**算法的具体步骤**

1.调用输入函数input()，完成完成课程总数n、n个课程编号以及课程之间的先修关系的输入。将其分别存储到整形数据n、字符数组a\[ \]\[ \]、b\[ \]\[ \]、c\[ \]\[ \]中。

2.调用IntGraph()函数，建立一个空图，然后用循环遍历数组a\[ \]\[ \]，调用InsertVerList()函数把n个课程编号插入到顶点表中，用循环嵌套遍历a\[ \]\[ \]和b\[ \]\[ \]、c\[ \]\[ \]，调用InsertEdge()函数将边的关系插入边表中，至此，邻接表的建立完成。

3.调用排序函数TopologicalSort()，把n、m、G传入该函数中，定义指针*stack指向开辟的n个空间，遍历顶点表找到入度为0的顶点，存入stack[]中，利用循环依次把stack[]中的顶点数据（即课程编号）取出存入数组d\[ \]\[ \]中，每取出一个顶点，利用循环遍历该顶点的边表，将该顶点指向的所有顶点入度减一，并判断这些顶点中是否有入度为0的顶点，若有则存入stack[]中。循环结束返回d\[ \]\[ \]中的课程编号个数count。

4.调用输出函数output()，把count和d\[ \]\[ \]传入该函数中，判断count和邻接表的顶点个数是否相等，若否，则输出无法进行不冲突的课程安排的提示，若相等，则用循环将d\[ \]\[ \]中排序好的课程编号依次输出到屏幕上。

**完整代码如下：**

    #include<stdio.h>
    #include<stdlib.h>
    #include<string.h>
    //#define max 100

    /*----------------------结构体声明-----------------------*/
    //边表结点结构体声明
    typedef struct EdgeNode					
    {
    	int index;
    	struct EdgeNode *next;
    }EdgeNode;
    //顶点表结构体声明
    typedef struct VertexList					
    {
    	int in;
    	char data[4];
    	struct EdgeNode *first;
    }adjList;
    // 图的结构体声明
    typedef struct graphAdjList					
    {	
    	int NumVertexes,NumEdges;
    	struct VertexList VerList[100];
    }graphAdjList;

    /*----------------------函数声明----------------------*/
    int input(int *n,char a[100][4],char b[100][4],char c[100][4]);
    void InitGraph(graphAdjList *G);
    void InsertVerList(char a[4],graphAdjList *G);
    int TopologicalSort(graphAdjList *G,char d[100][4]);
    void output(int count,graphAdjList *G,char d[100][4]);
    void InsertEdge(int t,int k,graphAdjList *G);
    /*----------------------输入函数----------------------*/
    int input(int *n,char a[100][4],char b[100][4],char c[100][4])
    {
    	int i;
    	printf("请输入课程总数:\n");
    	scanf("%d",n);
    
    	i=0;
    	while(i<(*n))
    	{	
    		printf("请输入第%d门课程编号：\n",i+1);
    		scanf("%s",&a[i][0]);
    		getchar();
    		i++;
    	}
    
    	i=0;
    	printf("请输入直接先修的课程编号，每次输入两个课程编号，前一个课程编号在后一个课程编号前修，中间以空格作为分隔，输入-1 -1时结束输入\n");	
    	do	
    	{		
    		scanf("%s %s",&b[i][0],&c[i][0]);
    		getchar();
    		i++;
    	}
    	while(strcmp(&b[i-1][0],"-1"));
    	return i-1;
    }
    /*----------------------图的初始化函数----------------------*/
    void InitGraph(graphAdjList *G)
    {
    	G->NumVertexes=0;
    	G->NumEdges=0;
    
    }
    /*----------------------插入顶点函数----------------------*/
    void InsertVerList(char a[4],graphAdjList *G)
    {
    	int i=G->NumVertexes++;
    
    	strcpy(G->VerList[i].data,a);
    	G->VerList[i].first=NULL;
    	G->VerList[i].in=0;
    }
    /*----------------------插入边函数----------------------*/
    void InsertEdge(int k,int t,graphAdjList *G)
    {
    		EdgeNode *p;
    		G->VerList[t].in++;
    		p=(EdgeNode *)malloc(sizeof(EdgeNode));
    		p->index=t;
    		p->next=G->VerList[k].first;
    		G->VerList[k].first=p;
    		G->NumEdges++;
    }
    /*----------------------拓扑排序函数----------------------*/
    int TopologicalSort(graphAdjList *G,char d[100][4])
    {
	EdgeNode *e;
	int i,k,gettop,m;	
	int top=0;						//用于栈指针下标索引
	int count=0;					//用于统计输出顶点个数
	int *stack;						//用于存储入度为0的顶点
	stack=(int *)malloc(G->NumVertexes*sizeof(int));
 
	for(i=0;i<G->NumVertexes;i++)
	{
		if(G->VerList[i].in==0)
		{
			stack[++top]=i;
		}
	}
	
	m=0;
	while(top!=0)
	{

		gettop=stack[top--]; 
		strcpy(d[m++],G->VerList[gettop].data);

		count++;

		for(e=G->VerList[gettop].first;e;e=e->next)
		{
			k = e->index;
			if(!(--G->VerList[k].in))
			{
				stack[++top]=k;
			}
		}
	}
    return count;
    }
    /*----------------------输出函数----------------------*/
    void output(int count,graphAdjList *G,char d[100][4])
    {
    	int i;
    	if(count<G->NumVertexes)
    		printf("输入的课程关系构成回路，不满足拓扑排序的要求，程序结束\n");
    	else
    	{	
    		printf("课程的安排为：\n");
    		for(i=0;i<G->NumVertexes;i++)
    			printf("%s ",d[i]);
    	}
    	printf("\n");
    }
    
    
    /*----------------------打印邻接表----------------------*/
    
    void PrintfGraphAL(graphAdjList *G)
    {
    	EdgeNode *p;
    	int i;
    	printf("下标 入度  数据  边表\n");
    for (i = 0; i < G->NumVertexes; i++)
    {
    printf(" %d %d%s",i,G->VerList[i].in, G->VerList[i].data);
    p = G->VerList[i].first;
    while (p)
    {
    printf("→ %d", p->index);
    p = p->next;
    }
    printf("\n");
    }
    }

    /*----------------------主函数----------------------*/
    void main()
    {
    	int n,m,count,i,j,k,t;
    	char a[100][4],b[100][4],c[100][4],d[100][4];
    	graphAdjList G;
    	m=input(&n,a,b,c);
    	InitGraph(&G);
    	for(i=0;i<n;i++)
    		InsertVerList(a[i],&G);
    
    	for(i=0;i<m;i++)
    	{
    		for(j=0;j<n;j++)
    		{
    			if(!(strcmp(b[i],a[j])))
    				k=j;
    			else if(!(strcmp(c[i],a[j])))
    				t=j;
    		}
    		InsertEdge(k,t,&G);
    	}
    //	PrintfGraphAL(&G);
    	count=TopologicalSort(&G,d);
    	output(count,&G,d);
    }

**运行结果**

<img src="../../../../../img/blogs/teaching plan/01.png">

<img src="../../../../../img/blogs/teaching plan/02.png">