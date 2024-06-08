深入理解计算机系统——第三章 Machine-Level Representation of Programs  
      
> [图解系统](https://www.xiaolincoding.com/os/)  
> [程序的机器级表示（一）](https://blog.csdn.net/tangfatter/article/details/122051638?spm=1001.2014.3001.5502)  
> [程序的机器级表示（二）](https://blog.csdn.net/tangfatter/article/details/122097188?spm=1001.2014.3001.5502)  
> [程序的机器级表示（三）](https://blog.csdn.net/tangfatter/article/details/122148111?spm=1001.2014.3001.5502)  
> [程序的机器级表示（四）](https://blog.csdn.net/tangfatter/article/details/122149973?spm=1001.2014.3001.5502)  
> [深入理解计算机系统学习笔记](https://zhuanlan.zhihu.com/p/478993532)  
> [程序的机器级表示](https://note.youdao.com/ynoteshare/index.html?id=fc45065e632e1d7a6f07ab4b11152fbe&type=note&_time=1659404552299)  
> [深入理解计算机系统（CSAPP）全书学习笔记（详细）](https://zhuanlan.zhihu.com/p/455061631)  
> [深入理解计算机系统 课程视频](https://www.bilibili.com/video/BV1iW411d7hd?p=5&share_source=copy_web)  
> [汇编语言各种指令的解释与用法](https://www.cnblogs.com/Sumarua/p/11698360.html)  
> [cmov条件传送指令](https://blog.csdn.net/npu2017302288/article/details/109171692)  
> [第18章-x86指令集之常用指令 ](https://www.cnblogs.com/mazhimazhi/p/15241450.html)  
      
# 3.2 Program Encoding  
C 程序从源文件生成可执行文件的过程：  
      
1. The `C Preprocessor` expands the source code to **include** any files specified with `#include` commands and to expand any `macros`, specified with `#define` declarations.  
      
2. The **compiler** generates **assembly code** versions of the source files (*.s).  
      
3. The **assembler** converts the assembly code into **binary object-code** files (*.o). **Object code** is one form of machine code—it contains binary representations of all of the instructions, but the **addresses of global values are not yet filled in**.  
      
4. The **linker** merges these object-code files along with code implementing **library functions** (e.g., `printf`) and generates the final **executable code** file (as specified by the command-line directive -o p). **Executable** code is the second form of **machine code** we will consider—it is the exact form of code that is executed by the **processor**.  
    
## 3.2.1 Machine-Level Code  
1. The format and behavior of a **machine-level program** is defined by the `instruction set architecture`, or `ISA`, defining the processor state, the format of the instructions, and the effect each of these instructions will have on the state.  
      
2. The memory addresses used by a machine-level program are `virtual addresses`, providing a memory model that appears to be a very large byte array.  
       
Whereas C provides a model in which objects of different **data types** can be declared and allocated in memory, **machine code** views the **memory** as simply a **large byte-addressable array**.  
      
指令集介绍：  
> [CPU 指令集](https://www.cnblogs.com/yilang/p/10967374.html)  
> [What Is an Instruction Set Architecture?](https://www.arm.com/glossary/isa)  
      
虚拟地址介绍：  
> [Windows内存体系（1） -- 虚拟地址空间](https://winsoft666.blog.csdn.net/article/details/79610915?spm=1001.2101.3001.6650.5&utm_medium=distribute.pc_relevant.none-task-blog-2~default~OPENSEARCH~default-5-79610915-blog-125882845.pc_relevant_multi_platform_whitelistv3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~OPENSEARCH~default-5-79610915-blog-125882845.pc_relevant_multi_platform_whitelistv3&utm_relevant_index=10)  
> [virtual address ](https://www.techtarget.com/whatis/definition/virtual-address#:~:text=A%20virtual%20address%20is%20a%20binary%20number%20in,to%20a%20hard%20disk%20or%20internal%20flash%20drive.)  
      
# 3.3 Data Formats  
> [线路位宽与 CPU 位宽](https://xiaolincoding.com/os/1_hardware/how_cpu_run.html#线路位宽与-cpu-位宽)  
    
Due to its origins as a `16-bit architecture` that expanded into a `32-bit` one, *Intel* uses the term **“word”** to refer to a `16-bit` data type. Based on this, they refer to `32- bit` quantities as **“double words,”** and `64-bit` quantities as **“quad words.”**  
![Sizes of C data types in x86-64](https://img-blog.csdnimg.cn/930d3a7e9d4f4b06a8c40246f8e6fcbc.png)  
      
# 3.4 Accessing Information  
An x86-64 central processing unit (CPU) contains a set of **16 general-purpose registers** storing **64-bit values**. These registers are used to store **integer** data as well as **pointers**.  
![Integer registers](https://img-blog.csdnimg.cn/cb69209fe82f4a3e920b13297ddead7f.png)  
      
**整数寄存器**，用来存放`整型`和`指针`，有6个寄存器用来存放6个参数，如果参数超过6个，则放在`栈`上。  
      
## 3.4.1 Operand Specifiers  
`x86-64` 支持操作数形式如下：  
![Operand forms](https://img-blog.csdnimg.cn/657ffc0bc77f42a6815fd9548f098810.png)<br/>  
      
源操作数有三种：立即数（常数），寄存器，内存  
目标操作数：寄存器或者内存； 结果存放的位置  
      
立即数（immediate）：常数值。  
内存引用（memory reference）：根据有效地址获取内存的位置。  
      
1. 对于 ATT 形式的汇编代码，用 `$` 跟着一个整型（C 标准格式）表示立即数，如，：`$-577`，`$0x1F`，因此指令 `$-577` 表示常数 `-577`。 不同的指令允许的立即数的范围不同。  
      
2. 用 $r_{a}$ 表示一个寄存器， $R[r_{a}]$ 表示寄存器  $r_{a}$  的值，将一组寄存器看作数组 $R$，而寄存器  $r_{a}$  为数组的索引。  
      
3.  内存被看作一个大的**字节数组**，因此用 $M_b[Addr]$ 表示对内存中起始地址为 `Addr` 的 `b` 字节的数值的引用。这里去掉下标 `b`。  
      
4. $M[Imm]$ 是绝对寻址，直接给内存的地址（立即数），指令的格式为一个立即数 `Imm`。  
      
5. $M[R[r_{a}]]$ 是间接对寻址，获取寄存器 $r_{a}$  的值作为内存的起始地址，指令的格式为 $(r_{a})$。  
      
6. 对于指令格式 $Imm(r_{b}, r_{i}, s)$， $r_{b}$ 为基址寄存器， $r_{i}$ 为变址寄存器， $s$ 为比例因子，其值必须为 1，2，4或8（对应 byte，word，double word， quad word 大小）。基址和变址寄存器必须为 64 位寄存器（图 3.2 中最左边一列）。最后有效地址的计算公式为： $Imm + R[r_{b}] + R[r_{i}] \cdot s$。  
      
************  
      
示例：  
![practice3.1](https://img-blog.csdnimg.cn/1eb459aa1cb549b0a858740d674dbdaf.png)  
      
## 3.4.2 Data Movement Instructions  
`MOV` 指令：These instructions copy data from a **source** location to a **destination** location, **without any transformation**.  
      
四种形式：`movb, movw, movl, and movq`，效果相同，只是数据大小不同，分别为 `1,2,4,8` 字节：  
![Simple data movement instructions](https://img-blog.csdnimg.cn/3d13adad3dea4b9d8b1ec0440ddfc84b.png)<br/>  
      
**********  
      
示例：  
|指令  |解释  |  
|:--|:--|  
| movl $0x4050,%eax      |   Immediate--Register, 4 bytes|  
| movw %bp,%sp     |           Register--Register, 2 bytes|  
| movb (%rdi,%rcx),%al  |    Memory--Register, 1 byte|  
| movb $-17,(%esp) |Immediate--Memory, 1 byte|  
| movq %rax,-12(%rbp) |Register--Memory, 8 bytes|  
      
1. `MOV` 指令**源操作数和目标操作数不能同时为内存**，因为将数据从一个内存地址复制到另一个内存地址需要**两条指令**完成，需要先将源地址的数据放到一个寄存器中，再从该寄存器的数值写到目标内存地址。  
2. 指令和所用寄存器大小需匹配，如用 `movl` 时，寄存器用 32 位寄存器。  
3. 通常，`mov` 指令只会根据**目标操作数**来更新寄存器中的特定位，如用 `movabsq` 将立即数复制到64位寄存器中，则目标寄存器的 64 位都用到，更新数据，源数据没有 64 位，则高位用 0 填充；而对于 `movw`指令，目标寄存器位 16 位，因此数据只更新低 16 位，高位不更新；但 `movl` 是一个特例，更新低 32 位后，会将高 32 位填充为 0。见下面例子，第一次用 `movabsq` 指令后更新 `%rax` 的全部 64 位数，之后用 `movb` 指令，目标寄存器是 `%al` ，即 `%rax` 寄存器的低 8 位，源操作数 -1 的补码为 FF，因此低 8 位更新为 FF，其他高位不变：  
![mov](https://img-blog.csdnimg.cn/29cdd6c5916f4cf5b0f03f165f5657c5.png)  
4. 当用 `movq` 指令且**源操作数**为**立即数**时，**立即数**只能是 32 位的补码，而不能是 64 位，高位将用符号位扩展从而得到 64 位（第二章介绍过符号位扩展补码的高位，其结果不变）。  
5. 用 `movabsq` 指令时，源操作数可以是任意的 64 位立即数，目标操作数只能是寄存器。  
      
当目标操作数尺寸大于源操作数，可以在高位进行`零扩展`或者`符号位扩展`。  
      
零扩展 `movz` 系列指令：剩下的高位填充 0。  
![Zero-extending data movement instructions](https://img-blog.csdnimg.cn/1c21fc139485407ebbdca09843a612ca.png)<br/>  
**注意**：这里没有 `movzlq`，因为当使用 `movl` 指令时，且目标操作数为寄存器，那么寄存器的高4个字节会填充 `0`。  
      
**要求**：源操作数为寄存器或者内存地址，目标操作数位寄存器。  
      
符号位扩展 `movs` 系列指令：剩下的高位用符号位填充：  
![Sign-extending data movement instructions](https://img-blog.csdnimg.cn/54487123e3ae49c9ada244a55903b7eb.png)<br/>  
      
示例：  
![fig 3.7](https://img-blog.csdnimg.cn/0258d4cfa62d49f2a927d681413299cf.png)<br/>  
      
第一个参数 xp 和第二个参数 y 分别存在寄存 `%rdi` 和 `%rsi` 中。  
指令 2：根据 `xp` 地址从内存中取数值，存到寄存器 `%rax` 中。  
指令3：将 `%rsi` 的值，即 `y` 的值存到地址为 `xp` 的内存中。  
最后将寄存器 `%rax` 的值，即 `x` 作为返回值返回。  
      
上述的局部变量保存在寄存器而非内存中，寄存器比内存的访问速度更快。  
    
***********************************************  
      
**示例1：**  
![practice 3.2](https://img-blog.csdnimg.cn/aad6e0d9de7b4af79b82926c086bdd05.png)  
      
根据图 3.2 可知所用寄存器大小，题解中说对于 x86-64，内存引用总是 quad word。  
*********************************  
      
**示例2：**  
![practice 3.3](https://img-blog.csdnimg.cn/cc4672d32a7b455baad0e90265f305a4.png)  
      
*************************  
      
**示例3：**  
![practice 3.5](https://img-blog.csdnimg.cn/f740888a40644eb8a4ae8f4cba98d089.png)  
      
1. 因为参数类型为指针，用 8 字节寄存器。  
2. 根据图 3.2 可知，第一个到第三个参数分别为 %rdi，%dsi 和  %rdx，三个寄存器的值分别为 xp，yp，和 zp，为地址。  
3. 第一个指令源操作数为内存引用，内存地址为 xp，从内存地址 xp 处取数值，存放到寄存器  %r8 中，用 x 代表 %r8 的值，则该指令对应 `x = *xp`。  
4. 第二个和第三个指令同第一个类似，分别用 y 和 z 代表寄存器 %rcx 和 %rax 的值，则这两条指令分别为 `y = *yp` 和 `z = *zp`。  
5. 第四条指令源操作数 %r8 的值，即 x，目标操作数为内存地址 yp，则将 x 存放到内存地址为 yp 处，即 `*yp = x`。  
6. 第五条指令和第六条指令同第四条指令类型，分别为 `*zp = y` 和 `*xp = z`。  
      
因此对应的 C 代码如下：  
![solution 3.5](https://img-blog.csdnimg.cn/9f4a415eb8164d3888a6453995f18f70.png)  
      
## 3.4.4 Pushing and Popping Stack Data  
- **栈** （stack）是一种**数据结构**，遵循**先进后出**的规则，可以增加（push）或删除（pop）数据。  
- **栈**可以由**数组来实现**，数组的**末尾为栈顶**，添加或删除元素在**栈顶**操作。  
- **程序**用栈来管理**过程调用**与**返回的状态**。  
- 对于 `×86-64`，栈存在内存的某个区域，且向下增长，即栈底在高位地址处，当 push 指令添加数据时，**栈顶向低位扩展**，**栈指针** `%rsp` 指向**栈顶**位置。  
- 当使用 `pop` **弹出栈顶**的数据时，**实际数据仍在该地址**，只是**栈顶**的**指针**指向的**位置变了**（向高地址移动8）。  
      
`Push` 和 `Pop` 指令：  
![Push and pop instructions](https://img-blog.csdnimg.cn/f6a2698e6a6e4e66832c3192750bd9d8.png)  
      
栈操作介绍：  
![Illustration of stack operation](https://img-blog.csdnimg.cn/d8ffe7d2588b44b5adc1e006a703dcd0.png)  
      
# 3.5 Arithmetic and Logical Operations  
每一种类型的指令都会对应操作 4 种不同大小数据的指令。  
例如，加法指令分为 `addb`, `addw`, `addl`, 和 `addq` 分别操作大小为 1字节，2字节，4字节，和 8 字节大小的数据。  
      
The operations are divided into **four groups**: load effective address, unary, binary, and shifts.  
      
## 3.5.1 Load Effective Address  
`load effective address` 指令是 `movq` 指令的一个变体，从**内存读数据到寄存器**，但不是内存引用，而是**复制内存地址到寄存器**。  
![Integer arithmetic operations](https://img-blog.csdnimg.cn/189341473f364e0eaba6c1fb3e6da864.png)<br/>  
      
例如：**%rax** 的值为 **x**，则指令 `leaq 7(%rdx,%rdx,4), %rax` 表示将寄存器 **%rax** 的值设为 **7 + 5x**，这个过程**只复制内存地址而不访问内存**; 而如果是 `movq` 指令，则是将内存地址为 **7 + 5x** 的值设置为寄存器 **%rax** 的值。  
    
********************  
      
示例：  
![practice 3.7](https://img-blog.csdnimg.cn/45be708eb0af4895b74bce3facb8cd25.png)  
      
1. 第一个参数 x 存在寄存器 %rdi 中，第二个参数 y 存在寄存器 %rsi 中，第三个参数 z 存在寄存器  %rdx 中。  
2. 第一个指令源操作数为地址 10 * y，leaq 指令只复制地址而不是同 mov 指令一样访问内存，因此该指令将 %rbx 的值设置为 10 * y。  
3. 第二个指令将 %rbx 设置为 10 * y + z。  
4. 第三个指令将 %rbx 设置为 10 * y + z + x * y。  
5. 最后返回为 %rbx 的值 ？？因为 %rbx 是被调用者保存数据的寄存器 （callee saved）？  
      
## 3.5.2 Unary and Binary Operations  
图 3.10 中第一组为一元运算， `INC` 指令相当于 C 语言的自增运算，如果是 `incq (%rsp)` 则会使**栈顶**的地址加 8 字节。  
图 3.10 中第三组加减等操作为二元运算，例如 `subq %rax,%rdx ` 指令会将寄存器 `%rdx` 的值设置为 原始 `%rdx` 的值减去 `%rax` 的值。  
第三组的二元运算的源操作数（第一个操作数）可以是立即数，寄存器 和 内存地址，而目标操作数（第二个操作数）只能是寄存器或内存地址，同 mov 指令相同，**两个操作数不能同时为内存地址**。  
当第二个操作数为**内存地址**时，处理器从内存地址读数据，执行运算操作后，将结果写回到该内存地址。  
      
**************************************************  
      
示例：  
![practice 3.8](https://img-blog.csdnimg.cn/3012113e519b480b8db9561eea380ab2.png)<br/>  
      
答案如下：  
![solution 3.8](https://img-blog.csdnimg.cn/e325dfa8008841a792832c3109f6ebf4.png)  
      
1. 第一条指令，源操作数为寄存器 %rcx 的值 0x1，目标操作数为内存地址 0x100 （寄存器 %rax 的值），将 0x 100 的值设置为 0xFF ( 初始内存中的值) + 0x01 (%rcx 的值) = 0x100。  
2. 第二条指令，目标操作数为内存地址 8 + 0x100 = 0x108，执行减法指令：0xAB（内存地址为 0x108 的值） - 0x3（%rdx 的值）= 0xA8，最后将计算结果写回到内存地址 0x108 处。  
3. 第三条指令，目标操作数为内存地址 0x100 + 0x3 * 8 = 0x118，执行乘法指令：0x10  （立即数 16）* 0x11（内存地址为 0x118 的值） = 0x110，最后将计算结果 0x110 写回内存地址 0x118 处。  
4. 第四条指令，目标操作数为内存地址 0x100 + 0x10（立即数 16） = 0x110，从该地址取值 0x13，将加减1，得到 0x14，然后写回内存地址 0x110 处。  
5. 第五条指令，两个操作数均为寄存器，目标操作数为寄存器 %rax，%rax 的值 0x100 - %rdx 的值 0x3 = 0xFD，将值设置为寄存器 %rax 的值。  
      
## 3.5.3 Shift Operations  
图 3.10 中的最后一组为移位运算，第一个数表示需要**移位的数目**，第二个操作数为需要**移位的数**。  
第一个操作数可以是**立即数**或者**单字节的寄存器 %cl**。  
如果第二个操作数是寄存器 %cl，对于 x86-64，如果一个数有 w 位，m 满足 $2^m$ = $w$，则移位的数目为 %cl 的最低 m 位数据的值。  
例如 %cl 的值为 0xFF，那么 salb 指令移位的数目为 7，因为 salb 操作的数 8 位，m 为 3，%cl 的低三位为 111，即 7。  
      
左移指令 SAL 和 SHL 相同，都是向右边填充 0。  
右移指令 SAR 和 SHR 分别表示**算术右移**和**逻辑右移**，**算术右移**向高位填充**符号位**，**逻辑右移**向高位填充 **0**。  
      
*******************  
      
示例：  
![practice 3.9](https://img-blog.csdnimg.cn/fadc33a52c4e412b8689482c9aaf88fa.png)  
      
## 3.5.4 Discussion  
图 3.10 中的指令能用于**无符号数或者补码**计算，有符号和无符号只有**右移运算**有区别，第二章有介绍相关知识。  
      
示例 1：  
![fig 3.11](https://img-blog.csdnimg.cn/74c9f94d15b84160ae99d9717b0efcf9.png)  
      
上面计算第二个乘法语句 z * 48 使用了两条指令，先计算 3 * z，然后左移 4 位，因为左移 1 位相当于乘以 2，因此结果位 3 * z * 16 得到 z * 48。  
      
***********************  
      
示例2：  
![practice 3.10](https://img-blog.csdnimg.cn/319c42d2a3b04c7d933c6c4074482b61.png)  
    
*********************  
      
示例3：  
![practice 3.11](https://img-blog.csdnimg.cn/268b48c8b33c49658ea698e34d21b520.png)  
      
1. 该异或的指令可以将寄存器的值设置为 0，相同的位异或后为0，因此将 `%rcx` 的值与自己异或后为0，然后将结果 0 设置为 `%rcx` 的值。  
2. 可以直接设置该值为 0：`movq $0, %rcx`。  
3. 没明白，答案说**任何指令更新低位的 4 字节会造成高位 4 字节为 0 （？？没明白）**，因此异或时只用处理低 4 字节，用 xorl %ecx,%ecx；或者用 movl $0, %ecx，因为 movl 会将高位设置为 0 （前面有讲）。  
![solution 3.11](https://img-blog.csdnimg.cn/2574a997ca9d49d5b5e38e92a71f7285.png)  
      
## 3.5.5 Special Arithmetic Operations  
特殊的算数操作：  
![Special Arithmetic Operations](https://img-blog.csdnimg.cn/94ba3b3657554270a4e6d777a5c52d85.png)  
    
# 3.6 Control  
## 3.6.1 Condition Codes  
In addition to the **integer registers**, the CPU maintains a set of **single-bit condition code registers** describing **attributes** of the most recent **arithmetic** or **logical** operation.  
      
These registers can then be tested to perform **conditional branches**.  
      
3.5.1节中的 fig 3.10 除了 `leaq` 指令外都会改变下面的某些条件代码寄存器的值。  
      
![condition codes](https://img-blog.csdnimg.cn/a1a7fee6df6c435aa1efcb77a65d5eb6.png)CF: 进位标志  
      
**比较和测试指令**：  
> [汇编语言CMP（比较）指令：比较整数](http://c.biancheng.net/view/3561.html)  
> [汇编语言TEST指令：对两个操作数进行逻辑（按位）与操作](http://c.biancheng.net/view/3560.html)  
> [汇编语言各种指令的解释与用法](https://www.cnblogs.com/Sumarua/p/11698360.html)  
      
![Comparison and test instructions](https://img-blog.csdnimg.cn/505e28eb8cef429a8b7e4b1d81cfac57.png)  
      
**比较指令**：如果**相等**则**设置 ZF标志位**为 1。  
**测试指令**：和 `AND` 指令使用相同，但不会修改内容（前面可知 AND 指令会将计算结果更新到目标操作数中）。  
      
## 3.6.2 Accessing the Condition Codes  
`SET` 指令：  
![The set instructions](https://img-blog.csdnimg.cn/98ba1d660fed4319a2411ddf334735f3.png)  
      
`SET` 的目标操作数：图 3.2 中最右侧低位 1 字节的寄存器，或者单字节的内存地址，其结果为 0 或者 1。  
如果想得到 32 位 或者 64 位 的结果，需3.9要将高位清零。  
      
## 3.6.3 Jump Instructions  
`jump` 指令：  
![The jump instructions](https://img-blog.csdnimg.cn/a5b46e538a4c4e25bd4168b2742aa728.png)  
      
`jmp` 指令有两种跳转方式：  
- **direct jump**  
The jump target is  encoded as part of the instruction:  
![](https://img-blog.csdnimg.cn/c0d3f7955a6845578c762d72b7aa4898.png)  
      
- **indirect jump**  
The jump target is read from a register or a memory location.  
Indirect jumps are written using `*` followed by an operand specifier using one of the memory operand formats described in Figure 3.3.  
![1](https://img-blog.csdnimg.cn/84fe43f2e1734e1799147430c5367b35.png)  
      
## 3.6.4 Jump Instruction Encodings  
There are several **different encodings for jumps**.  
- **PC relative**  
They encode the **difference** between the **address of the target instruction** and the **address of the instruction immediately following the jump**. These offsets can be encoded using 1, 2, or 4 bytes.  
      
- **absolute address**  
It uses 4 bytes to directly specify the target.  
      
## 3.6.6 Implementing Conditional Branches with Conditional Moves  
> [cmov条件传送指令](https://blog.csdn.net/npu2017302288/article/details/109171692)  
> [第18章-x86指令集之常用指令 ](https://www.cnblogs.com/mazhimazhi/p/15241450.html)  
      
When the machine encounters a conditional jump (referred to as a “branch”), it cannot determine which way the branch will go until it has evaluated the branch condition.  
跳转指令效率低：机器遇到条件跳转时，无法知道跳转到哪条分支，因此会做分支预测来猜测可能会跳转到哪条分支，如果预测失败，则需要舍弃之前的工作然后执行正确的分支。  
      
Unlike **conditional jumps**, the processor can execute **conditional move** instructions without having to predict the outcome of the test.  
条件传送指令无需预测，效率更高。  
      
条件传送指令：  
![The conditional move instructions](https://img-blog.csdnimg.cn/6ef78327d22c42c8be98fec18f3a46ea.png)  
      
条件转移示例：  
如果要做如下判断：  
> v = test-expr ? then-expr : else-expr;  
      
写成如下形式：  
```cpp  
v = then-expr;  
ve = else-expr;  
t = test-expr;  
if (!t) v = ve;  
```  
      
**注意：**  
**1、不是所有情况都能用条件转移**  
如：  
```cpp  
long cread(long *xp)  
{  
    return (xp ? *xp : 0);  
}  
```  
如果用条件转移：  
```  
long cread(long *xp)  
Invalid implementation of function cread  
xp in register %rdi  
1 cread:  
2 movq (%rdi), %rax       v = *xp  
3 testq %rdi, %rdi        Test x  
4 movl $0, %edx Set       ve = 0  
5 cmove %rdx, %rax        If x==0, v = ve  
6 ret Return v  
```  
那么在第 2 行 movq 指令中，当 `xp` 为空指针时，也会获取其值，将会出错。  
      
**2、不是所有用条件转移的情况效率都会更高**  
如果条件判断语句中有大量的计算过程，那么会浪费大量的工作在错误的分支计算中。  
      
因此，只有当条件表达式的计算较简单时采用**条件转移**。  
      
## 3.6.7 Loops  
`do-while`，`while` 和 `for` 通过条件判断和跳转实现循环控制。  
      
## 3.6.8 Switch Statements  
A `switch` statement provides a **multiway branching capability** based on the value of an **integer index**.  
      
Not only do they make the C code more **readable**, but they also allow an **efficient** implementation using a data structure called a `jump table`.  
      
A `jump table` is an `array` where entry `i` is the address of a **code segment** implementing the action the program should take when the `switch` index equals `i`.  
      
The code performs an **array reference** into the `jump table` using the `switch` **index** to determine the target for a jump instruction.  
      
The **advantage** of using a `jump table` over a long sequence of `if-else` statements is that the time taken to perform the `switch` is independent of the number of `switch cases`.  
      
`Jump tables` are used when there are **a number of cases** (e.g., four or more) and they span a **small range of values**.  
      
用 `switch` 比用大量的 `if-else` 效率高。  
      
# 3.7 Procedures  
过程是一个抽象，隐藏实现的细节，提供接口使用。对于不同的编程语言，其叫法不同，但本质特征一样。  
      
示例：  
```cpp  
long mult2(long, long);  
void multstore(long x, long y, long *dest)  
{  
    long t = mult2(x, y);  
    *dest = t;  
}  
```  
      
汇编代码：  
```  
multstore:  
pushq %rbx  
movq %rdx, %rbx  
      
call mult2  
movq %rax, (%rbx)  
popq %rbx  
ret  
```  
      
当 `P` 调用过程 `Q`，将执行以下操作：  
**1、传递控制 Passing control**  
调用 `call` 指令，首先**减小栈指针地址**（8位）**更新栈顶位置**，然后将这条**调用指令之后的指令地址**写入**栈顶**（调用返回后执行的指令地址），**程序计数器**将被设置为**被调用函数的首地址**（该地址存在 `call` 指令中），该指令结合了 `jump` 和 `push` 的功能。  
      
当被调用的过程 `Q` 执行完成，会执行 `ret` 指令（或 `retq`），该指令就是逆转 `call` 指令的效果。它会假设栈顶有一个想要跳转的地址，然后将栈顶的地址弹出（`pop`指令，弹出后栈顶的指针会增加，**而该地址的内容不会消失，只是不属于栈的一部分**），然后将程序计数器设置为弹出的地址，因此程序会回到原来的地方继续执行。  
      
**2、传递数据 Passing data**  
存放参数的寄存器有6个：`%rdi, %rsi, %rdx, %rcx, %r8, %r9`，存放返回值的寄存器位 `%rax`。  
上述这些寄存器只能存放整型和指针，如果参数超过6个，则参数会被放入栈中。（参数放在寄存器中比栈中访问速度更快）  
      
**3、管理和释放内存 Allocating and deallocation memory**  
被调用的函数 `Q` 必须为**局部变量分配空间**，并且在返回前释放存储空间。  
      
栈结构：  
![General stack frame structure](https://img-blog.csdnimg.cn/85a56a157885442aa32153ed52302ba7.png)  
      
## 3.7.1 The Run-Time Stack  
Using our example of procedure `P` calling procedure `Q`, we can see that while `Q` is executing, `P`, along with any of the procedures in the chain of calls up to `P`, is temporarily **suspended**.  
当 `Q` 正在被执行时，`P` 以及与其相关的调用 `P` 的过程都处于挂起状态。  
      
执行特定函数时，只需要引用该函数内部的数据或者已传递给它的值，而其他函数处于冻结状态，这种为**单线程运行模式**。  
      
**栈帧（stack frame）**：栈上用于特定 `call` 的每个内存块成为栈帧（It is a frame for a particular instance of a procedure, a particular call to a procedure）。  
      
需要在**栈**中为每个**被调用且未返回的过程**保留一个**栈帧**。  
      
通常一个**栈帧**由**两个指针**分隔，一个是**栈指针**（指向栈顶），另一个是**基指针**（base pointer)，由寄存器 `%rbp` 保存，**基指针是可选的**，指向当前栈帧的起始位置，即当前的栈底。  
      
用**递归**的弊端：会不断增加**栈帧**，需要很多空间，而大多数系统会**限制栈的深度**。  
      
当 `P` 调用 `Q` 时，会将 `Q` 返回后地址（返回后要执行的位置）放在栈上，该返回地址也被认为是 `P` 栈帧的一部分。  
      
大多数过程的**栈帧**有**固定的尺寸**，在**过程**开始执行时就分配好空间，但有些情况需要**可变大小的栈帧**，如过程中传递的参数数目大于6个时，会在 `Q` 被调用前将多余的参数存放在 `P` 的栈帧上。  
      
并非所以的**函数**都会用到**栈帧**，当一个函数的**全部局部变量**都存在**寄存器**中，且函数内部**不需要调用其他函数**，不需要**栈帧**。  
      
## 3.7.4 Local Storage on the Stack  
At times, however, `local data` must be stored in `memory`. Common cases of this include these:  
- There are **not enough registers** to hold all of the local data.  
- The address operator `&` is applied to a **local variable**, and hence we must be able to **generate an address** for it.  
- Some of the **local variables** are **arrays** or **structures** and hence must be **accessed by array or structure references**.  
      
## 3.7.5 Local Storage in Registers  
Although only one `procedure` can be **active** at a given time, we must make sure that when one `procedure` (the `caller`) calls another (the `callee`), the `callee` **does not overwrite** some register value that the `caller` planned to use later.  
      
当一个过程调用另一个过程时，必须保证**被调用者**不会**覆盖**掉**调用者**以后需要使用的**寄存器**。  
      
By convention, registers `%rbx, %rbp, and %r12–%r15` are classified as `callee saved registers`.  
      
当 `P` 调用 `Q`时，`Q` 必须保证那些在调用完后 `P` 仍需使用的寄存器的值在调用前后保持不变（如寄存器 `%rdi` （第一个参数的值）)，可以有两种方式：  
1、保证 `Q` 不会改变这些寄存器的值。  
2、在 `Q` **被调用前**先将这些寄存器的值压入**栈**中，然后在**调用结束**后**弹出**这些值。这种方式在压入栈中是在**栈帧**中创建的区域被标记为 `被保存的寄存器(Saved register)`。  
      
临时存放寄存器的值有两种：  
- **Caller Saved**  
调用者在**调用前**将**临时值**存放在它的**栈帧**中。  
- **Callee Saved**  
**被调用者**在执行前将临时值存在它的栈帧中，然后在返回到调佣处前恢复这些值。  
      
寄存器 `%r10` 和 `%r11` 为 `Caller Saved` 寄存器，用于存放任何可以被`函数`修改的**临时值**。  
      
寄存器 `%rbx, %rbp` 和 `%r12–%r15` 为 `Callee Saved` 寄存器，当一个函数要改变这些寄存器的值时，必须先压入栈中保存再在返回时从栈中弹出恢复数据  
      
示例：  
![Code demonstrating use of callee-saved registers](https://img-blog.csdnimg.cn/f53b4598f9394a938f609e86342eafb6.png)  
      
上述代码使用 `callee-saved` 寄存器 `%rbp`来存放 `x` 的值，用寄存器 `%rbx` 来存放 `Q(y)` 计算的结果。在函数的最开始，先将这两个寄存器的内容压入栈中，最后在函数的结果再弹出栈的内容到寄存器中。  
注意压入栈和弹出栈时的顺序相反（先进后出）。  
      
# 3.8 Array Allocation and Access  
**机器代码里没有数组**的概念，数组即为**连续的字节**集合。  
    
## 3.8.1 Basic Principles  
For data type `T` and `integer constant N`, consider a declaration of the form `T A[N]`;  
      
标注数组起始地址为 $x_{A}$，该数组将分配一块连续的内存空间，大小为 $L \cdot N$，其中 `L` 为数组元素类型 `T` 的大小（bytes)，`N` 为数组元素的个数。  
      
标志符 `A` 将被用作一个指向数组起始地址的**指针**，数组元素的索引在 `0` 到 `N-1` 区间，第 `i` 个元素的索引为 $x_{A} + L  \cdot i$。  
      
The memory referencing instructions of `x86-64` are designed to simplify array access.  
      
假设 `E` 是一个元素为 `int` 型的数组，`E` 的地址存在寄存器 `%rdx` 中，而索引 `i` 则存在寄存器 `%rcx` 中，如果想获取 `E[i]`，如下指令：  
```  
movl (%rdx,%rcx,4),%eax  
```  
将实现 $x_{E} + 4i$，从该地址的内存处读取数据，然后存在寄存器 `%eax` 中（`%eax` 是 32 位存返回值的寄存器，`%rax` 是64位 存放返回值寄存器）。  
      
比例因子（**scaling factors**）的值 可以是 1，2，4 和 8，分别对应前面说的四种数据大小，上面例子为 4，表示数组元素大小为 4 字节。  
      
## 3.8.2 Pointer Arithmetic  
C allows arithmetic on pointers, where the computed value is scaled according to the size of the data type referenced by the `pointer`.  
That is, if $p$ is a pointer to data of type $T$ , and the value of $p$ is $x_{p}$, then the expression $p+i$ has value $x_{p} + L \cdot  i$, where $L$ is the size of data type $T$ .  
      
示例：  
![Examples](https://img-blog.csdnimg.cn/c06edb8c9a884a1b9f0f2435005e14da.png)  
上述例子返回值的存放：the result being stored in either register `%eax` (for data) or register `%rax` (for pointers).  
      
上述返回结果为 `int` 类型的数组元素值时，用 `movl` 和寄存器 `%eax` ，而返回值为 `int *` 指针时，为 `leaq` （`leaq` 指令不是引用内存而是复制地址）和寄存器 `%rax`。  
      
最后一个例子展示计算有相同结构的两个指针相减，返回结果类型为 `long`，数值等于两个地址的差值除以元素类型的大小，即为两个地址相差的元素个数。  
      
## 3.8.3 Nested Arrays  
多维数组：对于数组 `T D[R][C]`，数组元素 `D[i][j]` 在内存中的地址为：  
&$D[i][j]$ = $x_{D} + L(C \cdot i + j)$  
      
其中 `L ` 是数据类型 `T` 的大小（bytes)。  
      
数组的结构如下图：  
![Nested Arrays](https://img-blog.csdnimg.cn/ded133cfbcc94d468bbf1c3dff4922fe.png)  
      
## 3.8.4 Fixed-Size Arrays  
示例：下面例子计算两个数组的相乘，数组 A 的第 i 行每个元素分别乘以数组 B 的第 k 列的每个元素，下面代码展示了优化前和优化后的两种实现方式。  
```c  
#define N 16  //通过 #define 来声明常量，便于修改尺寸  
//typedef 声明 fix_matrix 为一个二维数组，有 N 行，N列，元素类型为 int  
typedef int fix_matrix[N][N];  
```  
      
![fig 3.37](https://img-blog.csdnimg.cn/639623655b1943db944cf32c3c0494f4.png)  
      
优化后的汇编代码如下：  
![](https://img-blog.csdnimg.cn/2b287f4bbf284465a41450ceafd270ca.png)  
![](https://img-blog.csdnimg.cn/c3b263b0e74f451494b89dca79de46a6.png)  
      
`salq` 为左移指令，对于 &$A[i][0]$，根据公式 &$D[i][j]$ = $x_{D} + L(C \cdot i + j)$，可得到 $x_{A} + 64i$，而 `i` 左移 `6` 位即为 $i * 2^6$。  
      
13 行处比较的结果会保存在 `ZF` 标志中，相等则为1，然后根据 `jne` 指令，即判断 `ZF` 标志，不为 `0`，即不相等就跳转到标签 `L7` 处。  
      
## 3.8.5 Variable-Size Arrays  
Historically, C only supported **multidimensional arrays** where the **sizes** (with the possible exception of the first dimension) **could be determined at compile time**.  
      
Programmers requiring **variable-size arrays** had to allocate storage for these arrays using functions such as `malloc` or `calloc`, and they had to explicitly encode the mapping of **multidimensional arrays** into **single-dimension** ones via **row-major indexing**.  
      
可变大小的多维数组 `int A[expr1][expr2]` ，定义如下函数：  
```c  
int var_ele(long n, int A[n][n], long i, long j)  
{  
    return A[i][j];  
}  
```  
注意 `n` 要在 `A[n][n]` 前声明，汇编代码如下：  
![referencing function](https://img-blog.csdnimg.cn/6535e130644f471d896b6ab0302608e8.png)  
      
这里计算 $n \cdot i$ 用到 `imulq` 乘法指令，而非用左移指令。  
      
示例：  
![fig 3.38](https://img-blog.csdnimg.cn/029a74bf7acb4381a069b46c33e6efc2.png)  
      
优化后的汇编代码：  
![assembly code](https://img-blog.csdnimg.cn/4806aec55e6444eca05aced3438ff749.png)  
      
## 3.9 Heterogeneous Data Structures  
### 3.9.1 Structures  
The different components of a structure are **referenced by names**.  
      
The implementation of **structures** is similar to that of **arrays** in that all of the components of a **structure** are stored in a **contiguous region** of memory and a **pointer to a structure** is the address of its **first byte**.  
      
例如对以下结构体：  
```c  
struct rec  
{  
    int i;  
    int j;  
    int a[2];  
    int *p;  
};  
```  
      
其结构如下：  
![structure](https://img-blog.csdnimg.cn/e94b36f762b64876be8e312d903e1992.png)  
可见数组是嵌入在结构体中。  
例如变量 `r` 的类型为 `rec *`，获取结构体中成员的汇编代码：  
```  
Registers: r in %rdi  
1 movl (%rdi), %eax 		Get r->i  
2 movl %eax, 4(%rdi) 		Store in r->j  
```  
上述代码第一条为获取 `i` 的数值，然后存到返回值寄存器 `%eax` 中；  
第二条为将寄存器 `%eax` 的内容写入到变量 `r` 地址加 `4` 后的地址所在的内存处，即 `j` 的地址处，因此将 `j` 的数值设置为 `i` 的数值。  
      
To generate a **pointer to an object within a structure**, we can simply add the field’s offset to the structure address.  
For example, we can generate the pointer `&(r->a[1])` by adding offset $8 + 4 \cdot 1= 12$.  
For pointer `r` in register `%rdi` and long integer variable `i` in register `%rsi`, we can generate the pointer value `&(r->a[i])` with the single instruction:  
```  
Registers: r in %rdi, i %rsi  
1 leaq 8(%rdi,%rsi,4), %rax 		Set %rax to &r->a[i]  
```  
      
The selection of the different fields of a structure is handled completely at **compile time**.  
      
### 3.9.2 Unions  
**共用体**：allowing a **single object** to be **referenced** according to **multiple types**.  
      
**与结构体的区别**：Rather than having the different fields reference different blocks of memory, they all reference the **same block**.  
      
**共用体的大小**为共用体中**最大类型的变量的大小**。  
      
**共用体的使用场景**：  
1、One application is when we know **in advance** that the use of two different fields in a data structure will be **mutually exclusive**. Then, declaring these two fields as part of a union rather than a structure will **reduce the total space allocated**.  
提前知道需要使用几种不同的类型，且不同类型是互斥的使用。  
      
2、`Unions` can also be used to access the bit patterns of different data types.  
例如：需要做如下类型转换  
```c  
unsigned long u = (unsigned long) d;  
```  
通过以下方式实现：  
```c  
unsigned long double2bits(double d)  
{  
    union {  
        double d;  
        unsigned long u;  
    } temp;  
    temp.d = d;  
    return temp.u;  
};  
```  
The result will be that `u` will have the **same bit representation** as `d`, including fields for the sign bit, the exponent, and the significand.  
      
**注意事项**：  
When using unions to combine data types of **different sizes**, **byte-ordering** issues can become important.  
      
### 3.9.3 Data Alignment  
> [结构体字节对齐，C语言结构体字节对齐详解](http://c.biancheng.net/view/243.html)  
      
**数据对齐**：基于硬件的需求  
**Alignment restrictions** simplify the design of the hardware forming the interface between the `processor` and the `memory system`.  
      
使用数据对齐能提升内存系统的性能。  
      
对齐的规则：  
Their alignment rule is based on the principle that any primitive object of `K` bytes must have an address that is a **multiple of K**. We can see that this rule leads to the following alignments:  
![alignment rule](https://img-blog.csdnimg.cn/8326b402b5d641bfb618f9c1317ca498.png)  
      
具体 `K` 的大小是多少，由结构体中尺寸最大的类型对应的 `K` 决定。  
      
如：  
```c  
struct S1  
{  
    int i;  
    char c;  
    int j;  
};  
```  
对齐后如下，第二个数据 c 会填充 3 个字节来满足对齐：  
![alignment](https://img-blog.csdnimg.cn/8451accf600245158ede23efc481858f.png)  
      
有时编译器可能需要在结构体末尾填充，如：  
```c  
struct S2  
{  
    int i;  
    int j;  
    char c;  
};  
```  
对齐后：  
![alignment](https://img-blog.csdnimg.cn/7a9a01bd8aa8448a9b100435434f38b1.png)  
      
# 3.10 Combining Control and Data in Machine-Level Programs  
## 3.10.3 Out-of-Bounds Memory References and Buffer Overflow  
`GDB` 调试的命令：  
![Example gdb commands](https://img-blog.csdnimg.cn/c99e25f56ce6444a8e52ee1ee592a0c0.png)  
    
## 3.10.4 Thwarting Buffer Overflow Attacks (?)  
未看  
缓冲区溢出解决方案：  
1. Stack Randomization  
2. Stack Corruption Detection  
3. Limiting Executable Code Regions  
      
## 3.10.5 Supporting Variable-Size Stack Frames  
Some functions, however, require a **variable** amount of local storage.  
      
This can occur, for example, when the function calls `alloca`, a standard library function that can **allocate an arbitrary number of bytes** of storage on the **stack**.  
      
It can also occur when the code declares a `local array of variable size`.  
      
示例：  
```c  
long vframe(long n, long idx, long *q)  
{  
    long i;  
    long *p[n];  
    p[0] = &i;  
    for (i = 1; i < n; i++)  
        p[i] = q;  
              
    return *p[idx];  
}  
```  
上述代码包含可变尺寸的数组，数组 `p` 为包含 `n` 个指向 `long` 的指针，不同调用函数可能传递的 `n` 的值不同，而该数组需要在栈上分配 `8n` 字节，因此编译器无法知道为该函数的栈帧分配多少空间。  
      
此外，该函数用到了对局部变量 `i` 的地址的引用，因此该变量必须存在栈上。  
      
To manage a `variable-size stack frame`, `x86-64` code uses register `%rbp` to serve as a `frame pointer` (sometimes referred to as a `base pointer`, and hence the letters `bp` in `%rbp`).  
      
栈帧的结构如下：  
![Stack frame structure](https://img-blog.csdnimg.cn/f217d87a4b5f4c3987048a5c5ac232f8.png)  
      
We see that the code must **save the previous version** of `%rbp` on the `stack`, since it is a `callee-saved register`.  
It then keeps `%rbp` pointing to this position throughout the execution of the function, and it references `fixed-length local variables`, such as `i`, at offsets relative to `%rbp`.  
      
汇编代码如下：  
![Function requiring the use of a frame pointer](https://img-blog.csdnimg.cn/85bb4e80e18c41b79aaa7226056a2c85.png)  
      
1. 第 2 行，保存当前 `%rbp` 寄存器的值到栈中。  
2. 第 3 行，将 `%rbp` 寄存器的值设置为栈指针位置 `%rsp`。  
3. 第 4 行，将栈指针位置向下扩展16 字节，前 8 个字节存放局部变量 i，后 8 个字节未使用（unused）。  
4. 第 5 行，将寄存器 `%rax` 的值设置为 22+8n。  
5. 第 6 行，立即数 -16 的补码为 0xFFFFFFFFFFFFF0（假设用 5 位表示，则模为 32，32 - 16 = 16，因此其补码为 1 0000，对于补码高位用符号位扩展后结果不变，从而高位全部为1），因此 `%rax` 的值与 -16 相与的结果是将低 4 位设置为 0，其余高位不变，例如 0xF1 (241) 到 0xFF (255) 之间的数将变为 0xF0（240），而 240 + 16 = 256，这样相当于 `%rax` 的值变为最大的小于该值的能被 16 整除的数。对于 8n + 22，当 n 是奇数时，结果为 8n + 8；当 n 是 偶数时，结果为 8n + 16。  
6. 第 7 行，将 `%rsp` 的值减去 `%rax` 的值后作为 `%rsp` 的值，即栈指针向下扩展，扩展的大小为 `%rax` 的值。  
7. 第 8 行，设置寄存器 `%rax` 的值为 7 + `%rsp` 的值。  
8. 第 9 行，将 `%rax` 的值逻辑右移 3 位，高位补 0，即将该值除以 8，去掉余数。  
9. 第 10 行，将 `%r8` 的值设置为 `%rax` 的值 * 8，该位置即为数组 p 的起始位置。  
10. 第 11 行， 将 `%rcx` 的值设置为 `%r8` 的值。  
      
假设 n 为 5，s1 为 2065 和 n 为 6，s1 为 2064，则其他的值为：  
![solution 3.49](https://img-blog.csdnimg.cn/d0ad07c1bf49408aa4def92557a4377b.png)  
      
In earlier versions of `x86 code`, the frame pointer was used with every function call. With `x86-64 code`, it is used only in cases where the `stack frame` may be of **variable size**, as is the case for function `vframe`.  
      
# 3.11 Floating-Point Code  
未看  
存放浮点数据的寄存器：  
![Media registers](https://img-blog.csdnimg.cn/e99c04da6142486794cf0349f97204a8.png)  
      
Floating-point movement instructions:  
![Floating-point movement instructions](https://img-blog.csdnimg.cn/2b881d9d47de44549378a82199a44d5a.png)  
    
## 3.11.1 Floating-Point Movement and Conversion Operations  
浮点型转换操作：  
![floating-point conversion operations](https://img-blog.csdnimg.cn/181f2f5762b64fa7b6bcee56d2297c89.png)  
    
## 3.11.6 Floating-Point Comparison Operations  
浮点型比较：  
![Floating-Point Comparison Operations](https://img-blog.csdnimg.cn/f176bb63843e4c28853509ab105f90f5.png)  
