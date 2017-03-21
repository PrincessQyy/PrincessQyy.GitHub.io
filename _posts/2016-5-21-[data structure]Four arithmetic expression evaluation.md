---
layout:     post
title:      "【数据结构】四则运算表达式求值"
date:       2016-05-21 20:39:00
author:     "Yuki"
---

**背景**

在工资管理软件中，不可避免的要用到公式的定义及求值等问题。对于数学表达式的计算，虽然可以直接对表达式进行扫描并按照优先级逐步计算，但也可以将中缀表达式转换为逆波兰表达式，这样更容易处理。

**问题描述**

四则运算表达式求值，将四则运算表达式用中缀表达式（表达式树），然后转换为后缀表达式，并计算结果。

**基本要求**

(1)使用二叉树来实现。

**实现提示**

用一棵表达式树存储四则运算表达式，即中序遍历表达式树，可得到四则运算表达式。
利用二叉树后序遍历来实现表达式的转换，得到后缀逆波兰表达式，最后可以使用栈来求解后缀表达式的值。

输入输出格式：

输入：在字符界面上输入一个中缀表达式，回车表示结束。

输出：如果该中缀表达式正确，那么在字符界面上输出其后缀表达式和计算结果，其中后缀表达式中两相邻操作数之间利用空格隔开；如果不正确，在字符界面上输出表达式错误提示。

**物理数据类型**

题目首先要求输入表达式，由于表达式中有数字也有字符，所以用C语言中的字符数组	来定义变量。

因为选择了二叉树这个数据结构，所以用链表来定义。表达式结果的计算需要将数据压入栈中，并且栈的大小在表达式输入之后可以确定，所	以用顺序储存结构的线性表来定义栈。

**算法流程**

<img src="../../../../../img/blogs/Four arithmetic expression evaluation/01.png">

**算法的具体步骤**

模块详细设计：

1）输入模块：定义一个字符数组a[]，调用C语言scanf语句来接收表达式的输入。

