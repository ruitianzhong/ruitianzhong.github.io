---
title: "2023记录"
date: 2023-07-04T12:11:00+08:00
summary: "记录一下做了什么事情"
showtoc: false
categories:
- daily-record
---


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


