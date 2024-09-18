---
title: 一次解决Go编译问题的经过
comments: true
tags:
  - Go语言
  - 程序编译
  - 内存监控
  - 内存不足
  - 交叉编译
categories:
  - 软件安全
date: 2024-08-17 18:09:11
author: repoog
excerpt: 用Go语言编写一个小的程序之后，发现在Windows环境中编译正常，但在Linux环境下却编译失败，为了解决这个问题，不得不采用一些方法监控系统运行情况，最终发现的解决思路有两种：一种是内存监控，解决内存不足问题，一种是进行交叉编译，实现在Windows系统环境下编译Linux程序。
---

用Go语言编写了一个小的项目，项目开发环境是在本地的Windows环境中，一切单元测试和集成测试通过后，计划将项目部署到VPS服务器上自动运行，但在服务器上执行go run运行时，程序没有任何响应和回显，甚至main函数一开始的fmt.Println()都没有任何输出。

出现两个环境相同程序执行结果的不同，无外乎两类问题：配置问题和环境问题。

首先，检查了两个环境的Go语言配置和模块配置区别，包括Go语言版本差异（本地是1.22.4版本，服务器是1.23.0版本），并通过go clean -modcache和go mod tidy重新拉取了涉及的模块，发现问题依然存在。

其次，将程序中的几个单元测试文件单独执行，go test -v命令并没有任何输出，这意味着单元测试文件执行时候也有同样的问题，起初怀疑是和模块引用路径或程序中的路径或程序在Linux下的权限设置相关，但单元测试执行的问题排除了模块引用和路径问题，问题指向程序中数据库操作相关的函数。

因此，将主程序分步注释和执行，确定导致程序无响应的代码块，最终发现问题的根本在于：

``` go
import modernc.org/sqlite
```

这是一个操作SQLite数据库的第三方模块，是用Go语言编写，理论上是不存在环境兼容问题的。要查清楚问题的原因，需要单独编写一个程序验证该模块的问题，新的验证程序只引入了这个模块：

``` go
package main

import (
	"fmt"
	_ "modernc.org/sqlite"
)

func main() {
	fmt.Println("This is a test for sqlite")
}
```

通过go run单独执行该测试程序也出现相同现象，接下来通过go build -v手动编译检查模块引入过程中的问题，过程显示：

``` shell
modernc.org/sqlite/lib: /usr/local/go/pkg/tool/linux_amd64/compile: signal: killed
```

编译过程到此结束，意味着sqlite模块在编译过程中编译进程被杀死，导致后续程序均无法执行，结合htop工具检查下编译过程中编译进程和系统资源的情况：

![htop进程监控](/images/2024/09/htop-mem.png 'htop进程监控')

进一步再用dmesg | grep -i 'killed'，发现是由于Out of memory问题导致进程被杀死，和上图中的内存使用情况吻合（内存和Swap都满了），毕竟VPS服务器的内存有限，只有500Mb，Swap也只有265Mb。

问题的原因确定后，对应的解决思路就有两种：要么增加内存，要么在其他环境编译后移植可执行文件。

## 增加内存

由于内存本身有限，增加内存只能通过增加Swap解决，即在现有的/swap文件之外设置一个新的swap文件，并将新文件设置为新的swap。命令如下：

``` shell
falloate -l 1G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

另外，通过编辑/etc/fstab文件和添加下面命令，将新swap文件设置为系统启动时候自动加载的swap文件：

``` shell
/swapfile swap swap defaults 0 0
```

完成以上配置之后，重新在测试代码目录下执行go build -v便可以完成编译过程，经过htop对于资源利用的观测，实际编译modernc.org/sqlite至少需要1G的内存。

## 交叉编译

交叉编译是在Windows系统上编译Linux系统运行的可执行文件，这样可以利用Windows系统的性能完成编译工作，直接在Linux系统上运行可执行文件。

进行交叉编译需要设定Go语言的环境变量GOOS和GOARCH，前者是系统类型，比如windows、linux，后者是架构类型，比如amd64、arm。笔者使用的两个环境都是amd64的64位系统，因此只需要设置系统类型：

``` shell
go env -w GOOS=linux
```

之后再执行go build创建可执行文件，最后将可执行文件上传到VPS服务器执行即可。