+++
title = "linux中的socket"
description = ""
tags = [
    "linux",
    "socket"
]
date = "2024-10-26"
categories = [
    "Development",
    "linux"
]
menu = "main"

+++



# 简述

socket起源于Unix，而Unix/Linux基本哲学之一就是“一切皆文件”，都可以用“打开open –> 读写write/read –> 关闭close”模式来操作。Socket就是该模式的一个实现， socket即是一种特殊的文件，一些socket函数就是对其进行的操作（读/写IO、打开、关闭）.
说白了Socket是应用层与TCP/IP协议族通信的中间软件抽象层，它是一组接口。在设计模式中，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。

注意：其实socket也没有层的概念，它只是一个外观模式（Facade Pattern）的应用，让编程变的更简单。是一个软件抽象层。在网络编程中，大量用的都是通过socket实现的。

scoket 是一个通信的终端 , 在`<sys/socket.h>` c标准库中。



# 协议族

> int socket(int domain, int type, int protocol);

`socket()`函数创建了一个通信的终端， 并返回了一个`fd` (链接外部文件) 代表这个终端,  成功返回的 `fd` 将是当前程序打开编号中最低的`fd`.

**domain** 参数指定了通信的域, 它选择的协议族将会用于通信, 协议族定义在 `<sys/socket.h>` 中, 当前linux 内核可使用的协议族包括:



| Name         | Purpose                                                      | Man page           |
| ------------ | ------------------------------------------------------------ | ------------------ |
| AF_UNIX      | Local communication                                          | unix(7)            |
| AF_LOCAL     | Synonym for AF_UNIX                                          |                    |
| AF_INET      | IPv4 Internet protocols                                      | ip(7)              |
| AF_AX25      | Amateur radio AX.25 protocol                                 | ax25(4)            |
| AF_IPX       | IPX - Novell protocols                                       |                    |
| AF_APPLETALK | AppleTalk                                                    | ddp(7)             |
| AF_X25       | ITU-T X.25 / ISO-8208 protocol                               | x25(7)             |
| AF_INET6     | IPv6 Internet protocols                                      | ipv6(7)            |
| AF_DECnet    | DECet protocol sockets                                       | rds(7)             |
| AF_KEY       | Key management protocol,  originally developed for usage with IPsec | rds-rdma(7)        |
| AF_NETLINK   | Kernel user interface device                                 | netlink(7)         |
| AF_PACKET    | Low-level packet interface                                   | packet(7)          |
| AF_RDS       | Reliable Datagram Sockets (RDS)  protocol                    | rds(7) rds-rdma(7) |
| AF_PPPOX     | Generic PPP transport layer, for  setting up L2 tunnels (L2TP and PPPoE) |                    |
| AF_LLC       | Logical link   control (IEEE 802.2 LLC)  protocol            |                    |
| AF_IB        | InfiniBand native addressing                                 |                    |
| AF_MPLS      | Multiprotocol Label Switching                                |                    |
| AF_CAN       | Controller Area Network automotive   bus protocol            |                    |
| AF_TIPC      | TIPC, "cluster domain  sockets" protocol                     |                    |
| AF_BLUETOOTH | Bluetooth low-level socket protocol                          |                    |
| AF_ALG       | Interface to kernel crypto API                               |                    |
| AF_VSOCK     | VSOCK  (originally "VMWare VSockets") protocol for hypervisor-guest  communication | vsock(7)           |
| AF_KCM       | KCM   (kernel connection multiplexer) interface              |                    |
| AF_XDP       | XDP (express data path) interface                            |                    |



