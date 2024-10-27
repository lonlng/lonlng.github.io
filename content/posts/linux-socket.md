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



## API

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

  

### bind()

```c++
#include <sys/socket.h>

int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

#### 说明

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







## Socket options



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



### SO_REUSEADDR

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
