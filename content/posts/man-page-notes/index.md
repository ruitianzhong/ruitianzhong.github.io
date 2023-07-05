---
title: "Man Page Notes"
date: 2023-07-03T09:52:00+08:00
showtoc: false
summary: "Messy notes on man page"
categories:
- Linux
tags: 
- Docs Reading
---
*Last Modified :  2023-07-05*

# Motivation

Just to record some takeaways when reading man page

## vDSO

```bash
man vdso
```
Why does the vDSO exist at all?  There are some system calls the  kernel provides that user-space code ends up using frequently, to the point that such calls can  dominate  overall performance.   This is due both to the frequency of the call as well as the context-switch overhead that results from exiting user space and entering the kernel.


Note that the terminology can be confusing.  On x86 systems,the vDSO function used to determine the preferred method of making a system call is named  "__kernel_vsyscall",  but  on x86-64,  the  term "vsyscall" also refers to an obsolete way to ask the kernel what time it is or what CPU the caller is on.


The  base  address  of the vDSO (if one exists) is passed by the kernel to each program in the initial  auxiliary  vector(see getauxval(3)), via the AT_SYSINFO_EHDR tag.


### Finding the vDSO
You must not assume the vDSO is mapped at any particular location in the user's memory map. The base address will usually  be randomized at run time every time a new process image is created (at execve(2) time).  This is done for  security reasons, to prevent "return-to-libc" attacks.

### File Format
* Fully formed ELF image
* All symbols are versioned(using the GNU version format)
* Naming convention:the "gettimeofday" function is named "__vdso_gettimeofday"

### Miscellaneous
+ For x86_64, linux-vdso.so.1

+  System calls exported by vDSO will not appear in the trace output when using **strace** and will not be visible to seccomp filters

+ x86_64 & x86/x32 exported symbols(version: LINUX_2.6): 

  __vdso_clock_gettime,

  __vdso_getcpu,

  __vdso_gettimeofday,

  __vdso_time

+ i386 exported symbols:

  __kernel_sigreturn LINUX_2.5

  __kernel_rt_sigreturn LINUX_2.5

  __kernel_vsyscall LINUX_2.5

  __vdso_clock_gettime LINUX_2.6

  __vdso_gettimeofday LINUX_2.6
 
  __vdso_time LINUX_2.6

+ riscv functions(version:LINUX_4.15):

  __kernel_rt_sigreturn,

  __kernel_gettimeofday,

  __kernel_getcpu,

  __kernel_flush_icache,

  __kernel_clock_getres,

  __kernel_clock_gettime

## getauxval(3)
   
   TBD...




## brk & sbrk

+ *brk()* & *sbrk()* change the location of the program break.

+ *brk()* set the end of the data segment to the value specified by addr.

+ *sbrk()* increments the program's data space by increment bytes. sbrk(0) gets the current location of the program break.

+ *malloc(3)* memory allocation is the portable and comfortable way of allocating memory.

### code snippet
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
int main(){
       void * p1;
       p1 = sbrk(0);
       if(p1 == (void*)-1){
               perror("sbrk");
               exit(EXIT_FAILURE);
       }
       printf("current break : %p\n",p1);
       char * p2 = malloc(4096);
       if(p2 == 0) {
               perror("malloc");
               exit(EXIT_FAILURE);
       }
       p1 = sbrk(0);
       if(p1 == (void*)-1){
               perror("sbrk");
               exit(EXIT_FAILURE);
       }
      printf("current break : %p\n",p1);
      if (brk(p1+4096)){
                perror("brk");
                exit(EXIT_FAILURE);
      }
      printf("current break : %p\n",sbrk(0));
      exit(EXIT_SUCCESS);
}

```