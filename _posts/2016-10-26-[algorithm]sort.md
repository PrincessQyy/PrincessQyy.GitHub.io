---
layout:     post
title:      "【算法设计】排序算法"
date:       2016-10-26 14:07:00
author:     "Yuki"
---

**问题描述**

你要用C或python/JAVA/C++应用插入排序、归并排序、堆排序和快速排序去实现计算字数的程序，这些程序的目的是计算输入单词的总数量，以下给出样例。

输入
输入包含一系列被空格或标点、分行符（在一句话的末尾）分开的单词、数字和符号，请注意计字器要计数单词（忽视其他的数字、空格等），以出现次数（从最多的到最少的）和字典序（从a到z）排序。

对于本问题，我们认为单词是大小写敏感的（大写单词被放在小写单词前面），并且一个合成词或一个缩写应该被视为一个单词，比如“on-chip”是一个单词，“it’s”也是一个单词。

输出

输出包含一个数字，意思是总的单词数，接下来是一系列输出，每行包括一个单词（所有单词以字典序排列），它总共出现的次数，和它在文中出现的次序（即，该单词是文中第i个，忽视其他数字和空格）。比如，单词IC出现了一次，是文中第6个单词。注意，你不需要对每个单词出现次序排序。

**插入排序、归并排序、快速排序：**


    import os
    import re
    #插入排序
    def insertSort(key_words_list,count_list):
    temp1=count_list[0]
    temp2=key_words_list[0]

    for i in range(1,len(count_list)):
        if(count_list[i]>=count_list[i-1]):
            temp1=count_list.pop(i)
            temp2=key_words_list.pop(i)
            j=i-1
            #找插入位置操作
            while(j>=0 and (count_list[j]<temp1 or (count_list[j]==temp1 and key_words_list[j]>temp2))):
                    j-=1
               
            count_list.insert(j+1,temp1)
            key_words_list.insert(j+1,temp2)
    return key_words_list,count_list

	#快速排序
    def quickSort(key_words_list,count_list,left,right):
	    i=left
	    j=right
	    if(i>=j):
	        return key_words_list,count_list
	
	    key1=count_list[i]
	    key2=key_words_list[i]
	    
	    while i < j:
	        if(count_list[j]>key1 or (count_list[j]==key1 and key_words_list[j]<key2)):
	            temp1=count_list.pop(j)
	            temp2=key_words_list.pop(j)
	            count_list.insert(i,temp1)
	            key_words_list.insert(i,temp2)
	            i+=1
	        else:
	            j-=1
        

	    quickSort(key_words_list,count_list,left,i-1)
	    quickSort(key_words_list,count_list,i+1,right)
	         
	            
	           
	    return key_words_list,count_list

    #归并排序
    def mergeSort(key_words_list,count_list):
    #区间长度大于1才进行排序
	    if(len(key_words_list)>1):
	        mid=(len(key_words_list))//2
	        words_left=key_words_list[:mid]             
	        words_right=key_words_list[mid:]
	        count_left=count_list[:mid]
	        count_right=count_list[mid:]
	        #递归左切片和右切片
	        words_left,count_left=mergeSort(words_left,count_left)
	        words_right,count_right=mergeSort(words_right,count_right)
	        
	        return(merge(words_left,words_right,count_left,count_right))
	    else:
	        return key_words_list,count_list


    def merge(words_left,words_right,count_left,count_right):
	    words=[]
	    count=[]
	    i=0
	    j=0
	
	    while i<len(count_left) and j<len(count_right):
	        #排次数大小
	        if count_left[i]>count_right[j]:  
	            count.append(count_left[i])
	            words.append(words_left[i])
	            i+=1
	        #排字母顺序
	        elif (count_left[i]==count_right[j] and words_left[i]<words_right[j]):
	            count.append(count_left[i])
	            words.append(words_left[i])
	            i+=1
	        else:  
	            words.append(words_right[j])
	            count.append(count_right[j])
	            j+=1  
	
	    #将剩余数据追加到words和count中
	    words+=words_left[i:]  
	    words+=words_right[j:]
	
	    count+=count_left[i:]  
	    count+=count_right[j:]
	    return words,count


    #将读取的所有词汇提取到列表里
    file_in=open('test_input.dat','rb')
    words_list=file_in.read().decode('utf-8')
    file_in.close()
    

    #利用正则表达式提取符合要求的单词
    words_list = re.findall(r'e\.g.|i\.e.|i\.e|[a-zA-Z]+[a-zA-Z\-\'\/]*', words_list)

    #计算所有单词数量
    total=len(words_list)
    
    #将出现过的单词（不重复）和单词次数统计到列表key_word_list和count_list里
    key_word_list=[]
    count_list=[]

    for each_word in words_list:
    	if (each_word in key_word_list):
        	count_list[key_word_list.index(each_word)]+=1
    	else:
        	key_word_list.append(each_word)
        	count_list.append(1)

    #调用函数
    key_word_list,count_list=insertSort(key_word_list,count_list)
	key_word_list,count_list=mergeSort(key_word_list,count_list)
	key_word_list,count_list=quickSort(key_word_list,count_list)

    #将单词总数、单词出现的次数、单词位置存到文件中
    file_out=open("output.dat","w")
    file_out.write("%d\n" % total)
    
    for i in range(len(key_word_list)):
	    file_out.write("%s %d" % (key_word_list[i],count_list[i]))
	    t=0
	    for j in words_list:
	        t+=1
	        if(j==key_word_list[i]):
	            file_out.write(" %d" % t)
	    file_out.write("\n\n")
	file_out.close()   `


**堆排序**

    import os
    import re

    #自顶向下堆化
    def fixDown(count_list,key_words_list,k,n):   
	    N=n-1  
	    while 2*k<=N:  
	        j=2*k
	        #选出左右孩子节点中更小的那个
	        if (j<N) and (count_list[j]>count_list[j+1])or(count_list[j]==count_list[j+1] and key_words_list[j]<key_words_list[j+1]): 
	                j+=1  
	        if (count_list[k]>count_list[j])or(count_list[k]==count_list[j] and key_words_list[k]<key_words_list[j]):   
	                count_list[k],count_list[j]=count_list[j],count_list[k]
	                key_words_list[k],key_words_list[j]=key_words_list[j],key_words_list[k]
	                k=j
	        else:  
	            break
        
    #堆排序
    def heapSort(count_list,key_words_list):  
    	n=len(count_list)-1
	    #堆的初始化
	    for i in range(n//2,0,-1):  
	        fixDown(count_list,key_words_list,i,len(count_list))
	    #构建大顶堆
	    while n>1:  
	        count_list[1],count_list[n]=count_list[n],count_list[1]
	        key_words_list[1],key_words_list[n]=key_words_list[n],key_words_list[1]
	        fixDown(count_list,key_words_list,1,n)  
	        n-=1  
	    return count_list[1:],key_words_list[1:] 


    #将读取的所有词汇提取到列表里
    file_in=open('test_input.dat','rb')
    words_list=file_in.read().decode('utf-8')
    file_in.close()
    

    #利用正则表达式提取符合要求的单词
    words_list = re.findall(r'e\.g.|i\.e.|i\.e|[a-zA-Z]+[a-zA-Z\-\'\/]*', words_list)

    #计算所有单词数量
    total=len(words_list)

    #将出现过的单词（不重复）和单词次数统计到列表key_word_list和count_list里
    key_word_list=[]
    count_list=[]

    for each_word in words_list:
	    if (each_word in key_word_list):
	        count_list[key_word_list.index(each_word)]+=1
	    else:
	        key_word_list.append(each_word)
	        count_list.append(1)

    #为了让下标变成层序遍历的顺序，往列表前加一个元素
    key_word_list.insert(0,'zzz')
    count_list.insert(0,-1)
    #调用函数
    count_list,key_word_list=heapSort(count_list,key_word_list)
    
    #将单词总数、单词出现的次数、单词位置存到文件中
    file_out=open("output.dat","w")
    file_out.write("%d\n" % total)


    for i in range(len(key_word_list)):
	    file_out.write("%s %d" % (key_word_list[i],count_list[i]))
	    t=0
	    for j in words_list:
	        t+=1
	        if(j==key_word_list[i]):
	            file_out.write(" %d" % t)
	    file_out.write("\n\n")
	file_out.close() 

**运行结果的部分截图**


<img src="../../../../../img/blogs/sort/01.png">