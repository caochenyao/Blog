---
title: fork函数
date: 2016-06-25 14:28:19
categories: Linux
tags: Linux系统篇
---
一、初步认识fork函数
一个进程，包括代码、数据和分配给进程的资源。fork（）函数通过系统调用创建一个与原来进程几乎完全相同的进程，也就是两个进程可以做完全相同的事，但如果初始参数或者传入的变量不同，两个进程也可以做不同的事。
一个进程调用fork（）函数后，系统先给新的进程分配资源，例如存储数据和代码的空间。然后把原来的进程的所有值都复制到新的新进程中，只有少数值与原来的进程的值不同。相当于克隆了一个自己。
来看一个例子：

	  1 #include<stdio.h>
	  2 #include<unistd.h>
	  3 #include<stdlib.h>
	  4 
	  5 int main()
	  6 {
	  7     int count = 100;
	  8     pid_t pid = fork();
	  9 
	 10     if(pid == -1)
	 11     {
	 12         perror("fork error");
	 13         exit(1);
	 14     }
	 15     else if(pid == 0)
	 16     {
	 17         count++;
	 18         printf("child:%d,pid:%d,ppid:%d,count:%d\n",pid,getpid(),getppid(),count);
	 19     }
	 20     else
	 21     {
	 22         printf("parent:%d,pid:%d,ppid:%d,count:%d\n",pid,getpid(),getppid(),count);
	 23         sleep(5);
	 24     }
	 25 
	 26     return 0;
	 27 }
其结果如图所示：
![1](http://o6lb63nu0.bkt.clouddn.com/fork_1.png)
如果fork成功，子进程返回的值是0，父进程返回的值是子进程的进程号。
而且大家可以发现，在子进程中count进行了加一，但父进程的结果显示父进程的count并没有加一。那是因为父进程创建子进程时，子进程会获得父进程数据空间、堆和栈的副本(主要是数据结构的副本)，也就是说父进程会向子进程拷贝自己的数据空间、堆和栈(主要是数据结构)。所以两者的这些部分是独立的，而两者共享的是正文段(及代码段)。所以在子进程中count++对父进程的count没有影响。

二、深入了解fork函数
1.关于子进程通过getppid()获取的父进程进程号为1的问题
了解了fork函数的基本功能，再来看看这段代码：

	  1 #include<stdio.h>
	  2 #include<unistd.h>
	  3 #include<stdlib.h>
	  4 
	  5 int main()
	  6 {
	  7     int count = 100;
	  8     pid_t pid = fork();
	  9 
	 10     if(pid == -1)
	 11     {
	 12         perror("fork error");
	 13         exit(1);
	 14     }
	 15     else if(pid == 0)
	 16     {
	 17         count++;
	 18         printf("child:%d,pid:%d,ppid:%d,count:%d\n",pid,getpid(),getppid(),count);
	 19     }
	 20     else
	 21     {
	 22         printf("parent:%d,pid:%d,ppid:%d,count:%d\n",pid,getpid(),getppid(),count);
	 23     }
	 24 
	 25     return 0;
	 26 }
结果如下图所示：
![2](http://o6lb63nu0.bkt.clouddn.com/fork_2.png)
注意图中所圈的内容，子进程获取的父进程进程号为1，但其父进程的进程号却为6059，这又是什么原因呢？
仔细观察代码会发现虽然这个代码与之前的代码很相像，但在父进程这里少了一个sleep函数，那么说明少了这个sleep函数造成了这种现象。因为少了sleep函数，使得父进程在执行完后并没有等待子进程结束就急急忙忙退出了，此时子进程就变成了一个孤儿进程，这个孤儿进程由init接管，所以这里才显示子进程通过getppid()得到的父进程进程号为1。
2.关于循环fork进程
来看一下这段代码：

	  1 #include<stdio.h>
	  2 #include<unistd.h>
	  3 #include<stdlib.h>
	  4 
	  5 int main()
	  6 {
	  7     int i = 0;
	  8     pid_t pid;
	  9 
	 10     for(;i<2;i++)
	 11     {
	 12         pid = fork();
	 13         if(pid == -1)
	 14         {
	 15             perror("fork error");
	 16             exit(1);
	 17         }
	 18         else if(pid == 0)
	 19         {
	 20             printf("child:%d,pid:%d,ppid:%d,i:%d\n",pid,getpid(),getppid(),i);
	 21         }
	 22         else
	 23         {
	 24             printf("parent:%d,pid:%d,ppid:%d,i:%d\n",pid,getpid(),getppid(),i);
	 25             sleep(5);
	 26         }
	 27     }
	 28 
	 29     return 0;
	 30 }
结果如下图示：
![3](http://o6lb63nu0.bkt.clouddn.com/fork_3.png)
一定有六条信息，分析如下图：
![4](http://o6lb63nu0.bkt.clouddn.com/fork_4.png)
最初只有进程号为7003的进程，i=0时，该进程作为父进程创建子进程，进程号为7004；i=1时，父进程再次创建新的子进程，进程号为7006，而子进程也作为父进程创建出进程号为7005的子进程，所以最后的结果为六条信息。

