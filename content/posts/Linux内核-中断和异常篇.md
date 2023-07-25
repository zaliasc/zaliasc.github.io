---
title: "Linux内核-中断和异常"
date: 2023-07-25
draft: false
tags: ["Linux","system"]
---





# 中断和异常

## 中断

中断信号提供一种方式，让CPU去处理正常控制流之外的代码。

### 中断分类

中断

- 可屏蔽中断
- 不可屏蔽中断

异常：分为三类，取决于异常发生时eip保存的值

- fault，eip保存上一条故障的指令地址
- trap，eip保存上一条故障的指令地址随后要保存的地址，主要作用是程序调试
- abort，硬件故障

### 中断描述表 idt_table

idt_table 定义为一个 大小为 255 的数组

`gate_desc idt_table[NR_VECTORS] __page_aligned_data = { { { { 0, 0 } } }, };`

每一项 gate_desc（代码来自Linux code 2.6.39）

```
typedef struct desc_struct gate_desc;

struct desc_struct {
	union {
		struct {
			unsigned int a;
			unsigned int b;
		};
		struct {
			u16 limit0;
			u16 base0;
			unsigned base1: 8, type: 4, s: 1, dpl: 2, p: 1;
			unsigned limit: 4, avl: 1, l: 1, d: 1, g: 1, base2: 8;
		};
	};
} __attribute__((packed));
```

在 `trap_init`中对 idt_table 进行初始化：

```
void __init trap_init(void)
{
	int i;

#ifdef CONFIG_EISA
	void __iomem *p = early_ioremap(0x0FFFD9, 4);

	if (readl(p) == 'E' + ('I'<<8) + ('S'<<16) + ('A'<<24))
		EISA_bus = 1;
	early_iounmap(p, 4);
#endif

	set_intr_gate(0, &divide_error);
	set_intr_gate_ist(2, &nmi, NMI_STACK);
	/* int4 can be called from all */
	set_system_intr_gate(4, &overflow);
	set_intr_gate(5, &bounds);
	set_intr_gate(6, &invalid_op);
	set_intr_gate(7, &device_not_available);
#ifdef CONFIG_X86_32
	set_task_gate(8, GDT_ENTRY_DOUBLEFAULT_TSS);
#else
	set_intr_gate_ist(8, &double_fault, DOUBLEFAULT_STACK);
#endif
	set_intr_gate(9, &coprocessor_segment_overrun);
	set_intr_gate(10, &invalid_TSS);
	set_intr_gate(11, &segment_not_present);
	set_intr_gate_ist(12, &stack_segment, STACKFAULT_STACK);
	set_intr_gate(13, &general_protection);
	set_intr_gate(15, &spurious_interrupt_bug);
	set_intr_gate(16, &coprocessor_error);
	set_intr_gate(17, &alignment_check);
#ifdef CONFIG_X86_MCE
	set_intr_gate_ist(18, &machine_check, MCE_STACK);
#endif
	set_intr_gate(19, &simd_coprocessor_error);

	/* Reserve all the builtin and the syscall vector: */
	for (i = 0; i < FIRST_EXTERNAL_VECTOR; i++)
		set_bit(i, used_vectors);

#ifdef CONFIG_IA32_EMULATION
	set_system_intr_gate(IA32_SYSCALL_VECTOR, ia32_syscall);
	set_bit(IA32_SYSCALL_VECTOR, used_vectors);
#endif

#ifdef CONFIG_X86_32
	set_system_trap_gate(SYSCALL_VECTOR, &system_call);
	set_bit(SYSCALL_VECTOR, used_vectors);
#endif

	/*
	 * Should be a barrier for any external CPU state:
	 */
	cpu_init();

	x86_init.irqs.trap_init();
}
```

根据不同的 DPL ，可以将中断描述符分为几类：

![image-20230725213441157](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307252134905.png)

几种设置的区别在与：DPL、段选择器地址

使用下列函数进行中断门的插入：

![image-20230725213539464](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307252135199.png)

![image-20230725213552100](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307252135757.png)

## 硬件处理流程

当CPU执行完一条指令后，cs和eip包含了下一条将要执行的指令地址，在进入下一条指令的执行之前，CPU会检查执行上一条指令的过程中是否发生了中断或异常。如果发生执行以下动作：

