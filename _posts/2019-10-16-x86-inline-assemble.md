---
layout: post
title: x86 inline assemble Core dumped in Linux-5.0
categories: [c, gdb, assemble]
description: a fix to the inline assemble of x86 multilib gcc
keywords: c, inline assemble, gdb

---

# Linux-5.x 内核下 x86 内联汇编 Core Dumped 问题

Written by Tianyu

在做《庖丁解牛Linux内核分析》第 4.3 节当中，有关于 c++ 内联汇编的代码，同时这段汇编代码要求用 `gcc -m32` 来编译，即跨平台编译，用这种方式编译的可执行文件，运行的结果是 `core dumped`。代码如下所示

```c
#include <stdio.h>
#include <time.h>

int main() {
    time_t tt; 
    struct tm *t; 
    // tt = time(NULL);
    __asm__ __volatile__ (
            "mov $0,%%ebx\n\t"
            "mov $0xd,%%eax\n\t"
            "int $0x80\n\t"
            "mov %%eax,%0\n\t"
            :"=m"(tt)
            :
            );
    t = localtime(&tt);
    printf("time: %d : %d : %d : %d : %d : %d\n", t->tm_year, t->tm_mon, t->tm_mday, t->tm_hour, t->tm_min, t->tm_sec);
    return 0;
}

```

逻辑很简单，获取系统时间，调用 `localtime()` 获取格式化的时间信息，然后输出。代码中利用 `__asm __volatile__ ()` 来实现内联汇编，这段汇编的作用如它上方的那段注释所示，就是调用第 13 号系统调用，并将返回值放到 `tt` 变量当中。

然而编译并运行这段代码的结果如下

```shell
# 在运行 gcc -m32 之前需要确保安装了 gcc-multilib
$ gcc -m32 -o time-asm time-asm.c
$ ./time-asm 
Segmentation fault (core dumped)
```



### 问题描述