```c
/* Address families.  */
#define AF_UNSPEC	PF_UNSPEC
#define AF_LOCAL	PF_LOCAL
#define AF_UNIX		PF_UNIX
#define AF_FILE		PF_FILE
#define AF_INET		PF_INET
#define AF_AX25		PF_AX25
#define AF_IPX		PF_IPX
#define AF_APPLETALK	PF_APPLETALK
#define AF_NETROM	PF_NETROM
#define AF_BRIDGE	PF_BRIDGE
#define AF_ATMPVC	PF_ATMPVC
#define AF_X25		PF_X25
#define AF_INET6	PF_INET6
#define AF_ROSE		PF_ROSE
#define AF_DECnet	PF_DECnet
#define AF_NETBEUI	PF_NETBEUI
#define AF_SECURITY	PF_SECURITY
#define AF_KEY		PF_KEY
#define AF_NETLINK	PF_NETLINK
#define AF_ROUTE	PF_ROUTE
#define AF_PACKET	PF_PACKET
#define AF_ASH		PF_ASH
#define AF_ECONET	PF_ECONET
#define AF_ATMSVC	PF_ATMSVC
#define AF_RDS		PF_RDS
#define AF_SNA		PF_SNA
#define AF_IRDA		PF_IRDA
#define AF_PPPOX	PF_PPPOX
#define AF_WANPIPE	PF_WANPIPE
#define AF_LLC		PF_LLC
#define AF_IB		PF_IB
#define AF_MPLS		PF_MPLS
#define AF_CAN		PF_CAN
#define AF_TIPC		PF_TIPC
#define AF_BLUETOOTH	PF_BLUETOOTH
#define AF_IUCV		PF_IUCV
#define AF_RXRPC	PF_RXRPC
#define AF_ISDN		PF_ISDN
#define AF_PHONET	PF_PHONET
#define AF_IEEE802154	PF_IEEE802154
#define AF_CAIF		PF_CAIF
#define AF_ALG		PF_ALG
#define AF_NFC		PF_NFC
#define AF_VSOCK	PF_VSOCK
#define AF_KCM		PF_KCM
#define AF_QIPCRTR	PF_QIPCRTR
#define AF_SMC		PF_SMC
#define AF_XDP		PF_XDP
#define AF_MCTP		PF_MCTP
#define AF_MAX		PF_MAX
```





# API



这个列表是一个Berkeley套接字API库提供的函数或者方法的概要：

- `socket()` 创建一个新的确定类型的套接字，类型用一个整型数值标识，并为它分配系统资源。
- `bind()` 一般用于服务器端，将一个套接字与一个套接字地址结构相关联，比如，一个指定的本地端口和IP地址。
- `listen()` 用于服务器端，使一个绑定的TCP套接字进入监听状态。
- `connect()` 用于客户端，为一个套接字分配一个自由的本地端口号。 如果是TCP套接字的话，它会试图获得一个新的TCP连接。
- `accept()` 用于服务器端。 它接受一个从远端客户端发出的创建一个新的TCP连接的接入请求，创建一个新的套接字，与该连接相应的套接字地址相关联。
- `send()`和`recv()`,或者`write()`和`read()`,或者`recvfrom()`和`sendto()`, 用于往/从远程套接字发送和接受数据。
- `close()` 用于系统释放分配给一个套接字的资源。 如果是TCP，连接会被中断。
- `gethostbyname()`和`gethostbyaddr()` 用于解析主机名和地址。
- `select()` 用于修整有如下情况的套接字列表： 准备读，准备写或者是有错误。
- `poll()` 用于检查套接字的状态。 套接字可以被测试，看是否可以写入、读取或是有错误。
- `getsockopt()` 用于查询指定的套接字一个特定的套接字选项的当前值。
- `setsockopt()` 用于为指定的套接字设定一个特定的套接字选项。








## socket()

`socket()` 为通讯创建一个端点，为套接字返回一个文件描述符。 socket() 有三个参数：

- domain

   为创建的套接字指定协议集。 例如：

  - `PF_INET` 表示IPv4网络协议
  - `PF_INET6` 表示IPv6
  - `PF_UNIX` 表示本地套接字（使用一个文件）

- type

   如下：

  - `SOCK_STREAM` （可靠的面向流服务或流套接字）
  - `SOCK_DGRAM` （数据报文服务或者数据报文套接字）
  - `SOCK_SEQPACKET` （可靠的连续数据包服务）
  - `SOCK_RAW` (在网络层之上的原始协议)。

- protocol 指定实际使用的传输协议。 最常见的就是`IPPROTO_TCP`、`IPPROTO_SCTP`、`IPPROTO_UDP`、`IPPROTO_DCCP`。这些协议都在<netinet/in.h>中有详细说明。 如果该项为“`0`”的话，即根据选定的domain和type选择使用缺省协议。

如果发生错误，函数返回值为-1。 否则，函数会返回一个代表新分配的描述符的整数。

- 原型：

```
int socket(int domain, int type, int protocol)。
```



## bind()

`bind()` 为一个套接字分配地址。当使用`socket()`创建套接字后，只赋予其所使用的协议，并未分配地址。在接受其它主机的连接前，必须先调用bind()为套接字分配一个地址。`bind()`有三个参数：

- `sockfd`, 表示使用bind函数的套接字描述符
- `my_addr`, 指向sockaddr结构（用于表示所分配地址）的指针
- `addrlen`, 用socklen_t字段指定了sockaddr结构的长度

如果发生错误，函数返回值为-1，否则为0。



原型

