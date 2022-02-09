---
title: 操作系统常见面试题
author:
  name: Jugg Wang
  link: https://github.com/kekaiwang
date: 2022-02-08 09:30:00 +0800
categories: [操作系统, Interview]
tags: [面试, 操作系统, OS]
render_with_liquid: false
---

### 如何监听端口

- **`netstat` 的命令用来显示网络状态**

  - `netstat -anp`：显示系统端口使用情况
  - `netstat -nupl`：UDP类型的端口
  - `netstat -ntpl`：TCP类型的端口
  - `netstat -na|grep ESTABLISHED|wc -l`：统计已连接上的，状态为"established"
  - `netstat -l`：只显示所有监听端口
  - `netstat -lt`：只显示所有监听tcp端口
- `ps -ef | grep 80`： 查看指定端口占用

### 查看内存使用情况

1. **`free` 命令**

    - `total` 表示总共有 7822MB 的物理内存(RAM)，即7.6G。
    - `used` 表示物理内存的使用量，大约是 322M。
    - `free` 表示空闲内存;
    - `shared` 表示共享内存?;
    - `buff/cache` 表示缓存和缓冲内存量; Linux 系统会将很多东西缓存起来以提高性能，这部分内存可以在必要时进行释放，给其他程序使用。
    - `available` 表示可用内存;

2. **`cat /proc/meminfo`**

    - `MemTotal`, 总内存
    - `MemFree`, 空闲内存
    - `MemAvailable`, 可用内存
    - `Buffers`, 缓冲
    - `Cached`, 缓存
    - `SwapTotal`, 交换内存
    - `SwapFree`, 空闲交换内存
3. **`vmstat -s`**

4. **`top` 命令一般用于查看进程的CPU和内存使用情况**

### 查看磁盘使用情况

- **`df -h`**
- `df -h /usr`： 查看指定目录
- `du --max-depth=1 -h`: 查看当前目录文件夹的使用情况
- `du -sh /usr/`: 计算文件夹大小

### Nginx

Nginx由内核和模块组成，其中，内核的设计非常微小和简洁，完成的工作也非常简单，仅仅通过查找配置文件将客户端请求映射到一个locationblock（location是Nginx配置中的一个指令，用于URL匹配），而在这个location中所配置的每个指令将会启动不同的模块去完成相应的工作。

Nginx的模块从结构上分为核心模块、基础模块和第三方模块：

- **核心模块**：HTTP模块、EVENT模块和MAIL模块
- **基础模块**：HTTP Access模块、HTTP FastCGI模块、HTTP Proxy模块和HTTP Rewrite模块，
- **第三方模块**：HTTP Upstream RequestHash模块、Notice模块和HTTP Access Key模块。

Nginx的模块从功能上分为如下三类：

- **Handlers（处理器模块）**。此类模块直接处理请求，并进行输出内容和修改headers信息等操作。Handlers处理器模块一般只能有一个。
- **Filters （过滤器模块）**。此类模块主要对其他处理器模块输出的内容进行修改操作，最后由Nginx输出。
- **Proxies（代理类模块）**。此类模块是Nginx的HTTP Upstream之类的模块，这些模块主要与后端一些服务比如FastCGI等进行交互，实现服务代理和负载均衡等功能。

### fork() 的实现原理

`fork()` 函数是非常重要的函数，他从一个已存在的进程中创建一个新进程；新进程为子进程，而原进程称为父进程。

以当前进程作为父进程创建出一个新的子进程，并且将父进程的所有资源拷贝给子进程，这样子进程作为父进程的一个副本存在。父子进程几乎时完全相同的，但也有不同的如父子进程ID不同。

```c++
#include<unistd.h>
pid_t fork(void)
```

`pid_t` 是进程描述符，实质就是一个 `int`，**如果 fork 函数调用失败，返回一个负数，调用成功则返回两个值：0和子进程ID**。

- 该进程为父进程时，返回子进程的 pid
- 该进程为子进程时，返回0
- fork执行失败，返回 -1

fork() 系统调用通过复制一个现有进程来创建一个全新的进程。进程被存放在一个叫做任务队列的双向循环链表当中，链表当中的每一项都是类型为task_struct称为进程描述符的结构，也就是我们写过的进程PCB.

Tips：内核通过一个位置的进程标识值或PID来标识每一个进程。//最大值默认为32768，short int短整型的最大值.，他就是系统中允许同时存在的进程最大的数目。
可以到目录 /proc/sys/kernel中查看pid_max：

**当进程调用fork后，当控制转移到内核中的fork代码后，内核会做4件事情**:

1. 分配新的内存块和内核数据结构给子进程
2. 将父进程部分数据结构内容(数据空间，堆栈等）拷贝至子进程
3. 添加子进程到系统进程列表当中
4. fork返回，开始调度器调度

#### 为什么fork成功调用后返回两个值?

由于在复制时复制了父进程的堆栈段，所以两个进程都停留在fork函数中，等待返回。所以fork函数会返回两次,一次是在父进程中返回，另一次是在子进程中返回，这两次的返回值不同  
其中父进程返回子进程pid，这是由于一个进程可以有多个子进程，但是却没有一个函数可以让一个进程来获得这些子进程id，那谈何给别人你创建出来的进程。而子进程返回0，这是由于子进程可以调用getppid获得其父进程进程ID,但这个父进程ID却不可能为0，因为进程ID0总是有内核交换进程所用，故返回0就可代表正常返回了。
