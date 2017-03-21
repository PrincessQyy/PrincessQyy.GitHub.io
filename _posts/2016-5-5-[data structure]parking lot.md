---
layout:     post
title:      "【数据结构】停车场管理"
date:       2016-05-5 14:09:00
author:     "Yuki"
---

**问题描述**

设停车场是一个可停放 n 辆汽车的狭长通道，且只有一个大门可供汽车进出。汽车在停车场内按车辆到达时间的先后顺序，依次由北向南排列（大门在最南端，最先到达的第一辆车停放在车场的最北端），若车场内已停满 n 辆汽车，则后来的汽车只能在门外的便道上等候，一旦有车开走，则排在便道上的第一辆车即可开入；当停车场内某辆车要离开时，在它之后进入的车辆必须先退出车场为它让路，待该辆车开出大门外，其他车辆再按原次序进入车场，每辆停放在车场的车在它离开停车场时必须按它停留的时间长短交纳费用。试为停车场编制按上述要求进行管理的模拟程序。

**基本要求**

以栈模拟停车场，以队列模拟车场外的便道，按照从终端读入的输入数据序列进行模拟管理。每一组输入数据包括三个数据项：汽车“到达”或“离去”信息、汽车牌照号码以及到达或离去的时刻。对每一组输入数据进行操作后的输出信息为：若是车辆到达，则输出汽车在停车场内或便道上的停车位置；若是车辆离去，则输出汽车在停车场内停留的时间和应交纳的费用（在便道上停留的时间不收费）。

**测试用例（举例）**

设n = 2，输入数据为：（‘A’，1，5），（‘A’，2，10），（‘D’，1，15），（‘A’，3，20），（‘A’，4，25），（‘A’，5，30），（‘D’，2，35），（‘D’，4，40），（‘E’，0，0）。其中：‘A’表示车辆到达；‘D’表示车辆离去；‘E’表示输入结束。

**物理数据类型**

题目首先要求输入整数n，由于现在都是64位操作系统，绝大部分编译器整型数据都占4个或者8个字节，足以储存输入的整数，所以用C语言中的整型数据来定义变量。	用两个整型数据和一个字符型数据来接收输入的车辆信息。

车辆的基本信息有车辆的编号和停车时间，所以用结构体存放，停车场用栈来模拟，因为停车场大小是在一开始输入确定的，所以栈用顺序表来实现，便道用队列来模拟，因为队列有频繁的删除操作，所以队列用链表实现。

**程序的流程**

程序由四个模块组成：

（1）初始化模块：完成正整数n的输入，建立一个空栈和一个空的队列。

（2）输入模块：完成车辆信息的输入，存入整型数据a，b和字符c中，并对输入的车辆信息进行判断和根据要求作出相应的操作（即调用相应的函数）。

（3）进停车场模块：判断栈是否满，若未满，则将车辆信息压入栈内，若满了，则	   将车辆信息放入队列中。

（4）出停车场模块：从栈顶开始遍历栈将不出停车场的车辆弹出栈存入定义的数组  中，遍历到要出停车场的车辆，将该车辆弹出栈，然后将数组中的车辆信息逆出栈顺序压入栈中，最后把队列中第一辆车的信息取出来并压入栈里。

**算法的具体步骤**

模块详细设计：

1）初始化模块：定义一个整形变量n，调用C语言scanf语句来接收停车场停车位个数的输入，然后调用createStack(int n,stack0 *stack1)和createQueue(linkQueue *q)函数来完成空栈和空队列的创建，n为栈的长度，初始队列只有一个结点，栈和队列的建立用malloc()函数开辟空间。

2）输入模块：定义两个整型变量a,b和一个字符型变量c来接收车辆信息的输入，在循环条件中调用scanf语句完成车辆信息的循环输入，在输入字符E后结束输入，循环的内容是对输入的字符进行判断，若输入字符’A’，则调用arrive()函数，若输入字符’D’，则调用leave()函数。

3）进停车场模块：进停车场模块通过调用arrive(int a,int b,stack0，*stack1,linkQueue *q)函数来完成，在该模块中定义一个车辆结构体的对象car1，把车辆编号a和b存入car1中，然后对栈的长度进行判断，若栈的当前长度小于栈的总长度（n），则调用	入栈函数push(car0 car1,stack0 *stack1)函数将车辆信息压入栈内，并在屏幕上输出该车辆在停车场中的停车位置；否则，调用入队函数insertQueue(car0 car1,linkQueue *q)函数将车辆信息放入队列中，并在屏幕上输出该车辆在便道上的停车位置。

4）出停车场模块：出停车场模块通过调用leave(int a,int b,stack0 *stack1,linkQueue *q)函数来完成，首先定义一个车辆结构体指针，再定义一个车辆结构体数组，该指针指向栈顶，然后从栈顶开始遍历栈，若是不出停车场的车辆，则通过调用pop()函数将其弹出栈，并存入定义好的车辆结构体数组中，遍历到要出停车场的车辆，调用pop()函数将其弹出栈，，然后结束遍历。在屏幕上输出该车辆的停车时间和停车	费用，然后通过调用入栈函数push()把车辆结构体数组中的车辆信息逆出栈顺序一一压入栈内，最后判断队列中是否有车辆，若有，则将队列中第一辆车的时间修改为要出停车场的车辆出停车场的车的时间，然后调用push()函数将其压入栈内。

