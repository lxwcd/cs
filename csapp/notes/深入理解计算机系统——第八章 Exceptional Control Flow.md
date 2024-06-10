深入理解计算机系统——第八章 Exceptional Control Flow  
    
资源：  
> [视频课程](https://www.bilibili.com/video/BV1iW411d7hd?p=14)  
> [视频课件1](http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/14-ecf-procs.pdf)  
> [视频课件2](http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/15-ecf-signals.pdf)  
> [深入理解计算机系统（CSAPP）复习笔记——第八章](https://zhuanlan.zhihu.com/p/376568475)  
    
**control transfer:** From the time you first apply power to a processor until the time you shut it off, the **program counter** assumes a sequence of values $a_{0}, a_{1}, ... , a_{n}$, where each $a_{k}$ is the **address** of some corresponding **instruction** $I_{k}$. Each **transition** from $a_{k}$ to  $a_{k+1}$  is called a **control transfer**.  
    
**control flow**: A **sequence** of such **control transfers** is called the **flow of control**, or **control flow**, of the **processor**.  
    
最简单的控制流的类型是这一系列的指令是顺序执行的，没有跳转等情况。  
    
**exceptional control flow (ECF)**：控制流中的指令不是顺序执行，如发生跳转，函数调用，函数返回等。  
    
**理解 ECF 的重要性：**  
- ECF 是操作系统实现 I/O，进程和虚拟内存的基本机制，只有理解 ECF，才能理解这些概念。  
- 理解 ECF 帮助理解应用程序怎么和操作系统交互。  
- 理解 ECF 能帮助写应用程序。  
- 理解 ECF 能帮助理解并发。  
- 理解 ECF 能帮助理解软件的异常处理。  
    
本章主要介绍应用程序怎么和操作系统交互。  
    
# 8.1 Exceptions  
**Exceptions** are a **form of exceptional control flow** that are **implemented** **partly** by the **hardware** and **partly** by the **operating system**.  
    
An **exception** is an **abrupt change** in the **control flow** in response to some change in the **processor’s state**. Figure 8.1 shows the basic idea.  
![Anatomy of an exception](https://img-blog.csdnimg.cn/bf716a4ec38049dca5755551ee5424af.png)  
    
## 8.1.1 Exception Handling  
**exception number**：系统中每种类型的**异常**都有一个**唯一的非负的整数**，即**异常号**。有些**异常号**是**处理器的设计者**分配的，有些是**操作系统内核（操作系统中常驻内存的部分）** 的设计者分配的。  
    
系统启动时，操作系统会分配并初始化一个被称为**异常表**（exception table）的跳转表：  
    
![Exception table](https://img-blog.csdnimg.cn/5bae945feaa74c039ffedc7f6fd9c2b4.png)  
    
从图 8.2 可以看见，异常表的每个**条目**（entry）都包含该异常对应的 handler 的地址。  
    
在运行时（run time），如果处理器检测到有异常发生，并根据**异常类型**找到**异常号**后，则会通过**异常表**执行一个间接 procedure call (the exception）到一个操作系统子程序（the exception handler，专门设计处理这种特殊的事件)。  
    
图 8.3 展示怎么通过**异常号**找到 exception handler 的地址：异常表的起始地址存在 CPU 的一个叫**异常表基址寄存器**（exception table base register）中，该地址再加上**异常号**（异常表的索引号）则为 exception handler 的地址。  
    
一旦触发异常，剩下的工作由软件中的 exception handler 来完成。  
    
## 8.1.2 Classes of Exceptions  
异常有如下四种类型：![Classes of exceptions](https://img-blog.csdnimg.cn/9bd58b8d76494892af60b04dea8d8c0e.png)  
    
### Interrupts  
**中断**是**异步**发生的，来自 I/O 设备的信号，硬件中断的**异常处理程序**（exception handlers）被称为**中断处理程序**（interrupt handlers）。  
    
当中断处理完后，将控制返回给**中断前将要执行的下条指令**，效果是程序继续之前控制流中的指令执行，好像从未发生过中断。  
  
*************  
  
> 汇编语言（第四版）王爽 第12章  
  
中断分为内中断和外中断。  
  
CPU 接受到中断信息时，需要知道中断的来源，因此中断信息必须包含识别来源的编码。  
  
知道来源后，由中断处理程序来处理中断。  
  
#### 中断向量表  
中断向量表是计算机系统中一个非常重要的概念，它用于处理硬件和软件中断。中断是计算机处理器对外部事件或特定条件做出反应的一种机制。当中断发生时，处理器暂停当前的执行流程，转而执行一个专门处理该中断的程序，这个程序称为中断处理程序。中断处理完成后，处理器可以选择返回到被中断的位置继续执行原来的程序。  
  
中断向量表是一个包含了中断处理程序地址的表格，它为每个可能发生的中断提供了一个具体的处理程序的入口地址。当某个中断发生时，处理器会使用中断的编号或者类型作为索引，在中断向量表中查找相应的处理程序的起始地址，然后跳转到这个地址开始执行中断处理程序。  
  
中断向量就是中断处理程序的入口地址，中断向量表则是中断处理程序的入口地址的列表。  
中断向量表在内存中保存。  
CPU 将中断类型码作为中断向量表的表项号，定位相应的表项，从而得到中断处理程序的入口地址。  
  
中断向量表的详细讲解涉及到以下几个主要方面：  
  
1. **中断向量表的位置**：在不同的计算机系统中，中断向量表可能位于固定的内存地址，或者其位置可以在系统启动时进行配置。  
  
2. **中断向量表的结构**：虽然具体的结构可能因系统而异，但一般每个表项都包含中断处理程序的地址。  
  
3. **中断类型**：系统可能支持多种中断，例如硬件中断、软件中断、异常等。每种中断类型都会有一个或多个特定的处理程序。  
  
4. **中断处理过程**：当中断发生时，如何利用中断向量表查找并跳转到相应的处理程序，以及中断处理完毕后如何返回到原程序。  
  
#### 中断过程  
1. **中断发生**：当中断触发时，CPU完成当前正在执行的指令，然后开始中断序列。  
  
2. **保存现场**：CPU自动将当前的程序状态保存下来，以便中断处理完成后能返回到中断前的状态继续执行。在x86架构中，CPU会自动压入以下寄存器或标志的值到堆栈：  
  
   - **标志寄存器**（FLAGS或EFLAGS）：包含了许多状态位，如中断启用/禁用标志（IF）、方向标志（DF）等。  
   - **代码段寄存器**（CS）：当前正在执行的代码段标识。  
   - **指令指针寄存器**（IP或EIP/RIP）：当前执行的下一条指令的地址。  
  
3. **中断向量表查询**：CPU根据中断的类型或编号，到中断向量表（Interrupt Vector Table, IVT）或中断描述符表（Interrupt Descriptor Table, IDT，在保护模式下使用）查看相应中断处理程序的入口地址。  
  
4. **跳转执行**：CPU利用查找到的地址跳转到中断处理程序（Interrupt Service Routine, ISR）的起始位置开始执行。  
  
5. **中断处理**：ISR执行，完成对事件的处理。处理过程可能涉及到更多的寄存器，如数据段寄存器（DS）、栈段寄存器（SS）、基指针寄存器（BP），以及通用寄存器（AX, BX, CX, DX等）。这些寄存器的值如果在ISR中被改变，ISR需要负责保存和恢复它们。  
  
6. **中断返回**：ISR执行完成后，会通过一个特殊的中断返回指令（如`IRET`或`IRETD`）让CPU从堆栈中恢复被保存的寄存器的值（如EFLAGS, CS, EIP），这样CPU就可以回到ISR被调用前执行的代码继续执行。  
  
在x86-64或者说x64架构中，由于寄存器宽度的增加，相应的指令和寄存器也有所变化（如EFLAGS变为RFLAGS，EIP变为RIP），不过整个中断处理的基本流程是一致的。  
  
#### 单步中断  
单步中断是一种特殊类型的内中断，它允许调试器或其他系统软件在程序执行的每一步之后进行监控。单步中断通常用于调试目的，让开发者能够跟踪程序的执行流程，检查寄存器和内存状态，以及进行错误定位。  
  
在x86体系结构中，详细的单步中断处理过程通常涉及以下步骤：  
  
1. **设置标志**：在x86指令集中，特定的标志寄存器（FLAGS或EFLAGS）中有一个名为TF（Trap Flag）的位。当这个标志位被设置为1时，处理器将在执行每一条指令后产生一个单步中断。  
  
2. **执行指令**：CPU按正常流程执行一条指令。  
  
3. **中断触发**：指令执行完毕后，若TF位被置位，CPU自动触发Type 1中断，即单步中断。  
  
4. **保存程序状态**：CPU在中断发生时自动保存当前程序的执行状态，包括指令指针和代码段寄存器（IP或EIP/RIP和CS），以及FLAGS/EFLAGS寄存器的内容。  
  
5. **处理中断**：CPU通过中断向量表（IVT）或中断描述符表（IDT）找到单步中断处理程序的入口地址，并跳转过去执行。  
  
6. **中断处理程序（ISR）**：单步中断的ISR通常由调试器设置，它可能只是简单地检查CPU状态，也可能包含更复杂的逻辑来帮助调试。  
  
7. **恢复程序状态和继续执行**：ISR完毕后，通过`IRET`指令恢复之前保存的程序状态，继续执行下一条指令。  
  
开发者可以通过设置或清除TF位，控制程序是连续执行还是单步执行。单步执行对于分析程序行为、调试程序中的错误非常有用。  
  
在使用单步中断时，软件调试器通常在单步调试周期内把TF标志置位，以逐条执行指令；在连续执行时清除该标志。这允许在调试会话中灵活切换程序的执行模式。  
  
#### 中断是异步发生的  
"异步"指的是事件或动作的发生不需要等待前一个任务完成就可以开始。与之相对的是"同步"，同步执行会阻塞进程直到当前任务完成。所以，当说到"中断是异步发生"的时候，意味着中断可以在任何时间点发生，而不需要等待当前执行的任务完成。这允许系统能够更加灵活地响应外部事件，提高了系统的效率和响应速度。  
  
中断被称为异步发生，是因为中断可以在任何时间点由外部事件或系统内的其他原因触发，而与CPU的当前执行任务无关。这意味着中断不需要等待当前执行中的指令完成就可以发生，它的发生是不可预测的，与正在执行的程序流程异步。这样的特性使计算机系统能够更加灵活和及时地响应各种外部事件或内部状态的变化。  
  
#### 内中断和外中断  
**内中断（软件中断）**：  
- **定义**：内中断通常是由软件指令引起的，而不是由硬件事件直接触发。它可以被用于操作系统的系统调用、程序自身的异常处理等场景。  
- **触发方式**：内中断可以通过执行特定的中断指令（如x86指令集中的`INT`指令）来触发，也可以由程序运行过程中的异常情况（如除零错误、访问违规的内存页）自动触发。  
- **用途**：软件中断允许程序以一种受控的方式进入到操作系统内核，进行系统调用或处理异常情况，是操作系统提供系统服务的一种机制。  
  
**外中断（硬件中断）**：  
- **定义**：外中断是由硬件外设（如键盘、鼠标、网络接口卡等）产生的中断信号引起的，主要用于处理外设事件。  
- **触发方式**：当外设需要CPU注意时（例如，键盘被按下或者网络包到达网络卡），外设通过向CPU发送中断信号来触发外中断。这个信号通知CPU暂停当前任务，转而处理由外设引起的事件。  
- **用途**：外中断使得操作系统能够以非轮询的方式处理外围设备事件，极大地提高了CPU的工作效率和系统的响应能力。  
- **分类**：可屏蔽中断和不可屏蔽中断。  
  
### Traps and System Calls  
**陷阱**（traps）最重要的用途是在**用户程序**和**内核**之间提供一个类似**过程**的**接口**，即**系统调用（system call）**。**内核**是**常驻内存中的一部分**，为程序提供各种服务，但**内核是受保护**的，不能被应用程序**直接访问**，**内核**通过提供**接口**来为应用程序服务。  
![Trap handling](https://img-blog.csdnimg.cn/941eb9183d44416ba2ffc7f25b055c61.png)  
  
*****************  
  
补充：  
除0 错误 CSAPP 的表格中显示是 fault，有的地方又说是属于 trap？  
  
#### trap 是同步发生的  
Trap是同步发生的，因为它是指程序由于执行了某些操作（如执行了一个非法操作或访问了一个非法地址）导致的异常情况。与异步发生的中断不同，Trap是由程序自身的控制流引起的，通常是在执行某条指令时检测到错误条件而触发。因此，Trap的发生是可以预测的，且与程序的执行流程同步。  
  
由于Trap是由程序执行过程中的特定事件直接触发的，因此它是同步发生的，紧密关联于程序当前的执行状态。  
  
系统调用通常是通过特定的机制实现的，比如在x86架构下，程序通过执行`int`（中断指令）或`syscall`指令请求操作系统的服务，这些都可看作是trap。系统调用是同步的，因为执行系统调用的指令本身就会立即中断当前的程序流程，将控制权交由操作系统接管，并在操作系统处理完毕后，控制权会返回到程序中。  
  
确实有些系统调用的行为看起来像是异步的，如非阻塞I/O或者信号处理，但底层实现仍然是同步触发的trap。在这些情况下，操作系统可能会立即返回控制权给程序，而不是等到操作完全完成。这给用户留下了一种异步调用的印象，但从系统层面来看，系统调用的调用机制仍然是同步的。  
  
在这种情况下，即使系统调用中有些非阻塞的I/O调用给人一种异步的印象，但这并不改变trap本身是同步发生的事实。同步和异步的概念在这里指的是不同的上下文。  
  
同步（Synchronous）在trap的上下文中意味着trap的发生是由当前执行的指令直接引起的，即发生trap时，CPU会立即中断当前的执行路径，并按照预定的方式处理这个trap，该过程与正在执行的代码流程有直接的因果关系。这种情况下，CPU的控制流程是可以预测的，因为trap是由程序中的特定操作（如执行了系统调用指令）直接触发的。  
  
而非阻塞的I/O调用给人的异步印象，是因为当程序执行这样一个系统调用时，操作系统会立即返回控制权给调用者，而不是等待I/O操作完成。这种模式使得程序能够继续执行，而不必阻塞等待I/O操作的完成。在这种上下文中，"异步"（Asynchronous）意味着调用发起后，完成的具体时机和调用本身的执行流程是解耦的。  
  
要点是，即使某些系统调用操作表现为"异步"，trap的同步性质并未改变。当trap被触发时，处理trap的操作仍然是同步进行的，即CPU会立即中断当前任务，处理trap。这里的“同步”与“异步”反映了不同层面的概念：trap的处理与引发它的指令同步发生，而某些系统调用（如非阻塞I/O）的处理结果与调用本身异步发生。  
  
### Faults  
**故障**（faults）是由**错误**情况引起的，如果**故障处理程序**（fault handler）修正了错误，则重新执行引起故障的指令（非下一条指令）；如果无法修正错误，则返回到内核中的 abort routine，终止程序。  
![Fault handling](https://img-blog.csdnimg.cn/abff491c19834d6eb7b051a1e00ef759.png)  
  
***************  
  
#### fault 和 trap  
在计算机架构中，`fault`和`trap`是两种类型的中断，它们从根本上代表了硬件级别的异常和中断处理机制。  
  
**Fault（故障）**:  
- Fault是一种可以被修正的异常情况，典型地在异常发生时不会改变程序的正确性。当一个fault发生时，系统会自动尝试修正这个问题，并允许程序在异常指令处重新执行。例如，当一个程序尝试访问未被分配的内存时，操作系统可能会捕获到这个memory fault，随后为请求的内存区域分配物理内存，并允许程序继续执行。  
    
**Trap（陷阱）**:  
- Trap是一种由程序显示产生的异常，通常用于调试目的或系统调用实现。它表示程序在执行过程中到达一个特定的点，此时程序刻意要求操作系统进行干预。Trap通常是可预见的，发生后通常不会返回到导致trap的那条指令重新执行，而是继续执行trap处理例程之后的指令。Trap可以用于实现断点调试，当程序执行到开发者设置的断点时，产生trap，将控制权转交给调试器。  
  
**Fault和Trap的区别**主要在于它们被触发的条件和对系统的影响。Fault通常是由于程序的错误操作导致的，如访问非法内存；而trap则是程序正常执行流程中预设的中断点。在处理方式上，fault要求系统能够尽可能修正异常并允许程序继续执行；而trap则是将控制权转交给特定的处理程序，不一定回到原点继续执行。  
  
### Aborts  
**终止**（aborts）是**不可修复的致命错误**造成的结果，不会将控制权返回给应用程序。  
![Abort handling](https://img-blog.csdnimg.cn/9d2227bdeb3f4fb68b1fff76951ef048.png)  
    
## 8.1.3 Exceptions in Linux/x86-64 Systems  
以 **x86-64** 系统为例，有**256**种异常类型，见下图：  
![Examples of exceptions in x86-64 systems](https://img-blog.csdnimg.cn/9168f20dd96e41c99ffc6c6ca8f907e4.png)  
    
### Linux/x86-64 Faults and Aborts  
- **Divide error**  
当应用程序试图**除以0**或者除法指令的结果过大。Linux 对于这种错误会提示 "Floating exceptions"。  
    
- **General protection fault**  
一般保护故障，原因很多，如程序引用虚拟内存的一块未定义区域，或者程序试图写只读文本段。Linux 不会修复这类故障，提示为 "Segmentation faults"。  
    
- **Page fault**  
第九章介绍  
    
- **Machine check**  
**致命的硬件错误**造成的结果，在执行故障指令时检查到，不会将控制权返回给应用程序。  
    
### Linux/x86-64 System Calls  
Linux 有几百个系统**系统调用**，部分常用的系统调用见下图：  
![Examples of popular system calls in Linux x86-64 systems](https://img-blog.csdnimg.cn/1c7b195937494688b509370d0a63dd03.png)  
    
**system-level functions**：系统调用以及相关的包装函数（wrapper functions）。  
    
# 8.2 Processes  
**process**:  An instance of a running program.  
    
进程提供两个抽象：  
- **Logical control flow**  
每个程序似乎独占 CPU（provided by kernel mechanism called contex switching）。  
- **Private address space**  
每个程序似乎独占内存（provided by kernel mechanism called virtual memory）。  
    
Each **program** in the system **runs** in the **context** of some **process**.  
    
**上下文（context）**：由程序需要正确运行所需的**状态**组成。This **state** includes the **program’s code and data** stored in **memory**, its **stack**, the contents of its **general purpose registers**, its **program counter**, **environment variables**, and the set of **open file descriptors**.  
  
**************  
  
补充：  
进程提供两个抽象的目的是为了实现操作系统的多任务特性，即允许多个程序似乎是在同一时间并行运行。  
  
1. **每个程序独占CPU**: 这个抽象的目的是提供一种机制，让用户感觉到他们运行的程序是独占整个处理器的。这是通过操作系统中的调度程序实现的，后者会采取一种称为时间分片的策略。调度器在多个进程之间快速切换，分配CPU时间片，每个进程轮流使用CPU一小段时间。这种切换如此之快，以至于用户感觉到所有的程序似乎都在同时运行。  
  
2. **每个程序独占内存**: 为了保护和隔离，操作系统提供了每个程序都有自己独立的内存空间的假象。这通过使用内存管理技术（像是分页或者分段）来实现，使得每个进程都认为它拥有一段连续的地址空间。这种隔离确保了一个进程无法访问或者干扰到另一个进程的内存，有助于提高系统的稳定性和安全性。  
  
这两个抽象是现代操作系统的基础，使得操作系统可以在同一台机器上高效地运行多个程序，同时为用户提供良好的体验。  
    
## 8.2.1 Logical Control Flow  
**logical control flow**：程序计数器（PC）的值组成的序列。  
    
示例：  
![Logical control flows](https://img-blog.csdnimg.cn/64ea23b698cc4f43be9e8fcbc5489043.png)  
上图中三个进程交替的执行。  
    
## 8.2.2 Concurrent Flows  
**concurrent flow**：一个逻辑流在执行时和另一个逻辑流在时间上重叠。例如图 8.12 中进程 A 和进程 B，进程A 和进程 C 都是并发的运行，因为**进程A**未结束，**进程B**和**进程C**就开始运行了；但进程B和进程C不是并发运行。  
    
**multitasking**：多个进程之间轮流执行。  
    
**time slice**: **Each time period** that a **process** executes a **portion** of its **flow**. 图 8.12 中进程A有两个时间片。  
    
Thus, **multitasking** is also referred to as **time slicing**.  
    
**parallel flows**：Two **flows** are running **concurrently** on **different processor cores or computers**.  
    
## 8.2.3 Private Address Space  
对于一个 n-bit 地址的机器，**地址空间**是 $2^{n}$ 种可能地址的集合，范围为 $0, 1, ... 2^{n}-1$。  
    
一个进程为每个程序提供它自己的**私有地址空间**。  
This **space** is **private** in the sense that a byte of **memory** associated with a particular **address** in the **space** cannot in general be **read** or **written** by **any other process**.  
    
Figure 8.13 shows the **organization** of the **address space** for an **x86-64** Linux process.  
![Process address space](https://img-blog.csdnimg.cn/6ae93f3334f64ed9993cff2a16993a2e.png)  
    
## 8.2.4 User and Kernel Modes  
**处理器**通过一个**控制寄存器**中的 **mode** 位来赋予当前**进程**享有的**特权**。  
    
当**模式位**（mode bit）被设置后，处理器运行在**内核模式**（kernel mode）或叫**超级用户模式**（supervisor mode），在该模式下，**进程能执行任何指令，访问任何内存位置**。  
    
当**未设置模式位**，进程运行在**用户模式**（user mode），该模式下不能执行特权指令（privileged instructions），也不能直接运用内核的代码或数据，如果尝试做这些操作则会引起致命的**保护故障**，用户程序只能通过系统**调用接口**间接访问内核的代码和数据。  
    
**进程**在最初运行应用程序时处于**用户模式**，进程变为**内核模式**的唯一的方法是通过**异常**，如中断，故障或陷入系统调用，此时**控制权**会交给**异常处理程序**（exception handler），进程变为**内核模式**，handler 运行在内核模式，当控制权返回给应用程序后，进程又回到**用户模式**。  
    
linux 提供 `/proc` 文件系统，允许用户模式的进程访问内核数据结构的内容。  
    
## 8.2.5 Context Switches  
Processes are managed by a shared chunk of memory-resident OS code called the `kernel`.  
    
**context switch**：操作系统内核通过一种被称作 **上下文切换（context switch）** 的高阶异常控制流来实现**多任务**。  
    
**内核**为每个**进程**维持一个**上下文**，该**上下文**是**内核**需要**启动**一个被抢占的**进程**所需要的**状态**，包括 general-purpoe regisers，浮点寄存器，程序计数器，用户栈，状态寄存器等的值。  
    
**scheduling** ：在某个时刻，内核决定**抢占**（preempt）**当前进程**并开始一个**先前被抢占的进程**，这个决定被称为**调度**（scheduling），该过程是由内核中被称为**调度器**（scheduler）的代码执行的。  
    
当内核选择一个新的进程来运行时，称为内核**调度**（scheduled）该进程。  
    
在内核**调度**一个新的进程后，它通过一种被称为**上下文切换（context switch）** 的机制 **抢占**当前的进程然后将**控制转移**给新的进程：  
- 保存当前进程的上下文  
- 恢复某个之前被抢占的进程的上下文  
- 将控制转移给这个新的恢复的进程  
    
**上下文切换** 发生的情景：  
- 内核执行**系统调用**的时候。如果系统调用阻塞（block），如需要等待某个事件发生，则内核能让当前进程休眠，然后执行其他进程。  
- 由**中断**引起。如所有的系统都有某种机制来产生**周期性的定时器中断**，通常每 1ms 或 10ms，每次定时器产时中断，内核将判断当前进程是否已经运行了足够长的时间以及是否切换到另一个进程。  
    
示例：  
![Anatomy of a process context switch](https://img-blog.csdnimg.cn/17e13f8594754f3785ea43e5498e8e7a.png)  
    
初始进程 A 在**用户模式**运行，然后遇到一个 `read` 系统调用指令，因此变为**内核模式**，由于该**系统调用**需要很长时间**从磁盘读数据**，因此**内核**执行一个**上下文切换**，开始执行进程B，注意**在切换到进程B前，内核代表进程A在用户模式执行指令**（内核不是独立的进程）。  
    
在第一次上下文切换时，内核代表进程A在内核模式执行指令，然后在某刻开始**代表进程B**在**内核模式**执行执行，在**完成上下文切换**后，**内核代表进程B**在**用户模式**执行指令，接着进程B在用户模式执行指令直到磁盘发送中断信号表示已经将磁盘读内容传到内存，此时内核将再次进行上下文切换，将控制返回给进程A来执行系统调用之后的指令。  
    
## 8.3 System Call Error Handling  
Linux 系统级的函数遇到错误时，通常**返回-1**，然后设置一个全局整数变量 `errno` 来表明出错的原因。  
    
Hard and fast rule:  
- 检查系统级函数的返回结果  
- Only exception is the handful of functions that return void  
    
# 8.4 Process Control  
Unix provides **a number of system calls** for manipulating processes from C programs.  
## 8.4.1 Obtaining Process IDs  
每个**进程**都有一个**唯一的正数**表示的**进程 ID**（process ID, PID）。  
`getpid` 函数返回**调用该函数的进程，即当前进程**的 PID。  
`getppid` 函数返回**父进程**的 PID。  
    
## 8.4.2 Creating and Terminating Processes  
从程序员的角度看，进程有以下三种状态：  
- **Running**  
进程正在执行，或者等着被执行（最终会被内核调度）。  
    
- **Stopped**  
进程被**挂起**（suspended）且**不会被调度**。A process **stops** as a result of receiving a **SIGSTOP**, **SIGTSTP**, **SIGTTIN**, or **SIGTTOU** signal, and it **remains** **stopped** **until** it receives a **SIGCONT** signal, at which point it becomes running again.  
    
- **Terminated**  
进程**永久性的暂停**。三种情况造成进程终止：接收到一个信号，该信号的默认行为就是终止进程；从主程序返回；调用 **exit** 函数。  
    
### exit 函数：  
![exit](https://img-blog.csdnimg.cn/e6ff8607a8ff4553a49c591f73a96a36.png)  
    
- 无返回值  
- The **exit** function **terminates** the process with an **exit status** of **status**. (The other way to set the **exit status** is to **return an intege**r value from the **main** routine.)  
    
### fork 创建进程  
一个父进程能通过调用 fork 函数创建子进程：  
![fork](https://img-blog.csdnimg.cn/a726d6e90f624f149257d909a1820220.png)  
  
**int fork(void)**:  
- 调用一次但**返回两次**：在调用的进程（**父进程**）中返回**子进程的 PID**，在**子进程**中返回 **0**。  
- 子进程和父进程有相同的（但分开的）用户级**虚拟地址空间**，以及所有打开的**文件描述符**，只是 PID 不同（因为PID 是正数，因此可以通过返回值区分父进程和子进程）。  
    
示例：  
![fork](https://img-blog.csdnimg.cn/8f2fc86f503844b2a637e9e70ae140e1.png)  
上图所示，在第6行调用 fork 后，可以看到父进程和子进程的地址空间相同，有相同的变量值，代码等，但他们的地址空间是分开的，因此执行完 printf 后，两者的 x 值不同。  
    
### Modeling fork with Process Graphs  
A **process graph** is a useful tool for capturing the partial ordering of program statements:  
- Each **vertex** is the execution of a statement  
- a -> b means a happens before b  
- **Edges** can be labeled with current value of variables  
- **printf** vertices can be labeled with output  
- Each graph begins with a vertex with no inedges  
     
For a program running on a **single processor**, any **topological sort (total ordering of vertices where all edges point from left to right)** of the graph corresponds to a **feasible** total ordering. ([拓扑排序](https://blog.csdn.net/AC__dream/article/details/120234476))  
    
之前程序的进程图如下：  
![process graph](https://img-blog.csdnimg.cn/f4b5bfff079b4c77999fbdd7f62f2f23.png)  
    
从上图可以，**父进程**和**子进程**的 printf 语句可以**以任何顺序执行**（父进程先执行或者子进程先执行）。  
    
进程图也能帮助理解 nested fork 调用，如：  
![nested fork](https://img-blog.csdnimg.cn/e9ccc5ee6a4f4e4cac4b7982c4a1622a.png)  
    
## 8.4.3 Reaping Child Processes  
当进程**终止**（terminated）时，内核**不会立即**将它从系统中**移除**，进程保持一种**终止状态**（terminated state）直到被父进程**回收**（reaped）。  
    
父进程**回收**终止的子进程时，内核将**子进程**的 exit state 传递给**父进程**，然后**抛弃该子进程**。  
    
**zombie**：处于**终止状态**（terminated state）但**没有被回收**的进程成为**僵尸进程**。僵尸进程仍然**消耗系统资源**。  
    
当一个父进程终止，内核将调用 **init** 进程来回收那些**孤儿进程**（orphaned children）。**init** 的 PID 为1，是内核在系统启动时创建的，不会终止，也是**所有进程的祖先**。  
    
一个进程可以通过调用 **waitpid** 函数来**等待子进程终止**。  
![waitpid](https://img-blog.csdnimg.cn/0a57d12dac6145859ed0fb3a5057d415.png)  
    
- 默认情况，即 `options` 为 0，`waitpid` 将**挂起**调用它的进程的执行，直到**等待集合**（wait set）中的**一个子进程终止**。  
- 如果在调用 `waitpid` 时子进程已经终止，则该函数立即返回。  
- 以上两种情况子进程成功终止时 `waitpid` 返回令它返回的**子进程的 PID**，此时，**被终止的子进程已经被回收，内核会删除它在系统中的所有痕迹**。  
    
### Determining the Members of the Wait Set  
**等待集合（wait set）中的成员**由参数 `pid` 决定：  
- **pid > 0**  
等待集合只有**一个子进程**，该子进程的 PID 就是 pid 的值。  
- **pid = -1**  
等待集合中包含**父进程的所有子进程**。  
    
### Modifying the Default Behavior  
> [waitpid, wnohang, wuntraced. How do I use these](https://stackoverflow.com/questions/33508997/waitpid-wnohang-wuntraced-how-do-i-use-these)  
  
可用通过修改 `options` 参数为 **WNOHANG**,，**WUNTRACED** 和 **WCONTINUED** 的各种组合来修改默认行为：  
- **WNOHANG**  
如果目前在**等待集合**中**没有子进程终止**，则**立即返回 0**，而不用等待子进程终止。其他情况则和默认行为相同。  
    
- **WUNTRACED**  
将调用它的进程**挂起**（suspend）直到等待集合中有一个进程变成终止（terminated）或者停止（stopped）的状态，并返回造成该函数返回的子进程的 PID。  
    
- **WCONTINUED**  
将调用它的进程**挂起**（suspend）直到等待集合中有一个运行的进程变成终止（terminated）或者一个停止（stopped）的进程接收到 **SIGCONT** 信号而重新开始执行。  
    
- **WNOHANG | WUNTRACED**  
如果等待集合中没有终止或者停止的子进程，则立即返回0，否则返回已经终止或者停止的子进程的 PID。  
    
### Checking the Exit Status of a Reaped Child  
If the **statusp** argument is **non-NULL**, then **waitpid** encodes **status** information about the **child** that caused the return in **status**, which is the value pointed to by **statusp**. `wait.h` 中定义了几个宏来解释 **status** 参数：  
![status](https://img-blog.csdnimg.cn/c68f899df51343f98a25cf142f30d990.png)  
    
### Error Conditions  
如果调用 `waitpid` 函数的进程**没有子进程**，则 `waitpid` 函数返回 -1，并设置 `errno` 为 `ECHILD`；  
如果 `waitpid` 函数**被信号中断**，则返回 -1 并设置 `errno` 为 `EINTR`。  
    
### The wait Function  
The wait function is a simpler version of `waitpid`:  
![waitpid](https://img-blog.csdnimg.cn/2e9ffe55c1b74b3786047f83799d9e20.png)  
Calling `wait(&status)` is equivalent to calling `waitpid(-1, &status, 0)`.  
    
### Examples of Using waitpid  
示例1：  
![waitpid](https://img-blog.csdnimg.cn/71c99e9eb72f4b119a56e2f13c57b1eb.png)  
    
1. 11 行创建了 N 个子进程；  
2. 12 行每个子进程以不同的退出码退出；  
3. 15 行父进程调用 `waitpid` 来等待所有的子进程终止，**第一个参数** `pid` 为 -1，指**等待集合**中包含**所有的子进程**，**第二个参数** `statusp` 不为空，因此**返回信息**包含**子进程的状态信息**，**第三个参数** `options` 表示用**默认选择**，即**子进程终止才会返回**；  
4. 15 行第一个循环，返回子进程的 pid 为正数，进入循环，16 行检测状态，如果子进程是正常终止，则打印消息；  
5. 循环一直进行，直到**最后一个子进程终止**后，因为无子进程，15 行条件语句中 **pid** 返回 -1，因此跳出循环；  
6. 因为结束循环时 waitpid 调用无子进程，因此正常情况 `errno` 为 `ECHILD`；  
    
注意：这里**回收子进程没有顺序**的，也是无法预知顺序的。  
    
***************************  
    
**示例2：**  
![fig 8.19](https://img-blog.csdnimg.cn/59e512ba3f064d93984137d68647834d.png)  
图 8.19 与图 8.18 相比，变化是在 16 行，`waitpid` 的第一个参数不是 -1，而是一个特定子进程的 pid，因此等待集合只包含该子进程，保证了回收的顺序和创建的顺序一致。  
    
## 8.4.4 Putting Processes to Sleep  
The **sleep** function **suspends a process** for a **specified period of time**.  
![sleep](https://img-blog.csdnimg.cn/0b9f8a2d97fe43a8a7c4391586f20531.png)  
    
返回值：  
- **0**  
设定的时间已经达到  
- **剩下的秒数**  
设定的**时间未达到**，如 sleep 函数还未达到时间就被信号中断  
    
### pause  
**pause** 函数：让调用它的函数处于休眠（sleep）状态，直到进程接受到信号：  
```c  
#include <unistd.h>  
int pause(void);  //总是返回 -1  
```  
    
## 8.4.5 Loading and Running Programs  
### execve  
- The **execve** function **loads** and **runs** a **new program** in the **context** of the **current process**.  
- Overwrites code, data and stack  
- Retains PID, open files and signal context  
- Called once and never returns except if there is an error  
    
![execve](https://img-blog.csdnimg.cn/c5f2d18dbc72452e96871dd8f14a75c5.png)  
The **execve** function **loads** and **runs** the **executable object file** `filename` with the **argument list** `argv` and the **environment variable list** `envp`.  
    
**返回值**：成功则不返回，失败则返回 -1。  
    
第一个参数 `filename` 可以是二进制文件，或者以 `#!` 开头的脚本文件。  
    
第二个参数 `argv` 是一个指向数组的指针：  
![argv](https://img-blog.csdnimg.cn/91dca3e6ff474061be05c0a13d28aa7a.png)  
    
通常第一个元素 `argv[0]` 是可执行文件的名字。  
    
第三个参数环境变量 `envp` 也是一个指向数组的指针：  
![envp](https://img-blog.csdnimg.cn/4a05304929864106bba934744fbaecee.png)  
    
`envp` 的每个**数组元素**都是 `name=value` 的组合。  
    
在 `execve` 加载完文件后，调用 start-up 代码，最后将控制传递给主函数（执行 main 函数）。  
    
当 `main` 函数开始执行时，**栈**中的组织结构如下：  
![main](https://img-blog.csdnimg.cn/5cbde56cbba24849936b36f224201f9e.png)  
    
1. 栈的顶端是 start-up 函数 `libc_start_main` ；  
2. 然后是 `argv` 参数，每个元素 `argv[i]` 是一个指针，指针指向箭头所示的字符串区域；  
3. 接着是环境变量 `envp` ，也是指针，指向栈低端字符串区域；  
    
### getenv  
Linux provides several **functions** for manipulating the **environment array**:  
![getenv](https://img-blog.csdnimg.cn/47576333da254e82a021897914084f03.png)  
    
`getenv` 根据 `name` 看查找环境变量，找到则返回指向 `value` 的指针，否则返回空指针。  
    
### setenv  
![setenv](https://img-blog.csdnimg.cn/21eccdfb79964363bfdfc4107fdcc254.png)  
    
如果环境变量数组中有 `name=oldvalue` 形式，则 `unsetenv` 会删除它，然后 `setenv`  设置新名字，但需要 `overwrite` 参数不为 0；如果 `name` 不存在，则 `setenv` 则会在环境数组中添加一对 `name = newValue` 的组合。  
    
# 8.5 Signals  
A **signal** is a small **message** that notifies a **process** that an event of some type has occurred in the system.  
- **Akin** to **exceptions** and **interrupts**  
- **Sent** from the **kernel** (sometimes at the request of another process) to a **process**  
- **Signal type** is identified by **small integer IDs (1-30)**  
- Only **information** in a **signal** is its **ID** and the fact that it arrived  
- `SIGINT` 信号会中断键盘输入，可以按 `Ctrl` + `C` 发送该信号  
- linux 中用 `kill -L` 可查看信号类型  
    
![Signals](https://img-blog.csdnimg.cn/ad0c80b3190e450194e8faf940902dd2.png)<br/>  
    
## 8.5.1 Signal Terminology  
**Sending a signal:** The **kernel** **sends** (delivers) a **signal** to a **destination process** by **updating** some **state** in the **context** of the **destination process**.  
    
The signal is delivered for one of two **reasons**:  
- The **kernel** has **detected** a **system event** such as a divide-by-zero error or **the termination of a child process**.  
- **Another process** has invoked **kill** system call to **explicitly request the kernel to send a signal to the destination process**.  
    
**Receiving a signal:** A **destination process receives** a **signal** when it is **forced** by the **kernel** to react in some way to the **delivery** of the **signal**.  
     
Some possible ways to react:  
- **Ignore** the signal (do nothing).  
- **Terminate** the **process** (with optional core dump).  
- **catch** the **signal** by executing a user-level function called a **signal handler**.  
    
信号处理的过程：  
![信号处理](https://img-blog.csdnimg.cn/fc41a69fded1462fa41b5028d6078d47.png)  
    
**pending signal**: 已经发送但没有被接收的信号叫**待处理信号**（pending signal）。  
- 任何时候**一种类型最多只有一个待处理信号**  
- 如果一个进程已经有一个类型为 k 的待处理信号，则以后发送的类型为 k 的信号会被丢弃  
    
进程能选择性的 **阻塞（block）** 某个特定信号的接受，但一个信号被阻塞时，不会影响该信号的发送，但不会被接收，直到进程取消对其的阻塞。  
    
待处理信号最多只能被**接收**一次。  
    
**Kernel** maintains **pending and blocked bit vectors** in the **context** of each **process**.  
- **pending**: represent the set of **pending** signals  
	- Kernel **sets** bit k in **pending** when a signal of type k is **delivered**  
	- kernel **clears** bit k in **pending** when a signal of type k is **received**  
- **blocked**: represents the set of **blocked** signals  
	- Can be **set** and **cleared** by using the **sigprocmask** function  
	- Also referred to as the **signal mask**  
    
## 8.5.2 Sending Signals  
### Process Groups  
每个**进程**都属于一个**特定的进程组（process group）**，进程组由一个正整数 process group ID 标识。  
    
可以通过 `getpgrp` 函数来返回当前进程的 process group ID。  
```c  
#include <unistd.h>  
//Returns: process group ID of calling process  
pid_t getpgrp(void);  
```  
    
默认情况下，**子进程**和**父进程**属于同一个**进程组**，**进程组ID** 相同，但各自的**进程ID**不同。  
    
可以通过函数 `setpgid` 函数来修改自己或其他进程的进程组：  
```c  
#include <unistd.h>  
//Returns: 0 on success, −1 on error  
int setpgid(pid_t pid, pid_t pgid);  
```  
该函数将进程 `pid` 的进程组修改为 `pgid`，如果 `pid` 为0，则使用**当前进程的进程ID**（PID）；  
如果 `pgid` 为 0，则使用进程`pid` 的**PID**作为**进程组ID** 。  
    
例如：进程 15213 调用函数 `setpgid(0,0)`，则创建一个进程组ID为 15213 的进程组，并将进程 15213 加入到该组中。  
    
### Sending Signals with the /bin/kill Program  
The **/bin/kill** program **sends** an **arbitrary signal** to another **process**.  
    
下面命令发送信号 9（SIGKILL）到进程 15213：  
```bash  
linux> /bin/kill -9 15213  
```  
    
**PID为负数时**：下面命令发送信号 9（SIGKILL）到进程组 15213 中的每个信号：  
```bash  
linux> /bin/kill -9 -15213  
```  
    
### Sending Signals from the Keyboard  
Unix **shells** use the abstraction of a **job** to represent the **processes** that are created as a result of **evaluating a single command line**.  
    
任何时刻，最多有一个**前台作业（foreground job）** 和零个或多个**后台作业（background jobs）**。  
    
如：输入以下命令  
```bash  
linux> ls | sort  
```  
则会创建一个包含两个进程的前台作业，这两个进程由 Unix 管道（pipe）连接，一个运行 `ls` 程序，另一个运行 `sort` 程序。  
    
shell 为每个作业创建一个独立的进程组，进程组ID 为作业中的某个父进程的进程ID。  
![fig 8.28](https://img-blog.csdnimg.cn/0ad31c117f9b4a27961a7150e03d238a.png)  
    
按 `Ctrl+C ` 会造成内核发送 `SIGINT` 信号给前台进程组的每个信号，默认情况下将会 **终止（terminate）** 前台作业。  
    
按 `Ctrl+Z` 会造成内核发送 `SIGTSTP` 信号给前台进程组的每个信号，默认情况下将会 **暂停（stop）** 前台作业。  
    
### Sending Signals with the kill Function  
进程通过调用 `kill` 函数来发送信号到**自己和其他进程**。  
```c  
#include <sys/types.h>  
#include <signal.h>  
//Returns: 0 if OK, −1 on error  
int kill(pid_t pid, int sig);  
```  
1. `pid` 大于0，则该函数发送信号 `sig` 给进程ID为 `pid` 的进程 。  
2. `pid` 等于0，则该函数发送信号 `sig` 给**调用该函数的进程**的**进程组ID**中的**所有进程** （包括调用函数的进程）。  
3. `pid` 小于0，则该函数发送信号 `sig` 给**进程组ID**为 `-pid` 的所有进程 。  
    
### Sending Signals with the alarm Function  
进程可以通过调用 `alarm` 函数给自己发送 `SIGALRM` 信号。  
```c  
#include <unistd.h>  
//Returns: remaining seconds of previous alarm,  
//or 0 if no previous alarm  
unsigned int alarm(unsigned int secs);  
```  
`alarm` 函数让内核在 `secs` 秒后给调用它的进程发送 `SIGALRM` 信号。  
    
If **secs** is **0**, then **no new alarm** is scheduled.  
    
In any event, the **call to alarm** **cancels** any **pending alarms** and **returns** the **number of seconds remaining** until any pending alarm was due to be **delivered** (had not this call to alarm canceled it), or **0** if there were no pending alarms. （[alarm函数使用方法](https://blog.csdn.net/tjh1998/article/details/123680438)）  
    
## 8.5.3 Receiving Signals  
当**内核**将**进程 p** 从**内核模式**切换到**用户模式**时，会检测该进程的**未阻塞的待处理信号（unblocked pending signals）**。  
- 如果**没有符合条件的信号**，则内核将控制传给进程 p 的**逻辑控制流**中的**下一条指令**；  
- **如果有符合条件的信号**，则内核选择其中一个信号 k （通常是该信号集合中最小的数）然后强迫进程**接收**信号 k。  
进程接收信号后会触发某种**行为（action）**。  
然后对集合中剩下的信号重复上述操作，直到集合为空。（这个步骤书里没写，视频里讲的，所以会将集合中全部的信号都处理？）  
进程完成上述行为后，控制将会回到进程 p 逻辑控制流的下一条指令。  
    
**每种类型的信号都有预定的默认的行为**（见前面图 8.26）：  
- 终止（terminate）进程  
- 终止（terminate）进程并 [dumps core](https://baike.baidu.com/item/%E6%A0%B8%E5%BF%83%E8%BD%AC%E5%82%A8/16772089?fr=aladdin)  
- 暂停（stop）进程直到被信号 SIGCONT  重启  
- 忽略信号  
    
调用 `signal` 函数可以修改信号的默认行为（SIGSTOP 和 SIGKILL 两种信号除外）：  
```c  
#include <signal.h>  
typedef void (*sighandler_t)(int);  
    
//Returns: pointer to previous handler if OK,  
//SIG_ERR on error (does not set errno)  
sighandler_t signal(int signum, sighandler_t handler);  
```  
`signal` 函数改变信号 `signum` 的行为：  
- 如果 `handler` 是 `SIG_IGN`，则忽略类型为 `signum` 的信号  
- 如果 `handler` 是 `SIG_DFL`，则类型为 `signum` 的信号使用默认行为  
- 其他情况，`handler` 是一个**用户定义函数**的地址，称为 **信号处理程序（signal handler）**，当进程接收到类型为 `signum` 的信号时将调用该程序。  
    
**设置信号处理程序（installing the handler）**：通过传递处理程序的地址给信号函数来改变信号的默认行为。  
    
**捕获信号（catching the signal）**：调用信号处理程序。  
    
**处理信号（handling the signal）**：执行处理程序。  
    
示例：图 8.30 修改信号 `SIGINT` 的行为：  
![fig 8.30](https://img-blog.csdnimg.cn/a449d73c15e444ccaa9be57121e1b46b.png)<br/>  
    
*******************************  
    
图 8.31 展示信号处理程序被其他处理程序中断的例子：  
![fig 8.31](https://img-blog.csdnimg.cn/6372daf33703401ab3cf2a333f8af59d.png)  
    
## 8.5.4 Blocking and Unblocking Signals  
Linux 提供 **隐式（implicit）** 和 **显示（explicit）** 的机制来**阻塞信号**。  
    
- **Implicit blocking mechanism:** 默认情况下，内核会阻塞**当前处理程序正在处理**的**同类型**的**待处理信号**。  
- **Explicit blocking mechanism:** 应用程序能通过 `sigprocmask` 函数和它的辅助函数来**显示**的**阻塞**或者**解除阻塞**选择的信号。  
    
**显示阻塞机制**：  
![Explicit blocking mechanism](https://img-blog.csdnimg.cn/ddfa9d5e693646188f688bfd756f5fe5.png)  
    
`sigprocmask` 函数改变当前阻塞信号的集合（8.5.1 中讲的 blocked bit vector），具体行为依赖 `how` 的值：  
- **SIG_BLOCK**：将 `set` 中的信号添加到 blocked bit vector 中（(blocked = blocked | set)）  
- **SIG_UNBLOCK**：将 `set` 中的信号从 blocked bit vector 中移除（blocked = blocked & ~set）  
- **SIG_SETMASK**：blocked = set  
    
如果 `oldset` 是非空的值，则之前的 blocked bit vector 的值会存在 oldset 中。  
    
**操作信号`set`**：  
- `sigemptyset` 函数将 `set` 初始化为空集合  
- `sigfillset` 函数将每个信号添加到 `set` 集合中  
- `sigaddset` 函数将 `signum` 添加到 `set` 集合  
- `sigdelset` 函数从 `set` 集合中删除信号 `signum`  
- `sigismember` 函数当信号 `signum` 在 `set` 集合中时返回 1，否则返回 0  
    
示例：  
![fig 8.32](https://img-blog.csdnimg.cn/f912f547e2904b52aae24ce8c16064d0.png)<br/>  
    
## 8.5.5 Writing Signal Handlers  
### Safe Signal Handling  
**Handlers** run **concurrently** with the **main program** and share the **same global variables**, and thus can **interfere** with the **main program** and with **other handlers**.  
    
写安全的信号处理程序：  
- 处理程序越简单越好  
- 只调用**异步信号安全函数（async-signal-safe functions）**，这类函数是**可重入的（reentrant）**，也**不能被信号处理函数中断。**  
- 保存和恢复 `errno`，许多 Linux 的异步信号安全函数在返回错误时会设置 `errno`，在处理程序内部调用这类函数可能干扰其他依赖 `errno` 的程序；针对这种情况，可以在进入处理程序时保存 `errno` 到一个局部变量中，然后在处理程序返回前恢复它的值。这种方式只针对**处理程序会返回**的情况。  
- 通过**阻塞所有信号**来保护**共享全局数据结构**的访问，如果**处理程序**和**主程序**或者**其他处理程序**共享某个**全局数据结构**，则在访问（读或写）该数据结构时，处理程序和主程序等共享数据的程序应该**临时的阻塞所有的信号**。  
- 用 `volatile` 来声明全局变量，强迫编译器每次引用全局变量时**从内存中读值**。  
- 用 `sig_atomic_t` 声明标志，通常处理程序会写**全局标志（global flag）** 来记录信号的接收，主程序会周期性的读这个标志，响应信号，清除标志。对于这种方式共享的标志，C 提供了一个整数 `sig_atomic_t` 来实现 **原子的（atomic）** 读写，因为该过程可以用一条指令 `volatile sig_atomic_t flag;` 实现。  
    
**异步信号安全函数（async-signal-safe functions）**：  
![fig 8.33](https://img-blog.csdnimg.cn/d9be35b3f98f4de6bc47f01399486bfa.png)<br/>  
    
### Correct Signal Handling  
前面提到过 `pending bit vector` 中每种类型的信号只会包含一个，后面如果又有同类型的待处理信号会被丢弃，因此一个待处理的信号只能表示**至少有一个**该类型的信号到达。  
    
示例：  
![fig 8.36](https://img-blog.csdnimg.cn/34ed3b9fd62549be9cf3448a0957cfd1.png)  
    
上面例子输出结果如下：  
![](https://img-blog.csdnimg.cn/27fabef276e746768f1231fa7e4566f8.png)<br/>  
    
当子进程终止或者暂停时，内核会给父进程发送 `SICHILD` 信号，上述代码中用 `signal` 函数来修改父进程接收到 `SICHILD` 信号时的行为，用信号处理程序 handler1 来回收终止的子进程，在 handler1 中，`waitpid` 用 `if` 语句，即**只处理一个终止的子进程**。  
    
如果第一个子进程终止发送信号后由 handler1 正在处理，此过程中第二个子进程也终止发送了信号，因此成为待处理信号，然后第三个子进程也终止，由于第二个子进程还未处理，**该信号被丢弃**。  
    
因此第三个信号不会被回收。  
    
修改信号处理程序如下：  
![fig 8.37](https://img-blog.csdnimg.cn/fd054036c05544ebaf7ee4cf7f5772f8.png)  
    
变化是 `waitpid` 调用编程 `while` ，因此只要有终止的子进程，就处理，尽可能多的处理终止的子进程。  
    
### Portable Signal Handling  
不同的系统有不同的信号处理语义：  
- **The semantics of the signal function varies**  
有些老的 Unix 系统在信号 k 被信号处理程序捕获后会将信号的行为恢复为默认行为，而在其他系统，处理程序必须调用 `signal` 函数来显式的设置。  
    
- **System calls can be interrupted**  
**慢速系统调用（slow system call）**：一些系统调用如 `read，wait 和 accept` 会潜在的阻塞进程很长时间。在一些老的 Unix 版本，当处理程序捕获信号时，被中断的慢速系统调用在处理程序返回时不会继续执行，而是立即返回错误条件，因此程序员必须手动重启被中断的系统调用。  
    
可以使用 `Signal` 函数来解决上述问题：  
![fig 8.38](https://img-blog.csdnimg.cn/15261b6389e34bf6bc1fb1922ca88b0e.png)  
    
`Signal` 函数的调用方式和 `signal` 函数相同。  
    
其信号处理语义为：  
- 只有当前被处理的信号类型才会阻塞  
- 信号不会排队等待（前面提过）  
- 被中断的系统调用会尽可能的自动重启  
- 一旦设置了信号处理程序，将一直保持该设置，直到调用 `Signal` 函数且参数 `handler`  是 `SIG_IGN` 或 `SIG_DFL`。  
    
## 8.5.6 Synchronizing Flows to Avoid Nasty Concurrency Bugs  
第12章介绍。  
    
书中例子讲解见视频课程。  
    
## 8.5.7 Explicitly Waiting for Signals  
有时候主程序需要显式的等某个特定的信号处理函数运行。  
    
示例：  
![fig 8.41](https://img-blog.csdnimg.cn/18c4fe35022343869be7d09e02bd5a49.png)  
    
**5-10行**：信号 `SIGCHLD` 的处理程序，当子程序终止时，设置 `pid` 的值为子进程的 `PID`；  
    
**20-21行**：分别为信号 `SIGCHLD` 和  `SIGINT` 设置两个信号处理程序；  
    
**23行**：将 `SIGCHLD` 信号放到 `mask` 信号组中；  
    
**26行**：阻塞信号组 `mask` 中的所有信号，此时 `mask` 中的信号为 `SIGCHLD`；  
    
**27-28行**：创建一个子进程，如果失败则退出；  
    
**31行**：设置全局变量的值 `pid` 为 0；  
    
**32行**：将信号组 `prev` 的信号设置为 `blocked bit vector` 中的值，因为 `prev` 组中没有 `SIGCHLD` 信号，因此该信号被解除阻塞；  
    
**35-36行**：31 行设置了 `pid` 为0，因此在 32 行解除`SIGCHLD` 信号阻塞后，如果未回收子进程，即`SIGCHLD` 的处理程序未执行，则会一直执行 `while` 循环，等待子进程回收，这是 `pid` 大于 0，因此退出循环；  
    
**分析**：该程序无问题，但**35-36行**的 `spin loop` 浪费资源。  
    
**解决方案**：  
**方案一：** 如果将 **35-36行** 替换为：  
```c  
while (!pid) /* Race! */  
	pause(); //8.4.4 节介绍 pause 函数  
```  
`pause` 函数在 8.4.4 节有介绍：该函数会令当前进程暂停直到被信号中断。  
[C语言中的pause()函数和alarm()函数以及sleep()函数](https://www.jb51.net/article/71841.htm)  
    
可能遇到的问题：进入循环后，但在执行 `pause()` 前，接收到 `SIGCHLD` 信号，则 `pause` 可能会一致被阻塞，除非有 `SIGINT` 信号，此时 `pid` 非零，跳出循环。  
    
**方案二：** 如果将 **35-36行** 替换为：  
```c  
while (!pid) /* Race! */  
	sleep(1); //8.4.4 节介绍，休眠 1 秒钟  
```  
不会有 `pause` 函数出现的问题，但需要选择一个合适的休眠时间。  
    
**方案三：** 使用 `sigsuspend`  
```c  
#include <signal.h>  
//Returns: −1  
int sigsuspend(const sigset_t *mask);  
```  
该函数等价于一个**原子的（atomic）** 版本：  
```c  
sigprocmask(SIG_BLOCK, &mask, &prev);  
pause();  
sigprocmask(SIG_SETMASK, &prev, NULL);  
```  
在 `pause` 前阻塞所有的信号，因此 `pause` 不会被中断。  
    
# 8.6 Nonlocal Jumps  
（没仔细看，视频没讲这一部分）  
    
C 提供一种**用户级别**的**异常控制流**，成为**非本地跳转（nonlocal jump）**，它能将控制直接从一个函数转移给另一个当前执行的函数，而不需要经过正常的 `call-and-return` 步骤。  
    
非本地跳转有 `setjmp` 和 `longjmp` 函数提供。  
    
```c  
 #include <setjmp.h>  
int setjmp(jmp_buf env);  
int sigsetjmp(sigjmp_buf env, int savesigs);  
//Returns: 0 from setjmp, nonzero from longjmps  
```  
`setjmp` 函数保存 `env` 缓冲区的当前 `calling environment`，然后给 `longjmp` 使用，并返回 0.  
    
The **calling environment** includes the program counter, stack pointer, and general-purpose registers.  
    
The value that **setjmp** returns **should not be assigned to a variable**.  
    
`longjmp` 函数：  
```c  
#include <setjmp.h>  
void longjmp(jmp_buf env, int retval);  
void siglongjmp(sigjmp_buf env, int retval);  
//Never returns  
```  
`longjmp` 函数从`env` 缓冲区恢复 `calling environment`，然后触发一个从最近的初始化 `env` 的 `setjmp` 调用的返回，然后 `setjmp` 返回一个非零值 `retval`。  
    
The **setjmp** function is **called once** but **returns multiple times**: once when the **setjmp** is first called and the calling environment is stored in the **env** buffer, and once for each corresponding **longjmp** call.  
    
On the other hand, the **longjmp** function is called once but **never returns**.  
    
**应用：**  
- 从一个多级嵌套的函数调用中立即返回。  
-  Branch out of a signal handler to a specific code location, rather than returning to the instruction that was interrupted by the arrival of the signal.  
    
# 8.7 Tools for Manipulating Processes  
Linux 系统提供了大量监测和操作进程的有用工具：  
![Tools for Manipulating Processes](https://img-blog.csdnimg.cn/791f78ce5bd64d6fb5f9de6742eb9cd1.png)<br/>  
