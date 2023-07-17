---
title: "2023记录"
date: 2023-07-04T12:11:00+08:00
summary: "记录一下做了什么事情"
showtoc: false
categories:
- daily-record
---

## 2023/07/17
### KVM/ARM(接着昨天的写)
+ 中断虚拟化：ARM中有一个GIC（Generic Interrupt Controller），而虚拟化的ARM架构提供了vGIC（virtual Generic Interrupt Controller），而GIC又分为CPU的Interface和Ditributor，中断的处理过程中包括了EOF和ACK等机制，而在arm中，这两个操作都可以由硬件完成。
+ IO虚拟化：主要使用了virtio，对MMIO的读写会导致陷入KVM当中
+ **疑问1**：对于多个不同的VM，KVM内部是怎样管理的？
+ split-mode virtualization：ARM专门为那种运行在bare metal上的Hypervisor（比如说Xen）设计了Hyp mode，但对于KVM这种位于操作系统内核中的VMM来说，需要经过很大的修改才能够使得Kernel的代码运行在Hyp mode中（Hyp mode 和 Kernel mode有不一致的地方），因此设计者提出了split-mode virtualization。简单来说，在Hyp mode中运行lowvisor，在kernel mode当中运行highvisor，当从VM陷入到KVM时，先进入lowvisor，做一些必要的工作，然后转到处于kernel mode的highvisor，完成大部分的工作，而返回时则按照相反的顺序进行。有意思的是，开发的时候有个问题比较有意思：
  
> An important issue in developing KVM/ARM was how to get access to Hyp mode across the plethora of available ARM SoC platforms supported by Linux.

如果在一开始初始化Hyp mode需要BIOS提供相关支持（install trap handler），否则kernel启动的时候可能会crash，这需要很多的外部支持。
而最终选择的方法是让内核在Hyp mode模式下启动，在启动过程中KVM/ARM检测当前是Kernel Mode还是Hyp Mode，并且有不同的应对方法。
+ 提到了Catch 22（第二十二条军规）这个问题：想要给内核贡献这样一个重要功能需要先成为Known Contributor，但成为known contributor又要先贡献代码。
因此，作者认为，先从小的地方贡献（比如说清理内核代码、使得代码更加通用）
  


## 2023/07/16

### KVM/ARM
+ IO
+ 中断
+ CPU
+ 内存
+ KVM/ARM 合并进入内核主线的过程让人印象深刻
+ split mode virtualization（适应ARM的相关设计）
+ Code Reuse
  

## 2023/07/15

### 缓存一致性
+ 多核下non-inclusive 的cache中的数据需要通过一定手段保持一致
+ LLC（Last Layer Cache）
+ 解决方法directory-based & snoop-based
+ MSI协议：Modified、Shared、Invalid 读取的时候如果是在Shared状态则直接读取、写之前要将其他核心的cache设置为Invalid，注意传输旧的缓存行（原因：缓存一致性以缓存行为为粒度，而一个cacheline大小一般为64个字节，一般来说只会修改其中一个部分，因此要传输旧的缓存行）

### 内存一致性模型
+ 严格一致性模型（Strict Consistency）、顺序一致性模型（Sequential Consistency）、TSO（Total Store Order）、弱序一致性模型（Weak-order Consistency）
+ 严格一致性模型认为两个操作不可能同时发生，处理器核看到的是全局时钟下的顺序
+ 顺序一致性模型：不同核心看到的操作一致（全局顺序），每个核心自己的读写操作可见顺序与程序顺序相同，可能出现一个核上的更新另一个核上的读操作无法读到。
+ TSO 不保证写读的全局可见顺序
+ 弱序一致性：写写、写读、读写、读读都不保证
+ 前提/研究对象：不同地址而且没有依赖（数据依赖、地址依赖）
+ 利用内存屏障来解决问题
+ ARM -> Weak-order Consistency  x86 -> TSO

### 同步原语的可扩展性
+ 当CPU核数比较多的时候，对同一个Cache Line的争抢导致在内存一致性协议上开销增大，从而出现核数增加反而性能下降的情况
+ 解决方法：MCS队列
+ NUMA下Cohert锁：全局锁+NUMA节点上的锁，全局锁直到本NUMA节点上的所有请求得到满足后释放，需要解决饥饿问题和公平性问题：限制单个NUMA节点的请求。（尽可能减少跨NUMA节点的迁移）


### Side Channel & Covert Channel
+ Flush + Reload
+ Flush + Flush
+ Prime + Probe
+ Evict + Reload
+ Meltdown
+ Spectre

### TEE
+ Intel SGX（Secure Guard eXtension）、Enclave concept
+ 应用、OS、虚拟机级别的隔离
+ 两大特征：隔离执行 + 远程认证
+ 操作系统、Hypervisor不可信的情况下的解决方案

### MISC
+ Linux Test Project（syscall测试）
+ kernelci（Fengguang Wu？）（硬件兼容性）
+ selftest
+ 软件兼容性（POSIX相关的测试）
+ 性能测试？


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


