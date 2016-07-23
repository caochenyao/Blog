---
title: Linux信号量semop函数中的sem_UNDO
date: 2016-07-16 20:32:14
categories: Linux
tags: Linux系统篇
---
一、什么是信号量
为了防止出现因多个程序同时访问一个共享资源而引发的一系列问题，我们需要一种方法，它可以通过生成并使用令牌来授权，在任一时刻只能有一个执行线程访问代码的临界区域。临界区域是指执行数据更新的代码需要独占式地执行。而信号量就可以提供这样的一种访问机制，让一个临界区同一时间只有一个线程在访问它，也就是说信号量是用来调协进程对共享资源的访问的。
信号量是一个特殊的变量，程序对其访问都是原子操作，且只允许对它进行等待（即P(信号变量))和发送（即V(信号变量))信息操作。最简单的信号量是只能取0和1的变量，这也是信号量最常见的一种形式，叫做二进制信号量。而可以取多个正整数的信号量被称为通用信号量。这里主要讨论二进制信号量。
 
二、信号量的工作原理
由于信号量只能进行两种操作等待和发送信号，即P(sv)和V(sv),他们的行为是这样的：
P(sv)：如果sv的值大于零，就给它减1；如果它的值为零，就挂起该进程的执行
V(sv)：如果有其他进程因等待sv而被挂起，就让它恢复运行，如果没有进程因等待sv而挂起，就给它加1
举个例子，就是两个进程共享信号量sv，一旦其中一个进程执行了P(sv)操作，它将得到信号量，并可以进入临界区，使sv减1。而第二个进程将被阻止进入临界区，因为当它试图执行P(sv)时，sv为0，它会被挂起以等待第一个进程离开临界区域并执行V(sv)释放信号量，这时第二个进程就可以恢复执行。
 
三、Linux中信号量表示的资源操作
Linux提供了一组精心设计的信号量接口来对信号进行操作，它们声明在头文件sys/sem.h中。
函数semop用以操作一个信号量集，实质是通过修改sem_op指定对资源要进行的操作，semctl函数则是对信号量本身的值进行操作，可以修改信号量的值或者删除一个信号量，这是他们二者容易混淆的地方:
	
	int semop(int semid,struct sembuf semoparray[],size_t nops);
semid是通过semget函数返回的一个信号量集标识符ID，nops标明了参数semoparray所指向数组中的元素个数。semoparray是一个结构数组指针。结构体struct sembuf 用来说明要执行的操作。

	struct sembuf{
		unsigned short sem_num; //对应信号量集中的某一个资源
		short sem_op;  		    //指明所要执行的操作
		short sem_flg;		    //函数semop的行为
	}
在sembuf结构中，sem_num是相对应的信号量集中的某一个资源，所以它的值是一个从 0--相对应的信号量集的资源总数（ipc_perm.sem_nsems)之间的整数。
sem_op指明想要进行的操作，sem_flg说明函数semop的行为。sem_op的值是一个整数。
sem_op：
（1）> 0：释放相应的资源数，如果有两个信号量，释放信号量1，则其semval+1，对信号量这个无名结构体的操作，可以通过下面介绍的semctl函数实现。
（2）0：进程阻塞直到信号量的相应值为0，当信号量已经为0，函数立即返回。
（3）< 0：请求sem_op的绝对值的资源数。
sem_flg 参数：
该参数可设置为 IPC_NOWAIT 或 SEM_UNDO 两种状态。只有将 sem_flg 指定为 SEM_UNDO 标志后，semadj （所指定信号量针对调用进程的调整值）才会更新。   此外，如果此操作指定SEM_UNDO，系统更新过程中会撤消此信号灯的计数（semadj）。此操作可以随时进行---它永远不会强制等待的过程。调用进程必须有改变信号量集的权限。
sem_flg公认的标志是 IPC_NOWAIT 和 SEM_UNDO。如果操作指定SEM_UNDO，它将会自动撤消该进程终止时。
在标准操作程序中的操作是在数组的顺序执行、原子的，那就是，该操作要么作为一个完整的单元，要么不。如果不是所有操作都可以立即执行的系统调用的行为取决于在个人sem_flg领域的IPC_NOWAIT标志的存在。