深入理解计算机系统——第十一章 Network Programming

资源：
> [视频课程](https://www.bilibili.com/video/BV1iW411d7hd?p=21)
> [视频课件 1](http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/21-netprog1.pdf)
> [视频课件 2](http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/22-netprog2.pdf)
> [解读《深入理解计算机系统(CSAPP)》第11章网络编程](https://blog.csdn.net/FMC_WBL/article/details/123808645)
> [深入理解计算机系统 第11章 笔记整理](http://doraemonzzz.com/2021/09/22/2021-9-22-%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%B3%BB%E7%BB%9F-%E7%AC%AC11%E7%AB%A0-%E7%AC%94%E8%AE%B0%E6%95%B4%E7%90%86/)
> [[读书笔记]CSAPP：25[VB]网络编程1](https://zhuanlan.zhihu.com/p/129040027)
> [IP地址和MAC地址的区别和联系是什么？](https://www.zhihu.com/question/49335649)



# 11.1 The Client-Server Programming Model
每个网络应用都是依据**客户端-服务器**模型，一个应用由一个**服务器进程**和一个或多个**客户端进程**组成。

一个**服务器**管理某种**资源**，并通过操作这种**资源**为**客户端**提供某种**服务**。

**客户端-服务器模型**的基本操作是**事务**（transaction），由下面四步组成：
1. 客户端需要服务时，通过向服务器发送请求来初始化一个事务。
2. 服务器接收到请求后，解释（interpret）它，然后以合适的方式操作其资源。
3. 服务器向客户端发送响应（response），然后等待下一个请求。
4. 客户端接收响应并进行处理。

![fig 11.1](https://img-blog.csdnimg.cn/e80b8b6aed70405c872b43f983fab7d5.png)
<br/>

**注意**：客户端和服务器是**进程**而不是机器。

# 11.2 Networks
客户端和服务器通常运行在分开的主机（host）上，通过计算机网络（computer network）的硬件和软件资源来交流。

对于**主机**而言，**网络**就是一种 I/O 设备，充当数据源和数据接收方。

![fig 11.2](https://img-blog.csdnimg.cn/e5232bdfd03e41d5ba3589390d47c201.png)

<br/>

- 从上图可以看出，适配器（adapter）是插在 I/O 总线的扩展插槽（expansion slots）中，为网络（network）提供物理接口。
- 从网络接收的数据经过网络适配器，I/O 总线和内存总线最终复制到主存中，即通过 DMA 传输方式（见 6.1.1 节）。
- 数据也能从内存复制到网络。

******************

从**物理**层面上说，**网络**是依据地理远近组成的**层次系统**（hierarchical system）：

![network](https://img-blog.csdnimg.cn/65d7749a5dbc4c63ab00a60bb5fbf70a.png)
***********************

**以太网段（Ethernet segment）：**
- **以太网段**由一些**电缆**（通常为双绞线（twisted pairs of wires））和一个被称为**集线器**（hub）的小盒子组成，见下图所示：
![fig 11.3](https://img-blog.csdnimg.cn/f595fd26c4cf4b578a294d2c3d5d503f.png)
- **以太网段**通常覆盖的面积的范围很小，如一个房间或一栋建筑的一层。 
- 每个**电缆**有相同大小的带宽（bandwidth），通常为 100 Mb/s 或 1 Gb/s。
- **电缆**的一端连接到一个**主机的适配器**上，另一端连到**集线器**的一个端口上。
- **集线器**会将**每个端口**接收到的**数据**会**复制**给**其他的所有端口**，因此与**集线器**相连的**每个主机**能看到集线器上传输的所有数据。
- 每个**以太网适配器**有一个全球唯一的 48 位地址（**MAC address**），该地址存在**适配器**的**非易失性存储器**上。
-  A **host** can send **a chunk of bits** called a **frame** to **any other host** on the **segment**.
	- Each **frame** includes some **fixed number of header bits** that **identify** the **source** and **destination** of the **frame** and the **frame length**, followed by a **payload** of data bits.
	- **Every host adapter** sees the **frame**, but only the **destination host** actually reads it.

*****************************
**桥接以太网段（Bridged	Ethernet	Segment）**：
![fig 11.4](https://img-blog.csdnimg.cn/e47eea7ef64c4323b149146eddaced97.png)

- 通过一些**电缆**和被称为**网桥（bridges）**的小盒子，**多个以太网段**能连接成更大的**局域网**（LAN），称为**桥接以太网**（bridged Ethernets）。
- **桥接以太网**能覆盖更大的范围，如一栋建筑或学校。
- **网桥**比**集线器**能更好的利用带宽，它**不会无差别的复制数据**给所有的集线器，而是通过**自学习**后有选择的复制数据到对应的端口。例如主机 A 发送帧给主机 B，则网桥 X 会丢掉该帧的数据；如果主机 A 发送帧给主机 C，则网桥 X 复制帧给网桥 Y，然后网桥 Y 只会将帧复制给主机 C 所在的集线器端口。


**********************************

**互联网络（internet）：**
![fig 11.6](https://img-blog.csdnimg.cn/17fc41077aa14d919f73b4928babd7eb.png)

- **多个不兼容的局域网**可以通过一种被称为**路由器**（router）的计算机连接起来形成**互联网络（internet）**。
- 每个**路由器**对于它所连接的每个网络都有一个适配器（端口）。
- **路由器**也能连接高速点到点电话连接，这种是一种被称为**广域网**（wide area networks）的例子。

**注意**：
![Internet versus internet](https://img-blog.csdnimg.cn/cef56e898d15486684e3fe6d47da1f54.png)
<br/>

The crucial property of an **internet** is that it can consist of **different LANs and WANs** with radically **different** and **incompatible technologies**.

***********************************

**协议软件（protocol software）：** 协议软件运行在每个**主机**和**路由器**上，通过一种**协议**（protocol）来处理**不同网络之间的差异**。

**协议（protocol）：** 一系列规则，用来管理不同的**主机**和**路由器**在**网络**之间如何**传输数据**。
- **Provides a naming scheme**
	- An	**internet protocol** defines a **uniform format** for **host addresses**.	
	- Each **host (router)** is then assigned **at least one of these internet addresses** that **uniquely identifies** it.
- **Provides a	delivery mechanism**
	- An **internet protocol** defines a **standard transfer unit (packet)**. 
	- **Packet** consists of **header** and **payload**.
		- **Header**: contains info such as **packet size**, **source** and **destination address**.
		- **Payload**: contains **data** bits sent from **source host**.

<br/>

下图 11.7 展示主机和路由器怎么通过互联网协议在不兼容的局域网之间传输数据：
![fig 11.7](https://img-blog.csdnimg.cn/fce1f1beb0ff4854b0857810a2946e14.png)
<br/>

运行在主机 A 的客户端连接的是局域网 LAN1，需要给连接在局域网 LAN2 上的主机 B 的服务器发送数据，该过程如下：
1. 主机 A 上的客户端通过系统调用从客户端的虚拟地址空间复制数据到内核的缓冲区。
2. 主机 A 上的**协议软件**为**数据**添加一个 **internet header**（PH）和 **LAN1 帧头**（LAN1 frame header）从而组成一个 **LAN1 frame**，再将该 **LAN1 frame** 传输给 **LAN1 适配器（adapter）**。
    - **internet header** 记录**主机 B** 的 IP 地址，**LAN1 frame header** 记录**路由器**的 MAC 地址。
    - **LAN1 frame** 的 **payload** 就是一个 **internet packet**，而该 **internet packer** 的 **payload** 是实际传输的**数据 data**。
3. LAN1 适配器将该帧复制到网络上。
4. 当**帧**到达**路由器**时，路由器的 LAN1 **适配器**读该帧的数据然后传送给 **protocol software**。
5. **路由器**根据**帧**中的 **internet packet header (PH)** 中获取 **destination internet address**（主机 B 的 IP 地址），然后将其作为**路由表（routing table）**的索引来决定将这个**包（packet）**转发到哪个**端口**，此例中为 LAN2。路由器去掉旧的帧头 FH1，然后添加新的帧头 FH2，该帧头记录主机 B 的 MAC 地址，然后将该帧传递给 LAN2 适配器。以太网帧的目的地址是下一跳的 MAC 地址，因此会动态变化，而 IP 数据报的目的地址是最终目的主机的 IP 地址，固定不变。
6.  路由器的 LAN2 适配器将帧复制到网络。
7. 当帧达到主机 B后，主机 B的适配器读取该帧数据然后传送给协议软件。
8. 主机 B的协议软件剥离帧头 FH2 和包头 PH2，当服务器有读这些数据的系统调用时，协议软件将最终的数据复制到服务器的虚拟地址空间。

# 11.3 The Global IP Internet

Figure 11.8 shows the basic hardware and software organization of an Internet client-server application.

![fig 11.8](https://img-blog.csdnimg.cn/68b447185f8a4ebd901b25783ddf0d13.png)

## 11.3.1 IP Addresses
1. IP 地址是无符号 32 位整数。
2. TCP/IP 定义了统一的网络字节序，对于 IP 地址等整型数据采用[大端字节序](https://blog.csdn.net/weixin_44515978/article/details/121730568)。
3. IP 地址通常用**点分十进制**（dotted-decimal notation）表示，如 128.2.194.242。
4. IP 地址的结构如下：
![fig 11.9](https://img-blog.csdnimg.cn/34860976ae904369a861a37208e67115.png)

5. 主机字节序和网络字节序之间的转换如下：
![1](https://img-blog.csdnimg.cn/96b0ff760f2146688a1bfe32575b4d21.png)

6. IP 地址点分十进制字符串和二进制网络字节序之间转换函数：
![](https://img-blog.csdnimg.cn/4515f0b51fff4deabf653bf981583258.png)
函数名 `_` 后面的 `n` 表示 `network`，`p` 表示 `presentation`，这两个函数能操作 32 位的 IPv4 地址（AF_INET）或者 128 位的 IPv6 地址（AF_INET6）。
`inet_pton` 将点分十进制的字符串（src）转换为二进制的网络字节序（dst）。
`inet_ntop` 将二进制的网络字节序（src）转换为点分十进制的字符串（dst）。

## 11.3.2 Internet Domain Names
互联网定义便于记忆的**域名**（domain name）来代替难记忆的 IP 地址，并通过域名系统 DNS（domain name system）来管理域名和 IP 地址之间的映射，DNS 是一个**分布式的数据库系统**。

**域名**采用**层次树状结构**来命名，**不区分大小写**，其结构如下：

![](https://img-blog.csdnimg.cn/0bd2365bbc5347789a791a1632901842.png)

每个主机都有一个本地定义的域名 `localhost`，总是映射为**回送地址**（lookback address）127.0.0.1。

## 11.3.3 Internet Connections
Internet **clients** and **servers** communicate by sending and receiving **streams of bytes** over **connections**. 	Each connection	is:
- **Point-to-point**
连接一对进程。
-  **Full-duplex**
全双工模式，数据可以同时双向流动，同时发送和接收数据。
-  **Reliable**
**源进程**发送的数据基本能以**被目的进程**按照相同的字节序接收。

**连接**的末端就是套接字（socket）。
- 套接字的地址格式为：**IP 地址 : 端口**。（端口见 [计算机网络-谢希仁-第7版 第5章 运输层 5.09](https://blog.csdn.net/Lee567/article/details/127574147?csdn_share_tail=%7B%22type%22%3A%22blog%22%2C%22rType%22%3A%22article%22%2C%22rId%22%3A%22127574147%22%2C%22source%22%3A%22Lee567%22%7D)）
- **连接**由其两端的套接字地址唯一的确定，这一对套接字称为**套接字对**（socket pair），格式为 `(cliaddr:cliport, servaddr:servport)`，`cliaddr` 为客户端 IP 地址。
![fig 11.11](https://img-blog.csdnimg.cn/b83015f7b0344dfa9565dfb613b9b88e.png)

# 11.4 The Sockets Interface
The **sockets interface** is a set of functions that are used in conjunction with the Unix I/O functions to build network applications. 

![fig 11.12](https://img-blog.csdnimg.cn/5408801bdf82442fbc55a723e5d8a023.png)
<br/>


## 11.4.1 Socket Address Structures
**套接字：**
1. **从不同角度看套接字**
- 对于内核，套接字是通信的**端点**。
- 对于应用程序，套接字是一个有相应描述符的打开文件。

2. **客户端和服务器通过读写套接字描述符来通信**
![](https://img-blog.csdnimg.cn/87145318281845b59e98608902200c12.png)

3. **套接字和普通的 I/O 文件的区别**
The main distinction between regular file I/O and socket I/O is how the application "open" the socket descriptors.

*********************

**套接字地址结构：**
- **Generic socket address**
总共 16 字节，前两字节指明协议类型。
![](https://img-blog.csdnimg.cn/be3030e875bd4b94a4035d795ed93248.png)
![](https://img-blog.csdnimg.cn/7b83b2fe1c414c4ebf45f943d2c73205.png)

- **IP socket address structure**
总共 16 字节，前 2 字节协议类型为 `AF_INET`，表示使用 `IPV4` 协议；`sin_port` 占两个字节，表示端口号，**大端字节序**；`sin_addr` 占 4 字节，表示 IP 地址；最后的 8 个字节用来填充。
`sockadd_in` 可以看成 `sock_addr` 的子类。
![](https://img-blog.csdnimg.cn/349d6ffaad2247e58837bccbb4511b71.png)
![](https://img-blog.csdnimg.cn/e1b7d9ce28c04fc4b38378225e5b256c.png)

- **IP 地址**和**端口号**在网络中都是**大端字节序**。

## 11.4.2 The socket Function
**Clients** and **servers** use the **socket** function to create a **socket descriptor**.

![](https://img-blog.csdnimg.cn/94ae91c803fc4f6fb3aec07e5de55808.png)

```cpp
clientfd = Socket(AF_INET, SOCK_STREAM, 0);
```

- `AF_INET` 表名使用 32 位 IP 地址。
- `SOCK_STREAM` 表名套接字将是**连接**的端点。
- 返回的 `clientfd` 描述符还不能用来读写。

## 11.4.3 The connect Function
客户端通过 `connect` 函数和服务器建立连接。

![](https://img-blog.csdnimg.cn/f988bd22f65e432db11fbe3e48f3a9d4.png)

- `addr` 为服务器套接字地址。
- `addrlen` 为 `sizeof(sockaddr_in)`。
- 该函数会阻塞直到连接成功或出错。
- 如果连接建立成功，`clientfd` 则可以进行读写。
- 连接成功，则返回值为套接字对：`(x:y, addr.sin_addr:addr.sin_port)`，其中 `x` 为客户端 IP 地址，`y` 为客户端的短暂端口号。

## 11.4.4 The bind Function
The remaining sockets functions—**bind**, **listen**, and **accept**—are used by **servers** to establish connections with **clients**.

![](https://img-blog.csdnimg.cn/40bf963d91e847629128b6a3875be67a.png)

- `bind` 函数要求内核将服务器的套接字地址 `add` 和套接字描述符 `sockfd` 关联起来。
- `addrlen` 为 `sizeof(sockaddr_in)`。

## 11.4.5 The listen Function
默认情况下，内核认为由 `socket` 函数创建的描述符为**主动套接字**（active socket），将作为**客户端**的套接字。

**服务器**可以调用 `listen` 函数来通知内核该**描述符**将被**服务器**使用。

![](https://img-blog.csdnimg.cn/f1853fb13ff8418e9e91216aa2f2d095.png)

- `listen` 函数将 `sockfd` 从**主动套接字**转换为**监听套接字**（listening socket），用来接收**客户端**发送的连接请求。
- The `backlog` argument is a hint about the number of **outstanding connection requests** that the **kernel** should **queue up** before it starts to **refuse requests**.

## 11.4.6 The accept Function
**服务器**通过调用 `accept` 函数来等待**客户端**的**连接**请求。

![](https://img-blog.csdnimg.cn/57cc302b45634fd1afcffcecb6c3c229.png)

`accept` 函数等待来自**客户端**的**连接**请求到达**监听描述符** `listenfd`，然后将**客户端的套接字地址**填写到 `addr` 中，最后返回一个**已连接的描述符**用来和**客户端**通信。

**监听描述符**和**连接描述符**的区别：
- **监听描述符**是**客户端**请求**连接**的端点。
- **监听描述符**只**创建一次**，存在于**服务器的整个生命周期**。
- **连接描述符**是**客户端**和**服务器**创建的**连接**的端点。
- **连接描述符**在每次**服务器**接收一个**连接请求**时**创建**，只在**服务器**为**客户端**提供服务的过程中存在。

<br/>

监听和连接描述符的关系见下图：

![](https://img-blog.csdnimg.cn/63a38a2a437141f5843fb5808a5a4afc.png)

1. 服务器调用 `accept` 函数来等待连接请求到达监听描述符。
2. 客户端调用 `connect` 函数，向 `listenfd` 发送连接请求。
3. 服务器调用 `accecpt` 函数打开一个新的已连接的描述符 `connfd` ，在 `clientfd` 和 `connfd` 之间创建连接，返回 `connfd` 给应用程序。客户端和服务器能通信。

## 11.4.7 Host and Service Conversion
Linux provides some powerful **functions**, called **getaddrinfo** and **getnameinfo**, for **converting** back and forth between **binary socket address structures** and the **string representations of hostnames, host addresses, service names, and port numbers**.

###  The getaddrinfo Function
The **getaddrinfo** function **converts string representations of hostnames, host addresses, service names, and port numbers** into **socket address structures**. 

<br/>

![1](https://img-blog.csdnimg.cn/b77c617972554e6998bf03b398ecefdf.png)


<br/>

1. `host` 参数是域名或者数字形式的地址（如点分十进制的 IP 地址），可以为空 NULL。
2. `service` 参数是十进制的端口号或者服务名（如 http 协议），可以为空 NULL。
3. `host` 和 `service` 不能同时为空 NULL。
4. `hint` 是可选参数，是一个指向 `addrinfo` 结构体的指针，结构体如下，通常使用时，通过 `memset` 先将整个结构体清空，然后设置几个选择的字段。
![fig 11.16](https://img-blog.csdnimg.cn/33807ed821c34924835be4d63981c5eb.png)
5. Given **host** and **service** (the two components of a socket address), **getaddrinfo** returns a `result` that points to a **linked list of addrinfo structures**, each of which points to a **socket address structure** that corresponds to **host** and **service**.
**Clients**: walk this **list**, trying each **socket address** in turn, untill the **calls** to **socket** and **connect** succeed.	
**Servers**: walk the **list**	untill	**calls** to **socket** and **bind** succeed.	
![fig 11.5](https://img-blog.csdnimg.cn/9d45ce39006f4e389ffea2118c67b6ca.png)
6. By default, **getaddrinfo** can **return** both **IPv4** and **IPv6** socket addresses. Setting `ai_family` to **AF_INET** restricts the list to **IPv4** addresses. Setting it to **AF_INET6** restricts the list to **IPv6** addresses.
7. Helper functions:
- **freeadderinfo**	frees the entire linked list.
- **gai_strerror** converts error code to an error message.


### The getnameinfo Function
The **getnameinfo** function is the **inverse of getaddrinfo**. It converts a **socket address structure** to the corresponding **host** and **service** name strings. 
- It is the modern **replacement** for the obsolete **gethostbyaddr** and **getservbyport** functions.
- Unlike those functions, it is **reentrant and protocol-independent**.

![1](https://img-blog.csdnimg.cn/5a4b61467194434c845fd27a05a16e67.png)

示例：

![fig 11.7](https://img-blog.csdnimg.cn/5d0352744c754d07a66fc3034dd5e477.png)

# 11.5 Web Servers
## 11.5.1 Web Basics
- Web **clients** and **servers** interact using a text-based application-level protocol known as **HTTP** (hypertext transfer protocol). 
- Web content can be written in a language known as **HTML** (hypertext markup language).

![1](https://img-blog.csdnimg.cn/44d0f0cff1d14f30b0c08789adbdb5e5.png)

## 11.5.2 Web Content
1. To Web **clients** and **servers**, **content** is a sequence of bytes with an associated **MIME** (multipurpose internet mail extensions) type. 
![11.23](https://img-blog.csdnimg.cn/17346e3664f545e2a4d4e8742868744a.png)

2. Web **servers** provide content to **clients** in two different ways:
![2](https://img-blog.csdnimg.cn/118775db6818492dbb0c30c1ba06d778.png)

3. Every piece of **content** returned by a **Web server** is associated with some **file** that it manages. Each of these **files** has a **unique name** known as a **URL** (universal resource locator).
	- Clients and servers use different parts of the URL during a transaction. 
![3](https://img-blog.csdnimg.cn/d588b099a00e432fae52591c0951a6ca.png)

	- **URLs** for executable files can include program **arguments** after the filename. 
	A ‘**?**’ character **separates** the **filename** from the **arguments**, and each **argument** is separated by an ‘**&**’ character.
	For example, the URL `http://bluefish.ics.cs.cmu.edu:8000/cgi-bin/adder?15000&213` identifies an executable called `/cgi-bin/adder` that will be called with two argument strings: 15000 and 213. 


## 11.5.3 HTTP Transactions
1. **HTTP Requests**
An **HTTP request** consists of a **request line**, followed by **zero or more request headers**, followed by an **empty text line** that **terminates** the list of **headers**. 
- **Request line**
	- **Method**
![1](https://img-blog.csdnimg.cn/6198dba857bc45039ebea2d342a28c19.png)
	- **URI**
	[uri和url的区别与联系](https://blog.csdn.net/sinat_38719275/article/details/102607458)
![2](https://img-blog.csdnimg.cn/21c4982f708244fe80e754c16c6f55c6.png)
	- **Version**
	HTTP 版本，HTTP/1.0 或 HTTP/1.1。
- **request headers**
> 计算机网络 谢希仁 第7版

![3](https://img-blog.csdnimg.cn/09d2c6608ca24a21a82768a929adc260.png)
![4](https://img-blog.csdnimg.cn/e27e794a60d24d27b3f3f878f39c8f02.png)

2. **HTTP Responses**
An **HTTP response** consists of a **response line**, followed by **zero or more response headers**, followed by an **empty line** that **terminates the headers**, followed by the **response body**. 
- **Response line**
	- **Version**
	HTTP 版本。
	- **Status-code**
![5](https://img-blog.csdnimg.cn/ce46545c63054904b267920918e62ef3.png)

	- **Status-message**
	状态码的文字描述。
![6](https://img-blog.csdnimg.cn/0d0da0a05bd740d4ae434a06f9f28328.png)

- **Response headers**
![7](https://img-blog.csdnimg.cn/f5bfcf00edfb4b08baa18954d0013f07.png)

## 11.5.4 Serving Dynamic Content
见书和课件。

CGI 介绍：
![8](https://img-blog.csdnimg.cn/fe4db29b439f48f88fe5f3e3ebfe6cfc.png)

