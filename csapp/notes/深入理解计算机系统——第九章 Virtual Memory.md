> [视频课程](https://www.bilibili.com/video/BV1iW411d7hd?p=17)
> [视频课件](http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/schedule.html)
> [深入理解计算机系统——第9章 虚拟内存](https://blog.csdn.net/baidu_15952103/article/details/122462967)
> [虚拟内存](https://zhuanlan.zhihu.com/p/96098896)
> [虚拟地址转换[一] - 基本流程](https://zhuanlan.zhihu.com/p/65298260)
> [虚拟地址转换[二] - 具体实现](https://zhuanlan.zhihu.com/p/65348145)
> [彻底搞懂虚拟地址翻译为物理地址的过程](https://www.toutiao.com/article/6955273381021319712/?wid=1661925549685)
> [进程—内存描述符（mm_struct）](https://blog.csdn.net/qq_26768741/article/details/54375524)
> [malloc](https://docs.microsoft.com/zh-cn/previous-versions/visualstudio/visual-studio-2012/6ewkz86d(v=vs.110))
> [Heap - Part I](https://www.cnblogs.com/yxqxx/p/13883084.html)
> [CSAPP-MallocLab](https://blog.csdn.net/qq_45531291/article/details/122629911) 
> [CMU CSAPP笔记 第九章](https://blog.csdn.net/winter_wu_1998/article/details/81152540)
> 


# 9.1 Physical and Virtual Addressing
**physical address (PA)**：系统的主存是由 M 个连续的字节单元组成，每个字节都有一个唯一的**物理地址（physical address）**。第一个字节在地址 0，接着是地址1，地址2，等，见下图：
![fig 9.1](https://img-blog.csdnimg.cn/7c02a986e16d40fa91836a48ea8381e2.png)
<br/>

**virtual address (VA)**：CPU 产生一种**虚拟地址（VA）**，然后通过**地址翻译**转化为**物理地址**传送给**主存**。

**address translation**：将**虚拟地址**转化为**物理地址**的过程。

**memory management unit (MMU)**：**CPU** 中的**内存管理单元**，进行**地址翻译**。

![A system that uses virtual addressing](https://img-blog.csdnimg.cn/038624df0277480e8e81d1e6431f275c.png)
<br/>

# 9.2 Address Spaces
**address space**：地址空间是一个由**非负整数地址**组成的有序的集合：${0,1,2, ...}$ 。

**linear address space**：地址空间中的整数是**连续不间断**的。

**virtual address space**：CPU 从一个 $N$ = $2^n$ 个地址的**地址空间**中生成虚拟地址组成的集合：${0,1,2, ...，N-1}$；该虚拟地址空间称为 n-bit 地址空间，**现代操作系统通常有 32-bit 或 64-bit 的虚拟地址空间**。

**physical address space**：物理地址空间，对应系统中 M 字节的物理内存： ${0,1,2, ...，M-1}$；M 不需要是 2 的幂，但这里假设 $M$ = $2^m$。

The concept of an **address space** is important because it makes a clean distinction between **data objects (bytes)** and their **attributes (addresses)**. 

We can generalize and allow **each data object** to have **multiple independent addresses**, each chosen from a **different address space**.

This is the basic idea of **virtual memory**. **Each byte of main memory** has a **virtual address** chosen from the **virtual address space**, and a **physical address** chosen from the **physical address space**.

# 9.3 VM as a Tool for Caching
**虚拟内存**被组织成存储在**磁盘**上的 N 个连续的字节单元数组，每个字节都有唯一的**虚拟地址**作为**数组的索引**。

**磁盘**上**数组的内容**被**缓存在主存**中，这些数据被分割为**块（block）**的形式作为**磁盘**和**主存**的传输单元。(见第六章介绍)

**virtual pages (VPs)**：**虚拟内存系统**将虚拟内存分割为的**固定尺寸的块**称为**虚拟页**，每个虚拟页的尺寸为 $P$ = $2^p$。

**physical pages (PPs)**：**物理内存**被分割的**块**称为**物理页**，也是 P 字节，物理页也叫**页帧（page frames）**。


任何时刻，**虚拟页**被分为**三个不相交的子集**：
- **Unallocated**
还没被 VM 系统创建的页，没有数据，**不占空间**
- **Cached**
已缓存在**物理内存**中的**被分配**了的**页**
- **Uncached**
被分配但**没缓存在物理内存**中的页

见下图：
![fig 9.3](https://img-blog.csdnimg.cn/a309d4bccd894234b03cd46d5452d122.png)
## 9.3.1 DRAM Cache Organization
**约定**：
- Use the term **SRAM** cache to denote the L1, L2, and L3 cache memories between the **CPU and main memory**.
- Use the term **DRAM** cache to denote the **VM system’s cache** that caches **virtual pages in main memory**.

**DRAM** 未命中的代价大，因此虚拟页大，通常在 4KB 到 2MB 之间。

**DRAM** 缓存是**全相关**的（第六章 6.4.4 有讲），任何虚拟页能被放在任何物理页。

**DRAM** 缓存总是用 **write-back** 。(6.4.5)

## 9.3.2 Page Tables
**page table**：
- 存储在**物理内存**中的数据结构，用来将**虚拟页**映射到**物理页**；
- 页表是一个由 **页表条目（page table entries, PTEs）** 组成的数组；
- Each **page** in the **virtual address space** has a **PTE** at a **fixed offset** in the **page table**. 

假设每个 PTE 由一个 **valid bit** 和 **n-bit** 地址组成；
其中 **valid bit** 表示虚拟页是否缓存到 DRAM 中；
<br/>

![fig 9.4](https://img-blog.csdnimg.cn/b87a92ed754040f8b66e5ad2ebcc52c0.png)
<br/>



## 9.3.3 Page Hits
如下图 9.5 所示，当 CPU 读虚拟内存中 VP 2 的内容时，**地址翻译硬件**使用**虚拟地址**作为**索引**定位到**页表**中的 **PTE2** 的条目，该条目的**有效位**是 1，表示 VP 2 已被**缓存到内存**，因此用 **PTE 2** 中的**物理内存地址**（该地址指向 PP1 的起始位置）来查找数据的**物理地址**。
<br/>

![fig 9.5](https://img-blog.csdnimg.cn/0499adb4c8dd4e5a93ad2bd179508a50.png)
## 9.3.4 Page Faults
**page fault**：虚拟内存的术语中用 **缺页（page fault）** 来表示 **DRAM 缓存未命中**。

如下图所示：
<br/>
![page fault](https://img-blog.csdnimg.cn/448911056a494e029b2326926065c2b8.png)<br/>

1. CPU 引用了一个没有缓存在 DRAM 中的 VP 3 中的数据；
2. **地址翻译硬件**通过**页表**查找到 PET 3，发现其**有效位**为 0，因此**触发缺页异常**（page fault exception）；
3. **内核**中的一个**缺页异常处理程序**（page fault handler）选择存储在物理内存 PP 3 的 VP 4 作为 victim page；
4. 如果 VP 4 已经被修改过，则内核将它的内容复制回磁盘，否则内核将修改页表条目的有效位为0；
5.  然后内核从磁盘复制 VP 3 的内容放到内存的 PP 3 的地址处，然后更新 PTE 3，最后返回；
6. 处理程序返回时，它重启缺页指令，重新将 faulting virtual address 发送给地址翻译硬件，然后就能正常命中；


由于历史原因，虚拟内存系统使用和 SRAM 缓存不同的术语。虚拟内存的术语中：
- blocks 被称为 pages
- **transferring a page** between **disk and memory** is known as **swapping or paging**
- Pages are **swapped in** (paged in) from **disk to DRAM**, and **swapped out** (paged out) from **DRAM to disk**
- The **strategy** of **waiting** until the **last moment** to **swap in a page**, when a **miss** occurs, is known as **demand paging**

## 9.3.5 Allocating Pages
见下图：
![fig 9.8](https://img-blog.csdnimg.cn/65a5c2ed903744dbbbaac41071af0349.png)
<br/>

上图中分配一个新的虚拟页 VP 5，让 PTE 5 指向该地址。如 `malloc` 的调用会创建新的虚拟页。

## 9.3.6 Locality to the Rescue Again
The principle of **locality** promises that at any point in time they will tend to work on **a smaller set of active pages** known as the **working set** or **resident set**.

If the **working set size exceeds the size of physical memory,** then the program can produce an unfortunate situation known as **thrashing**, where pages are **swapped in and out continuously**.

# 9.4 VM as a Tool for Memory Management
In fact, operating systems provide a **separate page table**, and thus a **separate virtual address space**, for **each process**.

**多个虚拟页**可能被映射到**相同的共享的物理页**。
<br/>
![fig 9.9](https://img-blog.csdnimg.cn/460cf43abc384eedb1cc61798dbda227.png)<br/>

VM simplifies **linking** and **loading**, the **sharing of code and data**, and **allocating memory to applications**.
- **Simplifying linking**
**独立的地址空间**允许每个**进程**的**内存映像**使用**相同的基本格式**。第8章 8.2.3 中的图 8.13 中展示了每个进程在一个给定的 Linux 系统中有相似的内存格式。这种一致性简化了连接器的设计和实现，允许连接器生成独立于物理内存中代码和数据的最终地址的完全链接的可执行程序。

- **Simplifying loading**
虚拟内存使得**加载可执行和共享目标文件**到内存更容易。将**目标文件**中的 `.text` 和 `.data` 节加载到一个新的进程中时，Linux **加载器**会为**数据和代码段**分配**虚拟页**，设置他们的有效位为 **0** 来表示**未被缓存**，然后将**页表条目**指向**目标文件**的合适的位置。因此**加载器**不用从**磁盘复制数据到内存**，数据将会在**第一次被引用时**自动被调用（page in）。
**memory mapping**：将一组**连续的虚拟页**映射到**随机的文件**中的**随机位置**，称为**内存映射**。

- **Simplifying sharing**
**Separate address spaces** provide the **operating system** with a **consistent mechanism** for managing **sharing** between **user processes** and the **operating system** itself. 

- **Simplifying memory allocation**
虚拟内存为给用户进程**分配额外的内存**提供了一个简单的机制。如当用户调用 `malloc` 而需要额外的堆空间时，**虚拟地址**可以分配**连续的虚拟内存页**，然后映射到**随机的物理地址内存**中，不需要连续的物理内存页。

# 9.5 VM as a Tool for Memory Protection
![fig 9.5](https://img-blog.csdnimg.cn/0b18815612bf42718d279974088bc72f.png)<br/>

上图中，在每个 PTE 中增加了三个**许可位**：
- **SUP**
表示是否**进程**必须在**内核模式**运行才能访问该**页**
- **READ** 和 **WRITE**
控制对页的读和写

如果指令**违反**了这些**许可**，则 CPU 将触发一个**一般保护故障（general protection fault）**，将控制传递给内核中的**异常处理程序**，处理程序会发送一个 `SIGSEGV` 信号给违反许可的进程。Linux shells 通常将这种异常报告为 **segmentation fault**。

# 9.6 Address Translation
本节将使用的符号：
![fig 9.11](https://img-blog.csdnimg.cn/e3429c2425a64912a16533e0aa8cb375.png)
<br/>

MMU 利用**页表**将**虚拟地址空间**映射到**物理地址空间**的过程：
![fig 9.12](https://img-blog.csdnimg.cn/69885f1a05f248a29393b6738d2221ff.png)
<br/>

1. CPU 中的一个**控制寄存器 PTBR**（page table base register）指向当前的页表；
2. n-bit 的虚拟地址由两部分组成：p-bit virtual page offset (VPO) 和 (n − p)-bit virtual page number (VPN)。
3. MMU 利用 VPN 来选择 PTE，例如 VPN 0 选择 PTE 0 等。
4. 物理地址由两部分组成：physical page number (PPN) 和 the physical page offset (PPO)；PPO 和 VPO 相同，大小均为页的大小，数值也相同。

**VPN 和 VPO 的含义**：
假如虚拟地址的所有位均为 0，即第一个地址，VPN 为 0，指第 0 页，对应 PTE 0；而 VPO 也是 0，代表**第 0 页中第一个字节**；
如果第二个地址，即 VPN 为 0，VPO 为 1，则表示第 0 页中的第二个字节，仍在 PTE 0 页表条目中；
因此可以通过 VPN 作为 PTE 的索引，而 VPO 则表示在该页中的索引；


******************
图 9.13(a) 展示了当有 page hit 时 CPU 硬件的操作，该操作完全由硬件处理:
![fig 9.13(a)](https://img-blog.csdnimg.cn/9bcf846c9cd84d8e8f3d59673bb8d999.png)
<br/>

**Step 1:** 处理器生成一个虚拟地址并传送给 MMU；

**Step 2:** MMU 生成 PTE 地址，and requests it from the cache/main memory；

**Step 3:** 缓存/主存将 PTE 返回给 MMU;

**Step 4:** MMU 构建一个物理地址并传送给缓存/主存；

**Step 5:** 缓存/主存返回需要的数据给处理器；

**缺页**的操作需要硬件和操作系统内核合作处理：
![page fault](https://img-blog.csdnimg.cn/980ac4308b424e72bbe9017f73b001f7.png)
<br/>

**Step 1-3:** 步骤 1-3 和 page hit 的前三步相同

**Step 4:** 因为 PTE 的有效位为 0，因此 MMU 触发一个异常，将控制传给内核中的缺页异常处理程序

**Step 5:** 缺页异常处理程序从物理内存中找到一个 victim page，如果该页的内容被修改过，则将其内容更新到磁盘（page out）

**Step 6:**  缺页异常处理程序将数据写入到前面选择的 victim page（page in），然后更新 PTE

**Step 7:** 缺页处理程序返回到原来的进程，重启缺页指令，CPU 重新发送虚拟地址到 MMU，此时能命中并顺利执行

## 9.6.1 Integrating Caches and VM
大多数系统选择用物理地址来访问 SRAM 缓存，图 9.14 展示了将物理地址缓存和虚拟内存相结合：
![fig 9.14](https://img-blog.csdnimg.cn/a1e95e802c03446789bb0bf41be09620.png)
<br/>

注意：地址翻译在 catch lookup 前完成，因此缓存无需处理保护问题。

## 9.6.2 Speeding Up Address Translation with a TLB
每次 CPU 产生虚拟地址，都要查阅 PTE 来找到对应的物理地址，这将需要额外的 fetch from memory。

为了解决这个问题，许多系统在  MMU 中包含了一个小的关于 **PTE 的缓存**，称为**转译后备缓冲器（translation lookaside buffer，TLB）**。

- A **TLB** is a small, **virtually addressed cache** where each **line** holds a block consisting of **a single PTE**. 

- A **TLB** usually has a **high degree of associativity**.

见下图：
![TLB](https://img-blog.csdnimg.cn/21cbc870f66f4c8ebdfade76052d1679.png)
<br/>

下图展示了一个 TLB hit 的过程：

![fig 9.16](https://img-blog.csdnimg.cn/6b8c663410e24d5ba27f71bcc996b249.png)
<br/>

上图（a）中命中的步骤：
**Step 1:** CPU 产生一个虚拟地址
**Step 2-3:** MMU 从 TLB 中获取合适的 PTE
**Step 4:**  MMU 将虚拟地址翻译为物理地址并发送给缓存/主存
**Step 5:**  缓存/主存将数据返回给 CPU  

## 9.6.3 Multi-Level Page Tables

> [多级页表的原理](https://blog.csdn.net/forDreamYue/article/details/78887035?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-78887035-blog-118616334.t5_layer_eslanding_D_0&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-78887035-blog-118616334.t5_layer_eslanding_D_0&utm_relevant_index=2)
> [谈一谈内存管理，虚拟内存，多级页表](https://www.jianshu.com/p/45fa8bd131be)
> 

**假如地址空间为 32-bit，一页的大小为 4KB，一个 PTE 为 4字节**：
- 则**虚拟地址空间**的集合为 ${0,1,2, ...，2^{32}-1}$，即有 $2^{32}$ 个虚拟地址（9.2 节）；
- 根据每个**页**的大小为 4KB（ $2^{12}$ 字节），则总共有 $2^{32} / 2^{12}$，即 $2^{20}$ 个页；
- 由于每个页都对应一个 PTE （9.3.2 节）,而每个 PTE 的大小为 4 字节，则页表的大小为 $2^{20} \times 4$，即 4MB。

**使用多级页表减小页表的大小，例如上述条件使用两级页表：**
![fig 9.17](https://img-blog.csdnimg.cn/978850a5287d4f75ba9888af5cf66c8c.png)
<br/>

**虚拟地址空间**的形式：
1. 虚拟内存中前 2K 的页（前2048个页）用来分配代码和数据；
2. 接下来 6K 的页 不分配；
3. 接下来的 1023 个页不分配；
4. 接下来的一个页分配给用户栈；

**多级页表映射**：
- 第一级 PTE 用于映射 4M 的虚拟地址空间片（chunk），每个 chunk 由连续的 1024 个页组成（一个页 4KB，$4 \times 2^{10} \times 2^{10}$ = 4MB）。

- 因此，1024 个 PTE 能映射 4G 的内存空间。

- 如果 chunk i 中的每个页都未分配，则 level 1 中对应的的 PTE i 为空；

- 只要 chunk i 中有一个页分配了内容，则 level 1 中的 PTE i 指向 the base of level 2 page table （见上图）；

- 二级页表中的每个 PTE 映射虚拟内存中的一页，即 4K 的地址；

多级页表降低内存需求：
- 如果一级页表为空，则二级页表不存在；
- **只有一级页表需要一直在主存中**，二级页表可以在需要时由虚拟内存系统创建；只有最常用的二级页表才需要缓存在主存中；

多级页表示例：
![fig 9.18](https://img-blog.csdnimg.cn/d185c182201a4fab9401f326519491e5.png)

## 9.6.4 Putting It Together: End-to-End Address Translation
会涉及第 6 章，6.4 节的内容。

本节通过一个具体的例子描述地址翻译的过程，假设条件如下：
<br/>
![假设条件](https://img-blog.csdnimg.cn/3ed192a976c94e25be541fc32829f61f.png)
<br/>

下图 9.19 展示了虚拟地址和物理地址的形式：
![fig 9.19](https://img-blog.csdnimg.cn/60700d44ce1a4ee48b7492f9095605ea.png)
<br/>

1. **虚拟地址**和**物理地址**的**低6位**分别作为 VPO和 PPO（因为页的大小为 $2^{6}$ = 64 字节）；
2. **虚拟地址**的**高8位**和**物理地址**的**高6位**分别作为 VPN和PPN；

下图 9.20 展示了内存系统：
<br/>

![fig 9.20](https://img-blog.csdnimg.cn/9bf8a3b728284fb4adaaa445bd429c60.png)
<br/>

**TLB**：
- The TLB is **virtually addressed using the bits of the VPN**.
- TLB 是 PTE 的缓存，而 VPN 代表了 PTE 的索引号，VPN 有 8 位；
- TLB 有 4 组，因此用低2位来作为组的索引（00， 01， 10 ，11），表示为 TLBI，共 4 组，一组  4 行（4-way）；
- TLB 剩下的 6 位作为 标志位（tag），表示位 TLBT；

**Page table**：
- 页表只有一级，共有 256 页，图中仅展示 16 个 PTE；
- 图中每个 PTE 有一个 VPN 作为索引，但 VPN 并不存在于页表中，只在图中展示出来看；
- 无效的 PTE 中的 PPN 表示为**短线（—）**；

**Catche**：
- The direct-mapped cache is addressed by the fields in the physical address.
- 每个块是 4 字节，而一个 word 是一个字节，因此用物理地址的最低 2 位表示 block offset（CO）；
- 由于有 16 组，一组一行，因此用物理地址中接下来的 4 位作为组索引（CI）;
- 物理地址剩下的 6 位作为 tag （CT）;

**CPU 执行 load 指令，读地址 0x03d4 的一个字节（这里 1 word 是 1 字节）的过程**：
1. MMU 提取虚拟地址 0x03d4 中 VPN 的值：
![0x03d4](https://img-blog.csdnimg.cn/0774d7d2c15b482b8b2b75521f97530e.png)<br/>
上图为地址 0x03d4 对应的二进制，VPN 的值为 PTE 的索引号，这里为 0x0f。

2. 根据 VPN 的值从 TLB 中查看是否有对应 PTE 0x0f 的缓存
前面说过 TLBI 为 TLB 缓存组的索引，这里为 0x03，通过下图：
![TLB](https://img-blog.csdnimg.cn/e07d1d60d57f4ed9bcb4f361a02d8453.png)<br/>
可以找到 Set 3，然后根据 TLBT，即 Tag 查找 Set 3 中是否有相同 Tag 的块，找到第二行 Tag 相同，其有效位 Valid 为 1，则说明该块已被缓存，即命中。

3. 根据命中的块中 PPN 的值找到对应的物理地址
将 TLB 中 PPN 的值 0x0D，以及偏移 VPO 的值 0x14 组合得到物理地址 0x354（0011 0101 0100）。（前面可看到物理地址有 12 位，低 6 位为 PPO，高 6 位为 PPN，而 PPO 和 VPO 相同均为 0x14，因此低 6 位为 01 0100；高 6 位为 00 1101；因此组合一起为 0011 0101 0100，即0x354）

4. MMU 将物理地址发给缓存，提取物理地址最低的两位作为偏移 CO（0x00），接下来的 4 位作为索引 CI （0x5），剩下的 6 位为 tag CT（0x0D），找到缓存如下：
![Cache](https://img-blog.csdnimg.cn/1359fc1635704381939169e6d0509dd1.png)
<br/>
索引 0x05 对应的组只有一行，该行的有效位 Valid 为 1，表示数据有效，Tag 为 0x0D 与物理地址的 Tag 相同，因此命中，再根据物理地址的偏移 0x00 找到对应的块 Blk 0，读该块的内容 0x36（缓存中一块为 1 字节，而 1 word 也是 1 字节）。

5. 将找到的数据 0x36 返回给 MMU，然后 MMU 发送给 CPU。
  
上述过程是命中的情况，还有其他可能，如 PTE 有效但缓存中的块未命中等。


# 9.7 Case Study: The Intel Core i7/Linux Memory System
We conclude our discussion of virtual memory mechanisms with a case study of a **real system**: an **Intel Core i7** running **Linux**.

Figure 9.21 gives the highlights of the **Core i7 memory system**. 

The **processor package (chip)** includes **four cores**, a large **L3 cache** shared by **all of the cores**, and a **DDR3 memory controller**.

![fig 9.21](https://img-blog.csdnimg.cn/2e1d540f287847f68a439fa7d1d40511.png)
<br/>


<br/>

The **TLBs** are **virtually addressed**, and **4-way set associative**.
The **L1, L2, and L3 caches** are **physically addressed**, with a block size of 64 bytes.
**L1 and L2** are **8-way set associative**, and **L3** is **16-way set associative**. 

The **page size** can be configured at **start-up time** as either 4 KB or 4 MB. 
Linux uses **4 KB pages**.

## 9.7.1 Core i7 Address Translation
下图为 Core i7 地址返回过程：
<br/>

![fig 9.22](https://img-blog.csdnimg.cn/e8e88d4565204e42bc4d1a3a5b72baf9.png)
<br/>

Core i7 有**四级页表**，每个进程有独立的 page table hierarchy。

When a **Linux process** is **running**, the **page tables** associated with allocated **pages** are all **memory-resident**, although the Core i7 architecture allows these page tables to be **swapped in and out**. 

**CR3 控制寄存器**包含了 level 1 页表的**起始物理地址**，CR3 的值是每个进程上下文（content）的一部分，在每次上下文切换（context switch）时被恢复。

Figure 9.23 shows the **format of an entry** in a level 1, level 2, or level 3 **page table**. 
![fig 9.23](https://img-blog.csdnimg.cn/a93fa5f969e04424a31e06c57fa4ca3b.png)
<br/>

当 P 为 1（Linux 中通常为该值），12 - 51 位为 40-bit 的 PPN，为下级页表物理地址的首地址。 Notice that this imposes a 4 KB alignment requirement on page tables.  

*********************
Figure 9.24 shows the **format of an entry** in a **level 4 page table**. 
<br/>

![fig 9.24](https://img-blog.csdnimg.cn/02b3330700184d5dae5a2c0ee83934ef.png)
<br/>

When P = 1, the address field contains a **40-bit PPN** that points to the base of **some page** in **physical memory**. This imposes a 4 KB alignment requirement on physical pages. （每页的大小位 4KB）

**PTE 有三个许可位**：R/W，U/S 和 XD 。

**XD**：execute disable bit，64 位系统引入的，用来设置该 PTE 对应的页是否可执行，为 1 则不可执行。（如果有多级，则一个 PTE 可能对应很多页）

**XD** 位的作用：Allows the **operating system kernel** to **reduce the risk of buffer  overflow attacks** by **restricting execution to the read-only code segment**.

MMU 翻译每个虚拟地址时，也会更新两个位：
- MMU 设置 A 位
The MMU sets the **A** bit, which is known as a **reference bit**, each time a page is **accessed**. 内核能使用该位实现 **page replacement algorithm**。

- MMU 设置 D位
The MMU sets the **D** bit, or **dirty bit**, each time the **page is written to**.
当该页被选择为 victim 页时，内核查看该位可知该页的内容是否被修改过，从而决定是否需要写回。

内核能通过调用一个特殊的内核模式指令来清除 reference 或 dirty 位。

*****************
Figure 9.25 shows how the **Core i7 MMU** uses the **four levels of page tables** to **translate a virtual address to a physical address**. 
 <br/>

![fig 9.25](https://img-blog.csdnimg.cn/b200123e269d400bbef493facf058336.png)
<br/>

1. 虚拟地址的 36 位的 VPN 被分成 4 块，每部分 9 位，分别作为每一级页表的索引。
2. CR3 寄存器包含了第一级页表的起始物理地址。
3. 第一级页表的 12 - 51 位（40位）包含了第二级页表的起始物理地址。
4. 第四级页表的 12 - 51 位（40位）包含了对应页的起始物理地址（PPN）

## 9.7.2 Linux Virtual Memory System
一个虚拟内存系统需要硬件和内核的紧密配合。
本节描述一个真实的操作系统如何组织虚拟内存，以及如何处理缺页（page faults）。

Linux maintains a **separate virtual address space** for **each process** of the form shown in Figure 9.26. 
<br/>
![fig 9.26](https://img-blog.csdnimg.cn/828d519301b54e18b4091506c7285cd7.png)<br/>

The **kernel virtual memory** contains the **code and data structures** in the **kernel**.

**Some regions of the kernel virtual memory** are mapped to **physical pages** that are **shared by all processes**. 例如，每个进程共享内核的代码和全局数据结构。

Linux also **maps a set of contiguous virtual pages** (equal in size to the total amount of DRAM in the system) to the **corresponding set of contiguous physical pages**. 

**Other regions** of kernel virtual memory contain **data that differ for each process**. 

### Linux Virtual Memory Areas
> [进程—内存描述符（mm_struct）](https://blog.csdn.net/qq_26768741/article/details/54375524)

Linux organizes the **virtual memory** as a **collection of areas** (also called **segments**).

An **area** is a **contiguous chunk** of **existing (allocated) virtual memory** whose **pages** are **related in some way**. 例如 代码段，数据段等位不同的 areas。

任何已存在的虚拟页都在某个 area 中。

不在任何一个 area 中的虚拟页则不存在，不能被进程引用，不会消耗内存资源。内核不会跟踪不存在的虚拟页。

Figure 9.27 highlights the **kernel data structures** that **keep track of the virtual memory areas** in a process.
<br/>

![fig 9.27](https://img-blog.csdnimg.cn/33460095b4564757915952e3b34a05f4.png)<br/>

The kernel maintains a **distinct task structure** (task_ struct in the **source code**) for **each process** in the system. 

**task_ struct** 的每个元素要么**包含**要么**指向**内核需要运行进程的所有信息，如 PID，指向用户栈的指针，可执行目标文件的名字，程序计数器等。

**task_ struct** 中的一个条目 **mm** 指向表示虚拟内存的当前状态的一块区域 **mm_struct**。

**mm_struct** 中的 **pgd** 指向 level 1 table (the page global directory），就是第一级页表的起始物理地址，当内核运行该进程时，将 **pgd** 存入 **CR3 控制寄存器**中。

**mm_struct** 中的 **mmap** points to a list of **vm_area_structs** (area structs), each of which characterizes an **area of the current virtual address space**. 

**vm_area_structs** 结构中的内容有：
<br/>
![vm_area_structs](https://img-blog.csdnimg.cn/6c61b50c5fdc400da0a7042e8ae5daa3.png)<br/>

### Linux Page Fault Exception Handling
当 MMU 在翻译虚拟地址时触发**缺页异常**，将导致**控制**转移给**内核**的**缺页处理程序**，然后执行以下步骤：
1. **判断虚拟地址是否合法**
如果虚拟地址在某一个 area 中，即位于某个结构的  **vm_start** 和 **vm_end** 之间（处理程序将比较 **vm_area_structs** 结构中所有的**vm_start** 和 **vm_end** ），则有效。如果**无效**，缺页处理程序将触发一个 **segmentation falut** ，终止进程，见图 9.28 的 1 过程。
Because a **process** can **create an arbitrary number** of **new virtual memory areas** (using the mmap function described in the next section), a sequential search of the **list of area structs** might be very **costly**.  
So in practice, Linux superimposes a tree on the list, using some fields that we have not shown, and performs the search on this tree.

2. **判断内存访问是否合法**
判断进程是否能读，写或者执行该页。
如果不合法，则**缺页处理程序**将触发**保护异常**，**终止进程**，该过程见图 9.28 的  2 过程。

3. It handles the fault by **selecting a victim page**, **swapping out the victim page** if it is **dirty**, **swapping in** the **new page** and **updating the page table**. 

4. 缺页处理程序返回后，CPU 重启之前造成缺页故障的指令，此时能正常命中执行程序。

<br/>

![fig 9.28](https://img-blog.csdnimg.cn/dbae219cdbf5498fb39d47f7882bf47c.png)
# 9.8 Memory Mapping
Linux **initializes** the contents of a **virtual memory area** by associating it with an **object** on **disk**, a **process** known as **memory mapping**. **Areas** can be mapped to **one of two types of objects**:
- **Regular file in the Linux file system**
An **area** can be **mapped** to a **contiguous section of a regular disk file**, such as an **executable object file**. 
当 CPU 第一次 touch 某一页时，虚拟页才会 swap into 物理内存。当需要某一页的内容时才 swap in。
如果 area 比 file section 大，则 area 会将多余的部分用 0 填充。

- **Anonymous file**
An **area** can also be **mapped** to an **anonymous file**, **created** by the **kernel**, that contains **all binary zeros**. 
The **first time** the **CPU** **touches** a **virtual page** in such an **area**, the **kernel** finds an appropriate **victim page** in **physical memory**, **swaps out** the **victim page** if it is **dirty**, **overwrites** the **victim page** with **binary zeros**, and **updates** the **page table** to mark the **page** as **resident**. 
Notice that **no data** are actually **transferred** between **disk and memory**. 
For this reason, **pages in** areas that are mapped to **anonymous files** are sometimes called **demand-zero pages**.

In either case, **once a virtual page is initialized**, it is **swapped back and forth** between a special **swap file** maintained by the **kernel**. 

The **swap file** is also known as the **swap space** or the **swap area**.

At any point in time, the **swap space** bounds the **total amount of virtual pages** that can be **allocated** by the **currently running processes**.

## 9.8.1 Shared Objects Revisited
**Memory mapping** provides us with a clean mechanism for controlling how **objects** are **shared by multiple processes**.

A **virtual memory area** into which a **shared object** is mapped is often called a **shared area**. Similarly for a **private area**.

共享对象：only a **single copy** of the **shared object** needs to be stored in **physical memory**, even though the **object** is mapped into **multiple shared areas**. 

私有对象：**Private objects** are mapped into **virtual memory** using a clever technique known as **copy-on-write**. A private object begins life in exactly the same way as a shared object, **with only one copy of the private object stored in physical memory**.

- For each **process** that maps the **private object**, the **page table entries** for the corresponding **private area** are flagged as **read-only**, and the **area struct** is flagged as **private copy-on-write**. 
- 一旦有一个**进程**试图写**私有区域**的内容，触发**保护故障**，该**异常**会导致处理程序在物理内存中创建一个**新的私有区域副本**，然后**更新 PTE** 来指向该**新的副本**，最后**恢复该页的写许可**。

<br/>

![fig 9.29](https://img-blog.csdnimg.cn/2e25fa40d29e4b2eaa2261f43493e048.png)
<br/>

![fig 9.30](https://img-blog.csdnimg.cn/f570b105569a4b57b7c397888ffead55.png)
## 9.8.2 The fork Function Revisited
**fork** 函数用于创建新进程，如果当前进程调用该函数，内核会为其创建多个数据结构并分配一个唯一的 PID。

新进程的 **mm_struct** 结构，area struct 和页表都和当前进程相同。

每个进程的每页都标记为 **read-only** ，每个进程的 area struct 都设置为 **private copy on-write**，当有**进程写数据**时，和上节中介绍的**私有对象写数据**的过程相同。

## 9.8.3 The execve Function Revisited
Virtual memory and memory mapping also play key roles in the **process of loading programs into memory**. 

假设当前进程中的程序调用 **execve** 函数：
```c
execve("a.out", NULL, NULL);
```
第 8.4.5 节介绍过该函数，在当前进程中加载并运行程序 `a.out` 来代替当前进程的程序，其过程如下：
![steps](https://img-blog.csdnimg.cn/3ec924f8c3e648288e422132fdcf80d0.png)<br/>

![fig 9.31](https://img-blog.csdnimg.cn/83aae80ffb294ab0bc4a78cd6f2a0cfb.png)
## 9.8.4 User-Level Memory Mapping with the mmap Function
Linux 进程能利用 **mmap** 来创建新的虚拟内存区域，并将对象映射到这些区域。

```c
#include <unistd.h>
#include <sys/mman.h>
void *mmap(void *start, size_t length, int prot, 
int flags, int fd, off_t offset);
//Returns: pointer to mapped area if OK, MAP_FAILED (−1) on error
```

- **mmap**  函数要求**内核**创建一个**新的内存区域**，该内存区域最好是**起始地址**为 **start**，并将 **文件描述符（file descriptor）fd** 指定对象的**连续的一片（chunk)** 映射到该区域；该**连续的片**长度为 **length**，**起始地址**到 **fd** 指定对象的开始地址的偏移为 **offset**。

- The **start** address is merely a **hint**, and is usually specified as **NULL**.

- **prot** 参数包含了新的虚拟内存区域的**访问许可的位**(i.e., the **vm_prot** bits in the corresponding area struct)。
<br/>
![prot](https://img-blog.csdnimg.cn/6a663170506347f8b42d359a0398dfcb.png)<br/>

- **flags** 参数包含了描述被映射对象的类型的位。
	- 如果设置 **MAP_ANON** 位，则对象为匿名对象，虚拟页是 **demand-zero**。
	- 如果设置 **MAP_PRIVATE** 位，表示对象为私有对象。
	- 如果设置 **MAP_SHARED** 位，表示对象为共享对象。

Figure 9.32 depicts the meaning of these arguments.
<br/>

![fig 9.32](https://img-blog.csdnimg.cn/44cffc2d60244d1e97b0308ffdcbc3d8.png)
<br/>


示例：
```c
bufp = Mmap(NULL, size, PROT_READ, 
MAP_PRIVATE|MAP_ANON, 0, 0);
```
asks the **kernel** to create a **new read-only, private, demand-zero area of virtual memory** containing **size** bytes. 
If the **call** is **successful**, then **bufp** contains the **address of the new area**.

***********

**munmap** 函数用来**删除虚拟内存区域**：
```c
#include <unistd.h>
#include <sys/mman.h>
int munmap(void *start, size_t length);
//Returns: 0 if OK, −1 on error
```
删除区域的起始地址为 **start**，长度为 **length**。
引用已经删除的区域将导致 **segmentation faluts**。

# 9.9 Dynamic Memory Allocation
动态内存分配器维护进程的虚拟内存中一块称为 **堆（heap）** 的区域，见下图：
<br/>

![fig 9.33](https://img-blog.csdnimg.cn/bf99d88651ff46d6a35574d102b5e112.png)<br/>

Details vary from system to system, but without loss of generality, we will assume that the **heap** is an area of **demand-zero memory** that begins **immediately** after the **uninitialized data area** and **grows upward** (toward higher addresses). 

对于每个进程，内核维护一个变量 **brk** ，该变量指向**堆的顶端**。

**堆**是由一系列**不同大小的块（block）**组成的，每个**块**是**虚拟内存**上一片**连续的片（chunk）**。

这些**块**要么**已分配**（allocated），要么是**空闲状态**（free）。

**分配器**有两种基本的风格，这两种都需要应用程序**显式的分配块**，不同之处在于哪个**实体**（entity）负责**释放已分配的块**：
- **Explicit allocators**
显式的分配器需要应用程序**显式的释放**已分配的块，如 C 程序用 **free** 函数释放 **malloc** 分配的块。

- **Implicit allocators**
**隐式分配器**也叫**垃圾收集器（garbage collectors）**，分配器需要检测一个已被分配的块是否不再使用，然后释放该块。**自动释放未使用的块**的过程角**垃圾收集（garbage collection）**。

## 9.9.1 The malloc and free Functions
> [malloc](https://docs.microsoft.com/zh-cn/previous-versions/visualstudio/visual-studio-2012/6ewkz86d(v=vs.110))


C 标准库提供 **malloc** 作为显式分配器：
```c
#include <stdlib.h>
void *malloc(size_t size);
//Returns: pointer to allocated block if OK, NULL on error
```

被分配的内存大小至少是 size 个字节，可能会因为对齐的需求而分配内存比 size  大。

代码是在 32-bit 模式编译还是 64-bit 模式编译将有不同的对齐要求。

In **32-bit mode**, **malloc** returns a **block** whose **address** is always a **multiple of 8**. 

In **64-bit mode**, the address is always a **multiple of 16**.

如果 **malloc** 遇到问题，如请求的内存块比可用的虚拟内存大，则返回 NULL 并设 **errno.Malloc** 。

**malloc** 不会初始化内存。

**calloc** 函数会将分配的内存初始化为 0.

**realloc** 函数可以**修改**已经分配的块的大小。

**Dynamic memory allocators** such as **malloc** can **allocate** or **deallocate heap** memory **explicitly** by using the **mmap** and **munmap** functions, or they can use the **sbrk** function:
```c
#include <unistd.h>
void *sbrk(intptr_t incr);
//Returns: old brk pointer on success, −1 on error
```

The **sbrk** function **grows** or **shrinks** the **heap** by adding **incr** to the kernel’s **brk** pointer. 

如果成功，返回 **brk** 之前的值，失败则返回 -1 并设置 **errno** 为 **ENOMEM**。

如果 **brk** 为 0，则返回的为当前 **brk** 的值。

如果 **brk** 为 负数，则返回的为当前 **brk** 的值减去 **-incr** 个字节后的值，因此减小堆的尺寸。

**********

**free** 函数用于释放已分配的内存空间：
```c
#include <stdlib.h>
void free(void *ptr);
//Returns: nothing
```
**prt** 必须指向内存块的起始地址，否则失败。

***************
Figure 9.34 shows how an **implementation of malloc and free** might manage a (very) small **heap** of 16 words for a C program.  (这里 1 word 是 4 字节)
<br/>

![fig 9.34](https://img-blog.csdnimg.cn/9ed8d80e856b4b45b311d74cfcfeb8b8.png)<br/>

## 9.9.2 Why Dynamic Memory Allocation?
需要动态内存分配的原因：通常有些数据结构只有在运行时才知道大小。

## 9.9.3 Allocator Requirements and Goals
显式的分配器有严格的限制：
- **Handling arbitrary request sequences**
分配空间和释放块的顺序不一定是一致的，要能处理任意顺序的分配和释放。

- **Making immediate responses to requests**
分配器要能立即响应分配请求。

- **Using only the heap**
In order for the allocator to be scalable, any nonscalar data structures used by the allocator must be stored in the **heap** itself.

- **Aligning blocks (alignment requirement)**
The **allocator** must **align blocks** in such a way that they can **hold any type of data object**.

- **Not modifying allocated blocks**
一旦分配结束，分配器不能操作或移动已分配的块，只能操作空闲的块。

基于上述限制，分配器要实现以下目标：
- **Maximizing throughput**
吞吐率：单位时间完成的请求数。
可以通过**最小化**处理**分配和释放**请求的平均时间来最大化**吞吐率**。
It is not too difficult to develop **allocators** with reasonably good performance where the **worst-case running time** of an **allocate request** is **linear** in the number of **free blocks** and the **running time** of a **free request** is **constant**.

- **Maximizing memory utilization**
虚拟内存的大小并非无限的，系统上所有进程可分配的虚拟内存大小受磁盘上交换空间（swap space）数量的限制。


## 9.9.4 Fragmentation
The primary cause of **poor heap utilization** is a phenomenon known as **fragmentation**, which occurs when otherwise **unused memory** is not available to satisfy **allocate requests**.

- **Internal fragmentation**
分配的块比 payload 大。如由于对齐的需求，分配到块比实际请求大。


- **External fragmentation**
所有空闲内存加起来能满足分配请求，但没有一个单独的块能满足分配请求。

## 9.9.5 Implementation Issues
分配器要在吞吐率和利用率之间保持平衡必须考虑以下因素：
- Free block organization
怎么跟踪空闲块
- Placement
怎么选择合适的空闲块来分配新的块
- Splitting
在空闲块中分配了新的块后怎么处理空闲块剩下的部分
- Coalescing
怎么处理被释放的块

## 9.9.6 Implicit Free Lists
大多数分配器都嵌入一些数据结构来**区分块的边界**，以及**已分配的块**和**空闲块**。
<br/>

![fig 9.35](https://img-blog.csdnimg.cn/73065bee1fb146d18891b246702a578e.png)<br/>

A **block** consists of a **one-word header**, the **payload**, and **possibly** some **additional padding**. 

- **header**
大小：one-word
内容：块的尺寸（包含 header 和 padding），以及块是否已分配。 
如果有 double-word 对齐的限制，则块的尺寸总是 8 的倍数，低 3 位总是 0（8 是 1000，16 是 1 0000，24 是 1 1000，32 是 10 0000）。
因此可以将低 3 位留着存放其他信息，如上图中用**最低位**表示该块是否分配。
- **payload**
请求分配的大小。
- **padding**
padding 可能是任意的大小。请求分配的大小之外**额外的部分**，例如因为字节对齐的需要而多分配的部分。

注意：**malloc 返回的是指向 payload 起始地址的指针，不是指向 header**。

基于上图 9.35 的块格式，将堆组织成如下连续的已分配和空闲块的序列。
<br/>

![fig 9.36](https://img-blog.csdnimg.cn/857f4df926e841d68430286ed8bdbdea.png)<br/>

We call this organization an **implicit free list** because the **free blocks** are **linked implicitly** by the size fields in the headers. 


上图这种组织称为**隐式空闲链表（implicit free list）**，


## 9.9.7 Placing Allocated Blocks
**placement policy**：当应用程序请求分配一块内存空间时，分配器怎么选择合适的块有以下三种策略：
- **First fit**
**从头开始**查询空闲链直到找到**第一个**满足要求的块
优点：趋向将大的块保留在空闲链的后端
缺点：趋向于在空闲链的前端留下小的碎片空闲块
- **Next fit**
从上次查询到的地方开始查询空闲链，直到找到第一个满足要求的块
这种方案时 **first fit** 的替代方案
优点：可能比 **first fit** 快
缺点：内存利用率比 **first fit** 差
- **Best fit** 
检查每个空闲块，找出满足要求的最小的块
优点：内存利用率在三种中最好
缺点：花费时间长

## 9.9.8 Splitting Free Blocks
一旦分配器已经找到一个空闲块来分配内存，还需确定**分配器**使用该**空闲块中多大的部分**来**分配内存**，有两种方案：
- 使用全部的空闲块
优点：简单，操作快
缺点：可能产生内部碎片

- 将空闲块分为两部分，一部分分配内存，另一部分作为**新的空闲块**
 
## 9.9.9 Getting Additional Heap Memory
如果分配器没找到合适大小的空闲块，则：
1. 试图**合并相连的空闲块**成为更大的空闲块；
2. 如果已经合并后仍没有满足要求的空闲块，则分配器通过调用 **sbrk** 函数**向内核请求扩大堆**的空间；
3. 分配器将请求的额外的堆空间作为一个空闲快来分配内存。

## 9.9.10 Coalescing Free Blocks
当分配器释放一个已分配的块后，可能在该块**相邻的地方**有其他的**空闲块**，这些相邻的空闲块可能造成一种 **false fragmentation** 的现象，即有很多小的空闲块但每个小的空闲块都不能用。

因此，需要合并相连的空闲块。

什么时候合并空闲块：
- **immediate coalescing**
一旦有已分配的块被释放则合并；
问题：如果某个块重复的分配然后释放，则不适合采用这种方案；

- **deferred coalescing**
在释放块之后的某个时间合并，如请求分配块失败时，或者扫描整个堆时；

## 9.9.11 Coalescing with Boundary Tags
假设当前释放的空闲块为当前块（current block），则需要合并它前后的块：
- **合并后面的块**
因为当前块的 header 中有本块的大小，因此能找到下一个块的起始位置，然后根据下一个块 header 中包含的该块是否分配的信息可以确定是否合并下一个块。

- **合并前面的块**
根据前面介绍的块的格式**无法得知前面的块的位置**和**是否分配的信息**，除非从头开始扫描一遍堆。
Knuth 提出一种 **boundary tags** 的方法，即在**每个块的末尾**在添加一个 **footer（boundary tags）** ，该部分和 header 的内容相同，这样就能知道前面的块的信息。
**缺点**：浪费内存。
![fig 9. 39](https://img-blog.csdnimg.cn/c32503bbd90e42cf898679b33c8d7c6d.png)**改进方案**：每个块的 header 低位有三位可以使用，目前只用最低位表示当前块是否空闲，可以再用一位来表示前面的块是否空闲；因为只有空闲的块需要合并，因此**只有空闲的块**在末尾添加 ** footer**，如果前面的块是空闲的，则根据前面块的 **footer** 中块的尺寸合并空闲块；这样已分配的块不需要 **footer** 占用额外的内存。

## 9.9.12 Putting It Together: Implementing a Simple Allocator
we will work through the **implementation of a simple allocator** based on an **implicit free list** with **immediate boundary tag coalescing**.
The **maximum block size** is $2^{32}$ = 4 GB. The code is 64-bit clean, running without modification in 32-bit (gcc -m32) or 64-bit (gcc -m64) processes.

### General Allocator Design
隐式空闲链表的格式如下：

<br/>

![fig 9.42](https://img-blog.csdnimg.cn/174eb3777eec4b029636e6347f369acf.png)<br/>

一个小方块是 1 word。
1. 第一个 word 未使用；
2. 第2和第3个 word 为 **prologue block**，只有一个 header 和 一个 footer；该块在**初始化时创建**且**不会被释放**；
3. 接下来是一些 **regular blocks** 是由 **malloc** 分配；
4. 最后一个 word 为 **epilogue block**，只有一个 header。

The **prologue** and **epilogue** blocks are tricks that **eliminate the edge conditions** during **coalescing**. 

Our **allocator** uses a **model of the memory system** provided by the **memlib.c** package shown in Figure 9.41：
![fig 9.41](https://img-blog.csdnimg.cn/f4d99c6fc78d43ddb354cec17ef62aad.png)<br/>

分配器会导出下面三个函数给应用程序：
```c
extern int mm_init(void); // 成功返回 0，失败返回 -1
extern void *mm_malloc (size_t size);
extern void mm_free (void *ptr);
```

### Basic Constants and Macros for Manipulating the Free List
Figure 9.43 shows some basic **constants and macros** that we will use throughout the **allocator code**:
![fig 9.43](https://img-blog.csdnimg.cn/b330a6c6129d4ae083742d04620b071b.png)<br/>

1. word 为 4 字节，double word 是 8 字节。
2. **PACK(size, alloc)** 用于将块的大小和最后一个 allocated bit （表示该块是否分配）组合，用于放到 header 和 footer 中。
3. **GET_SIZE( p )** 得到 p 指向的地址（**header** 或 **footer**）中**块的大小**，因为块的最后三位为 0，做特殊用途，只有前面的位表示大小的值，因此和 **~0x7（最后三位为0，其余全为 1）** 进行**与**运算，得到前面的位。
4. **GET_ALLOC( p )** 则得到 **header** 或 **footer** 中最后一位，即表示该块是否分配的位。
5.  **block pointer (bp)** 是指向 **payload** 的起始地址。
6. **FTRP(bp)** 返回 **footer** 的地址，**bp** 为 **payload** 起始位置的地址，**GET_SIZE(HDRP(bp))** 为块的大小（包含 header 和 footer，以字节为单位），因此 **bp**  + **块的大小** - (**header 加上 footer 的大小, 即 8 字节**) = **footer 的地址**。
7. **NEXT_BLKP(bp)** 为下个块 **payload** 的地址，非 **header** 处的地址；(char *)(bp) - WSIZE 即为 **当前块 header** 的地址，**bp** 的地址 **加上块的大小** 即为下一个块 **payload** 的地址。
8. **NEXT_BLKP(bp)** 为前一个块 **payload** 的地址，非 **header** 处的地址；(char *)(bp) + DSIZE 即为 **前一个块 footer** 的地址，**bp** 的地址 **减去前一个块的大小** 即为前一个块 **payload** 的地址。

已知当前块的指针 bp，计算下个块的大小：
```c
size_t size = GET_SIZE(HDRP(NEXT_BLKP(bp)));
```

### Creating the Initial Free List
Before calling **mm_malloc** or **mm_free**, the application must **initialize** the **heap** by calling the **mm_init** function (Figure 9.44).
![fig 9.44](https://img-blog.csdnimg.cn/ed565a8829cf44bbb525464b961c6c4c.png)
<br/>

1. **m_init** 函数分配 4 word 的大小，第一个 word 初始化为 0；
2. 第二个 word 用于 **Prologue block** 的 **header**，大小为 8 字节，最后一位为 1 表示已分配；
3. 第三个 word 用于 **Prologue block** 的 **footer**，大小为 8 字节，最后一位为 1 表示已分配；
4. 第四个 word 用于 **Epilogue block** 的 **header**，大小为 0 字节，最后一位为 1 表示已分配；

然后调用 **extend_heap** 函数来扩展空的堆，该函数如下：
![fig 9.45](https://img-blog.csdnimg.cn/51dc49fa71644e3b8ff9595d6f47dde7.png)
<br/>

**extend_heap** 函数在以下两种情况下调用：
- 初始化堆
- **mm_malloc** 分配内存时不能找到合适的块

1. 分配的内存 8 字节（2 words）对齐，如请求的大小为 3 word，则会分配 4 words，即 16 字节。
2. 将分配的块的 header 和 footer 初始化，最后一位为 0。
3. 在末尾加上一个 **epilogue block**。
4. 如果前面有空闲的块，则合并后返回空闲块的起始地址。

注意： bp 为当前块 payload 的起始地址，非 header 的地址。
### Freeing and Coalescing Blocks
An application **frees a previously allocated block** by calling the **mm_free** function (Figure 9.46):
![fig 9.46](https://img-blog.csdnimg.cn/01a2563cf60f4363bf9b7d01ef423ef7.png)
<br/>

注意： bp 为当前块 payload 的起始地址，非 header 的地址。
合并块时由四种情况：
- 前一个块和后一个块均已分配
- 前一个块已分配，后一个块为空闲状态
- 前一个块空闲，后一个块已分配
- 前后的块均空闲

### Allocating Blocks
An application **requests a block of size bytes of memory** by calling the **mm_malloc** function (Figure 9.47). 
![fig 9.47](https://img-blog.csdnimg.cn/64ecc2521bf840808cfb64dfa72c9910.png)
<br/>

- 函数的参数 **size** 只请求的内存大小，单位为字节，因此是 payload 的大小，实际分配时要加上 **header 和 footer**，同时考虑 8 字节对齐的需求。
- 如果请求的大小不超过 8 字节，因为 header 和 footer 总共 2 words，即 8 字节，因此分配的内存大小为 16 字节。
- 如果请求的大小超过 8 字节，则基于对齐的要求，分配最小满足对齐的尺寸；如请求大小为 9，则需要 9 + 8 = 17 字节，基于对齐要求，分配 24 字节。

其他接口实现代码参考：[CSAPP-MallocLab](https://blog.csdn.net/qq_45531291/article/details/122629911) 

## 9.9.13 Explicit Free Lists
> [内存管理：显式空闲链表](https://zhuanlan.zhihu.com/p/378352199)

隐式空闲链表缺点：block allocation time is linear in the total number of heap blocks. 因为分配内存时要找到合适的空闲块。

显式空闲链表：
![fig 9.48](https://img-blog.csdnimg.cn/1d9d205fccdf479c9fe013e38e091437.png)
<br/>

显式空闲链表的空闲块中增加了两个指针，分别指向前一个和后一个空闲块的地址。
这样用 **first-fit** 方案查找空闲块时不需要查找已分配的块，通过双链表的形式将空闲块连在一起。
使用 **first-fit** 时查找空闲块的时间只与堆中全部的空闲块数目线性相关。

However, the **time to free a block** can be either **linear** or **constant**, depending on the **policy** we choose for **ordering the blocks in the free list**.
- **last-in first-out (LIFO)**
最新释放的空闲块放在链表的开始处，这样查找空闲块时从上次释放的块查找。
In this case, **freeing a block** can be performed in **constant time**.
If boundary tags are used, then **coalescing** can also be performed in **constant time**.

- **maintain the list in address order**
链表中块的顺序和地址的顺序一致，每个块的地址小于它后面的一个空闲块。
In this case, freeing a block requires a **linear-time search** to **locate the appropriate predecesso**r. 
The **trade-off** is that **address-ordered first fit** enjoys **better memory utilization** than **LIFO-ordered first fit**, approaching the **utilization of best fit**.

## 9.9.14 Segregated Free Lists
单个链表，分配内存时查找空闲块的时间和空闲块总数成线性关系，可以通过使用多个链表来减小分配时间。

**Segregated Free Lists**：维护多个空闲链表，让每个链表中每个块的大小基本相同。每个空闲链有自己的一个 size，即链表中块的大小，这些 size 组成一个数组，数组的元素递增。当需要分配一个大小为 n 的块时，就找到适合的链表。

# 9.10 Garbage Collection
 **garbage collector**：垃圾回收器是一种动态存储分配器，用来**自动释放不再被程序使用的块**。

## 9.10.1 Garbage Collector Basics
A **garbage collector** views memory as a directed **reachability graph** of the form shown in Figure 9.49.
<br/>

![fig 9.49](https://img-blog.csdnimg.cn/08a3dbff8b8b466683e9db2d29ece774.png)
<br/>

- 图中每个节点是一个块。
- **root note**: correspond to locations **not in the heap** that **contain pointers into the heap**.
- A directed edge p → q means that some location in block p points to some location in block q. 
- These **locations** can be **registers**, **variables** on the **stack**, or **global variables** in the read/write data area of virtual memory.
- We say that a node p is **reachable** if there exists a **directed path** from any **root node to p**. 
- At any point in time, the **unreachable nodes** correspond to garbage that can **never be used again** by the application.
- 垃圾回收器回收图中的垃圾块。

## 9.10.2 Mark&Sweep Garbage Collectors
**Mark**：利用块 header 的最低三位中的一位作为 mark bit（如果是8字节对齐），用来标记该块是否是 reachable。

**Sweep**：扫描所有的块，根据 mark bit 释放不是 reachable 的块。
<br/>
![fig 9.51](https://img-blog.csdnimg.cn/008fe4e7bef5422b87ef19e60923e690.png)<br/>

- **ptr** is defined as typedef **void *ptr**.
- **ptr isPtr(ptr p)**：如果 p 指向某个已分配的块中的某个数据，则返回一个指向该块起始地址的指针，否则返回 NULL。
- **int length(ptr b)**：返回 b 指向的块的长度，包含 header，以 word 为单位。

**mark 函数**：对每个 root note 都调用 mark 函数，如果 p 不是指向一个**已分配**但**未标记的块**则返回，否则标记该块，然后以 word 为单位对块中每个子块递归调用 mark 函数，最后将该 root note 下的所有已分配的块全部标记。

**sweep 函数**：释放**未标记但已分配**的块。

# 9.11 Common Memory-Related Bugs in C Programs
## 9.11.1 Dereferencing Bad Pointers
引用一个错误的地址，如：
```c
scanf("%d", val);
//应该是 scanf("%d", &val)
```

## 9.11.2 Reading Uninitialized Memory
While **bss** memory locations (such as uninitialized global C variables) are always **initialized to zeros by the loader**, this is **not true** for **heap** memory. 

![9.11.2](https://img-blog.csdnimg.cn/fdc95cf9c6d54e4c859fe86af08b85ac.png)<br/>

## 9.11.3 Allowing Stack Buffer Overflows
![9.11.3](https://img-blog.csdnimg.cn/90449f11b9e14df49fa37b01c74b7f0a.png)
## 9.11.4 Assuming That Pointers and the Objects They Point to Are the Same Size
![fig 9.11.4](https://img-blog.csdnimg.cn/0ff853cced384d47a98d0bd37ce08791.png)
<br/>

第 5 行应该是  `sizeof(int *)` 而不是 `sizeof(int)`。

## 9.11.5 Making Off-by-One Errors

![9.11.5](https://img-blog.csdnimg.cn/7101fbaae705493ea70130f86f363afe.png)
第 7 行 循环执行 n+1 次，而非 n 次。

## 9.11.6 Referencing a Pointer Instead of the Object It Points To
![9.11.6](https://img-blog.csdnimg.cn/336bbdd9739f4c0e98fad5ff325d2040.png)
第 6 行 `*` 和 `--` 相同的优先级，但结核性是从右到左，因此先执行 `--`。

## 9.11.7 Misunderstanding Pointer Arithmetic
![9..11.7](https://img-blog.csdnimg.cn/6a40cc966abb4550b47ac3d82d219029.png)
## 9.11.8 Referencing Nonexistent Variables
![9.11.8](https://img-blog.csdnimg.cn/8a06deafab024cd0911acfc72e9aa90b.png)<br/>

This function returns a pointer (say, p) to a **local variable** on the **stack** and then **pops its stack frame**. 

Although p still points to a valid memory address, it no longer points to a valid variable. 

## 9.11.9 Referencing Data in Free Heap Blocks
![9.11.9](https://img-blog.csdnimg.cn/c1b09e62962a417eaa607feb4278b654.png)

14 行中 x 已被释放，不能引用其内容。

##  9.11.10 Introducing Memory Leaks

![9.11.10](https://img-blog.csdnimg.cn/31719b1ab64d4f22bbcf8a6695be1f2a.png)

没有释放分配的内存。








