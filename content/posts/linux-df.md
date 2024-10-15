+++
title = "linux中的文件描述符(fd)"
description = ""
tags = [
    "linux",
    "fd"
]
date = "2024-10-15"
categories = [
    "Development",
    "linux",
]
menu = "main"
+++

什么是fd?
在linux 中多数资源以文件形式存在, 文件描述符就是文件对应的编号。
为什么存在 fd
fd的作用
1. 描述符是文件描述符，文件描述符是进程和文件系统之间的一个接口，它用来表示进程和文件系统之间的一个连接。