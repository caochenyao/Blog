---
title: FILE结构体
date: 2016-06-24 10:20:52
categories: Linux
tags: Linux系统篇
---
struct file结构体定义在/linux/include/linux/fs.h(Linux 2.6.11内核)中，其原型是：

	struct file {
	        /*
	         * fu_list becomes invalid after file_free is called and queued via
	         * fu_rcuhead for RCU freeing
	         */
	        union {
	                struct list_head        fu_list;
	                struct rcu_head         fu_rcuhead;
	        } f_u;
	        struct path             f_path;
	#define f_dentry        f_path.dentry
	#define f_vfsmnt        f_path.mnt
	        const struct file_operations    *f_op;
	        atomic_t                f_count;
	        unsigned int            f_flags;
	        mode_t                  f_mode;
	        loff_t                  f_pos;
	        struct fown_struct      f_owner;
	        unsigned int            f_uid, f_gid;
	        struct file_ra_state    f_ra;
	        unsigned long           f_version;
	#ifdef CONFIG_SECURITY
	        void                    *f_security;
	#endif
	        /* needed for tty driver, and maybe others */
	        void                    *private_data;
	#ifdef CONFIG_EPOLL
	        /* Used by fs/eventpoll.c to link all the hooks to this file */
	        struct list_head        f_ep_links;
	        spinlock_t              f_ep_lock;
	#endif /* #ifdef CONFIG_EPOLL */
	        struct address_space    *f_mapping;
	};
文件结构体代表一个打开的文件，系统中的每个打开的文件在内核空间都有一个关联的struct file。它由内核在打开文件时创建，并传递给在文件上进行操作的任何函数。在文件的所有实例都关闭后，内核释放这个数据结构。在内核创建和驱动源码中，struct file的指针通常被命名为file或filp。一下是对结构中的每个数据成员的解释：
一、

	union {
	    struct list_head fu_list;
	    struct rcu_head rcuhead;
	}f_u;
其中的struct list_head定义在 linux/include/linux/list.h中，原型为：

	struct list_head {
	        struct list_head *next, *prev;
	};
用于通用文件对象链表的指针。

	struct rcu_head定义在linux/include/linux/rcupdate.h中，其原型为：
	/**
	* struct rcu_head - callback structure for use with RCU
	* @next: next update requests in a list
	* @func: actual update function to call after the grace period.
	*/
	struct rcu_head {
	        struct rcu_head *next;
	        void (*func)(struct rcu_head *head);
	};
二、

	struct path             f_path;
被定义在linux/include/linux/namei.h中，其原型为：

	struct path {
	        struct vfsmount *mnt;
	        struct dentry *dentry;
	};
在早些版本的内核中并没有此结构，而是直接将path的两个数据成员作为struct file的数据成员

	struct vfsmount *mnt的作用是指出该文件的已安装的文件系统，
	struct dentry *dentry是与文件相关的目录项对象。
三、

	const struct file_operations    *f_op;
被定义在linux/include/linux/fs.h中，其中包含着与文件关联的操作，如:

	loff_t (*llseek) (struct file *, loff_t, int);
	ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
	ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
当打开一个文件时，内核就创建一个与该文件相关联的struct file结构，其中的*f_op就指向的是具体对该文件进行操作的函数。例如用户调用系统调用read来读取该文件的内容时，那么系统调用read最终会陷入内核调用sys_read函数，而sys_read最终会调用于该文件关联的struct file结构中的f_op->read函数对文件内容进行读取。
四、

	atomic_t                f_count;
atomic_t被定义为：

	typedef struct { volatile int counter; } atomic_t;
volatile修饰字段告诉gcc不要对该类型的数据做优化处理，对它的访问都是对内存的访问，而不是对寄存器的访问。 
本质是int类型，之所以这样写是让编译器对基于该类型变量的操作进行严格的类型检查。此处f_count的作用是记录对文件对象的引用计数，也即当前有多少个进程在使用该文件。
五、

	unsigned int            f_flags;
当打开文件时指定的标志，对应系统调用open的int flags参数。驱动程序为了支持非阻塞型操作需要检查这个标志。
六、

	mode_t                  f_mode;
对文件的读写模式，对应系统调用open的mod_t mode参数。如果驱动程序需要这个值，可以直接读取这个字段。
mod_t被定义为：

	typedef unsigned int __kernel_mode_t;
	typedef __kernel_mode_t         mode_t;
七、

	loff_t                  f_pos;
当前的文件指针位置，即文件的读写位置。
loff_t被定义为：

	typedef long long       __kernel_loff_t;
	typedef __kernel_loff_t         loff_t;
八、

	struct fown_struct      f_owner;
	struct fown_struct在linux/include/linux/fs.h被定义，原型为:
	struct fown_struct {
	        rwlock_t lock;          /* protects pid, uid, euid fields */
	        struct pid *pid;        /* pid or -pgrp where SIGIO should be sent */
	        enum pid_type pid_type; /* Kind of process group SIGIO should be sent to */
	        uid_t uid, euid;        /* uid/euid of process setting the owner */
	        int signum;             /* posix.1b rt signal to be delivered on IO */
	};
该结构的作用是通过信号进行I/O时间通知的数据。
九、

	unsigned int            f_uid, f_gid;
标识文件的所有者id，所有者所在组的id.
十、

	struct file_ra_state    f_ra;
	struct file_ra_state结构被定义在/linux/include/linux/fs.h中，原型为：
	struct file_ra_state {
	        pgoff_t start;                  /* where readahead started */
	        unsigned long size;             /* # of readahead pages */
	        unsigned long async_size;       /* do asynchronous readahead when
	                                           there are only # of pages ahead */
	        unsigned long ra_pages;         /* Maximum readahead window */
	        unsigned long mmap_hit;         /* Cache hit stat for mmap accesses */
	        unsigned long mmap_miss;        /* Cache miss stat for mmap accesses */
	        unsigned long prev_index;       /* Cache last read() position */
	        unsigned int prev_offset;       /* Offset where last read() ended in a page */
	};
文件预读状态，文件预读算法使用的主要数据结构，当打开一个文件时，f_ra中出了perv_page(默认为－1)和ra_apges(对该文件允许的最大预读量)这两个字段外，其他的所有西端都置为0。
十一、

	unsigned long           f_version;
记录文件的版本号，每次使用后都自动递增。
十二、

	#ifdef CONFIG_SECURITY
	        void                    *f_security;