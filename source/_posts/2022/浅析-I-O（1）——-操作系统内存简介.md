---
title: 浅析 I/O（1）—— 操作系统内存简介
author: ratears
categories:
	- [Operating-Systems,I/O]
tags:
	- Operating-Systems
	- I/O
date: 2022-09-19 21:56:50
updated: 2022-09-19 21:56:50
---

# 操作系统的应用与内核

&emsp;&emsp;现代计算机是由硬件和操作系统组成，我们的应用程序要操作硬件（如往磁盘上写数据），就需要先与内核交互，然后再由内核与硬件交互；

&emsp;&emsp;操作系统可以划分为：**内核**与**应用**两部分；

&emsp;&emsp;内核提供进程管理、内存管理、网络等底层功能，封装了与硬件交互的接口，通过系统调用提供给上层应用使用。



<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/6a59c5e5-a44b-444d-b5de-22a47319e41a.5r3m1k43n280.webp" width="80%">

<br>

<br>

<br>

# 内核空间与用户空间

&emsp;&emsp;现在操作系统都是采用虚拟地址空间，那么对32位操作系统而言，它的寻址空间（虚拟存储空间）为4G（2的32次方）。操作系统的核心是内核，独立于普通的应用程序，可以访问受保护的内存空间（内核空间），也有访问底层硬件设备的所有权限。

&emsp;&emsp;为了保证用户进程不能直接操作内核（kernel），保证内核的安全，操心系统将虚拟空间划分为两部分，一部分为**内核空间**，一部分为**用户空间**。内核空间是操作系统内核访问的区域，独立于普通的应用程序，是**受保护的内存空间**。用户空间是普通应用程序可访问的内存区域。

&emsp;&emsp;针对linux操作系统而言，将最高的1G字节（从虚拟地址0xC0000000到0xFFFFFFFF），供内核使用，称为内核空间，而将3G字节（从虚拟地址0x00000000到0xBFFFFFFF），供各个进程使用，称为用户空间。



<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.23luuh3fhlwg.webp" width="60%">

&emsp;&emsp;**用户态的程序不能随意操作内核地址空间，即使用户的程序崩溃了，内核也不受影响。这样对操作系统具有一定的安全保护作用。**

<br>

<br>

<br>

# CPU指令等级

&emsp;&emsp;其实早期操作系统是不区分内核空间和用户空间的，但是应用程序能访问任意内存空间，如果程序不稳定常常把系统搞崩溃，比如清除操作系统的内存数据。后来觉得让应用程序随便访问内存太危险了，就按照CPU 指令的重要程度对指令进行了分级；



<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.5dnygfmt0ow0.webp" width="40%">



&emsp;&emsp;CPU指令分为四个级别：Ring0~Ring3，linux 只使用了 Ring0 和 Ring3 两个运行级别，进程运行Ring3级别的指令时运行在用户态，指令只访问用户空间，而运行在 Ring0级别时被称为运行在内核态，可以访问任意内存空间。

<br>

<br>

<br>

# 进程的内核态和用户态

&emsp;&emsp;**当进程运行在内核空间时，它就处于内核态；当进程运行在用户空间时，它就处于用户态。**

- 那什么时候运行再内核空间什么时候运行再用户空间呢？

> 当我们需要进行IO操作时，如读写硬盘文件、读写网卡数据等，进程需要切换到内核态，否则无法进行这样的操作，无论是从内核态切换到用户态，还是从用户态切换到内核态，都需要进行一次上下文的切换。一般情况下，应用不能直接操作内核空间的数据，需要把内核态的数据拷贝到用户空间才能操作。

> 比如我们 Java 中需要新建一个线程，调用 start() 方法时，基于Hotspot Linux 的JVM 源码实现，最终是调`pthread_create`系统方法来创建的线程，这里会从用户态切换到内核态完成系统资源的分配，线程的创建。

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.ymdgdg0666o.webp" width="60%">

- 当一个任务（进程）执行系统调用而陷入内核代码中执行时，称进程处于内核运行态（内核态）

> Tips：除了系统调用可以实现用户态到内核态的切换，软中断和硬中断也会切换用户态和内核态。

- 在内核态下：进程运行在内核地址空间中，此时 CPU 可以执行任何指令。运行的代码也不受任何的限制，可以自由地访问任何有效地址，也可以直接进行端口的访问。
- 在用户态下：进程运行在用户地址空间中，被执行的代码要受到 CPU 的很多检查，比如：进程只能访问映射其地址空间的页表项中规定的在用户态下可访问页面的虚拟地址。

<br>

<br>

<br>

# 术语解释

## 核心态/内核态(Kernel model)和用户态(User model)

&emsp;&emsp;核心态(Kernel model)和用户态(User model)，CPU会在两个model之间切换。

- 核心态代码拥有完全的底层资源控制权限，可以执行任何CPU指令，访问任何内存地址，其占有的处理机是不允许被抢占的。内核态的指令包括：启动I/O，内存清零，修改程序状态字，设置时钟，允许/终止中断和停机。内核态的程序崩溃会导致PC停机。
- 用户态是用户程序能够使用的指令，不能直接访问底层硬件和内存地址。用户态运行的程序必须委托系统调用来访问硬件和内存。用户态的指令包括：控制转移，算数运算，取数指令，访管指令（使用户程序从用户态陷入内核态）。

<br>

<br>

<br>

## 进程切换

&emsp;&emsp;为了控制进程的执行，内核必须有能力挂起正在CPU上运行的进程，并恢复以前挂起的某个进程的执行。这种行为被称为进程切换。因此可以说，任何进程都是在操作系统内核的支持下运行的，是与内核紧密相关的。从一个进程的运行转到另一个进程上运行，这个过程中经过下面这些变化：

> 1. 保存处理机上下文，包括程序计数器和其他寄存器。
> 2. 更新PCB信息。
> 3. 把进程的PCB移入相应的队列，如就绪、在某事件阻塞等队列。
> 4. 选择另一个进程执行，并更新其PCB。
> 5. 更新内存管理的数据结构。
> 6. 恢复处理机上下文。

<br>

<br>

<br>

## 文件描述符(fd, File Descriptor)

&emsp;&emsp;FD用于描述指向文件的引用的抽象化概念。文件描述符在形式上是一个非负整数。实际上，它是一个索引值，指向内核为每一个进程所维护的该进程打开文件的记录表。当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。在程序设计中，一些涉及底层的程序编写往往会围绕着文件描述符展开。但是文件描述符这一概念往往只适用于UNIX、Linux这样的操作系统。

<br>

<br>

<br>