**完整代码如下：**

    #include<stdio.h>
    #include<stdlib.h>
    typedef int type;

    /*-------------------------结构体定义-------------------------*/
    
    typedef struct car			//车辆信息
    {
    	type number;
    	type time;
    }car0;
    
    typedef struct stack				//栈
    {
    	car0 *top;
    	car0 *base;
    	int length;
    }stack0;
    
    typedef struct Qnode			//队列
    {
    	car0 carw;
    	struct Qnode *next;
    }Qnode0,*QueuePrt;
    typedef struct
    {
    	QueuePrt front,rear;
    }linkQueue;
    /*-------------------------函数声明-------------------------*/
    void createStack(int n,stack0 *stack1);
    void push(car0 car1,stack0 *stack1);
    void pop(car0 *car1,stack0 *stack1);
    void createQueue(linkQueue *q);
    void insertQueue(car0 car1,linkQueue *q);
    void deleteQueue(car0 *car1,linkQueue *q);
    void arrive(int a,int b,stack0 *stack1,linkQueue *q);
    void leave(int a,int b,stack0 *stack1,linkQueue *q);
    
    /*-------------------------栈的创建，进出栈-------------------------*/
    
    void createStack(int n,stack0 *stack1)				//栈的创建
    {
    	stack1->length=n;
    	stack1->base=(car0 *)malloc(n*sizeof(car0));
    	stack1->top=stack1->base;
    }
    
    
    void push(car0 car1,stack0 *stack1)			//进栈
    {
    	car0 *c;
    
    	c=stack1->top;
    	c->number=car1.number;				////相当于*(stack1->top)=car1;
    	c->time=car1.time;
    	
    	stack1->top++;
    
    }


    void pop(car0 *car1,stack0 *stack1)			//出栈
    {
    	car0 *c;
    	
    	stack1->top--;
    	c=stack1->top;
    	car1->number=c->number;
    	car1->time=c->time;
    	
    }

    /*-------------------------队列的创建，进出队-------------------------*/
    
    void createQueue(linkQueue *q)			//队列的创建
    {
    	q->front=q->rear=(Qnode0 *)malloc(sizeof(Qnode0));
    	q->front->next=NULL;
    }
    
    void insertQueue(car0 car1,linkQueue *q)			//入队
    {
    	QueuePrt p;//节点指针P
    	p=(QueuePrt)malloc(sizeof(Qnode0));
    
    	q->rear->carw.number=car1.number;
    	q->rear->carw.time=car1.time;
    	
    	q->rear->next=p;
    	q->rear=p;//rear后移
    	
    
    }

    void deleteQueue(car0 *car1,linkQueue *q)			//出队
    {
    
    
    	if(q->front==q->rear)
    	{
    		printf("没车了  呵呵\n");
    		return;
    	}
    	car1->number=q->front->carw.number;
    	car1->time=q->front->carw.time;
    
    	q->front=q->front->next;
    	
    }
    /*-------------------------停车场-------------------------*/
    
    void arrive(int a,int b,stack0 *stack1,linkQueue *q)
    {
    	int i=0;
    	car0 car1;
    	QueuePrt p;
    	p=q->front;
    	car1.number=a;
    	car1.time=b;
    	if((stack1->top)-(stack1->base)<(stack1->length))
    	{
    		push(car1,stack1);
    		printf("%d号车停放在停车场的第%d个位置\n",a,stack1->top-stack1->base);
    	}
    	else
    	{
    		insertQueue(car1,q);
    		while(p!=(q->rear))
    		{
    			p=p->next;
    			i++;
    		}
    		printf("%d号车停放在便道的第%d个位置\n",a,i);
    	}
    }

    void leave(int a,int b,stack0 *stack1,linkQueue *q)
    {
    	car0 car1,carhehe[10];
    	car0 *p;
    	int i=0;
    	p=stack1->top-1;
    	for(i=0;i<stack1->length;i++)
    {
		if(p->number!=a)
		{
			pop(&car1,stack1);
			carhehe[i].number=car1.number;
			carhehe[i].time=car1.time;
			p--;
		}
		else
		{
			pop(&car1,stack1);
			break;
		}
	}
	printf("%d号车停放时间为：%d小时,停车费用为:%d元\n",car1.number,b-car1.time,(b-car1.time)*3);
	i--;
	while(i>=0)
	{
		car1.number=carhehe[i].number;
		car1.time=carhehe[i--].time;
		push(car1,stack1);
	}
	if(q->front!=q->rear)
	{

		deleteQueue(&car1,q);
		car1.time=b;
		push(car1,stack1);

	}
	

	
    }

    /*-------------------------主函数-------------------------*/
    int main()
    { 
    	int n,a,b;
    	char c;
    	linkQueue q;
    	stack0 stack1;
    	scanf("%d",&n);	
    	getchar();
    	createStack(n,&stack1);
    	createQueue(&q);
    
    	while(scanf("(%c,%d,%d)",&c,&a,&b) && c!='E')
    	{
    		getchar();
    		if(c=='A')
    			arrive(a,b,&stack1,&q);
    		if(c=='D')
    			leave(a,b,&stack1,&q);
    	}	
    	return 0;
    }

**运行结果**


<img src="../../../../../img/blogs/parking lot/01.png">

<img src="../../../../../img/blogs/parking lot/02.png">