---
title: OSTEP | 进程基础
date: 2020-11-15 09:20:00
tags:
  - 操作系统
categories:
  - 计算机
  - 计算机基础
---
**进程**就是运行中的程序。

操作系统通过虚拟化CPU来提供了一种假象：仿佛有无数的CPU似的。操作系统让一个进程只运行一个时间片，然后切换到其他进程。这就是时分共享（`time sharing`）技术。操作系统会通过一些低级机制（比如上下文切换）和高级策略（比如调度策略）以高效地实现虚拟化。

<!-- more -->

## 进程API
后续会详细讨论进程API，这里先简单介绍一下。操作系统为了完成必要的功能，API必须要包含以下内容：
* 创建（`create`）：必须包含一些创建新进程的方法。
* 销毁（`destroy`）：对应创建，还必须提供销毁进程的方法，不管是程序自行退出还是用户强制终止某个应用。
* 等待（`wait`）：等待进程停止运行非常有用。
* 其他控制（`miscellaneous control`）：除了销毁和等待之外，还需要一些其他控制，比如暂停和恢复进程运行。
* 状态（`status`）：获取进程信息，比如运行了多久、处于什么状态等等。

## 进程创建
下面是一个简单的示意图  
![](/images/OSTEP0401.PNG)

操作系统运行程序必须做的第一件事是将代码和静态数据（初始化变量）加载到内存中（现代操作系统往往是惰性加载，要理解其是如何工作的，需要了解分页和交换的机制），然后创建和初始化栈（运行时的栈，存放着局部变量、函数参数和返回地址），接着执行与I/O设置相关的其他工作，至此准备工作就完成了。最后启动程序，指令跳转到`main()`入口，将CPU控制权交给新创建的进程。

## 进程状态
进程可以处于以下三种状态之一。
* 运行（`running`）：正在执行指令。
* 就绪（`ready`）：准备好了，但是操作系统还没有选择让其运行。
* 阻塞（`blocked`）：进程执行了某个操作（比如发起I/O请求）会进入阻塞，直到其他事件发生（比如I/O完成）才会准备运行。

三者关系如下图所示：  
![](/images/OSTEP0402.PNG)

## 数据结构
操作系统和其他程序一样也是一个程序，有一些关键的数据结构来跟踪各种相关信息。下面使用xv6内核的代码来说明。
``` C
// the information xv6 tracks about each process
// including its register context and state
struct proc {
    char *mem;      // Start of process memory
    uint sz;        // Size of process memory
    char *kstack;   // Bottom of kernel stack
                    // for this process

    enum proc_state state;      // Process state
    int pid;                    // Process ID
    struct proc *parent;        // Parent process
    void *chan;                 // If !zero, sleeping on chan
    int killed;                 // If !zero, has been killed
    struct file *ofile[NOFILE]; // Open files
    struct inode *cwd;          // Current directory
    struct context context;     // Switch here to run process
    struct trapframe *tf;       // Trap frame for the
                                // current interrupt
};
```

上下文切换时，需要保存寄存器的信息。
``` C
// the registers xv6 will save and restore
// to stop and subsequently restart a process
struct context {
    int eip;
    int esp;
    int ebx;
    int ecx;
    int edx;
    int esi;
    int edi;
    int ebp;
};
```

除了运行、就绪和阻塞外，还有一些其他状态。僵尸状态是很有用的，它允许其他进程（通常是父进程）检查程序的返回码，并查看是否成果，完成之后调用系统API告诉操作系统可以清理这个正在结束的进程的所有相关数据结构。
``` C
// the different states a process can be in
enum proc_state { UNUSED, EMBRYO, SLEEPING, 
                  RUNNABLE, RUNNING, ZOMBIE };
```

## 模拟作业
这一章后面的模拟作业挺好的，说明了不同调度策略对性能的影响是很大的。
* 前面五题说明当有I/O操作时，放弃CPU给其他进程，总体效率会提升。
* 后面四题说明当有I/O完成时，立即响应完成事件，总体效率比较好。
