---
title: "Linux内核-进程篇"
date: 2023-07-25
draft: true
tags: ["Linux","system"] 
---

## 进程标识与创建

### TaskStruct结构

![image-20230612171305862](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211737407.png)



### 进程状态

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

### 内核栈

<img src="https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211738057.png" alt="image-20230612173301075" style="zoom:50%;" />

大小为8K，task_struct 和 thread_info 可以很方便的互相寻址

<img src="https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211738477.png" alt="image-20230612173316818" style="zoom:50%;" />

### 进程切换



### 进程创建

clone

fork

vfork

## 进程调度

todo.

## Reference

[1] 深入理解LINUX内核
