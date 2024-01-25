深入理解计算机系统——第十章 System Level IO  
  
> [视频课程](https://www.bilibili.com/video/BV1iW411d7hd?p=16)  
> [视频课件](http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/16-io.pdf)  
> [General overview of the Linux file system](https://tldp.org/LDP/intro-linux/html/sect_03_01.html)  
> [What is the difference between path and directory?](https://unix.stackexchange.com/questions/131561/what-is-the-difference-between-path-and-directory)  
  
输入输出（Input/output）是在**主存**和**外部设备**之间复制数据的过程。  
  
  
# 10.1 Unix I/O  
- A linux file is a sequence of $m$ bytes: $B_{0}, B_{1}, ... , B_{m-1}$.  
  
- **All I/O devices**, such as networks, disks, and terminals, are **modeled as files**, and all **input and output** is performed by **reading and writing the appropriate files**.   
	- /dev/sda2  &emsp;  &emsp;   (usr disk partion)  
	- /dev/tty2       &emsp;   &emsp;     (terminal)  
- Even the **kernel** is reprensented as a **file**.  
	- /boot/vmlinuz-3.13.0-55-generic &emsp;  &emsp;  (kernel image)  
	- /proc &emsp;   &emsp;  &emsp;  &emsp;  &emsp;  &emsp;  &emsp;  &emsp;  &emsp;  &emsp;  &emsp;    (kernel data structure)  
	  
This elegant **mapping** of **devices to files** allows the **Linux kernel** to export a **simple, lowlevel application interface**, known as **Unix I/O**, that enables all **input and output** to be performed in a **uniform and consistent way**:  
  
- **Opening files**  
**应用程序**通过要求**内核**打开相应的**文件**来访问一个 I/O 设备。  
**内核**返回一个小的**非负整数**，即**描述符（descriptor）**，用来标志该文件。  
**内核**跟踪打开的文件的**全部信息**。  
**应用程序**只跟踪**描述符**。  
Linux shell 创建的每个进程创建完就有三个打开的文件：standard input (descriptor 0)，standard output (descriptor 1) 和 standard error (descriptor 2)。  
The header file **<unistd.h>** defines **constants STDIN_ FILENO, STDOUT_FILENO, and STDERR_FILENO**, which can be used **instead of the explicit descriptor values**.  
  
- **Changing the current file position**  
对于每个打开的文件，内核维护一个 file position $k$，该值初始为 0。  
The **file position** is a **byte offset** from the **beginning** of a **file**.   
应用程序能通过 **lseek** 函数来设置当前文件的 file position $k$。  
  
- **Reading and writing files**  
**读文件**：从文件中复制 n （n > 0）个字节到内存，起始位置为 k。如果一个文件有 m 字节，而 k > m，则触发 end-of-file (EOF) ，应用程序能检测到这种情况。  
**写文件**：从内存复制 n （n > 0）个字节到文件，起始位置为 k，复制完后更新 k。  
  
- **Closing files**  
当应用程序**访问完文件**后，通知**内核**来**关闭文件**，此时**内核释放文件打开时创建的数据结构**然后将该**文件的描述符恢复到可用描述符的池**中。  
无论**进程**因为什么原因**终止**，**内核**都会**关闭进程打开的全部的文件**并**释放内存资源**。  
  
# 10.2 Files  
每个 **Linux 文件**都有一个**类型**来表明它在系统中的角色：  
- **regular file**  
普通文件包含**任意数据**。应用程序将普通文件分为以下两类：  
1）**text file**  
仅包含 ASCII 和 Unicode 字符。  
Linux text 文件由一系列的 text lines 组成，每行是一系列以 newline character ('\n') 结尾的字符。   
newline character 和 ASCII 的 feed character (LF) 相同，数值为 0x0a。  
2）**binary file**  
其他文件，如 JPEG image，object file 等。  
**内核不区分这两种文件**。  
  
- **directory**  
A **directory** is a **file** consisting of **an array of links**, where each **link** maps a **filename** to a **file**, which may be **another directory**.  
Each **directory** contains at least **two entries**:   
1）**. (dot)** is a **link** to the directory **itself**.  
2）**.\. (dot-dot)** is a **link** to the **parent directory** in the **directory hierarchy** .  
能通过 **mkdir** 命令创建目录，用 **ls** 命令查看目录的内容，用 **rmdir** 命令删除目录。  
  
- **socket**  
A **socket** is a **file** that is used to **communicate with another process** across a **network**.  
  
- **其他文件**  
Other file types include **named pipes**, **symbolic links**, and **character** and **block devices**.  
  
The **Linux kernel** organizes all **files** in a **single directory hierarchy** anchored by the **root** directory named **/ (slash)**.   
  
Each **file** in the system is a **direct** or **indirect** **descendant** of the **root** directory.   
  
Figure 10.1 shows a portion of the **directory hierarchy** on our Linux system.  
<br/>  
![fig 10.1](https://img-blog.csdnimg.cn/36b3ddba076d4ce69861281bcef1eb55.png)  
<br/>  
  
As part of its **context**, each **process** has a **current working directory** that identifies its **current location** in the directory hierarchy.   
  
可以用 **cd** 命令修改当前工作目录。  
  
**Locations** in the directory hierarchy are specified by **pathnames**.  
  
A **pathname** is a **string** consisting of an **optional slash** followed by **a sequence of filenames separated by slashes**.   
  
**Pathnames have two forms**:  
- **absolute pathname**  
An **absolute pathname** **starts** with a **slash** and denotes a path from the **root node**.  
例如 `hello.c` 的绝对路径为 `/home/droh/hello.c`  
- **relative pathname**  
A **relative pathname** **starts** with a **filename** and denotes a path from the **current working directory**.  
例如 `/home/bryant` 是当前的工作目录，则  `hello.c` 的相对路径为 `../droh/hello.c`。  
  
[What is the difference between path and directory?](https://unix.stackexchange.com/questions/131561/what-is-the-difference-between-path-and-directory)  
  
# 10.3 Opening and Closing Files  
进程用 **open** 函数打开**已存在的文件**或**创建新文件**：  
```c  
#include <sys/types.h>  
#include <sys/stat.h>  
#include <fcntl.h>  
int open(char *filename, int flags, mode_t mode);  
//Returns: new file descriptor if OK, −1 on error  
```  
  
- **filename**  
The **open** function **converts** a **filename** to a **file descriptor** and **returns** the **descriptor number**.  
  
- **flags**  
**flags** 参数用来表示进程打算**怎么访问文件**：  
![flags](https://img-blog.csdnimg.cn/478252c87e1a46e18b5a6baaa8d027e2.png)  
对于**写**操作，**flags** 参数也能通过**或**运算符组合下列指令：  
![additional instructions for writing](https://img-blog.csdnimg.cn/24319083eef6434abdfee7d2dd99cbdc.png)- **mode**  
**mode** 参数用来声明对**新文件的访问许可**：  
![fig 10.2](https://img-blog.csdnimg.cn/4322672517074e0d805b1b1994133af7.png)  
  
As part of its **context**, each **process** has a **umask** that is set by calling the **umask** function.   
  
进程通过**open**创建一个新文件时，如果设置了**mode**参数，则文件的**访问许可位**为**mode & ~umask**。  
  
例如：  
```  
#define DEF_MODE S_IRUSR|S_IWUSR|S_IRGRP|S_IWGRP|S_IROTH|S_IWOTH  
#define DEF_UMASK S_IWGRP|S_IWOTH  
  
umask(DEF_UMASK);  
fd = Open("foo.txt", O_CREAT|O_TRUNC|O_WRONLY, DEF_MODE);  
```  
`mode` 设置权限是文件的所有者，所有者所在的组和其他用户均能读写文件；  
`umask` 设置的权限为所有者所在的组和其他用户写文件权限；  
`mode & ~umask` 为最终的权限，即所有者能读写文件，所有者所在的组和其他用户只能读文件。  
  
*****************  
进程通过**close**函数关闭一个打开的文件：  
```c  
#include <unistd.h>  
int close(int fd);  
//Returns: 0 if OK, −1 on error  
```  
Closing a **descriptor** that is **already closed** is an **error**.  
   
# 10.4 Reading and Writing Files  
Applications perform **input** and **output** by calling the **read** and **write** functions, respectively.  
```c  
#include <unistd.h>  
ssize_t read(int fd, void *buf, size_t n);  
//Returns: number of bytes read if OK, 0 on EOF, −1 on error  
  
ssize_t write(int fd, const void *buf, size_t n);  
//Returns: number of bytes written if OK, −1 on error  
```  
- **fd**  
文件描述符  
- **buf**  
内存位置  
- **n**  
读或写的最大字节数  
  
**read**：从 **fd** 指定的文件的 current file position 处读最多 n 个字节到内存位置 **buf** 处。  
**write**：从内存位置 **buf** 处复制最多 n 个字节到 **fd** 指定的文件的 current file position 处。  
  
应用程序可以通过调用 **lseek** 函数来修改 current file position。  
  
以下情况可能造成**读或者写的字节数小于 n**：  
- **Encountering EOF on reads**  
已经读到文件末尾，可读的字节数少于请求的字节数。  
- **Reading text lines from a terminal**  
可能每次只读一行，遇到换行符停止读。  
- **Reading and writing network sockets**  
11章介绍  
  
# 10.5 Robust Reading and Writing with the Rio Package  
The **Rio (Robust I/O) package** provides convenient, robust, and efficient I/O in applications such as network programs that are subject to **short counts**.  
  
![Rio](https://img-blog.csdnimg.cn/7b9dfd767df7429bbfc6aa67ceb54c87.png)<br/>  
  
##  10.5.1 Rio Unbuffered Input and Output Functions  
Applications can **transfer data directly** between **memory and a file** by calling the **rio_readn** and **rio_writen** functions.  
```c  
#include "csapp.h"  
ssize_t rio_readn(int fd, void *usrbuf, size_t n);  
ssize_t rio_writen(int fd, void *usrbuf, size_t n);  
//Returns: number of bytes transferred if OK, 0 on EOF (rio_readn only), −1 on error  
```  
参数的含义和 **read** 和 **write** 相同，但 **rio_readn** 只有读到**文件末尾**才可能出现读的字节数小于 n，而 **rio_writen** 不会出现写的字节数小于 n 的情况。  
  
Calls to **rio_readn** and **rio_writen** can be **interleaved arbitrarily** on the **same descriptor**.  
  
Figure 10.4 shows the code for **rio_readn** and **rio_writen**.   
Notice that each function **manually restarts the read or write function** if it is **interrupted** by the **return** from an **application signal handler**.   
<br/>  
  
![rio_readn](https://img-blog.csdnimg.cn/cf76e0268d9348a6942bef4448e7d11b.png)![rio_writen](https://img-blog.csdnimg.cn/fed0c52d4dfd44d4a6f7b1a396b00bb3.png)<br/>  
  
读文件时可能遇到已经读到文件末尾的情况，因此实际读到的数据大小比 `n` 小，即 `nleft` 仍大于0。  
写文件时，写的数据大小一定是 `n`，因此 `nleft` 大于 0，但 `write` 返回值为 0，说明出现错误。  
  
## 10.5.2 Rio Buffered Input Functions  
```c  
#include "csapp.h"  
void rio_readinitb(rio_t *rp, int fd);  
//Returns: nothing  
  
ssize_t rio_readlineb(rio_t *rp, void *usrbuf, size_t maxlen);  
ssize_t rio_readnb(rio_t *rp, void *usrbuf, size_t n);  
//Returns: number of bytes read if OK, 0 on EOF, −1 on error  
```  
- **rio_readinitb**  
The **rio_readinitb** function is called **once per open descriptor**.   
It **associates** the **descriptor fd** with a **read buffer of type rio_t** at address **rp**.  
- **rio_readlineb**  
The **rio_readlineb** function **reads** the **next text line** from **file rp** (**including** the terminating **newline** character), copies it to **memory** location **usrbuf**, and **terminates** the **text line** with the **NULL** (zero) character.  
The **rio_readlineb** function **reads at most maxlen-1 bytes**, leaving room for the terminating **NULL** character.   
Text lines that **exceed maxlen-1** bytes are **truncated** and **terminated** with a **NULL** character.  
- **rio_readnb**  
The **rio_readnb** function **reads up to n bytes** from **file rp** to **memory location usrbuf**.   
Calls to **rio_readlineb** and **rio_readnb** can be **interleaved** arbitrarily on the **same descriptor**.  
  
例子：  
![fig 10.5](https://img-blog.csdnimg.cn/31d2923220124077a313d4fea925de11.png)  
  
<br/>  
  
  
************************  
1. `Rio_readinitb(&rio, STDIN_FILENO);` 为初始化过程，只执行一次；  
2. 将文件描述符 `fd` 赋值给 `rp-rio_fd`；  
3. `rio_cnt` 初始化为 0，表示缓冲区中未读的字节数为 0，因此需要填充缓冲区；  
4. `rio_bufptr` 指向未读的缓冲区的起始位置，此时初始化为 `rio_buf`，即缓冲区的起始位置。  
  
![1](https://img-blog.csdnimg.cn/69babdfefd7d48f6808b471ec863b429.png)  
<br/>  
  
![fig 10.6](https://img-blog.csdnimg.cn/708a42646303456f96efe2fd728d658a.png)  
  
<br/>  
  
****************************************  
1. `rio_read` 函数是`read` 函数的带缓冲版本，读 `n` 字节的数据到 `usrbuf` ；  
2. 初始时 `rp` 中 `rio_cnt` 为 0，因此第 5 行中进入 `while` 循环；  
3. 第 7 行和第 8 行，向  `rio_buf` 缓冲区中读数据，从 `rio_fd` 文件的当前文件位置处读数据到 `rio_buf` 中，读取数据为 `rio_buf` 的大小；读的数据大小赋值给 `rio_cnt`，这里从文件读数据到缓冲区，但该数据并未被应用程序读取使用，只是读到内部缓冲区，因此缓冲区中未读取的数据大小为 `rio_cnt`；  
4. 如果 `rio_cnt` 大于 0，即读到数据到缓冲区，则更新缓冲区中未被应用程序读取的起始位置为缓冲区的起始位置；  
5. 如果缓冲区中还未被应用程序读取的数据小于 `n`，则只赋值缓冲区中未被读取的数据到 `usrbuf`；  
6. 第 23 和 24 行更新缓冲区中未被读取的数据起始位置和大小；  
7. 返回读取到 `usrbuf` 中数据大小。  
  
![fig 10.7](https://img-blog.csdnimg.cn/053fbf9285ff485eab13373f2bc712b3.png)  
  
****************************  
  
1. `Rio_readlineb(&rio, buf, MAXLINE)` 读数据到 `buf` ，读取的最大字节数为 `MAXLINE-1`，最后一个为空字符；  
2. `rio_read(rp, &c, 1)` 读取一个字节的数据，如果 `rio_buf` 缓冲区中有未被读取的数据，则直接从缓冲区读数据；  
3. 如果读取成功，则将读到的字符存赋值给 `*bufp`，`bufp` 指针向后加 1；（[Do you know what *p++ does in C?](https://denniskubes.com/2012/08/14/do-you-know-what-p-does-in-c/)）   
4. 如果读到换行符，则退出循环，停止读数据；  
5. 读完后将最后一个字符设置为空字符，返回读到 `usrbuf` 缓冲区的字符（包括结尾空字符）。   
  
![fig 10.8](https://img-blog.csdnimg.cn/649fbc2dc168404d87f42e0b04e1f779.png)  
  
<br/>  
  
****************************************************  
`rio_readnb` 是 `rio_readn` 的带缓冲的版本：  
  
![fig 10.8](https://img-blog.csdnimg.cn/083a3dcf0d684b1c931680bd3fe8eaac.png)  
  
<br/>  
  
# 10.6 Reading File Metadata  
An application can **retrieve** information about a **file** (sometimes called the file’s **metadata**) by calling the **stat** and **fstat** functions.  
```c  
#include <unistd.h>  
#include <sys/stat.h>  
int stat(const char *filename, struct stat *buf);  
int fstat(int fd, struct stat *buf);  
//Returns: 0 if OK, −1 on error  
```  
  
The **stat** function takes as **input** a **filename** and **fills** in the **members** of a **stat structure** shown in Figure 10.9.   
  
![stat](https://img-blog.csdnimg.cn/d6513f7679374613a5173d2b137bddad.png)<br/>  
  
The **fstat** function is similar, but it takes a **file descriptor** instead of a **filename**.   
  
The **st_mode** member encodes both the **file permission bits** (Figure 10.2) and the **file type** (Section 10.2).  
Linux defines **macro** predicates in **sys/stat.h** for determining the **file type** from the **st_mode** member:  
![file type](https://img-blog.csdnimg.cn/ca34247d79ba450fb64907109e8b1945.png)  
示例：  
![fig 10.10](https://img-blog.csdnimg.cn/43f84c8ae2774f4c893b3a16954abad8.png)<br/>  
  
# 10.7 Reading Directory Contents  
Applications can **read the contents of a directory** with the **readdir family** of functions.  
```c  
#include <sys/types.h>  
#include <dirent.h>  
DIR *opendir(const char *name);  
//Returns: pointer to handle if OK, NULL on error  
```  
The **opendir** function takes a **pathname** and **returns** a **pointer** to a **directory stream**.  
A **stream** is an **abstraction** for an **ordered list of items**, in this case **a list of directory entries**.  
  
*****************  
  
```c  
#include <dirent.h>  
struct dirent *readdir(DIR *dirp);  
//Returns: pointer to next directory entry if OK, NULL if no more entries or error  
```  
Each call to **readdir** returns a **pointer** to the **next directory entry** in the **stream** dirp, or **NULL** if there are no more entries.   
Each directory entry is a **structure** of the form:  
```c  
struct dirent {  
ino_t d_ino; /* inode number, file location */  
char d_name[256]; /* Filename */  
};  
```  
On **error**, **readdir** **returns** **NULL** and sets **errno**.  
  
The **only way** to **distinguish** an **error** from the **end-of-stream condition** is to **check** if **errno** has been **modified** since the call to **readdir**.  
*******************  
  
```c  
#include <dirent.h>  
int closedir(DIR *dirp);  
//Returns: 0 on success, −1 on error  
```  
The **closedir** function **closes** the **stream** and **frees up any of its resources**.  
![fig 10.11](https://img-blog.csdnimg.cn/ebcf389c2b874e70b16a0f7314f91c64.png)<br/>  
  
# 10.8 Sharing Files  
The **kernel** represents **open files** using three related data structures:  
- **Descriptor table**  
每个进程都有一个独立的描述符表（descriptor table），该表条目的索引是该进程打开文件的描述符。  
每个条目指向 file table 的一个条目（entry）。  
  
- **File table**  
The **set** of **open files** is represented by a **file table** that is **shared by all processes**.  
Each **file table entry** consists of (for our purposes) the **current file position**, a **reference count** of the **number of descriptor entries** that **currently point to it**, and a **pointer** to an **entry in the v-node table**.  
当关闭一个描述符时，对应的文件表条目中的引用计数减一。  
只有当引用计数减为零时内核才会删除文件表条目。  
  
- **v-node table**  
**v-node table** 也是被所有进程共享的。  
每个条目包含 **stat** 结构体的大多数信息，其中包括 **st_mode** 和 **st_size** 两个成员。  
  
Figure 10.12 shows an example where **descriptors 1 and 4** reference **two different files** through **distinct open file table entries**.  
  
![fig 10.12](https://img-blog.csdnimg.cn/adfb40503adb414aadecb0206a4f3345.png)  
<br/>  
  
图 10.12 中两个文件描述符 **fd1** 和 **fd4** 引用两个不同的文件，对应文件表条目 A 和 B，这两个文件表条目最终指向不同的 **v-node table entry**。  
  
***********************  
  
**Multiple descriptors** can also **reference** the **same file** through **different file table entries**, as shown in Figure 10.13.  
![fig 10.13](https://img-blog.csdnimg.cn/9bd977f3963b48158040b14bb616852b.png)  
<br/>  
  
上图描述符1 和 4 最后的 **v-node table entry** 相同，表示他们引用的文件相同。  
注意两个描述符有不同的文件表条目，如果每个文件表条目中文件位置不同，则可以读文件中的不同位置数据。  
这种情况**可能**产生的原因有：对相同的文件调用两次 **open** 函数。  
  
**************************  
  
假设一个父进程初始是图 10.12 的情况，则其调用 **fork** 函数创建子进程后将变成下图 10.14 的情形：  
<br/>  
![fig 10.14](https://img-blog.csdnimg.cn/01343485465447329b07235c07028354.png)<br/>  
  
前面说过子进程会复制父进程的描述表，因此指向的文件表条目相同，文件表条目中的引用计数加一。  
父进程和子进程都要关闭描述符，才能让文件表的引用计数减为零，内核才会删除对应的文件表条目。  
  
# 10.9 I/O Redirection  
**Linux shells** provide **I/O redirection operators** that allow users to **associate standard input and output** with **disk files**.  
  
如在 Linux 终端输入如下命令：  
```shell  
ls > foo.txt  
```  
则会执行 **ls** 命令，将结果输出到 **foo.txt** 文件中。  
  
可以通过 **dup2** 函数来实现重定向操作：  
```c  
#include <unistd.h>  
int dup2(int oldfd, int newfd);  
//Returns: nonnegative descriptor if OK, −1 on error  
```  
**dup2** 函数将描述符表条目 **oldfd** 复制到描述符表条目 **newfd** ，**newfd** 之前的内容会被覆盖。  
如果 **newfd** 对应的文件已经打开，则会先关闭文件再复制。  
  
假设初始是图 10.12 的情形，在调用 **dup2(4,1)** 后为下图 10.15 所示：  
![fig 10.15](https://img-blog.csdnimg.cn/114523b3286e44ddb2eabcd2fdbf25ee.png)  
<br/>  
  
描述符1 从指向 File A 改为指向 File B，File A 中引用计数为 0，File B 中的引用计数增加到 2。  
File A 已经被关闭，其对应的 **v-node table** 条目被删除。  
  
# 10.10 Standard I/O  
The C language defines **a set of higher-level input and output functions**, called the **standard I/O library**, that provides programmers with a **higher-level alternative** to **Unix I/O**.  
  
The library (**libc**) provides functions for **opening and closing files** (**fopen** and **fclose**), **reading** and **writing** bytes (**fread** and **fwrite**), **reading** and **writing** **strings** (**fgets** and **fputs**), and **sophisticated formatted I/O (scanf and printf)**.  
  
The **standard I/O library** models an **open file** as a **stream**.   
  
To the **programmer**, a **stream** is a **pointer to a structure of type FILE**.   
  
Every **ANSI C program begins with three open streams**, **stdin, stdout, and stderr**, which correspond to **standard input, standard output, and standard error**, respectively:  
  
```c  
#include <stdio.h>  
extern FILE *stdin; /* Standard input (descriptor 0) */  
extern FILE *stdout; /* Standard output (descriptor 1) */  
extern FILE *stderr; /* Standard error (descriptor 2) */  
```  
  
A **stream** of type **FILE** is an **abstraction** for a **file descriptor** and a **stream buffer**.  
  
The **purpose** of the **stream buffer** is the **same** as the **Rio read buffer**: to **minimize** the **number of expensive Linux I/O system calls**.  
  
# 10.11 Putting It Together: Which I/O Functions Should I Use?  
Figure 10.16 summarizes the **various I/O packages** that we have discussed in this chapter.  
  
![fig 10.16](https://img-blog.csdnimg.cn/ef740c5c1eb749b28b7531e0872452a9.png)<br/>  
  
原则：  
- **Use the standard I/O functions whenever possible**  
- **Don’t use scanf or rio_readlineb to read binary files**  
Functions like **scanf** and **rio_readlineb** are designed specifically for reading **text** files.  
For example, **binary files** might be littered with many **0xa** bytes that have **nothing** to do with **terminating text lines**.  
- **Use the Rio functions for I/O on network sockets**  
  
**Standard I/O streams** are **full duplex** in the sense that programs can perform **input and output** on the **same stream**.   
  
However, there are poorly documented restrictions on streams that interact badly with **restrictions on sockets**:  
  
![restrictions ](https://img-blog.csdnimg.cn/6b2bd2a9ad624a1d97e69f6606707239.png)<br/>  
  
![note](https://img-blog.csdnimg.cn/8cc103b4fe33468fb321c9087ff3b314.png)  
