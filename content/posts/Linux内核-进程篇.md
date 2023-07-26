---
title: "Linux内核-进程篇"
date: 2023-07-21
draft: false
tags: ["Linux","system"] 
---

## 进程和线程

进程定义为程序执行的一个实例，也是操作系统分配资源的对象实体。

## 进程描述符 TaskStruct结构

![image-20230612171305862](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211737407.png)



## 进程状态

`可运行状态- task running`

`可中断的等待状态- task interruptible`

<img src="https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211737047.png" alt="image-20230612171518628" style="zoom: 50%;" />

`不可中断的等待状态- task uninterruptible`

<img src="https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211737204.png" alt="image-20230612171534570" style="zoom: 50%;" />

`暂停状态- task_stopped`

<img src="https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211738569.png" alt="image-20230612171550684" style="zoom:50%;" />

`跟踪状态- task_traced`

<img src="https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211738204.png" alt="image-20230612171605734" style="zoom:50%;" />

两种EXIT_STATE状态

`僵死状态- EXIT_ZOMBIE`

<img src="https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211738539.png" alt="image-20230612171810588" style="zoom:50%;" /><img src="https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211738355.png" alt="image-20230612171820766" style="zoom:50%;" />



`僵死撤销状态- EXIT_DEAD`

<img src="https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211738160.png" alt="image-20230612171857858" style="zoom:50%;" />

## 进程标识

进程和进程描述符之间有严格的对应关系，用户使用PID对进程进行标识。

PID顺序编号，新进程的PID一般为前一个进程的PID + 1。

PID有上限（缺省为32767），可以通过向`/proc.sys/kernel/pid_max`写入值来减小上限。

到达上限后循环使用最小的值。

内核中使用位图来进行管理，在新版的Linux内核中有了更高效的方式：radix-tree（待查证：TODO）。

多线程应用中所有的线程具有和领头线程相同的PID，存储在线程的tgid字段中。领头线程的PID和tgid的值相同。

## pidhash表

内核需要一种方式来组织PID和TaskStruct的对应关系。

Linux使用了四个散列表：

PID、tgid、pgrp、session

![image-20230725102703845](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307251027522.png)

通过PID散列表的形式可以便捷的删除一个线程组中的所有线程。

![image-20230725103043061](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307251030761.png)

## 进程描述符与内核栈

<img src="https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211738057.png" alt="image-20230612173301075" style="zoom:50%;" />

大小为8K，task_struct 和 thread_info 可以很方便的互相寻址

<img src="https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211738477.png" alt="image-20230612173316818" style="zoom:50%;" />

## 进程的组织方式

### 等待队列

等待队列中的元素表示等待某一条件的进程，当条件变为真时，由内核对对待队列中的进程进行唤醒。

等待队列的数据结构

```
struct wait_queue_entry {
	unsigned int		flags;
	void			*private;
	wait_queue_func_t	func;
	struct list_head	entry;
};

struct wait_queue_head {
	spinlock_t		lock;
	struct list_head	head;
};

typedef struct wait_queue_head wait_queue_head_t;
```

flags表示进程的唤醒方式，这里是导致惊群现象的根本原因：

flag == 1 互斥进程，只唤醒一个进程

flag == 0 非互斥进程，会唤醒等待链表中所有进程

func代表唤醒进程的方式，回调函数



在将进程插入等待链表时，调用add_wait_queue插入队首，add_wait_queue_exclusive插入队尾，并设置相应的flag。



使进程进入等待的几种方式：

- sleep_on
- interruptible_sleep_on
- sleep_on_timeout, interruptible_sleep_on_timeout
- prepare_to_wait, prepare_to_wait_exclusive, finish_wait
- wait_event, wait_event_interruptible

注意区分两种状态 task interruptible 和 task uninterruptible



唤醒进程的一些宏：

- wake_up
- wake_up_nr
- wake_up_all
- wake_up_interruptible
- ...

wake_up宏等价于：

![image-20230725104717704](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307251047627.png)

当扫描到互斥进程时，停止唤醒后续进程。（互斥进程插入时总是位于尾部）

## 进程资源限制

每个进程可以分配的资源是有限制的，在 signal->rlim 字段中指定。

## 进程切换

内核将正在运行的进程挂起，并恢复某个挂起进程的执行，我们把这个过程叫做进程切换：context-switch

### 硬件上下文

进程之前共享CPU硬件寄存器，因此在执行进程切换之前，必须执行寄存器内容的保存与恢复。我们把必须更新的寄存器数据叫做硬件上下文，硬件上下文是可执行上下文的一部分。

在Linux中，进程硬件上下文一部分存放在TSS段，一部分存放在内核态堆栈中。

Linux使用软件执行进程切换：通过一组move指令进行切换。



**进程切换发生在内核态**。执行进程切换前，用户态进行所使用的寄存器内容都保存在内核态堆栈上。



### TSS段

80x86体系结构下会为每个CPU创建一个特殊的段：TSS段

- CPU从用户态进入内核态时，从TSS获取内核态堆栈的地址

task_struct中包含thread_struct字段，用来保存进程的硬件上下文，包含了大部分的CPU寄存器，但不包含eax、ebx等通用寄存器。

### 进程切换过程

进程切换只可能发生在一个位置：schedule()函数

每个进程切换由两部组成：

1、切换全局页目录，加载新的地址空间：CR3寄存器

2、切换内核态堆栈和硬件上下文，switch_to 宏

## 进程创建

todo.

## Reference

[1] 深入理解LINUX内核
