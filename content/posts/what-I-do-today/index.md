---
title: "2023记录"
date: 2023-07-04T12:11:00+08:00
summary: "记录一下做了什么事情"
showtoc: false
categories:
- daily-record
---

## 2023/07/14

### 不同类型的操作系统
+ 简要结构内核
+ 宏内核
+ 微内核（内核态功能比较少，提供IPC、进程/线程等功能，大多数功能在用户态以服务的形式实现，比如，缺页中断处理可以在用户态进行，减小攻击面，某个模块崩溃的时候缩小影响面），比较有代表性的是MACH、L4微内核
+ Exokernel，也叫做外核，内核主要提供资源抽象和硬件的多路复用
+ Mutilkernel 异构计算

### NUMA
+ 内存控制器容易称为瓶颈，一些内存控制器分布在不同的核心上，本地和远端访问内存时间的不同的架构称为NUMA
+ 远端访问需要通过互联总线（interconnect）来与远端节点通信
+ NUMA节点由一个或多个CPU核心构成
+ NUMA-aware: 1、提供NUMA感知的接口，让应用决定 2、尽可能将内存分配在本地节点
+ Linux：绑定模式、优先模式（从最近的NUMA节点分配）、交错模式，libnuma封装了相关调用
  
 ### IPC(Inter-Process Communication)
+ 建立连接的方式
+ Naming Service
+ 权限检查（Capability）
+ 如何提高IPC的性能（减少拷贝的次数、参数传递的方式、LRPC的迁移线程模型）
+ IPC的几种方式（共享内存、信号量Semaphore、信号、管道等）
+ Android Binder IPC（Binder IPC模块、线程池、动态增加线程数量以及设置线程数量的上限、Handle的抽象、服务发现）

### cond_wait() & cond_signal() & cond_broadcast()
+ 生产者消费者问题
+ cond_wait() 放在while循环的原因？
+ cond_signal()需要每次调用？
+ cond_signal 和 释放锁的顺序调换过来会怎么样？

## 2023/07/13
### 文件系统的崩溃一致性（Crash Consistency）
+ redo log & undo log & WAL(Write Ahead Log)（至少要写两次）
+ Soft Update & 依赖追踪（循环依赖）& 撤销操作（重新开机时可以一边修复一边接收新的请求）
+ 同步写入（低效的方法）
+ 写时拷贝（容易出现写放大的现象，尤其是写入数据比较小的情况下，如1字节）
+ Ext4 文件系统不同的日志模式（ordered、journal、writeback）（针对文件数据，非元信息）
+ Log-structured File System（LFS）& Wear Leveling & Segment（一种折衷的方法）
+ FUSE（用户态文件系统）：容易调试、和内核隔离、可以使用丰富的第三方库



## 2023/07/12

### 科学、技术、工程和应用
今天看了一个视频，有一点印象比较深刻，讲的是科学、技术、工程和应用之间的区别。科学主要解决的是**是什么**和**为什么**，而技术主要解决的是怎么做，而工程解决的是如何多快好省地解决问题。举个例子：对于造一把刀这件事来说，科学指出什么样的材料**可以**造一把能用地刀（陶瓷或钢铁），而技术指出怎样造这样一把刀，而工程则是低成本地大批量生产这样的刀，而应用则类似于‘切西瓜的n种方法’。

### 系统虚拟化
最近读了一下SJTU的《操作系统原理和实现》，总结一下系统虚拟化的部分的内容

#### 一个演进的VMM（Virtual Machine Monitor）
前提：当前ISA下所有敏感指令都是特权指令，都会引起下陷

这一小节首先提出了一个最小的VMM0，在这个VMM中客户操作系统内核只能够执行用户ISA下的指令。VMM1在VMM0的基础上添加了时钟中断，本质上还是利用在用户态执行特权指令会最终会进入VMM中这个特点，来做一些信息的更新或者说行为的模拟，比如更新时钟中断的间隔这个指令本身是特权指令，导致虚拟机陷入虚拟机监视器，虚拟机监视器在这个过程中更新实际上位于内存的时间间隔。VMM2添加了虚拟机内核态和用户态之间切换的功能（设置一个变量指示当前是内核态还是用户态，从而在陷入VMM的时候可以将一些工作交给虚拟机系统来做）。VMM3增加了虚拟机多线程，但VMM本身不需要作任何改动。VMM4则通过宿主系统的线程来模拟VMM的vCPU（virtual CPU），从而重用宿主操作系统上的一些调度策略，简化了实现。

