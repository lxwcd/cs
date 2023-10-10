深入理解计算机系统——第十二章 Concurrent Programming


资源：
> [视频课程](https://www.bilibili.com/video/BV1iW411d7hd?p=23)

# 12.1 Concurrent Programming with Processes
The simplest way to build a concurrent program is with processes, using familiar functions such as fork, exec, and waitpid. 

例如搭建一个并发的服务器提供服务，当接收一个客户端的连接请求后，创建一个子进程来处理该客户端的连接，然后继续监听其他客户端的连接请求，如果后续又有客户端建立连接，再创建一个新的子进程来处理新客户端的连接

![](img/2023-10-10-20-51-12.png)


# 12.2 Concurrent Programming with I/O Multiplexing
假如服务器要处理两个独立的 I/O 事件：客户端连接请求和用户键盘输入命令，则可以通过 I/O 复用来实现

I/O复用（I/O Multiplexing）是一种机制，允许一个进程或线程同时监控多个I/O操作，而无需阻塞或轮询每个I/O操作

例如通过 select 函数来实现 I/O 复用：

![](img/2023-10-10-21-12-56.png)

select 函数的第二个输入参数中 fd_set 是描述符集合（descriptor set），类似一个 bit vector of size n：
$b_{n-1},...,b_{1},b_{0}$
如果描述符 k 是该描述符集合的成员，则 $b_{k}$ 为 1
通过 `the FD_ZERO, FD_SET, FD_CLR, and FD_ISSET` 这些宏来设置这种类型变量的值或做其他操作
该集合被称为 read set，即读取集合

select 函数的第一个参数 n 为描述符集合的基数

select 函数使用读取集合来指定要监视的描述符，并阻塞等待至少一个描述符准备好进行读取操作。
然后 select 函数会修改读取集合，将准备好进行读取操作的描述符放入准备集合（ready set），并返回准备集合的基数。
为了正确使用 select 函数，每次调用前都需要更新读取集合，以确保包含需要监视的描述符。

![](img/2023-10-10-21-33-45.png)

见上面示例，将连接客户端的描述符和接收标准输入的描述符都加入 read set 中
然后通过 FD_ISSET 判断哪个描述符处于 ready set 集合中，如果在该集合，则处理对应的事件

上面程序的缺点：
If you type a command to standard input, you will not get a response until the server is finished with the client. 
A better approach would be to multiplex at a finer granularity, echoing (at most) one text line each time through the server loop.

## 12.2.1 A Concurrent Event-Driven Server Based on I/O Multiplexing