```c++
#include <sys/socket.h>

int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

**说明**

当使用 `socket(2)` 创建套接字时，它存在于名称空间（地址系列）中，但没有为其分配地址。 `bind()` 将 addr 指定的地址分配给文件描述符 **sockfd** 引用的套接字。 **addrlen** 指定 **addr** 指向的地址结构的大小（以字节为单位）。 传统上，此操作称为 “为套接字分配名称”。

1. 套接字在创建时并没有地址，使用`bind()`函数将指定的地址（`addr`）赋给套接字。`addrlen`指定了地址结构的大小（字节数）。
2. 通常在SOCK_STREAM套接字接收连接之前，需要先使用`bind()`来指定一个本地地址。
3. 各种地址族（如AF_INET，AF_INET6等）在命名绑定时的规则不同，具体信息可以参考各自手册。
4. `bind()`函数根据地址族不同会传递不同的地址结构。`sockaddr`结构通常如下：

```c
struct sockaddr {
    sa_family_t sa_family;
    char        sa_data[14];
}
```



**返回值：** 成功时返回0，出错时返回-1，并设置errno以指示错误。

**可能的错误：**

- EACCES: 地址受保护，用户不是超级用户。
- EADDRINUSE: 地址已被使用。
- EBADF: sockfd不是有效的文件描述符。
- EINVAL: 套接字已经绑定到一个地址或`addrlen`错误。
- ENOTSOCK: 文件描述符sockfd不是一个套接字。



示例:

```c++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <sys/un.h>
#include <unistd.h>

#define MY_SOCK_PATH "/somepath"
#define LISTEN_BACKLOG 50

#define handle_error(msg)   \
    do                      \
    {                       \
        perror(msg);        \
        exit(EXIT_FAILURE); \
    } while (0)

int main(void)
{
    int sfd, cfd;
    socklen_t peer_addr_size;
    struct sockaddr_un my_addr, peer_addr;

    sfd = socket(AF_UNIX, SOCK_STREAM, 0);
    if (sfd == -1)
        handle_error("socket");

    memset(&my_addr, 0, sizeof(my_addr));
    my_addr.sun_family = AF_UNIX;
    strncpy(my_addr.sun_path, MY_SOCK_PATH,
            sizeof(my_addr.sun_path) - 1);

    if (bind(sfd, (struct sockaddr *)&my_addr,
             sizeof(my_addr)) == -1)
        handle_error("bind");

    if (listen(sfd, LISTEN_BACKLOG) == -1)
        handle_error("listen");

    /* Now we can accept incoming connections one
       at a time using accept(2). */

    peer_addr_size = sizeof(peer_addr);
    cfd = accept(sfd, (struct sockaddr *)&peer_addr,
                 &peer_addr_size);
    if (cfd == -1)
        handle_error("accept");

    /* Code to deal with incoming connection(s)... */

    if (close(sfd) == -1)
        handle_error("close");

    if (unlink(MY_SOCK_PATH) == -1)
        handle_error("unlink");
}

```



## listen()

当socket和一个地址绑定之后，`listen()`函数会开始监听可能的连接请求。然而，这只能在有可靠数据流保证的时候使用，例如：数据类型(`SOCK_STREAM`,`SOCK_SEQPACKET`)。

listen()函数需要两个参数：

- `sockfd`, 一个socket的描述符.
- `backlog`, 一个决定监听队列大小的整数，当有一个连接请求到来，就会进入此监听队列，当队列满后，新的连接请求会返回错误。当请求被接受，返回 0。反之，错误返回 -1。

原型:

```
int listen(int sockfd, int backlog);
```



## accept()

当应用程序监听来自其他主机的面对数据流的连接时，通过事件（比如Unix select()系统调用）通知它。必须用 `accept()`函数初始化连接。 Accept() 为每个连接创立新的套接字并从监听队列中移除这个连接。它使用如下参数：

- `sockfd`,监听的套接字描述符
- `cliaddr`, 指向sockaddr 结构体的指针，客户机地址信息。
- `addrlen`,指向 `socklen_t`的指针，确定客户机地址结构体的大小 。

返回新的套接字描述符，出错返回-1。进一步的通信必须通过这个套接字。

Datagram 套接字不要求用accept()处理，因为接收方可能用监听套接字立即处理这个请求。

- 函数原型：

```
int accept(int sockfd, struct sockaddr *cliaddr, socklen_t *addrlen);
```

## connect()

`connect()`系统调用为一个套接字设置连接，参数有文件描述符和主机地址。

某些类型的套接字是无连接的，大多数是UDP协议。对于这些套接字，连接时这样的：默认发送和接收数据的主机由给定的地址确定，可以使用 send()和 recv()。 返回-1表示出错，0表示成功。

- 函数原型：

```
int connect(int sockfd, const struct sockaddr *serv_addr, socklen_t addrlen);
```



## recv()



 recv, recvfrom, recvmsg - 从socket 中接受消息

**语法：**

```c++
#include <sys/socket.h>
ssize_t recv(int sockfd, void buf[.len], size_t len,
                 int flags);
