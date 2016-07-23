---
title: 匿名管道与命名管道
date: 2016-07-15 21:08:40
categories: Linux
tags: Linux系统篇
---
之前的博客我们已经了解了管道这一概念，在这篇博客中我们将探讨匿名管道与命名管道。
管道(Pipe)实际是用于进程间通信的一段共享内存. 创建管道的进程称为管道服务器，连接到一个管道的进程为管道客户机。
一、匿名管道
匿名管道，顾名思义，其没有一个标识符来标识它，所以匿名管道只能用于有血缘关系的进程间通信
下面是匿名管道的一段代码：

	  1 #include<unistd.h>
	  2 #include<stdio.h>
	  3 #include<errno.h>
	  4 #include<sys/types.h>
	  5 #include<sys/wait.h>
	  6 #include<string.h>
	  7 
	  8 int main()
	  9 {
	 10     int pipe_fd[2];
	 11     if(pipe(pipe_fd) < 0)
	 12     {
	 13         perror("pipe");
	 14     }
	 15     
	 16     pid_t pid = fork();
	 17     if(pid == 0)
	 18     {
	 19         //child
	 20         close(pipe_fd[0]);
	 21         const char* msg = "I'm a child\n";
	 22         int count = 5;
	 23         int i = 1;
	 24         while(1)
	 25         {
	 26             write(pipe_fd[1],msg,strlen(msg));
	 27             if(count-- == 0)
	 28                 break;
	 29             sleep(1);
	 30             printf("write data:%d\n",i++);
	 31         }
	 32         close(pipe_fd[1]);
	 33     }
	 34     else
	 35     {
	 36         //father
	 37         close(pipe_fd[1]);
	 38         char buf[128];
	 39         while(1)
	 40         {
	 41             memset(buf,'\0',sizeof(buf));
	 42             ssize_t s = read(pipe_fd[0],buf,sizeof(buf)-1);
	 43             if(s > 0)
	 44             {
	 45                 printf("client->server:%s",buf);
	 46             }
	 47             else if(s == 0)
	 48             {
	 49                 printf("read pipe erro!!!\n");
	 50                 break;
	 51             }
	 52             else
	 53             {
	 54                 break;
	 55             }
	 56         }
	 57         close(pipe_fd[0]);
	 58 
	 59         int status = 0;
	 60         pid_t ret = waitpid(pid,&status,0);
	 61         if(ret == pid)
	 62         {
	 63             printf("Wait child succeed...\n");
	 64         }
	 65     }
	 66 
	 67     return 0;
	 68 }
运行结果如图：
![1](http://o6lb63nu0.bkt.clouddn.com/namepipe_01.png)
二、命名管道
命名管道(Named Pipes)是在管道服务器和一台或多台管道客户机之间进行单向或双向通信的一种命名的管道。一个命名管道的所有实例共享同一个管道名，但是每一个实例均拥有独立的缓存与句柄，并且为客户——服务通信提供有一个分离的管道。实例的使用保证了多个管道客户能够在同一时间使用同一个命名管道。
命名管道具有很好的使用灵活性，表现在：
1) 既可用于本地，又可用于网络。
2) 可以通过它的名称而被引用。
3) 支持多客户机连接。
4) 支持双向通信。
5) 支持异步重叠I/O操作。

下面是命名管道的代码部分：
客户端：

	  1 #include<stdio.h>
	  2 #include<sys/types.h>
	  3 #include<sys/stat.h>
	  4 #include<fcntl.h>
	  5 #include<string.h>
	  6 
	  7 int main()
	  8 {
	  9     int fd = open("./myfifo",O_WRONLY);
	 10     if(fd < 0)
	 11     {
	 12         perror("open");
	 13         return 1;
	 14     }
	 15 
	 16     char buf[128];
	 17     while(1)
	 18     {
	 19         memset(buf,'\0',sizeof(buf));
	 20         printf("Please Enter# ");
	 21         fflush(stdout);
	 22         ssize_t s = read(1,buf,sizeof(buf));
	 23         if(s > 0)
	 24         {
	 25             buf[s-1]='\0';
	 26             write(fd,buf,strlen(buf));
	 27         }
	 28     }
	 29     close(fd);
	 30 
	 31     return 0;
	 32 }
服务器：

	  1 #include<stdio.h>
	  2 #include<sys/types.h>
	  3 #include<sys/stat.h>
	  4 #include<fcntl.h>
	  5 #include<string.h>
	  6 
	  7 int main()
	  8 {
	  9     if(mkfifo("./myfifo",S_IFIFO | 0644) < 0)
	 10     {
	 11         perror("mkfifo");
	 12         return 1;
	 13     }
	 14 
	 15     int fd = open("./myfifo",O_RDONLY);
	 16     if(fd < 0)
	 17     {
	 18         perror("open");
	 19         return 2;
	 20     }
	 21 
	 22     char buf[128];
	 23     while(1)
	 24     {
	 25         memset(buf,'\0',sizeof(buf));
	 26         ssize_t s = read(fd,buf,sizeof(buf)-1);
	 27         if(s > 0)
	 28         {
	 29             printf("client# %s\n",buf);
	 30         }
	 31         else
	 32         {
	 33             break;
	 34         }
	 35     }
	 36 
	 37     close(fd);
	 38     return 0;
	 39 }
运行结果如图：
![2](http://o6lb63nu0.bkt.clouddn.com/namepipe_02.png)