#### CPU 虚拟化（不可虚拟化架构）
+ 解释执行
+ 动态二进制翻译（基本块的概念、缓存）
+ 扫描-翻译（在翻译中替换掉不下陷的敏感指令，有些敏感指令不下陷到内核导致VMM无法接入）
+ 半虚拟化（Hypercall，virio中也用到了类似的机制）
+ 硬件虚拟化（Intel VT-x，一套寄存器）（ARM 的则有多组寄存器，切换时不需要专门对虚拟机中的寄存器进行保存）

#### 内存虚拟化
+ 影子页表
+ 直接页表（半虚拟化，通过hypercall等机制）
+ 硬件第二阶段地址翻译
+ 内存气球（Memory Ballooning），解决Semantic Gap
  
#### IO虚拟化
+ 软件模拟
+ 半虚拟化的方式（virio 简化驱动实现，一次陷入提交更多的数据）
+ 设备直通，解决安全问题（**IOMMU**），从硬件上实现不同虚拟机共享一个设备（对虚拟机而言看起来是独占）--**SR-IOV**


#### 中断虚拟化
+ 物理中断
+ 虚拟中断


#### KVM 和 QEMU

+ KVM是机制，QEMU是策略（QEMU还要负责IO虚拟化等内容）
+ KVM以/dev/kvm 的形式暴露出来，QEMU通过ioctl系统调用与其交互

## 2023/07/04

+ x86_64里面有一条endbr64指令，看了一些相关的资料，发现里面的内容还是很多的，Intel把它叫做Contro-flow Enforcement Technology（CET），简单来说，跳转指令执行后如果跳转到的指令不是endbr64的话，处理器抛出异常，这是一种JOP（Jump Oriented Programming）攻击的缓解措施（Mitigation）。

  在这之前的历史也十分有趣，在这之前还有一种攻击手段叫做ROP（Return Oriented Programming），简单来说就是利用程序本身漏洞改写保存在栈上函数的返回地址，从而使得函数返回时跳转到一个错误的地方，一种防护方法就是在原有的栈之外设立一个影子栈（Shadow Stack），入栈出栈操作在两个栈上同时进行，如果出栈的结果不一致，处理器抛出错误，阻断了ROP攻击。只不过还没怎么看过具体实现，感觉有时间可以看看......
  
+ 看了一些资料发现ASLR（Address Space Layout Randomization）技术也是可以被绕过的，loadtime、runtime等不同阶段都会有一些信息（code locator）可以被攻击者利用来猜出其他模块被加载到的地址，比如栈上会存储caller函数的地址，如果能够获得这个地址，的确是可以猜到一些信息，从而实施JOP这样的攻击。

+  看了一下musl malloc()的实现，主要还是通过brk()系统调用来获得内存，*cat /proc/${pid}/maps* 可以看到它所在的区域叫做Heap，第一次调用brk（0）的时候，获得当前heap地址。

+  **疑问1**:
   musl ldso里struct dso::deps的作用？


## 2023/07/03
+ 读了一下ld.so的Man Page，了解到`LD_DEBUG` & `LD_SHOW_AUXV`这两个环境变量的作用, musl和glibc的ld.so（动态连接器都是可以通过命令行启动）.

+ 知道了Kernel是怎样向动态链接器传递信息的(*Auxiliary Vector*),而Dynamic Linker可以通过esp寄存器的值（由内核设置）找到到Auxiliary Vector的起始地址，而这个Auxiliary Vector会包括一些只有Kernel知道并由Kernel填写信息，比如说主程序加载的位置（**AT_BASE**）


+ 看了一下musl libc的动态链接器最开始的汇编（*_dlstart*），做的事情很简单，也就是将*esp*寄存器的值和 *.DYNAMIC*段的地址作为参数传递（按照X86_64参数传递的*Calling Convention*）给 _dlstart_c函数(用C语言编写)，然后_dlstart_c开始后面的一系列操作.
  
+ 动态连接器一开始对自己进行重定位的过程很有意思.
  
+ **疑问1**:
  有一些动态库可能会遇到多级的依赖，比如说：a.out依赖于libfunc.so,而libfunc.so又依赖于libc.so,musl libc的动态链接器是怎样处理这种情况的?我在LD_DEBUG=symbols:binding 的情况下用了一下glibc的ldso,发现它好像只解析a.out到libfunc.so的符号引用,而libfunc.so到libc.so的引用似乎是通过一种Lazy Binding的方式在运行时解决?


+ **疑问2**:
  看了一下musl ldso的代码,对malloc & calloc等函数的处理很特别(外部实现对内部实现的替换?),感觉需要深入了解一下?


