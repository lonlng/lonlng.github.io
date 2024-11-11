+++
title = "debian 安装指定版本的软件并忽略更新"
description = ""
tags = [
    "Debian",
    "Linux",
    "apt"
]
date = "2024-10-26"
categories = [
    "Linux",
    "Debian"
]
menu = "main"

+++



## apt安装指定版本

```bash
sudo apt install package=version
```

version是软件版本号，package是要安装的软件

## 忽略更新软件包

使用 `apt-mark` 命令

```bash
sudo apt-mark hold package
```

package是要安装的软件



查看hold软件列表

```bash
sudo apt-mark showhold
```



解除hold

```bash
sudo apt-mark unhold package
```

package是要安装的软件
