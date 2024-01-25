深入理解计算机系统——第四章 Processor Architecture  
  
参考资源：  
> [四、处理器体系结构](https://zhuanlan.zhihu.com/p/478993532)  
> [处理器体系架构（Y86系统）](https://blog.csdn.net/tangfatter/article/details/122182118?spm=1001.2014.3001.5502)  
  
本章主要介绍处理器硬件的设计，并定义了一套简单的指令集 **“Y86-64”** ，便于学习。   
  
We call this the **“Y86-64”** instruction set, because it was inspired by the **x86-64** instruction set.   
Compared with **x86-64**, the **Y86-64** instruction set has fewer data types, instructions, and addressing modes.  
  
# 4.1 The Y86-64 Instruction Set Architecture  
We have seen that a **processor** must **execute a sequence of instructions**, where each **instruction** performs some **primitive operation**, such as adding two numbers.   
  
An **instruction** is encoded in **binary form** as a sequence of **1 or more bytes**.   
The **instructions** supported by a **particular processor** and their `byte-level` **encodings** are known as its `instruction set architecture` (ISA).   
  
Different “families” of processors, such as Intel `IA32` and `x86-64`, `IBM/Freescale Power`, and the `ARM` processor family, have **different** `ISAs`.   
  
**A program compiled for one type of machine will not run on another**.   
  
On the other hand, there are many **different models of processors** within **a single family**.   
  
Each manufacturer produces processors of ever-growing performance and complexity, but the **different models** remain **compatible** at the `ISA` level.   
  
Defining an instruction set architecture, such as `Y86-64`, includes defining the different components of its state, the set of instructions and their encodings, a set of programming conventions, and the handling of exceptional events.  
  
## 4.1.1 Programmer-Visible State  
Each instruction in a `Y86-64` program can **read** and **modify** some part of the **processor state**.   
  
This is referred to as the `programmer-visible state`, where the **“programmer”** in this case is either **someone writing programs in assembly code** or a **compiler generating machine-level code**.  
  
There are **15** program registers: `%rax, %rcx, %rdx, %rbx, %rsp, %rbp, %rsi, %rdi, and %r8 through %r14`. (We omit the `x86-64` register `%r15` to simplify the instruction encoding.)   
  
Each of these stores a `64-bit word`.   
  
Register `%rsp` is used as a **stack pointer** by the push, pop, call, and return instructions.   
  
**Otherwise, the registers have no fixed meanings or values.**   
  
There are **three single-bit condition codes**, `ZF, SF, and OF`, storing information about the effect of the **most recent arithmetic or logical instruction**.   
  
**The program counter (PC)** holds the address of the instruction **currently** being executed.  
  
The **memory** is conceptually a **large array of bytes**, holding both **program and data**.   
  
`Y86-64` programs **reference memory** locations using `virtual addresses`. （第九章讲虚拟地址）  
  
A final part of the **program state** is a status code `Stat`, indicating the overall state of program execution.   
正常或者异常的状态。  
  
见下图：  
![Y86-64 programmer visible state](https://img-blog.csdnimg.cn/0352d65ab3f24fc7bcf3fa1704591505.png)<br/>  
  
## 4.1.2 Y86-64 Instructions  
- The `x86-64` `movq` instruction is split into **four different instructions**: `irmovq, rrmovq, mrmovq, and rmmovq`, explicitly indicating the form of the source and destination. The source is either **immediate** （i）, **register**  （r), or **memory**（m).  
  
- There are **four integer operation instructions**. These are `addq, subq, andq, and xorq`. They operate only on **register** data, whereas `x86-64` also allows operations on **memory** data. These instructions set the **three condition codes** `ZF, SF, and OF` (zero, sign, and overflow).  
  
![Y86-64 instruction set](https://img-blog.csdnimg.cn/aba868c6249b4cfeb16e0079eb0b9326.png)  
  
- The **seven jump instructions** (shown in Figure 4.2 as `jXX`) are `jmp, jle, jl, je jne, jge, and jg`.   
  
- There are **six conditional move instructions** (shown in Figure 4.2 as `cmovXX`): `cmovle, cmovl, cmove, cmovne, cmovge, and cmovg`.   
  
- The `call` instruction **pushes the return address on the stack** and **jumps to the destination address**. The `ret` instruction returns from such a `call`.  
  
- The `pushq and popq` instructions implement `push and pop`, just as they do in `x86-64`.  
  
- The `halt` instruction **stops instruction execution**. For `Y86-64`, executing the `halt` instruction causes the **processor to stop**, with the `status code` set to `HLT`.   
  
## 4.1.3 Instruction Encoding  
Each instruction requires between  `1 and 10 bytes`, depending on which fields are required. （见图 4.2 所示）  
  
Every instruction has an **initial byte** identifying the `instruction type`. This byte is split into `two 4-bit parts`: the `high-order`, or `code`, part, and the `low-order`, or `function`, part.   
4.1.2 中图 4.2 所见第一个字节，高4位为指令代码（范围为 `0 - 0xB`），低4位为指令功能。如 `halt` 的第一个字节，高四位和低四位都为 `0`。  
  
关于**第一个字节低四位**的**功能部分（fuction part）**，只有当有一系列相关的有 **相同指令代码（code part）** 片段时才有意义。  
见下图 4.3 所示，展示的为第一个字节，最左边对应图 4.2 中 `QPq` 指令，四个指令的高四位均为 6，低四位不同代表不同的功能，另外两个分支指令和Move指令类似。  
  
![Function codes for Y86-64 instruction set](https://img-blog.csdnimg.cn/e0f8317f82b04c1ab46db021bd1d4efc.png)  
<br/>  
  
每个寄存器都有个`ID (register identifier)` 用来表示寄存器的类型，其中 `oxF` 表示没有寄存器。见下图 4.4 所示：  
  
The numbering of registers in `Y86- 64` matches what is used in `x86-64`. The program **registers** are **stored** within the `CPU` in a `register file`, a small `random access memory` where the **register IDs serve as addresses**.   
  
![Y86-64 program register identifiers](https://img-blog.csdnimg.cn/f769d602f1ef4080a5b250471748381e.png)  
<br/>  
  
*********************************  
有些指令需要额外的寄存器，见 图 4.2，如 `rrmovq` 需要额外两个寄存器，一个存放源数据（rA），另一个存放目标数据（rB），而对于寄存器 `irmovq`，由于源数据是立即数，无寄存器，因此第二个字节的高四位区域为 `0xF` ，表示无寄存器，低四位区域称为 `rB`，该字节区域为 `register specifier byte`，位于第一个字节后。  
  
**********************************  
Some instructions require an **additional 8-byte constant word**. This word can serve as the **immediate data** for `irmovq`, the **displacement** for `rmmovq` and `mrmovq` address specifiers, and the `destination` of `branches` and `calls`.   
  
Note that `branch` and `call` `destinations` are given as **absolute addresses**, rather than using the `PC-relative addressing` seen in `x86-64`.   
  
As with `x86-64`, all integers have a **little-endian encoding**.  
  
有些指令需要额外的 8 字节，如图 4.2 中指令 `irmovq` 的第 3 ～ 10 个字节区域来存放立即数。  
****************************  
  
**示例**：指令 `rmmovq %rsp,0x123456789abcd(%rdx)` （十六进制）  
1. 根据图 4.2，第一个字节表示指令类型，值为 `40`；  
  
2. 第二个字节中，高四位 `rA` 区域为源寄存器，即 `%rsp`，从图 4.4 可以找到其 ID 为 4，而低4位 `rB` 区域为基址寄存器 `%rdx`，其 ID 为 2，因此第二个字节为 `42`；（如果还有变址寄存器和比例因子呢？）  
  
3. 最后的 8 个字节区域存放常数 `0x123456789abcd`，将高位填充 `0`，即为 `00 01 23 45 67 89 ab cd`，写成小端序为 `cd ab 89 67 45 23 01` 。  
  
4. 综上，最后得到该指令的二进制码为 `4042cdab896745230100`。  
  
## 4.1.4 Y86-64 Exceptions  
图 4.1 中可知 `programmer-visible state for Y86-64` 有个状态码 `Stat` 用来描述程序执行的状态，其可能的值如下图所示：  
![Y86-64 status codes](https://img-blog.csdnimg.cn/11fd767d6ba74f37a273c7b7afe5ad37.png)  
  
对于 `Y86-64`, 设计当处理器遇到异常时停止执行。  
  
但在更复杂的设计中，处理器遇到异常时会调用异常处理程序（ invoke an exception handler）来处理特定的异常。(第八章介绍)  
  
# 4.2 Logic Design and the Hardware Control Language HCL  
  
## 4.2.1 Logic Gates  
逻辑门是数字电路中的基本计算元件：  
  
![Logic gate types](https://img-blog.csdnimg.cn/fc917de85e3b44ceaddc7f2777cfc2ab.png)  
## 4.2.2 Combinational Circuits and HCL Boolean Expressions  
多个逻辑门组合起来构成一个组合电路，特点如下：  
- 每个逻辑门的输入必须连到以下三种中的一种：  
(1) one of the system inputs (known as a primary input)  
(2) the output connection of some memory element  
(3) the output of some logic gate  
  
- 多个逻辑门的输出不能连在一起。  
  
- The network must be acyclic. That is, there cannot be a path through a series of gates that forms a loop in the network. Such loops can cause ambiguity in the function computed by the network。  
  
示例1：  
![Combinational Circuits ](https://img-blog.csdnimg.cn/b1d7962e0d4b406d90977edbba5b7153.png)  
上述组合电路实现的功能为：如果两个输入相等则输出 1，否则输出 0。  
用 `HCL`  语言描述如下：  
```hcl  
bool eq = (a && b) || (!a && !b);  
```  
Unlike C, however, we do not view this as performing a computation and assigning the result to some memory location. Instead, **it is simply a way to give a name to an expression**.  
  
  
示例2 ：  
![single-bit multiplexor](https://img-blog.csdnimg.cn/aaf46ff96c854e6eaf635abc13c4cda2.png)  
该组合电路实现的功能为 `single-bit` 多路复用器，`s` 为控制信号，当 `s` 为 `1` 时，输出的结果为 `a`，当 `s`  为 `0` 时，输出的结果为 `b`。  
  
`HCL` 语言描述为：  
```hcl  
bool out = (s && a) || (!s && b);  
```  
  
注意 `HCL` 语言和 `C` 语言区别：C 语言逻辑表达式可能只需要判断部分就能得出结论，那么剩下的表达式不会执行，而组合逻辑没有该规则，不会只判断部分。  
  
## 4.2.3 Word-Level Combinational Circuits and HCL Integer Expressions  
Figure 4.12 shows a combinational circuit that tests whether **two 64-bit words** `A` and `B` are **equal**.   
  
![Word-level equality test circuit](https://img-blog.csdnimg.cn/783a5b09fd4e4f19b8177251e7d44a4a.png)  
  
对上图，如果 `A` 的每个位都和 `B` 的相应的位相等，则 `A` 和 `B` 相等，输出位 1。  
  
In `HCL`, we will declare any **word-level signal as an int**, without specifying the word size.   
  
图 4.12 可以表示为 `bool Eq = (A == B);`  
  
## 4.2.5 Memory and Clocking  
`Combinational circuits`, by their very nature, **do not store any information**. Instead, they simply **react to the signals** at their inputs, **generating outputs** equal to some function of the inputs.   
  
To create **sequential circuits**—that is, systems that have **state** and perform **computations** on that state—we must introduce devices that **store information represented as bits**.   
  
Our storage devices are all controlled by a **single clock**, a **periodic signal** that determines when **new values** are to be **loaded** into the devices.   
  
We consider two classes of **memory devices**:  
- **Clocked registers** (or simply **registers**) store **individual bits or words**. The clock signal controls the loading of the register with the value at its input.  
  
- **Random access memories** (or simply **memories**) store **multiple words**, using an **address** to select **which word should be read or written**.   
  
Examples of **random access memories** include (1) **the virtual memory system** of a processor, where a combination of hardware and operating system software make it appear to a processor that it can access any word within a large address space; and (2) **the register file**, where **register identifiers** serve as the **addresses**.   
  
In a `Y86-64` processor, the **register file** holds the 15 program registers (`%rax` through `%r14`).  
*********************  
  
As we can see, the word **“register”** means two slightly different things when speaking of **hardware** versus **machine-language programming**.   
  
- **In hardware**, a **register** is directly connected to the rest of the circuit by its input and output wires.   
  
- **In machine-level programming**, the **registers** represent a small collection of **addressable words** in the `CPU`, where the addresses consist of **register IDs**.  
  
  
见图 4.16 所示，对于硬件的寄存器，大多数时间会保持固定的状态（图中的 x），其输出即为当前的状态。  
只要始终信号处于低电平（0），则寄存器的状态将保持不变。  
当时钟信号处于高电平（1），寄存器的输入信号将传输到寄存器中更新其状态（y），因此其输出的结果更新为 y。  
  
![Register operation](https://img-blog.csdnimg.cn/dc7452d3cb124477b6499a6375d4fae4.png)  
<br/>  
  
Our `Y86-64 processors` will use `clocked registers` to hold the `program counter (PC)`, the `condition codes (CC)`, and the `program status (Stat)`.  
  
典型的寄存器文件如下：  
![typical register file](https://img-blog.csdnimg.cn/b4980d23309b48379cee845f9f687bc9.png)  
  
This `register file` has **two read ports**, named `A` and `B`, and one **write port**, named `W`. Such a multiported random access memory allows **multiple read and write operations to take place simultaneously**.   
  
如果将 `srcA` 的值设置为 `3`，根据图 4.4，则会读寄存器 `%rbx`，而该寄存器的值将会作为 `valA` 的结果输出。  
  
至于写输入到寄存器文件中，每次当**时钟信号处于高电平**时，输入到 `valW` 接口的数据会写入接口 `dstW` 接口的数值（寄存器ID）所代表的寄存器中。如果 `dstW` 的值位 `oxF` （无寄存器）,则不会写入数据。  
  
如果寄存器文件同时读写同一个寄存器，则当时钟信号处于高电平时，读端口的输出数据从旧值变为新值。  
  
# 4.3 Sequential Y86-64 Implementations  
As a first step, we describe a processor called `SEQ` (for “sequential” processor). On each clock cycle, `SEQ` performs all **the steps** required to process a **complete instruction**.  
  
因此这种设计每个时钟周期很长，时钟频率低。  
  
## 4.3.1 Organizing Processing into Stages  
- **取指令（Fetch）**  
根据**程序计数器**的值作为**内存地址**，从**内存中读取指令**。获取指令后，将提取指令的**第一个字节**（图 4.2 和 图4.3 中介绍），高4位指定代码 `icode`，低四位为指令功能 `ifun`，也可能根据需要用到后面字节的功能（见前面 4.1 节介绍），然后计算`valP`的值（程序计数器 PC的值 + 获取指令的长度）。  
  
- **译码（Decode）**  
译码阶段从寄存器文件读最多两个操作数，操作数的数目根据指令功能不同，有的指令有源操作数和目的操作数（rA 和 rB），然后获取相应寄存器的值 `valA and/or valB`。（见4.2.5 节寄存器文件介绍，和后面的例子）  
  
- **执行（Excute）**  
算数逻辑单元（ALU）根据指令（ifun 的值）执行操作，计算内存引用的有效地址，或者增加或减小栈指针。  
  
- **访存（Memory）**  
向内存中读或写数据，读的数据为 `valM`。  
  
- **写回（Write back）**  
将执行的结果写到寄存器文件中。  
  
- **更新程序计数器（PC update）**  
将程序计数器的值设置为下条指令的地址。  
  
*********************  
为了简化硬件电路的复杂性，在设计时尽量让更多的不同指令共享硬件。例如每个处理器只有一个算术逻辑单元，可以处理不同类型的指令。  
  
Figure 4.17 to illustrate the processing of different `Y86-64` instructions.  
  
![Sample Y86-64 instruction sequence](https://img-blog.csdnimg.cn/2cc13f10f94b4de9ae5644520866a1bb.png)  
<br/>  
  
***************  
图 4.18 展示了三种类型指令的处理过程。  
  
![Computations in sequential implementation of Y86-64 instructions](https://img-blog.csdnimg.cn/5065b73b0ff74e2eb4c6cb05c8ed921a.png)  
<br/>  
  
整数操作指令（`OPq rA, rB`）：  
- **取指令**：根据 4.1.2 的内容可知，第一个字节内容为 `icode:ifun`，而四个`整型操作（`addq, subq, andq, and xorq`）有相同的 `icode` z（6），只需要两个字节，无常数，因此 `valP` 的值为 `PC + 2`。  
- **译码**：读取两个操作数，分别将`rA` 和 `rB` 代表的寄存器的值分别赋给 `valA` 和 `valB`。  
- **执行**：根据 `ifun` 中代表的操作，对两个操作数执行计算，并将结果赋给 `valE`，并设置条件码 `CC`（`ZF, SF, OF`）。  
- **写回**：将结果 `valE` 写到 `rB` 所代表的寄存器中。（第三章的 图 3.10 可知四种操作的结果更新为目标操作数 rB 的值）  
- **更新程序计数器**：将 `valP` 的值赋给 `PC`。  
  
`rrmovq` 指令：  
**取指令**过程和上述整数操作相同，**译码阶段**，只需 `rA` 代表的寄存器中的值，**执行**时将 `valA` 的值加 `0` 作为结果，然后再**写回**时将结果赋给 `rB` 代表的寄存器，实现将一个寄存器的结果赋值给另一个寄存器。  
  
`irmovq` 指令：  
**取指**过程中增加常数存放区域，将**常数**赋值给 `valC`，`valP` 的值为 `PC + 10`（多了 8 字节的常数区域）。 然后在**执行**的阶段直接将**常数**加 `0` 作为结果，在**写回**阶段将结果赋值给 `rB` 代表的寄存器，最后更新 `PC` 的值。  
  
<br/>  
  
**********************  
  
`rmmovq` 和 `mrmovq` 的操作过程如下：  
  
![Computations in sequential implementation of Y86-64 instructions](https://img-blog.csdnimg.cn/1296e9c8205a4cd9a2ecc7e769b57254.png)<br/>  
  
和前面 `rrmovq` 的区别是在`访存` 阶段多了向**内存**中**写入**或者**读数据**的过程，内存地址为 `valC`，该值保存在常量区域，起始地址为指令的第三个字节开始处，大小为 8 字节（$M_8[PC+2]$）。  
  
疑问：这里 `rmmovq` 中的 rB 是基址寄存器，如果还有变址寄存器呢？  
	  
<br/>  
  
 ************************  
   
`push` 和 `popq` 的操作过程如下：  
  
![Computations in sequential implementation of Y86-64 instructions](https://img-blog.csdnimg.cn/83693234bd464f139877c8cc8ded9253.png)<br/>  
  
和前面的区域之处是在译码阶段用到了栈指针 `%rsp`，执行阶段将栈指针的值`减8`（向下扩展栈空间，`pushq`）或者`加8`（向上缩减栈空间，弹出数据 `popq`）。  
  
<br/>  
  
**********************  
控制转移指令 `jumps, call, 和 ret`：  
  
![Computations in sequential implementation of Y86-64 instructions ](https://img-blog.csdnimg.cn/58097993531c44459d0a278cdceeac66.png)<br/>  
  
跳转指令 `jXX` 不需要 `register specifier byte` （`rrmovq` 指令第二个字节的内容），因此第二个字节存放常数；  
在执行阶段，检查条件码 `CC` 和 `ifun` 中指令的功能来判断是否满足条件决定是否跳转，结果为 `1-bit` 的信号 `Cnd`；  
最后在更新计数器阶段，如果 `Cnd`为 `1`，则跳转到 `valC` 处（跳转分支的位置）,否则执行下一条指令 `valP`。  
  
调用和返回指令 （`call` 和 `ret`）：需要将下条要执行的指令地址压入或弹出栈。  
  
## 4.3.2 SEQ Hardware Structure  
In `SEQ`, all of the processing by the **hardware units** occurs within a **single clock cycle**.  
  
下面流程图的操作过程是从下向上的过程。  
  
![Abstract view of SEQ, a sequential implementation](https://img-blog.csdnimg.cn/94c6cd815f1b4b49ae3af3be97956eed.png)<br/>  
   
Figure 4.23 gives a **more detailed view of the hardware** required to implement `SEQ` :  
  
![Hardware structure of SEQ, a sequential implementation](https://img-blog.csdnimg.cn/5717d8ac74d4489697fc090f6845fcf1.png)  
## 4.3.3 SEQ Timing  
Our implementation of `SEQ` consists of **combinational logic and two forms of memory devices**: `clocked registers` (**the program counter and condition code register**) and `random access memories` (the **register file**, the **instruction memory**, and the **data memory**).   
  
`The program counter` is **loaded** with a **new instruction address** every **clock cycle**.   
  
`The condition code register` is **loaded** only when an `integer operation instruction is executed`.   
  
`The data memory` is **written** only when an `rmmovq, pushq, or call instruction is executed`.  
  
`The two write ports of the register file` allow `two program registers` to be **updated** on **every cycle**, but we can use the special register ID `0xF` as a port address to indicate that no write should be performed for this port. (rA 和 rB)  
  
  
**principle: No reading back**  
The processor never needs to read back the `state` updated by an instruction in order to complete the processing of this instruction.  
   
The color coding in **Figure 4.25** indicates how the circuit signals relate to the different instructions being executed.  
  
![Tracing two cycles of execution by SEQ](https://img-blog.csdnimg.cn/602856e757434c60a5a48e9665b1bed4.png)  
## 4.3.4 SEQ Stage Implementations  
`HCL` 语言使用的常量：  
  
![Constant values used in HCL descriptions](https://img-blog.csdnimg.cn/5d03698c229b44eeac70b29c06bd3d2c.png)  
- **Fetch Stage**  
  
![SEQ fetch stage](https://img-blog.csdnimg.cn/e7f423fc14a94fb9995a4f7cdb9ba6ab.png)  
- **Decode and Write-Back Stages**  
The **register file** has **four ports**. It supports up to two simultaneous reads (on ports A and B) and two simultaneous writes (on ports E and M).   
这部分内容前面讲过。  
![SEQ decode and write-back stage](https://img-blog.csdnimg.cn/fbb420c1cd3048bcb276a4e774848026.png)  
- **Execute Stage**  
The execute stage includes the **arithmetic/logic unit (ALU)**. This unit performs the operation `add, subtract, and, or exclusive-or` on inputs `aluA and aluB` based on the setting of the `alufun` signal.  
![SEQ execute stage](https://img-blog.csdnimg.cn/cab74c753305430c8b15cec247cb754c.png)  
The hardware unit labeled **“cond”** uses a **combination of the condition codes and the function code** to determine whether a conditional branch or data transfer should take place (Figure 4.3).  
  
- **Memory Stage**  
**The memory stage** has the task of either **reading** or **writing** program data.  
![SEQ memory stage](https://img-blog.csdnimg.cn/74f49d22bf8543b09e54b95603142c7c.png)  
- **PC Update Stage**  
The final stage in SEQ generates the new value of the program counter (see Figure 4.31).   
![SEQ PC update stage](https://img-blog.csdnimg.cn/01aa2e3aa1ec4c1ba3f49e5d1414b8f0.png)  
# 4.4 General Principles of Pipelining  
A key feature of **pipelining** is that it increases the `throughput` of the system (i.e., the number of customers served per unit time), but it may also slightly increase the `latency` (i.e., the time required to service an individual customer).  
  
Figure 4.32(a) shows an example of a simple **nonpipelined** hardware system.   
  
![Unpipelined computation hardware](https://img-blog.csdnimg.cn/17c3d512687b4b93982c1f401f413176.png)<br/>  
  
In this implementation, we must complete one instruction before beginning the next.  
  
**************************************  
将一个指令分为三个阶段 A，B 和 C，指令1 执行完阶段A，进入阶段B 后，指令2 即可开始执行阶段A，而不用等指令1 全部执行完。  
  
**Three-stage pipelined computation hardware**:  
  
![Three-stage pipelined computation hardware](https://img-blog.csdnimg.cn/453a5d030beb4746aeb9bef28a79d68c.png)  
  
![Three-stage pipeline timing](https://img-blog.csdnimg.cn/42a6584c32bb48e589a32fd97e362f09.png)  
## 4.4.2 A Detailed Look at Pipeline Operation  
**Figure 4.35** traces the circuit activity between times 240 and 360, as instruction `I1` (shown in dark gray) propagates through stage `C`, `I2` (shown in blue) propagates through stage `B`, and `I3` (shown in light gray) propagates through stage `A`.   
  
![One clock cycle of pipeline operation](https://img-blog.csdnimg.cn/0192bb8d2dba4ffc900a684dfba95843.png)<br/>  
  
As the clock rises and falls repeatedly, the different instructions flow through the stages of the pipeline without interfering with one another.  
  
## 4.4.3 Limitations of Pipelining  
### Nonuniform Partitioning  
图 4.35 所示的将操作过程分成三个阶段的方法有局限性：从一个阶段到下一个阶段之间会有延迟（50 - 150ps)。  
  
However, the rate at which we can operate the clock is limited by the **delay of the slowest stage**.   
  
以图 4.36 为例：  
![Limitations of pipelining due to nonuniform stage delays](https://img-blog.csdnimg.cn/66751aed503949898befbb70faf7d6f9.png)<br/>  
  
上图可看见每个阶段所用的时间不一致。  
  
Devising a partitioning of the system computation into a series of stages having **uniform delays** can be a major challenge for hardware designers.  
  
Often, some of the **hardware units in a processor**, such as the ALU and the memories, **cannot be subdivided into multiple units with shorter delay**.   
  
### Diminishing Returns of Deep Pipelining  
图 4.37 所示将操作过程分成6个阶段，每个阶段 50ps，可以看见最小的时钟周期缩短了。  
  
![Limitations of pipelining due to overhead](https://img-blog.csdnimg.cn/79d742d22070462da804f87fb293b29d.png)  
<br/>  
  
Modern processors employ very deep pipelines (15 or more stages) in an attempt to maximize the processor clock rate.   
  
处理器将指令执行过程分成大量很小的步骤，因此每个步骤的延迟也很小。  
  
## 4.4.4 Pipelining a System with Feedback  
有些情况指令之间有依赖关系，如下图，第二条指令要用到第一条指令的数据：  
  
![data dependency](https://img-blog.csdnimg.cn/d9ccf0a003fa43d5b02700cafb2fbdef.png)<br/>  
  
这种有依赖关系的指令执行过程见下图：  
![Limitations of pipelining due to logical dependencies](https://img-blog.csdnimg.cn/299878b24fd747a888d3760c9ecd8aab.png)  
  
# 4.5 Pipelined Y86-64 Implementations  
本章介绍设计一个 `pipelined Y86-64` 处理器。  
  
## 4.5.1 SEQ+: Rearranging the Computation Stages  
首先做一个改变，将**更新程序计数器**的步骤从**最后一步**改到**第一步**（`SEQ+`）。（后面 4.5.4 有讲这么做的目的）  
  
It is often used to balance the delays between the different stages of a pipelined system.   
  
## 4.5.2 Inserting Pipeline Registers  
在 SEQ+ 的不同阶段插入管道寄存器（pipeline regiesters）来设计 `PIPE− ` 处理器：  
  
![SEQ+ hardware structure](https://img-blog.csdnimg.cn/cfae63a11a6a42dcba97996a4e00f519.png)  
  
![Hardware structure of PIPE−, an initial pipelined implementation](https://img-blog.csdnimg.cn/a95b330f41034e49b23c642aaa8834cb.png)  
  
<br/>  
  
管道寄存器标签介绍：  
- `F`: holds a `predicted` value of the **program counter**.  
  
- `D`: sits between the `fetch` and `decode` stages. It holds information about the most **recently fetched instruction** for processing by the `decode` stage.  
  
- `E`: sits between the `decode` and `execute` stages. It holds information about the **most recently decoded instruction** and the values read from the `register file` for processing by the `execute` stage.  
  
- `M`: sits between the `execute` and `memory` stages. It holds the results of the **most recently executed instruction for processing by the memory stage**. It also holds information about **branch conditions** and **branch targets** for processing **conditional jumps**.  
  
- `W`: sits between the `memory stage` and the `feedback paths` that supply the computed results to the `register file` for writing and the return address to the `PC` selection logic when completing a `ret` instruction.  
  
示例：  
![Example of instruction flow through pipeline](https://img-blog.csdnimg.cn/d4fe0f29b1e0457c8ecd304a9e383b2f.png)  
## 4.5.3 Rearranging and Relabeling Signals  
对于不同阶段的状态码 （标签为`Stat`），为了进行区分，加上前缀：`D_stat, E_stat, M_stat, 和 W_stat`。   
  
未看  
  
## 4.5.4 Next PC Prediction  
Our goal in the **pipelined design** is to *issue* a new instruction on **every clock cycle**, meaning that on **each clock cycle**, a new instruction proceeds into the **execute stage and will ultimately be completed**.   
  
Achieving this goal would yield a **throughput** of one **instruction** per **cycle**.  
  
To do this, we must **determine the location of the next instruction** right after **fetching the current instruction**.  
  
但如果遇到条件分支或者`ret`指令，将无法在取指令阶段知道下条指令的地址（见 4.3.1 节的图 4.21 说明）。  
  
因此，对于上述情况，需要**预测程序计数器（PC）的下条指令的地址**（3.6.3 节介绍跳转指令时提到过）；该过程称为`分支预测（branch prediction）`。  
  
## 4.5.5 Pipeline Hazards  
如前面提到过，有时候指令之间有依赖关系，主要有两种：  
（1）数据依赖  
如下条指令需要用到本条指令的结果  
（2）控制依赖  
如跳转，调用指令和返回指令等，下条指令的地址由本条指令决定。  
  
如果依赖可能造成管道计算错误（预测错误），则称为 **冒险（hazards）**。  
  
Like **dependencies**, **hazards** can be classified as either **data hazards** or **control hazards**.   
  
**************************  
  
**示例1：**  
![Pipelined execution of prog1 without special pipeline control](https://img-blog.csdnimg.cn/a5217657f449430495f842fce71e4d2f.png)<br/>  
  
第一个示例可以正确得到结果，第一条指令在第`5`个时钟周期向寄存器写入数据，第二条指令则在第`6`个时钟周期向寄存器写数据，而到第`7`个时钟周期 `addq` 指令才开始执行译码阶段，此时两个操作数均已更新。  
  
该示例能正确执行，因为加入了三个 `nop` 指令在指令之间创建**延迟**。  
<br/>  
  
**********************  
  
**示例2：**  
![Pipelined execution of prog2 without special pipeline control](https://img-blog.csdnimg.cn/2752d0478f8f430bab907de1d54cc2d1.png)<br/>  
  
该示例相比示例1减少了一个 `nop` 指令，在第 `6` 个时钟周期，指令2处于写回阶段，但**写到寄存器中的数据要在下一个始终周期的开始（有高电平）时才更新数据**，而此时 `addq` 指令已经处于译码阶段，因此计算错误。  
  
<br/>  
  
**********************  
### Avoiding Data Hazards by Stalling  
One very general technique for **avoiding hazards** involves **stalling**, where the **processor holds back one or more instructions in the pipeline until the hazard condition no longer hold**s.  
  
例如示例2做如下修改：  
  
![Pipelined execution of prog2 using stalls](https://img-blog.csdnimg.cn/e929aaa129d84ab2a2be1cc5fc3892db.png)<br/>  
  
在执行 `addq` 指令前插入一个气泡 （bubble），在第7个时钟周期重复执行译码指令。   
  
A `bubble` is like a **dynamically generated `nop` instruction**—it **does not cause any changes** to the registers, the memory, the condition codes, or the program status.  
  
这种方式的缺陷：  
This will cause the pipeline to **stall** for up to three cycles, **reducing the overall throughput** significantly  
   
<br/>  
  
**********************  
### Avoiding Data Hazards by Forwarding  
示例2做如下修改：  
  
![Pipelined execution of prog2 using forwarding](https://img-blog.csdnimg.cn/d281a5b2d8dd4935bd52e19b86cc6016.png)<br/>  
  
根据前面的图 4.18 可知，在执行代码阶段，寄存器 `%rax` 的值被写入到端口E（W_valE)，因此正在第六个时钟周期执行 `addq` 的译码指令时，直接读取端口E的数据而非读取 `valB` 的数值。  
  
This technique of **passing a result value directly from one pipeline stage to an earlier one** is commonly known as `data forwarding` (or simply `forwarding`, and sometimes `bypassing`).   
  
Data forwarding requires **adding additional data connections and control logic** to the basic hardware structure.  
  
  
下图 5.42 展示 `PIPE` 的结构，能通过 `forwarding` 处理 hazards。  
  
![Hardware structure of PIPE, our final pipelined implementation](https://img-blog.csdnimg.cn/24ce1959eab945ffb747698c5dc843cc.png)  
  
<br/>  
  
**********************  
### Load/Use Data Hazards  
有一种情况不能通过 `forwading` 的方式解决，如：  
  
![Example of load/use data hazard](https://img-blog.csdnimg.cn/a25f7c6ef84d4c1b9ef97658a2c4129c.png)<br/>  
上图中，在7个时钟周期时，指令 `addq` 执行译码阶段，需要寄存器 `%rax` 的值，该值需要指令 `mrmovq` 在第8个时钟周期内存访问阶段才会得到（`valM`），因此无法使用 `forwarding` 的方法。  
<br/>  
  
**图 4.54 展示一种使用 `stalling` 和 `forwarding` 的组合方法**：  
  
![Handling a load/use hazard by stalling](https://img-blog.csdnimg.cn/d7734e9035014683a4881568cff79a38.png)  
  
<br/>  
  
上图在执行 `addq` 指令前插入一个气泡，因此在第7个时钟周期，`addq` 译码阶段能通过 `forwarding` 方式在`mrmovq` 的内存访问阶段获取数值。  
  
This use of a `stall` to handle a `load/use hazard` is called a `load interlock`. `Load interlocks` combined with `forwarding` suffice to handle all possible forms of data hazards.  
  
<br/>  
  
**********************  
### Avoiding Control Hazards  
**Control hazards** arise when the processor **cannot reliably determine the address** of the **next instruction** based on the current instruction in the **fetch stage**.   
  
**Control hazards** can only occur in our pipelined processor for `ret and jump` instructions.   
  
示例：  
  
![Processing mispredicted branch instructions](https://img-blog.csdnimg.cn/cf8bb405f40a476786beba4fa9e0fb75.png)  
<br/>  
  
上图中，指令2在译码阶段，进行分支预测，执行指令3，到四个时钟周期指令2到执行阶段才能判断分支预测是否正确，而此时已经错误的执行了指令3的两个阶段。  
  
At this point, the **pipeline** can simply ***cancel*** (sometimes called ***instruction squashing***) the two **misfetched instructions** by **injecting bubbles into the decode and execute stages** on the following cycle while also **fetching the instruction following the jump instruction**.  
  
被错误执行的指令将从**管道消失**且不会对 `programmer-visible state` 有任何影响。  
  
分支预测失败很浪费时间。  
  
<br/>  
  
## 4.5.6 Exception Handling  
内部异常：  
 (1) a `halt` instruction,  
 (2) an instruction with an **invalid combination of instruction and function code**  
(3) an attempt to access an **invalid address**, either for instruction `fetch` or `data read or write`.   
  
<br/>  
  
In a pipelined system, exception handling involves several subtleties.   
  
- 多个指令同时发生异常。  
The basic rule is to **put priority on the exception triggered by the instruction** that is **furthest** along the pipeline.   
  
- 在错误的分支预测中发生的异常，随后错误执行的指令会被取消。  
The pipeline control logic will **cancel this instruction**, but we want to avoid raising an exception.  
  
- A third subtlety arises because a pipelined processor **updates different parts of the system state in different stages**.   
在同一个时钟周期，一个指令发生异常，而另一个指令使用异常指令的错误数据。  
  
解决方案：使用状态码(`Stat`)  
  
The **exception status** propagates through the **pipeline** with the rest of the information for that instruction, **until it reaches the write-back stage**.  （因此前面提到分支预测失败不会影响到状态码）  
  
At this point, the **pipeline control logic** detects the occurrence of the exception and **stops execution**.  
  
To avoid having any updating of the `programmer-visible state` by instructions **beyond the excepting instruction**, the **pipeline control logic** must disable any updating of the **condition code register** or the **data memory** when an instruction in the **memory** or **write-back stages** has caused an **exception**.   
  
When an exception occurs in one or more stages of a **pipeline**, the information is simply stored in the **status fields of the pipeline registers**.   
<br/>  
  
## 4.5.7 PIPE Stage Implementations  
 `PIPE` uses the same set of hardware units as the earlier sequential designs, with the addition of **pipeline registers**, some **reconfigured logic blocks**, and additional **pipeline control logic**.   
  
Many of the **logic blocks** are identical to their counterparts in `SEQ` and `SEQ+`, except that we must choose proper versions of the **different signals** from the **pipeline registers** (written with the pipeline register name, written in **uppercase**, as a **prefix**) or from the **stage computations** (written with the first character of the **stage name**, written in **lowercase**, as a **prefix**).  
  
示例：  
  
![HCL code for the logic ](https://img-blog.csdnimg.cn/3732758607944a9ea46a6af7ea275971.png)<br/>  
  
They differ only in the **prefixes** added to the `PIPE signals`: `D_` for the source values, to indicate that the **signals** come from `pipeline register D`, and `d_` for the **result value**, to indicate that it is generated in the `decode stage`.  
（4.5.2 和 4.5.3 节有讲）  
  
<br/>  
  
### PC Selection and Fetch Stage  
`PIPE` `PC` selection and `fetch` logic：  
![PIPE PC selection and fetch logic](https://img-blog.csdnimg.cn/cdd3d0ceebde467b9472bf4711bcf1b7.png)  
<br/>  
The `PC selection logic` chooses between **three program counter sources**.   
  
As a **mispredicted branch** enters the **memory stage**, the value of `valP` for this instruction (indicating the address of the following instruction) is read from `pipeline register M (signal M_valA)`.   
  
When a `ret` instruction enters the `write-back stage`, the return address is read from **pipeline register W (signal W_valM)**.   
  
All other cases use the `predicted value of the PC`, stored in `pipeline register F (signal F_predPC)`:  
![1](https://img-blog.csdnimg.cn/3ea0cee39e674d2a8e67ed6efa422daf.png)  
<br/>  
  
Unlike in `SEQ`, we must split the computation of the instruction status into **two parts**.   
  
In the **fetch stage**, we can **test for a memory error** due to an out-of-range instruction address, and we can **detect an illegal instruction or a `halt` instruction**.   
  
Detecting an invalid data address must be **deferred to the memory stage**.  
<br/>  
  
### Decode and Write-Back Stages  
Figure 4.58 gives a detailed view of the **decode and write-back logic for PIPE**.  
![PIPE decode and write-back stage logic](https://img-blog.csdnimg.cn/a4cac1d002f643b7847fcdc2442f6260.png)  
<br/>  
  
Observe that the **register IDs** supplied to the **write ports** come from the `write-back stage` (signals W_dstE and W_dstM), rather than from the **decode stage**.   
  
This is because we want the **writes** to occur to the **destination registers** specified by the instruction in the **write-back stage**.  
  
The block labeled **“Sel+Fwd A”** serves two roles：  
- It merges the `valP` signal into the `valA` signal for later stages in order to reduce the amount of state in the **pipeline register**.   
- It also implements the `forwarding logic` for source operand `valA`.  
  
There are `five different forwarding sources`, each with a data word and a destination `register ID`:  
  
![five different forwarding sources](https://img-blog.csdnimg.cn/57c4d01b4a2e40419b6c2a8b6f7257b9.png)  
  
If **none of the forwarding conditions hold**, the block should select `d_rvalA`, the value read from register `port A`, as its output.  
  
Putting all of this together, we get the following `HCL` description for the new value of `valA` for `pipeline register E`:  
  
![HCL description](https://img-blog.csdnimg.cn/46575d6029004e23ae1c56f3f75aa6b3.png)<br/>  
  
The **priority** given to the five forwarding sources in the above `HCL` code is very important.   
  
The logic in the `HCL` code above first tests the forwarding source in the **execute stage**, then those in the **memory stage**, and finally the sources in the **write-back stage**.   
  
### Execute Stage  
Figure 4.60 shows the execute stage logic for `PIPE`.  
![PIPE execute stage logic](https://img-blog.csdnimg.cn/1d24a3c0951e4df886506011ebde716a.png)  
<br/>  
  
### Memory Stage  
Figure 4.61 shows the memory stage logic for `PIPE`.   
![PIPE memory stage logic](https://img-blog.csdnimg.cn/6d5b22e20fb8457ea8abca4c5aed5f60.png)  
<br/>  
  
## 4.5.8 Pipeline Control Logic  
This logic must handle the following **four control cases** for which other mechanisms, such as **data forwarding** and **branch prediction**, do not suffice:  
- **Load/use hazards**  
The pipeline must **stall** for **one cycle** between an instruction that **reads a value from memory** and an instruction that uses this value.  
- **Processing `ret`**  
The pipeline must **stall** until the `ret` instruction reaches the **write-back stage**.  
- **Mispredicted branches**  
分支预测失败后能取消错误的指令并重新执行正确的指令。  
- **Exceptions**  
When an instruction causes an **exception**, we want to **disable the updating of the programmer-visible state** by later instructions and `halt` execution once the excepting instruction reaches the **write-back stage**.  
<br/>  
  
### Pipeline Control Mechanisms  
Figure 4.65 shows **low-level mechanisms** that allow the **pipeline** **control logic** to **hold back an instruction** in a **pipeline register** or to **inject a bubble** into the pipeline.  
  
![Additional pipeline register operations](https://img-blog.csdnimg.cn/2c1c1c32d57b4bdb808a1410907c0fd3.png)  
<br/>  
  
The table in Figure 4.66 shows the actions the **different pipeline stages** should take for each of the three special conditions.   
![Actions for pipeline control logic](https://img-blog.csdnimg.cn/03b772e8e93c4499a1622c2098464d2e.png)  
<br/>  
  
如图 4.5 所示，假设每个管道寄存器有两个控制输入 `stall` 和 `bubble`。  
  
正常模式：两个输入均为0，在时钟信号变为高电平时，寄存器更新输入信号作为最新状态；  
  
当 `stall` 信号为 `1` 时，禁止更新状态，寄存器保持之前的状态；  
  
当 `bubble` 信号为 `1` 时，the state of the register will be set to some fixed **reset configuration**, giving a state equivalent to that of a `nop` instruction. The particular pattern of ones and zeros for a pipeline register’s reset configuration depends on the set of fields in the pipeline register.   
  
<br/>  
  
### Control Logic Implementation  
Figure 4.68 shows the overall structure of the pipeline control logic.  
  
![PIPE pipeline control logic](https://img-blog.csdnimg.cn/7fea1fcb67af4f9fb4a00038e419e2a6.png)  
## 4.5.9 Performance Analysis  
A **return** instruction generates **three bubbles**, a **load/use hazard** generates one, and a **mispredicted** branch generates two.   
  
We can quantify the effect these penalties have on the overall performance by computing an estimate of the **average** number of **clock cycles** `PIPE` would require **per instruction** it executes, a measure known as the `CPI` (for **“cycles per instruction”**).  
  
If the stage processes a total of $C_{i}$ instructions and $C_{b}$ bubbles,  
![CPI](https://img-blog.csdnimg.cn/ddcb9a46da6a4ba5894add8f9cb23f3b.png)  
![CPI](https://img-blog.csdnimg.cn/7227411137734a67a92e5240e4d6f524.png)  
