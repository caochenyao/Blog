---
title: Linux缓冲机制——进度条的简单实现
date: 2016-06-06 08:35:25
categories: Linux
tags: Linux系统篇
---
一、关于printf中'\n'刷新缓冲区的问题
   首先看这样的一段代码：

	  1 #include<stdio.h>
	  2 #include<unistd.h>
	  3 
	  4 int main()
	  5 {
	  6     printf("hello world\n");
	  7     sleep(3);                                                
	  8     return 0;
	  9 }
   运行这段程序的结果是输出"hello world"，再睡眠3秒；
   而再运行这一段代码，只有一点小小的差别，结果却不同：

	  1 #include<stdio.h>
	  2 #include<unistd.h>
	  3 
	  4 int main()
	  5 {
	  6     printf("hello world"); \\去掉了换行符'\n'
	  7     sleep(3);                                                
	  8     return 0;
	  9 }
   这段代码与上一段代码的差别在于少了一个换行符'\n'，但是结果却不同，该代码的运行结果是先睡
   眠了3秒，再输出"hello world"；
   那么问题来了，为什么少了一个换行符会造成两种不同的运行结果，因为printf是一个行缓冲函数，
   先写到缓冲区，满足条件后，才将缓冲区刷新到对应文件中，而换行符起到一个刷新缓冲区的作用。
   而刷新缓冲区的条件：
   1.缓冲区填满
   2.写入的字符中有'\n' '\r'
   3.调用fflush手动刷新缓冲区
   4.调用scanf要从缓冲区中读取数据时，也会将缓冲区内的数据刷新
   5.程序结束
二、实现进度条
   实现代码：

	  1 #include<stdio.hiiiiiii>                                                                                              
	  2 #include<string.h>
	  3 #include<unistd.h>
	  4 
	  5 void proc()
	  6 {
	  7     int rate = 0;
	  8     char bar[102];                      //进度条
	  9     char icon[5] = "|/\\-";             //动画效果-旋转
	 10     memset(bar,'\0',sizeof(char)*102);
	 11     while(rate < 101)
	 12     {
	 13          bar[rate]= '=';
	 14          printf("[%-101s][%d%%][%c]\r",bar,rate,icon[rate%4]);
	 15          fflush(stdout);                //刷新缓冲区
	 16          usleep(100000);                //睡眠时间0.1s
	 17          rate++;
	 18     }
	 19     printf("\n");
	 20 }
	 21 
	 22 int main()
	 23 {
	 24     proc();
	 25     return 0;
	 26 }
   编写makefile：
    
	 1 proc:proc.c
	 2     gcc -o proc proc.c                                       
	 3 .PHONY:clean
	 4 clean:
	 5     rm -rf proc
   运行结果
   ![1](http://o6lb63nu0.bkt.clouddn.com/task1.png)