ssize_t recvfrom(int sockfd, void buf[restrict .len], size_t len,
                 int flags,
                 struct sockaddr *_Nullable restrict src_addr,
                 socklen_t *_Nullable restrict addrlen);
ssize_t recvmsg(int sockfd, struct msghdr *msg, int flags);
```

**说明：**

**recv()**调用通常仅在已连接的套接字上使用（参见connect（2））。它相当于调用：

```c++
recvfrom(fd, buf, len, flags, NULL, 0);
```



## send()

send, sendto, sendmsg -  给socket发送消息

**语法：**

```c++
#include <sys/socket.h>
ssize_t send(int sockfd, const void buf[.len], size_t len, int flags);
ssize_t sendto(int sockfd, const void buf[.len], size_t len, int flags,
               const struct sockaddr *dest_addr, socklen_t addrlen);
ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags);
```





## readv()

readv, writev, preadv, pwritev, preadv2, pwritev2 - 将数据读取或写入多个缓冲区

**语法：**

```c++
#include <sys/uio.h>
ssize_t readv(int fd, const struct iovec *iov, int iovcnt);
ssize_t writev(int fd, const struct iovec *iov, int iovcnt);
ssize_t preadv(int fd, const struct iovec *iov, int iovcnt,
                off_t offset);
ssize_t pwritev(int fd, const struct iovec *iov, int iovcnt,
                off_t offset);
ssize_t preadv2(int fd, const struct iovec *iov, int iovcnt,
                off_t offset, int flags);
ssize_t pwritev2(int fd, const struct iovec *iov, int iovcnt,
                off_t offset, int flags);
```



**writev()**系统调用将iov描述的数据的**iovcnt**缓冲区写入与文件描述符fd关联的文件。**writev()**系统调用的工作方式与write（2）类似，只是写出了多个缓冲区。



>  iovec - Vector I/O data structure
>
> ```c++
> #include <sys/uio.h>
> struct iovec {
>     void   *iov_base;  /* Starting address */
>     size_t  iov_len;   /* Size of the memory pointed to by iov_base. */
> };
> ```
>
> 描述一个内存区域，从**iov_base**地址开始，大小为**iov_len**字节。系统调用使用这种结构的数组，其中数组的每个元素表示一个内存区域，整个数组表示内存区域的向量。该数组中**iovec**结构的最大数量受**IOV_MAX**（在<limits.h>中定义，或可通过调用sysconf（_SC_IOV_MAX）访问）的限制。





# Socket options



下面列出的套接字选项可以通过使用 `setsockopt(2) `进行设置，并使用 `getsockopt(2)` 进行读取，并将所有套接字的套接字级别设置为 `SOL_SOCKET`。除非另有说明，否则 `optval `是一个指向 int 的指针。

使用说明:

```c++
#include <sys/socket.h>

int getsockopt(int sockfd, int level, int optname,
                      void optval[restrict *.optlen],
                      socklen_t *restrict optlen);

int setsockopt(int sockfd, int level, int optname,
                      const void optval[.optlen],
                      socklen_t optlen);
```



## SO_REUSEADDR

指示用于验证`bind(2)`调用中提供的地址的规则应允许重用本地地址。对于`AF_INET socket`，这意味着套接字可能会绑定，除非有一个活动的侦听套接字绑定到该地址。当侦听套接字绑定到具有特定端口的`INADDR_ANY`时则不可能为任何本地地址绑定到此端口。参数是一个整数布尔标志。



> **NOTES**:
>
> Linux 将只允许使用 `SO_REUSEADDR `选项重用端口，前提是在执行 `bind(2)` 到端口的上一个进程和想要重用该端口的进程中设置了此选项。 这与某些实现 （例如 FreeBSD） 不同，在一些实现中，只有后面的进程需要设置 `SO_REUSEADDR` 选项。
>  通常，这种差异是不可见的，因为例如，服务器进程被设计为始终设置此选项。



```c++
#include <sys/socket.h>
void main(){
    int socket_fd = socket(PF_INET, SOCK_STREAM, 0);
    int flag = 1;
    setsockopt(socket_fd, SOL_SOCKET, SO_REUSEADDR, &flag, sizeof(flag));
}
```





# 参考链接:

[socket(2) - Linux manual page](https://man7.org/linux/man-pages/man2/socket.2.html)

[socket(7) - Linux manual page](https://man7.org/linux/man-pages/man7/socket.7.html)
