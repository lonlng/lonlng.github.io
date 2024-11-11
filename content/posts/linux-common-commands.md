+++
title = "linux常用命令"
description = ""
tags = [
    "linux"
]
date = "2024-10-26"
categories = [
    "linux"
]
menu = "main"

+++



**查看内核版本**

```bash
# 仅查内核版本
uname -r

# 查询更多详细信息
uname -a

# 查询系统文件
cat /proc/version
cat /etc/os-release
```



**查看cpu信息**

```bash
# 显示系统的 CPU 信息，包括核数、线程数等
lscpu

# 查看cpu信息 显示每个物理 CPU 的核数
cat /proc/cpuinfo

# 查看cpu内核数
nproc
```