2）判断模块：通过调用judge()函数完成，首先判断表达式的第一个字符是否为数字或’(‘，若否，则结束程序并输出错误提示，若是，则遍历字符数组，判断字符数组中是否只有数字、运算符、小数点，左、右括号出现，若否，则结束程序并输出错	误提示，若是，则判断数字、运算符、小数点左右的字符是否合法，若否，则结束	程序并输出错误提示，并定义变量t来累计左、右括号的数量，遇到左括号t+1，遇到右括号t-1，	表达式遍历结束后通过判断t是否等于0来判断输入的括号是否匹配，若否，则结束程序并输出错误提示，若是，则将表达式传入findsymbol()函数进行下一步的操作。

3）建树模块：首先调用findsymbol()函数，利用递归依次找到运算级最低的运算符并传入createNode()函数中，完成二叉树结点的创建，然后调用transfer()函数，把表达式中的数据存放到浮点型数组double b[]中，然后调用createLeaf()函数，从头结点开始遍历二叉树，若结点为空，则完成数据的存放和二叉树叶子结点的建立，否则判断若结点处存放的是运算符，则分别递归遍历二叉树的左子树和右子树，直到找到空结点将数据存放进去，至此，二叉树创建完毕。

4）计算模块：首先调用createStack()函数，完成栈的创建，然后调用calculate()函数，	通过后序遍历二叉树，若遍历到数据则将数据压入栈中，若遍历到运算符则把两个数据弹出栈，然后将数据和该运算符的运算结果压入栈中。遍历完毕后，调用出栈函数把最众计算结果弹出栈，并在屏幕上输出。

**完整代码如下**

    #include<stdio.h>
    #include<stdlib.h>
    static int mark=0;
    /*------------------------结构体定义------------------------*/
    typedef struct treeNode   //树的结点
    {
    	double data;
    	char symbol;
    	struct treeNode *LeftChild,*RightChild;	
    }node0;
    
    typedef struct stack				//栈
    {
    	double *top;
    	double *base;
    	int length;
    }stack0;
    
    /*------------------------函数声明------------------------*/
    void judge(char a[],node0 **p);
    void findsymbol(char a[],int right,int left,node0 **p);
    void createNode(char a,node0 **p);
    int transfer(char a[],node0 **p);
    void createLeaf(double b[],node0 **p);
    void postorder(node0 **p);
    void calculate(node0 **p,stack0 *stack1);
    void createStack(int n,stack0 *stack1);
    void push(double a,stack0 *stack1);
    void pop(double *a,stack0 *stack1);

    /*------------------------判断函数------------------------*/
    void judge(char a[],node0 **p)
    {
    	int i=0,t=0;
    	if(!((a[0]>='0'&&a[0]<='9')||a[0]=='('))
    	{
    		printf("输入的表达式第一个字符必须是数字或'(',程序结束\n");
    		exit(0);
    	}
    	while(a[i]!='\0')
    	{
    		
    		if(a[i]>='0' && a[i]<='9')
    			i++;
    		else if(a[i]=='+'||a[i]=='-'||a[i]=='*'||a[i]=='/'||a[i]=='.')
    		{
    			if(a[i+1]=='\0')
    			{
    				printf("输入的表达式中运算符位置不正确,程序结束\n");
    				exit(0);
    			}
    			if(!(a[i+1]>='0'&&a[i+1]<='9'||a[i+1]=='('))
    			{
    				printf("输入的表达式结构不正确,程序结束\n");
    				exit(0);
    			}
    			if(a[i]=='/'&&a[i+1]=='0')
    			{
    				printf("表达式中除号后面不能为0,程序结束\n");
    				exit(0);
    			}
    
    			i++;
    		}
    
    		else if(a[i]=='(')
    		{
    			if((a[i-1]>='0'&&a[i-1]<='9')||a[i+1]=='+'||a[i+1]=='-'||a[i+1]=='*'||a[i+1]=='/'||a[i+1]=='\0')
    			{
    				printf("输入的表达式结构不正确,程序结束\n");
    				exit(0);
    			}
    			t++;
    			i++;
    		}
    		else if(a[i]==')')
    		{
    			if((a[i+1]>='0'&&a[i+1]<='9')||a[i-1]=='+'||a[i-1]=='-'||a[i-1]=='*'||a[i-1]=='/')
    			{
    				printf("输入的表达式结构不正确,程序结束\n");
    				exit(0);
    			}
    			t--;
    			i++;
    		}
    		else
    		{
    			if(a[i]!='.')
    			{
    				printf("输入的表达式含非法字符,程序结束\n");
    				exit(0);
    			}
    		}	
    	}
    	
    	if(a[i]=='\0')
    	{
    		if(t==0)
    			findsymbol(a,i-1,0,p);	
    		else
    		{
    			printf("输入的表达式中括号不匹配,程序结束\n");
    			exit(0);
    		}
    	}	
    }

    /*------------------------找运算符函数------------------------*/
    void findsymbol(char a[],int right,int left,node0 **p)
    {
    	int i=0,t=0,k=0,s=100,m=0;
    	for(i=right;i>=left;i--)
    	{
    		if(a[i]==')')
    		{
    			t++;
    		}
    		if(a[i]=='(')
    		{
    			t--;
    		}
    		if(a[i]=='+' || a[i]=='-'||a[i]=='*' || a[i]=='/')
    		{
    			m=1;
    			if(t<s)
    			{
    				s=t;
    				k=i;
    			}
    			else if((a[k]=='*'||a[k]=='/')&&(a[i]=='+'||a[i]=='-')&&(t==s))
    			{
    				s=t;
    				k=i;
    			}
    		}
    		
    	}	
    	
    	if(m)
    	{
    			createNode(a[k],p);
    			findsymbol(a,k-1,left,&((*p)->LeftChild));
    			findsymbol(a,right,k+1,&((*p)->RightChild));
    	}
    }

    /*------------------------数据转移函数-------------------------*/
    int transfer(char a[],node0 **p)
    {
    	int i=0,j=0,k=0;
    	double b[100];
    	char c[10];
    	while(a[i]!='\0')
    	{
    		if((a[i]>='0'&&a[i]<='9')||a[i]=='.')
    			c[j++]=a[i];
    		else
    		{
    			c[j]='\0';
    			
    			if(j>0)
    			{
    				b[k++]=atof(c);
    				j=0;
    			}
    		}
    		if(a[i+1]=='\0')
    		{
    			c[j]='\0';
    			
    			if(j>0)
    			{
    				b[k++]=atof(c);
    				j=0;
    			}
    		}
    
    		i++;
    	}
    	createLeaf(b,p);
    	return(k);
    }
    
    /*------------------------符号结点创建函数------------------------*/
    void createNode(char a,node0 **p)
    {
    	if(*p==NULL)
    	{
    			
    		*p=(node0*)malloc(sizeof(node0));
    		(*p)->symbol=a;
    		(*p)->LeftChild=NULL;
    		(*p)->RightChild=NULL;
    	//	printf("\n%c\n",(*p)->symbol);
    	}		
    }

    /*------------------------数据叶子创建函数------------------------*/
    void createLeaf(double b[],node0 **p)
    {
    	if(*p==NULL)
    	{
    		*p=(node0*)malloc(sizeof(node0));
    		(*p)->data=b[mark++];
    		(*p)->LeftChild=NULL;
    		(*p)->RightChild=NULL;
    	}
    	else if((*p)->symbol=='+' || (*p)->symbol=='-' || (*p)->symbol=='*' || (*p)->symbol=='/' )
    	{
    		createLeaf(b,&((*p)->LeftChild));
    		createLeaf(b,&((*p)->RightChild));
    	}
    }

    /*------------------------后序遍历函数------------------------*/
    void postorder(node0 **p)
    {
    	if(*p!=NULL)
    	{
    		postorder(&((*p)->LeftChild));
    		postorder(&((*p)->RightChild));
    		if((*p)->symbol=='+' || (*p)->symbol=='-' || (*p)->symbol=='*' || (*p)->symbol=='/' )
    			printf("%c ",(*p)->symbol);
    		else
    			printf("%0.2lf ",(*p)->data);
    	}
    }

    /*------------------------计算函数------------------------*/
    void calculate(node0 **p,stack0 *stack1)
    {
    	double a,b;
    	char c;
    	if(*p!=NULL)
    	{
    		calculate(&((*p)->LeftChild),stack1);
    		calculate(&((*p)->RightChild),stack1);
    		if((*p)->symbol=='+' || (*p)->symbol=='-' || (*p)->symbol=='*' || (*p)->symbol=='/' )
    		{
    			pop(&a,stack1);
    			pop(&b,stack1);
    			c=(*p)->symbol;
    			switch(c)
    			{
    			case '+':
    				push(a+b,stack1);
    				break;
    			case '-':
    				push(b-a,stack1);
    				break;
    			case '*':
    				push(a*b,stack1);
    				break;
    			case '/':
    				push(b/a,stack1);
    				break;
    			}	
    		}
    		else
    			push((*p)->data,stack1);
    	}
    }

    /*------------------------栈的操作函数------------------------*/
    void createStack(int n,stack0 *stack1)				//栈的创建
    {
    	stack1->length=n;
    	stack1->base=(double *)malloc(n*sizeof(double));
    	stack1->top=stack1->base;
    }
    
    
    void push(double a,stack0 *stack1)			//进栈
    {
    	*(stack1->top)=a;
    	stack1->top++;
    }
    
    
    void pop(double *a,stack0 *stack1)			//出栈
    {
    	stack1->top--;
    	*a=*(stack1->top);		
    }

    /*------------------------主函数------------------------*/
    void main()
    {
    	char a[100];
    	double s;
    	int n;
    	stack0 stack1;
    	node0 *node1=NULL;
    	printf("请输入中缀表达式：\n");
    	scanf("%s",a);
    	judge(a,&node1);
    	n=transfer(a,&node1);
    	printf("后缀表达式为：\n");
    	postorder(&node1);
    	createStack(n,&stack1);
    	calculate(&node1,&stack1);
    	pop(&s,&stack1);
    	printf("\nresult is %0.2lf\n",s);
    }

**运行结果**


<img src="../../../../../img/blogs/Four arithmetic expression evaluation/02.png">


<img src="../../../../../img/blogs/Four arithmetic expression evaluation/03.png">


<img src="../../../../../img/blogs/Four arithmetic expression evaluation/04.png">


<img src="../../../../../img/blogs/Four arithmetic expression evaluation/05.png">