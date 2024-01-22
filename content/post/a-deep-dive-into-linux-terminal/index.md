---
title: "What exactly happen after pressing Ctrl-C in Linux terminal？"
subtitle: "在`Linux`终端中按下Ctrl-C后具体发生了什么？"

summary: "在`Linux`终端中按下Ctrl-C后具体发生了什么？"

date: 2024-01-22T23:13:00+08:00

lastmod: 2024-01-22T23:13:00+08:00

draft: false

featured: false

authors:
  - admin

categories:
  - Linux
---


```c
#include <fcntl.h>
#include <signal.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/ioctl.h>
#include <sys/types.h>
#include <unistd.h>
void sigint_handler(int sig) {
  printf("receive SIGINT\n");
  exit(EXIT_SUCCESS);
}

int main() {

  int ret = fork();
  if (ret == -1) {
    perror("fork");
    exit(EXIT_FAILURE);
  }
  if (ret != 0) {
    exit(EXIT_SUCCESS);
  }
  struct sigaction act;
  act.sa_sigaction = NULL;
  act.sa_flags = 0;
  act.sa_handler = sigint_handler;
  sigemptyset(&act.sa_mask);

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

  ret = ioctl(fd, TIOCSCTTY, 1);
  if (ret != 0) {
    perror("ioctl");
    exit(EXIT_FAILURE);
  }

  puts("successfully get the controlling terminal\n");
  pid_t p = getpid();
  ret = ioctl(fd, TIOCSPGRP, &p);
  if (ret != 0) {
    perror("ioctl");
    exit(EXIT_FAILURE);
  }
  printf("Now in %s\n", terminal);

  while (1) {
    sleep(1);
    printf("continue...\n");
  }

  return 0;
}
```