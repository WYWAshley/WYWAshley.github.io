---
layout: post
title: x86 inline assemble Core dumped in Linux-5.0
categories: [c, gdb, assemble]
description: a fix to the inline assemble of x86 multilib gcc
keywords: c, inline assemble, gdb

---

## x86 内联汇编 Core Dumped

Written by Tianyu

在做《庖丁解牛Linux内核分析》[^note1]第 4.3 节当中，有关于 c++ 内联汇编的代码，同时这段汇编代码要求用 `gcc -m32` 来编译，即跨平台编译，用这种方式编译的可执行文件，运行的结果是 `core dumped`。代码如下所示

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

首先是系统的一些信息

* Operating System：Ubuntu 18.04.1
* Kernel Version: Linux 5.0.0-31-generic
* Machine: x86_64

然后是遇到问题的具体描述

> 在进行 c 语言内联汇编的时候，用 `gcc -m32` 进行跨平台编译，得到的可执行文件无法运行，会得到 `Segmentation fault (core dumped)`



### 解决思路

我和我的同事 [Simon](https://github.com/SimonSungm) 探讨了这个问题，他碰到过类似的情况，那个是在操作系统实验当中，由于 gcc 的版本问题，导致编译出的可执行文件无法正常运行。



于是我查看了 gcc 的版本

```shell
$ gcc -v
gcc version 8.3.0
```

已经是最新的版本了。



接着我考虑用 gdb 去调试一下程序，看看到底是在哪一步出错了。

```shell
$ gcc -m32 -g -o time-asm time-asm.c
$ gdb time-asm
```



> 在这里可能会遇到一个问题，就是如果 gdb 和 gcc 的版本不匹配，那么会导致代码连 main() 函数都无法运行到。

**对于这种情况，请读者自行配置匹配的 gcc, g++, gdb。我配置的是 gcc/g++ 8.3.0, gdb 8.3。**



接下来进行调试

```shell
$ gcc -m32 -g -o time-asm time-asm.c 
$ gdb time-asm 
GNU gdb (GDB) 8.3
......
Reading symbols from time-asm...
(gdb) b main
Breakpoint 1 at 0x5c8: file time-asm.c, line 4.
(gdb) r
Starting program: /home/zty/dev/cfile/time-asm 

Breakpoint 1, main () at time-asm.c:4
4	int main() {
(gdb) step
8	    __asm__ __volatile__ (
(gdb) step
16	    t = localtime(&tt);
(gdb) step

Program received signal SIGSEGV, Segmentation fault.
0x56555440 in localtime@plt ()
```

从调试结果可以看到，在第 16 行代码处发生了 Segmentation Fault。这说明，**内联汇编代码的执行并没有问题**。



那么，问题就集中在为什么 `localtime(&tt)` 这个函数调用会 Crash，我于是将上面一段代码的内联汇编部分改写成了正常的 API 调用 `tt = time(NULL)`，结果当然是可以运行的。我关心的是这两个文件（仅进行 `time()` 系统调用的手段不一样）的汇编代码有何不同。



使用 `gcc -S` 命令可以生成汇编文件，查看两个汇编文件的代码之后，发现的不同如下所示

```assembly
	// time.s				|	// time-asm.s
	subl	$12, %esp		|	mov $0,%ebx
	pushl	$0				|	mov $0xd,%eax
	call	time@PLT		|	int $0x80
	addl	$16, %esp		|
```

左边就是开辟新的 stack 空间，然后把 0 作为参数传入 `time()` 函数，最后再把 stack 顶部指针 %esp 归为原位。



右边的就是把 0 赋值给 %ebx 寄存器，然后把系统调用号 13 传给 %eax，最后触发中断向量 0x80。



逻辑上都没有太大的问题，我也看不出有什么不一样的地方，而且两边执行完之后，系统调用的返回值都应该放在 %eax 寄存器里面，那么为了确定之后执行的 `localtime()` 函数传入的是正确的值，我们再次用 gdb 调试一下。

```assembly
7	    tt = time(NULL);
(gdb) info registers 
eax            0x0                 0
ecx            0xffffce80          -12672
edx            0xffffcea4          -12636
ebx            0x56556fcc          1448439756
eip            0x565555f3          0x565555f3 <main+42>
(gdb) step
8	    t = localtime(&tt);
(gdb) info registers 
eax            0x5da8477d          1571309437
ecx            0x5da8477d          1571309437
edx            0x0                 0
ebx            0x56556fcc          1448439756
eip            0x56555603          0x56555603 <main+58>
```

上述是 **time.c** 编译的可执行文件在执行 `tt=time(NULL)` 前后的寄存器对比，执行前后，存储内容发生变化的有 eax，ecx，edx，还有 eip。其中 eip 为指令地址寄存器，在这里不需要关注。

```assembly
7	    __asm__ __volatile__ (
(gdb) info registers 
eax            0x0                 0
ecx            0xffffce90          -12656
edx            0xffffceb4          -12620
ebx            0x56556fd0          1448439760
edi            0x0                 0
eip            0x565555d3          0x565555d3 <main+42>
(gdb) step
15	    t = localtime(&tt);
(gdb) info registers 
eax            0x5da847b2          1571309490
ecx            0xffffce90          -12656
edx            0xffffceb4          -12620
ebx            0x0                 0
eip            0x565555e2          0x565555e2 <main+57>
eflags         0x246               [ PF ZF IF ]

```

上述是 `time-asm.c` 编译的可执行文件在执行内联汇编前后的寄存器对比，执行前后，存储内容发生变化的是 eax，ebx，eip。



根据逻辑判断，eax 记录的是函数的返回值，确实应该改变。那么有什么是**不应该改变的寄存器值**呢？我猜测可能是因为 ebx 值发生改变，导致之后的函数调用出错。



### 真相大白

通过 gdb 工具改变 ebx 寄存器的值

```assembly
(gdb) set $ebx=0x56556fd0
(gdb) step
16	    printf("time: %d : %d : %d : %d : %d : %d\n", t->tm_year, t->tm_mon, t->tm_mday, t->tm_hour, t->tm_min, t->tm_sec);
(gdb) step
time: 2019 : 9 : 17 : 18 : 51 : 30
17	    return 0;
```

成功的调用了 `localtime()` 函数并打印时间。



因此问题就是出在 `%ebx` 寄存器的值发生改变，和原先的不一致。那么为什么会有这种情况呢？为什么实验服务器 Linux 4.15 的 kernel 下可以正常运行，而我本地的 Linux 5.0 kernel 不能够运行？



带着这个疑问我咨询了导师[申文博](http://wenboshen.org/)教授，他指出**内联汇编并不是标准的 c 语言写法，一般汇编语言会单独写成一个文件，若是一定要写内联汇编，那就要严格的按照相关的标准编写。**



这是因为编译器有自己的编译规则和优化手段，我们在内联汇编当中使用的寄存器，很有可能会被后续的 gcc 汇编代码所用到，那么如果改变了这个寄存器的值（如本次实验当中的 ebx 寄存器），则会导致意想不到的错误（如 Segmentation Faults 等）。一般在 gcc 的库当中，如果是用汇编语言写的，一定会有对于寄存器的值的保存，利用 push 和 pop 的方法，在 **Caller** 和 **Callee** 两端都对寄存器的内容进行保护，所以内联汇编不建议使用。



### 总结

c 语言的内联汇编应该有非常严格的标准，为了操作方便而随意使用寄存器的做法是非常不可取的，套用申老师的话来说，“在 Linux 4.15 上能够成功运行是撞了大运”，同样一个二进制文件在不同的 kernel 版本上不能运行，从结果而言当然是不尽人意的。



对于通用寄存器的保护（eax，ebx，ecx，edx）要和栈指针 ebp 一样重视，同时如果以后遇到了相似的问题，也可以参考本文的方法去解决。



### 参考

[^note1]: Here is the *text* of the **footnote**.





































[^footnote]: 