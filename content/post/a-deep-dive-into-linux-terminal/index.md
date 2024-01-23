---
title: "What exactly happen after pressing Ctrl-C in Linux terminal？"
subtitle: "在`Linux`终端中按下Ctrl-C后具体发生了什么？"

summary: "在`Linux`终端中按下Ctrl-C后具体发生了什么？"

date: 2024-01-22T23:13:00+08:00

lastmod: 2024-01-23T13:11:00+08:00

draft: false

featured: true

authors:
  - admin

categories:
  - Linux
---


## 概览

在Web领域有一个比较常见的问题:浏览器地址栏输入网址后具体发生了什么？这个问题把网络、浏览器、HTTP、TCP等知识都给包含了，而且还能根据回答作进一步的追问。

那么，在Linux中有没有类似的问题？我构造了一个类似的问题："在`Linux`终端中按下Ctrl-C后具体发生了什么？"，并对这个问题做出了一定程度上的解答。在我看来，这个问题同样涉及到不少方面：Linux的`pgid(Process Group ID)`、`sid(Session ID)`、Shell、相关系统调用、进程等内容都需要有所了解。

本文先介绍了对这一问题的一种比较模糊的解答，然后再介绍理解这个问题需要的一些知识，再大致介绍按下Ctrl-C后发生的事情。最后，通过一些演示程序加深对其中一些细节的理解。


## 模糊的解答

运行下面的无限循环，linux终端就好像卡在那里，按键盘Ctrl+C后程序就可以退出，在`Linux`终端中按下Ctrl-C后具体发生了什么？
```bash
$ gcc foo.c && ./a.out
```

```c
int main(){
  while(1)
  ;
}
```

可以进行粗略地猜测，按下Ctrl-C这个瞬间，键盘应该向系统发起了一次中断的请求，操作系统识别到是Ctrl-C后将这个信息以某种方式传送到我们正在运行程序的进程，收到这个信息后，进程终止退出。

问题来了，操作系统怎么向进程传递这个信息的？如果懂一些Linux的信号机制（`signal(7)`），那么就知道OS通过将信号递送到进程来实现信息的传递。

也就是说大概操作系统将键盘中断转换为递送到进程的信号,如果进程没有注册相关的处理函数，就会在信号的作用下退出（中断->操作系统在进程相关的数据结构上标记信号->再次调度该进程时进入信号处理的相关流程）。

在研究这个问题时，我的认识也就停留在这种程度，但还是有疑问：按下Ctrl+C后，操作系统怎么知道向哪一个进程递送这样一个信号？比如说，在Linux中开了多个窗口，在某个窗口下按下Ctrl+C后，操作系统是怎样将相关信号递送到当前窗口下正在运行的程序？

## 前置知识


更详细的信息可以参考credentials(7)
### Process Group 和 Session
> Sessions  and  process groups are abstractions devised to support shell job control.

从文档中可以看出sessions和proces groups都是为了支持shell job control（作业控制）.

同一个进程组中的所有进程都有同一个process group id(`pgid`)，shell每次运行一条命令/一个程序的时候都会创建一个新的进程组（如何创建请看下文）。当进程的pid和pgid相等的时候，该进程就是进程组的领头进程（process group leader）。当一个信号发送给进程组的时候（通过kill等系统调用），所有进程组内的进程都会收到该信号。


> If a session has a controlling terminal, and the CLOCAL flag for that terminal is not set, and a terminal hangup occurs, then the session leader is sent a SIGHUP signal.If a process that is a session leader terminates, then a SIGHUP signal is sent to each process in the foreground process group of the controlling terminal. --setsid(2)

同一个Session中的进程具有相同的session ID。此外规定，一个进程组中的所有的进程都具有相同的session ID（如何保证这个要求请看下文），因此可以认为，session和process group有着1:n的关系，一个session可能对应多个process group，一个process group对应一个session group。

