---
title: "深入理解Linux内核读书笔记"
date: 2023-06-04
draft: false
tags: ["Linux","system"]
---

## 第二章 内存寻址

### 地址分类

- 逻辑地址：段+偏移量组成。
- 线性地址，即虚拟地址，一个32位无符号整数，表示高达4GB的地址空间。
- 物理地址，用于内存芯片级内存单元寻址。

`逻辑地址 --分段--> 线性地址 -->MMU-->物理地址`



### 分段机制

此部分可以参考 https://zhuanlan.zhihu.com/p/324210723 进行学习

一个逻辑地址由段选择符（16位）和段内偏移（32位）组成。段选择符存储在段寄存器中。80x86提供了6个段寄存器：

- cs：代码段寄存器，指向包含程序指令的段。
- ss：栈段寄存器，指向包含当前程序栈的段。
- ds：数据段寄存器，指向包含静态数据或者全局数据段。
- es fs gs：一般用途，可以指向任意段。

<img src="https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211737128.png" alt="image-20230612094608246" style="zoom: 50%;" />

每个段通过一个`8B`大小的`段描述符`来描述特征。段描述符存储在全局描述符表（GDT）或局部描述符表（LDT）中。GDT通常只有一个；如果进程创建了GDT中没有的段，就拥有自己的LDT。GDT的地址和大小存放在`gdtr`控制寄存器，LDT的地址和大小存放在`ldtr`控制寄存器。



为每个段寄存器提供了一个非编程寄存器，当段寄存器内容发送变化时，将从LDT或GDT中加载对应的段描述符到对应的非编程寄存器。

段寄存器存储了段描述符在描述符表中的索引。

<img src="https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211737540.png" alt="image-20230612095038781" style="zoom: 33%;" />

`线性地址 = [ 段描述符 ]+段内偏移量`

### Linux中的分段

Linux下的逻辑地址与线性地址是一致的

![image-20230612093924214](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211737211.png)

用户态的所有进程，都通过`用户代码段(__USER_CS)`和`用户数据段(__USER_DS)`来寻址代码和数据。内核态的所有进程，都通过`内核代码段(__KERNEL_CS)`和`内核数据段(__KERNEL_DS)`来寻址代码和数据。

四个段都是从0x0开始，即Linux下，逻辑地址和线性地址是一致的。

CPU 的特权等级发生变化时（用户态 <--> 内核态），相应的段寄存器也要更新，（DPL字段代表访问这个段的最低优先级）。



### 硬件分页机制

https://zhuanlan.zhihu.com/p/480796773

`线性地址->物理地址`

一个关键任务：比较请求访问的类型与线性地址访问权限，若访问无效，产生缺页异常



一级页表

<img src="https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211737773.png" alt="image-20230612100911754" style="zoom: 33%;" />

多级页表

<img src="https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211737288.png" alt="image-20230612100956332" style="zoom:33%;" />

第一级表称为页目录，存放在一页 4K 大小的页面中，具有 2^10 个 4 字节长度的表项。 这些表象指向对应的二级表。 线性地址的最高 10 位（31-22）用作以及表中的索引。

第二级称为页表，长度也是 4K 大小的一个页面，最多有 1K 个 4 字节的表项。 每个 4 字节的表项含有相关页面的 20 位物理基地址。 二级页表使用线性地址的中间 10 位（21-12）作为表项索引值，以获取含有页面 20 物理地址基地址的表项。 该20位页面物理基地址和线性地址中的第12位（页内偏移）组合在一起就得到了分页转换过程的输出值，即对应的的最终物理地址。

对于给定的线性地址，CR3 寄存器指定页目录表的基地址。线性地址的高10位用于索引这个页目录表，以获得指向相关第二级页表的指针。线性地址空间中间10位用于索引二级页表，以获得物理地址的高20位。线性地址的第12位直接作为物理地址的第12位，从而组成一个完整的32位物理地址。



### Linux分页机制

<img src="https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211737965.png" alt="image-20230612165643253" style="zoom:50%;" />

## 第三章 进程

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