1. **确定中断/异常向量 i**
2. 读取`idtr`寄存器指向的 IDT 表中的第 i 项：中断描述表的起始地址加上中断向量号乘以8
3. 从 `gdtr`中获取 GDT 的地址，并在 GDT 中查找，以读取 IDT 表项所标识的段描述符，**获取中断/异常处理程序的基地址**。
4. 检查中断请求的特权级是否满足：比较 CPL（CS寄存器中） 和 DPL（段描述符中），不满足则产生 General protection 异常。
5. 如果发生了特权级的变化，即 CPL 与 DPL 不同（被中断的程序和中断处理程序在不同特权级），那么必须开始使用新的特权级相关的栈：
   1. 读取 tr 寄存器，访问被中断进程的 TSS 段。
   2. 使用新的特权级相关的栈，装载 ss (栈段寄存器) 和 esp （栈指针寄存器）
   3. 将被中断程序的 ss 和  esp 推入到中断处理程序的栈中
6. **在栈中保存 eflags、cs、eip寄存器**
7. 保存硬件出错码（如果异常产生了的话）
8. **装载 cs 和 eip 寄存器，指向中断处理程序**

开始执行中断/异常处理程序

处理完成后执行 iret 指令：

1. 弹出 eflags、cs、eip寄存器
2. 检查 CPL 与 DPL，不同则需要 弹出 ss 和 esp 寄存器
3. 返回被中断程序的执行

## 源码分析

以系统调用为例，在触发硬件中断后，跳转到 system_call （entry_32.S）

```
ENTRY(system_call)
	RING0_INT_FRAME			# can't unwind into user space anyway
	pushl_cfi %eax			# save orig_eax
	SAVE_ALL
	GET_THREAD_INFO(%ebp)
					# system call tracing in operation / emulation
	testl $_TIF_WORK_SYSCALL_ENTRY,TI_flags(%ebp)
	jnz syscall_trace_entry
	cmpl $(nr_syscalls), %eax
	jae syscall_badsys
syscall_call:
	call *sys_call_table(,%eax,4)
	movl %eax,PT_EAX(%esp)		# store the return value
syscall_exit:
	LOCKDEP_SYS_EXIT
	DISABLE_INTERRUPTS(CLBR_ANY)	# make sure we don't miss an interrupt
					# setting need_resched or sigpending
					# between sampling and the iret
	TRACE_IRQS_OFF
	movl TI_flags(%ebp), %ecx
	testl $_TIF_ALLWORK_MASK, %ecx	# current->work
	jne syscall_exit_work

restore_all:
	TRACE_IRQS_IRET
restore_all_notrace:
	movl PT_EFLAGS(%esp), %eax	# mix EFLAGS, SS and CS
	# Warning: PT_OLDSS(%esp) contains the wrong/random values if we
	# are returning to the kernel.
	# See comments in process.c:copy_thread() for details.
	movb PT_OLDSS(%esp), %ah
	movb PT_CS(%esp), %al
	andl $(X86_EFLAGS_VM | (SEGMENT_TI_MASK << 8) | SEGMENT_RPL_MASK), %eax
	cmpl $((SEGMENT_LDT << 8) | USER_RPL), %eax
	CFI_REMEMBER_STATE
	je ldt_ss			# returning to user-space with LDT SS
restore_nocheck:
	RESTORE_REGS 4			# skip orig_eax/error_code
irq_return:
	INTERRUPT_RETURN
```

其中 

call *sys_call_table(,%eax,4) 执行对应系统调用程序

RESTORE_REGS 4	恢复之前保存的寄存器

INTERRUPT_RETURN 就是 iret 指令，从中断程序返回

## 中断处理

中断处理的主要部分在 \_\_do\_\_IRQ 中完成

## 软中断及 tasklet

todo.



## Reference

[1] 深入理解LINUX内核

[2] [CPU中断那些事](https://open.toutiao.com/a6951563322730840615/?utm_source=vivoliulanqi&utm_medium=webview&utm_campaign=open&label=related_news&item_id=6951563322730840615&gy=b691c991707e75f40b6951b1cceca90329bdb717425a006fa5011b560d941a44c4eae172607b0a013717b8e4a11d6a5d9470f31a174313de3a01795104f7f461&crypt=1408&req_id=20210622223520010212142018131CAE6A&fr=normal&isRelated=1&isNews=1&vivoRcdMark=1&from_gid=6937076362490479112&channel_id=88805669586)