当进程的pid和进程所在session的sid相同时，该进程就是该session的领导（session leader）。如果一个session当中的session leader结束，那么所有和控制终端关联的前台进程组都会发送一个SIGHUP信号。有趣的是，可以从上面的文档中可以看出SIGHUP信号的一些历史渊源，它与（物理的）terminal卡住有关。

在不进行特别配置的情况下，向shell所在进程发送SIGUP也会使得包括在后台运行的进程退出，根据bash(1)文档，这是因为在退出的时候shell还会向所有作业发送SIGUP，这相当于bash在OS的基础上又多做了一些工作。

> The shell exits by default upon receipt of a SIGHUP.  Before exiting, an interactive shell resends  the  SIGHUP  to  all jobs,  running  or stopped.  Stopped jobs are sent SIGCONT to ensure that they receive the SIGHUP.  To prevent the shell from sending the signal to a particular job, it should be removed from the jobs table with the disown builtin (see SHELL BUILTIN COMMANDS below) or marked to not receive SIGHUP using disown -h.




### `pgid`和`sid`


> A child created by fork(2) inherits its parent's  session ID and process group ID.  A process's session ID and process group  ID  are  preserved  across  an  execve(2). Sessions  and  process groups are abstractions devised to support shell job control.   -- credentials(7)

pgid（Process Group ID）进程组ID用于区分不同的进程组，当调用`fork()`的时候，父进程和子进程的pgid相同.

foo.c:
```c
#include <sys/types.h>
#include <unistd.h>

int main() {
  int ret = fork();
  while (1)
    ;
}
```
运行：

```bash
./a.out & # running in background
ps -o comm,pgid,sid,pid,ppid
COMMAND            PGID     SID     PID    PPID
bash                519     519     519     518
a.out               867     519     867     519
a.out               867     519     868     867
ps                  870     519     870     519

```
这里a.out中执行了一次fork，a.out对应的pgid都是867，注意到a.out的pgid和bash(也就是Shell)对应的pgid不一样,也印证了上文提到的"shell每次运行一条命令/一个程序的时候都会创建一个新的进程组".


#### 如何改变`pgid`

有一个系统调用`int setpgid(pid_t pid,pid_t pgid)`可以改变setpgid这个要求，该调用将进程号为pid的进程的进程组号修改为pgid,
如果pid等于零，则默认将调用该函数的进程的进程组号改为pgid。

该调用将进程从一个进程组移动到另外一个当中，需要满足两个进程组在同一个session当中，这样限制以后满足了同一个进程组的进程必定在同一个session当中这个之前提到的*约束条件*。

`setpgid()`示例程序(这有助于理解bash在新进程启动一个程序之前是怎么样改变它的pgid)：
```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>
int main() {
  int ret = fork();
  if (ret == 0) {
    printf("child process pid:%d pgid:%d\n", getpid(), getpgid(0));
    if (setpgid(0, 0)) {
      perror("setpgid");
      exit(EXIT_FAILURE);
    }
    printf("child process pgid:%d\n", getpgid(0));

    exit(EXIT_SUCCESS);

  } else {
    printf("parent process pid:%d pgid:%d\n", getpid(), getpgid(0));
    exit(EXIT_SUCCESS);
  }
}
```
程序输出：
```bash
$ ./a.out
parent process pid:1262 pgid:1262
child process pid:1263 pgid:1262
child process pgid:1263
```

#### 如何改变`sid`

通过系统调用`pid_t setsid(void)`可以改变当前进程的session id，
它要求当前进程不是所在进程组的leader（group leader process），它会使得该进程位于一个新的进程组当中（进程组号为该进程的pid），同时也把它移动到一个新的session当中（session id为该进程的pid）。最后该session初始时不与任何控制终端相关联（controlling terminal，下文会介绍）。

