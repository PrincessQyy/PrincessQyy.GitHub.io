---
layout:     post
title:      "【数据结构】无向图简单路径"
date:       2016-05-21 14:39:00
author:     "Yuki"
---

**背景**

简单路径：如果一条路径上的顶点除了起点和终点可以相同外，其它顶点均不相同，则称此路径为一条简单路径。

**问题描述**

若用无向图表示高速公路网，其中顶点表示城市，边表示城市之间的高速公路。试设计一个找路程序，获取两个城市之间的所有简单路径。

**基本要求**

（1）输入参数：结点总数，结点的城市编号（4位长的数字，例如电话区号，长沙是0731），连接城市的高速公路（用高速公路连接的两个城市编号标记）。

（2）输入 要求取所有简单路径的两个城市编号。

（3）将所有路径（有城市编号组成）输出到用户指定的文件中。

**物理数据类型**

题目首先要求输入整数n，因为n为正整数，用C语言中的整型数据来定义变量。用三	个二维字符数组来存储输入的城市编号和城市之间的公路。
因为输入的城市之间存在一对多、多对一的关系，且我们只关心城市之间是否有公路，	不考虑公路的方向，所以用邻接矩阵来存储无向图，方便于公路的标记。

**程序的流程**

程序由四个模块组成：

（1）输入模块：调用input()函数，完成结点总数n、n个城市编号以及城市之间的公路(用高速公路连接的两个城市编号标记)的输入。

（2）建图模块：调用CreateMGraph()函数，建立一个空图，然后调用void InserVer(	)向顶点表中添加新的结点，再调用void InsertEdge()函数，向邻接矩阵中添加边的信息，至此，邻接矩阵表示的无向图创建完毕。

（3） 查找模块：调用Find()函数，找到输入的要求简单路径的两个城市编号在顶点表	中的位置，传入DFS()函数中完成简单路径的查找和存放。

（4）输出模块：调用output()函数，把查找情况（查找成功与否和查找成功时的所有的	简单路径）输出到屏幕上。

**算法的具体步骤**

1.调用输入函数input()，完成完成结点总数n、n个城市编号以及城市之间的公路的输入。将其分别存储到整形数据n、字符数组a\[ \]\[ \]、b\[ \]\[ \]、c\[ \]\[ \]中。

2.调用建图函数CreateMGraph()，把顶点个数、边数初始化为0，邻接矩阵也全部初始化为0，完成空图的创建。利用循环，调用void InserVer()向顶点表中添加新的结点，并更新图的顶点个数。同理，再调用void InsertEdge()函数，向邻接矩阵中添加边的信息，若两个城市之间存在公路，则它们下标对应的邻接矩阵的值变为1，并更新图的边数。至此，邻接矩阵表示的无向图创建完毕。

3.调用查找函数Find()，找到输入的要求简单路径的两个城市编号在顶点表中的位置k	和t，传入DFS()函数中，，在对k做过访问标记后，选择一条从k出发的未检测过的	边(k，y)。若发现顶点y已访问过，则重新选择另一条从k出发的未检测过的边，否则	沿边(k，y)到达未曾访问过的y，对y访问并将其标记为已访问过并将其存放到数组out\[ \]\[ \]中；然后从y开始搜索，直到搜索完从y出发的所有路径，即访问并存放完所有从y出发可达的顶点之后，才回溯到顶点x，并且再选择一条从x出发的未检测过的边。重	复上述过程直至从k出发到t为止的所有边都已检测并存放过。完成简单路径的查找和	存放。

4.调用输出函数output()，把存放路径的数组out\[ \]\[ \]传入该函数中，判断路径数量是否为0，若不为0，则在屏幕上输出两个城市间所有的简单路径，若为0，则在屏幕上输出两城市间没有简单路径的提示。

