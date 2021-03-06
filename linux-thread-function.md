---
title: 线程安全和可重入函数
date: 2016-07-17 15:30:24
categories: Linux
tags: Linux系统篇
---
一、线程安全与可重入函数
(一)线程安全
线程安全的概念比较直观。一般说来，一个函数被称为线程安全的，当且仅当被多个并发线程反复调用时，它会一直产生正确的结果。
1)确保线程安全： 
要确保函数线程安全，主要需要考虑的是线程之间的共享变量。属于同一进程的不同线程会共享进程内存空间中的全局区和堆，而私有的线程空间则主要包括栈和寄 存器。因此，对于同一进程的不同线程来说，每个线程的局部变量都是私有的，而全局变量、局部静态变量、分配于堆的变量都是共享的。在对这些共享变量进行访 问时，如果要保证线程安全，则必须通过加锁的方式。 
2)线程不安全的后果： 
线程不安全可能导致的后果是显而易见的——共享变量的值由于不同线程的访问，可能发生不可预料的变化，进而导致程序的错误，甚至崩溃。
(二)可重入函数
可重入的概念基本没有比较正式的完整解释，多数的文档都只是说明什么样的情况才能保证函数可重入，但没有完整定义。按照Wiki上的说法，“A computer program or routine is described as reentrant if it can be safely executed concurrently; that is, the routine can be re-entered while it is already running.”根据笔者的经验，所谓“重入”，常见的情况是，程序执行到某个函数foo()时，收到信号，于是暂停目前正在执行的函数，转到信号处理 函数，而这个信号处理函数的执行过程中，又恰恰也会进入到刚刚执行的函数foo()，这样便发生了所谓的重入。此时如果foo()能够正确的运行，而且处 理完成后，之前暂停的foo()也能够正确运行，则说明它是可重入的。 
1)确保可重入： 
要确保函数可重入，需满足以下几个条件： 
1、不在函数内部使用静态或全局数据 
2、不返回静态或全局数据，所有数据都由函数的调用者提供。 
3、使用本地数据，或者通过制作全局数据的本地拷贝来保护全局数据。 
4、不调用不可重入函数。 
2)不可重入的后果： 
不可重入的后果主要体现在象信号处理函数这样需要重入的情况中。如果信号处理函数中使用了不可重入的函数，则可能导致程序的错误甚至崩溃。
二、线程安全与可重入函数的联系与区别
可重入与线程安全并不等同。一般说来，可重入的函数一定是线程安全的，但反过来不一定成立。
（1）线程安全是在多个线程情况下引发的，而可重入函数可以在只有一个线程的情况下来说。
（2）线程安全不一定是可重入的，而可重入函数则一定是线程安全的。
（3）如果一个函数中用到了全局或静态变量，那么它不是线程安全的，也不是可重入的。
（4）如果将对临界资源的访问加上锁，则这个函数是线程安全的，但如果这个重入函数若锁还未释放则会产生死锁，因此是不可重入的。
（5）线程安全函数能够使不同的线程访问同一块地址空间，而可重入函数要求不同的执行流对数据的操作互不影响使结果是相同的。
（6）如果我们对它加以改进，在访问全局或静态变量时使用互斥量或信号量等方式加锁，则可以使它变成线程安全的，但此时它仍然是不可重入的，因为通常加锁方式是针对不同线程的访问，而对同一线程可能出现问题。 
（7）如果将函数中的全局或静态变量去掉，改成函数参数等其他形式，则有可能使函数变成既线程安全，又可重入。