`setsid()`示例程序

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>
int main() {
  int ret = 0;

  ret = setsid();
  if (ret) {
    puts("expected to fail");
  }

  ret = fork();
  if (ret) {
    exit(EXIT_SUCCESS);
  } else {
    printf("sid:%d pid:%d pgid:%d before setsid\n", getsid(0), getpid(),
           getpgid(0));
    ret = setsid();
    if (ret == -1) {
      perror("setsid");
      exit(EXIT_FAILURE);
    }
    printf("sid:%d pid:%d pgid:%d after setsid\n", getsid(0), getpid(),
           getpgid(0));
    exit(EXIT_SUCCESS);
  }
}
```
输出结果
```bash
$ ./a.out
expected to fail
sid:969 pid:2073 pgid:2072 before setsid
sid:2073 pid:2073 pgid:2073 after setsid
```


### 终端(Terminal) / `tty`

很久以前，tty是有物理存在的，可以参考知乎上的[看得见摸得着的TTY——电传打字机](https://zhuanlan.zhihu.com/p/108206742)

到了现代，已经没有物理上的tty了，往往是软件模拟出来的tty（和图形显示配合），如下图。为了降低理解的复杂性，作为tty的用户，我们仍然可以将tty看作是物理设备，而不用去管模拟的实现。如果要深究其中的原理，pts(4)是一个比较好的起点。

![Pseudo terminal](pts.png)

* 查看当前tty的命令

```bash
$ tty
/dev/pts/2
```


```c
// verified on Ubuntu 22.04 for Windows Subsystem Linux(WSL)
// root privilege is needed
#include <fcntl.h>
#include <signal.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/ioctl.h>
#include <sys/types.h>
#include <unistd.h>
// SIGINT handler
void sigint_handler(int sig) {
  printf("receive SIGINT\n");
  exit(EXIT_SUCCESS);
}

int main() {
  // ensure process calling setsid() is not the process group leader
  int ret = fork();
  if (ret == -1) {
    perror("fork");
    exit(EXIT_FAILURE);
  }
  if (ret != 0) {
    // parent process exits
    exit(EXIT_SUCCESS);
  }
  struct sigaction act;
  act.sa_sigaction = NULL;
  act.sa_flags = 0;
  act.sa_handler = sigint_handler;
  sigemptyset(&act.sa_mask);
  // register signal handler
  ret = sigaction(SIGINT, &act, NULL);
  if (ret) {
    perror("sigaction");
    exit(EXIT_FAILURE);
  }

  printf("old session id:%d parent pid:%d\n", getsid(0), getppid());

  pid_t new_session_id = setsid();
  if (new_session_id == -1) {
    perror("setsid");
    exit(EXIT_FAILURE);
  }
  printf("new session id:%d\n", new_session_id);

  printf("process id:%d\n", getpid());

  int buf[100];
  char terminal[] = "/dev/pts/3";
  int fd = open(terminal, O_RDONLY);

  if (fd == -1) {
    perror("open");
    exit(EXIT_FAILURE);
  }
  // More details in ioctl_tty(2) in Man page.
  // set the controlling terminal(already existed) of calling process() which is
  // stolen from other process caller should have the CAP_SYS_ADMIN capability
  ret = ioctl(fd, TIOCSCTTY, 1);
  if (ret != 0) {
    perror("ioctl");
    exit(EXIT_FAILURE);
  }

  puts("successfully get the controlling terminal\n");
  pid_t p = getpid();
  // set the foreground process for the controlling terminal so that signal from
  // the tty/pts can be received.
  ret = ioctl(fd, TIOCSPGRP, &p);
  if (ret != 0) {
    perror("ioctl");
    exit(EXIT_FAILURE);
  }
  // without changing file descriptor table, `stdout` is still associated with
  // current terminal
  printf("Now in %s\n", terminal);
  // pressing ctrl-c in terminal associated with /dev/pts/3 will result in the
  // end of current process.
  //  "receive SIGINT" is printed in current terminal
  while (1) {
    sleep(1);
    printf("continue...\n");
  }

  return 0;
}
```
## Resources and References