**完整代码如下：**


    #include<stdio.h>
    #include<stdlib.h>
    #include<string.h>
    #define max 100
    typedef char VerType;
    static int out[max][max],arry[max];
    static int num=0;


    /*-----------------------结构体声明-----------------------*/
    typedef struct MGraph					
    {
    	int NumVertexes,NumEdges;
    	VerType VertexList[max][max];
    	int EdgeMA[max][max];
    }MGraph;

    /*-----------------------函数声明-----------------------*/
    int input(int *n,char a[100][5],char b[100][5],char c[100][5]);
    void CreateMGraph(MGraph *MG);
    void InserVer(char a[5],MGraph *MG);
    void InsertEdge(char b[5],char c[5],MGraph *MG);
    void Find(char d[5],char e[5],MGraph *MG);
    void DFS(int k,int t,MGraph *MG,int mark[max],int r,int b[max]);
    void output(MGraph *MG);
    /*-----------------------输入函数-----------------------*/
    int input(int *n,char a[100][5],char b[100][5],char c[100][5])
    {
    	int i;
    	printf("请输入结点总数:\n");
    	scanf("%d",n);
    	getchar();
    	i=0;
    	while(i<(*n))
    	{	
    		printf("请输入第%d个城市编号：\n",i+1);
    		scanf("%s",&a[i][0]);
    		getchar();
    		i++;
    	}

	i=0;
	printf("请输入连接城市的公路，每次输入两个城市编号，中间以空格作为分隔，输入-1 -1时结束输入\n");	
	do	
	{		
		scanf("%s %s",&b[i][0],&c[i][0]);
		getchar();
		i++;
	}
	while(strcmp(&b[i-1][0],"-1"));
	return i-1;
    }
    /*-----------------------建图函数-----------------------*/
    void CreateMGraph(MGraph *MG)
    {
    	int i,j;
    	MG->NumVertexes=0;
    	MG->NumEdges=0;
    	for(i=0;i<max;i++)
    	{
    		for(j=0;j<max;j++)
    			MG->EdgeMA[i][j]=0;
    	}
    }
    /*-----------------------插入顶点函数-----------------------*/
    void InserVer(char a[5],MGraph *MG)
    {
    	int i=MG->NumVertexes;
    	strcpy(MG->VertexList[i],a);
    	MG->NumVertexes++;
    }
    /*-----------------------插入边函数-----------------------*/
    void InsertEdge(char b[5],char c[5],MGraph *MG)
    {
    	int i,k,t;
    	for(i=0;i<MG->NumVertexes;i++)
    	{
    		if(!strcmp(b,MG->VertexList[i]))
    			k=i;
    		else if(!strcmp(c,MG->VertexList[i]))
    			t=i;
    	}
    	MG->EdgeMA[k][t]=1;
    	MG->EdgeMA[t][k]=1;
    	MG->NumEdges++;	
    }
    /*-----------------------寻找函数-----------------------*/
    void Find(char d[5],char e[5],MGraph *MG)
    {
    	int i,k,t,mark[max]={0},b[max];
    	for(i=0;i<MG->NumVertexes;i++)
    	{
    		if(!strcmp(d,MG->VertexList[i]))
    			k=i;
    		if(!strcmp(e,MG->VertexList[i]))
    			t=i;
    	}
    	DFS(k,t,MG,mark,0,b);
    }
    /*-----------------------DFS函数-----------------------*/
    void DFS(int k,int t,MGraph *MG,int mark[max],int r,int b[max])
    {
    	int i,j=0;
    	if(k==t&&r>0)
    	{
    		for(i=0;i<r;i++)
    			out[num][i]=b[i];
    		out[num][i]=t;
    		arry[num]=r+1;
    		num++;
    		return;
    		
    	}
    
    	mark[k]=1;
    	b[r++]=k;
    
    	while(j<MG->NumVertexes)
    	{
    		if(MG->EdgeMA[k][j]==1 && (mark[j]==0 || j==t))
    		{
    			
    			DFS(j,t,MG,mark,r,b);
    		}
    		j++;		
    	}	
    	mark[k]=0;
    }
    /*-----------------------输出函数-----------------------*/
    void output(MGraph *MG)
    {
    	int i,j;
    	if(num==0)
    		printf("两城市间没有简单路径\n");
    
    	else
    	{
    		printf("简单路径为：\n");
    		for(i=0;i<num;i++)
    		{
    			for(j=0;j<arry[i];j++)
    				printf("%s ",MG->VertexList[out[i][j]]);
    			printf("\n");
    		}
    	}
    }
    /*-----------------------打印邻接矩阵函数-----------------------*/
    void print(MGraph *MG)
    {
    	int i,j;
    	printf("顶点表为：");
    	for(i=0;i<MG->NumVertexes;i++)
    		printf("%s ",MG->VertexList[i]);
    	printf("\n");
    	for(i=0;i<MG->NumVertexes;i++)
    	{
    		for(j=0;j<MG->NumVertexes;j++)
    		{
    			printf("%d ",MG->EdgeMA[i][j]);
    			
    		}
    			printf("\n");
    	}
    		printf("\n");
    }
    /*-----------------------主函数-----------------------*/
    void main()
    {
    	int n,m,i;
    	char a[100][5],b[100][5],c[100][5],d[5],e[5];
    	MGraph MG;
    	
    	m=input(&n,a,b,c);
    	CreateMGraph(&MG);
    	for(i=0;i<n;i++)
    		InserVer(a[i],&MG);
    	for(i=0;i<m;i++)
    		InsertEdge(b[i],c[i],&MG);
    	printf("请输入要取所有简单路径的两个城市编号，中间以空行作为分隔：\n");
    	scanf("%s %s",d,e);
    	getchar();
    	Find(d,e,&MG);
    	output(&MG);
    }


**运行结果**

<img src="../../../../../img/blogs/simple path of undirected graph/01.png">

<img src="../../../../../img/blogs/simple path of undirected graph/02.png">

<img src="../../../../../img/blogs/simple path of undirected graph/03.png">