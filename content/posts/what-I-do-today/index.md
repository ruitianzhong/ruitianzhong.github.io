---
title: "2023记录"
date: 2023-07-04T12:11:00+08:00
summary: "记录一下做了什么事情"
showtoc: false
categories:
- daily-record
---


## 2023/07/03
+ 读了一下ld.so的Man Page，了解到`LD_DEBUG` & `LD_SHOW_AUXV`这两个环境变量的作用, musl和glibc的ld.so（动态连接器都是可以通过命令行启动）.

+ 知道了Kernel是怎样向动态链接器传递信息的(*Auxiliary Vector*),而Dynamic Linker可以通过esp寄存器的值（由内核设置）找到到Auxiliary Vector的起始地址，而这个Auxiliary Vector会包括一些只有Kernel知道并由Kernel填写信息，比如说主程序加载的位置（**AT_BASE**）


+ 看了一下musl libc的动态链接器最开始的汇编（*_dlstart*），做的事情很简单，也就是将*esp*寄存器的值和 *.DYNAMIC*段的地址作为参数传递（按照X86_64参数传递的*Calling Convention*）给 _dlstart_c函数(用C语言编写)，然后_dlstart_c开始后面的一系列操作.
  
+ 动态连接器一开始对自己进行重定位的过程很有意思.
  
+ **疑问1**:
  有一些动态库可能会遇到多级的依赖，比如说：a.out依赖于libfunc.so,而libfunc.so又依赖于libc.so,musl libc的动态链接器是怎样处理这种情况的?我在LD_DEBUG=symbols:binding 的情况下用了一下glibc的ldso,发现它好像只解析a.out到libfunc.so的符号引用,而libfunc.so到libc.so的引用似乎是通过一种Lazy Binding的方式在运行时解决?


+ **疑问2**:
  看了一下musl ldso的代码,对malloc & calloc等函数的处理很特别(外部实现对内部实现的替换?),感觉需要深入了解一下?


