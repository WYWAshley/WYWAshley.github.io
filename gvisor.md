# gVisor

Written by Tianyu from ZJU.

[TOC]

## gVisor Setup

### docker 安装

Docker的安装主要参照Docker官网的教程

* [Ubuntu](<https://docs.docker.com/install/linux/docker-ce/ubuntu/>)
* [CentOS](<https://docs.docker.com/install/linux/docker-ce/centos/>)
* [MacOS](<https://docs.docker.com/docker-for-mac/install/>)
* [Windows](<https://docs.docker.com/docker-for-windows/>)

**注意：可能在安装的过程当中需要翻墙**

在安装完成之后，运行

```shell
$ sudo docker run hello-world
```

即可看到hello-world image在docker当中run的结果。

#### 安装须知

##### 用户权限

通过运行以下语句，可以将当前的用户添加进可以运行**docker**的用户当中。

```shell
$ sudo groupadd docker
$ sudo gpasswd -a ${USER} docker
$ sudo systemctl restart docker
$ reboot
```

##### 国内镜像设置

参照[docker构建国内镜像服务](<https://blog.csdn.net/xxb249/article/details/79469534>)

笔者采用的是阿里云。如果翻墙速度较快的话，用**docker**官方的仓库也行。

#### 学习资料

* [Docker——从入门到实践](<https://yeasy.gitbooks.io/docker_practice/>)
* [Docker教程](<https://www.w3cschool.cn/docker/>)
* [Learn Docker & Containers using Interactive Browser-Based Scenarios](<https://www.katacoda.com/courses/docker>)

### gVisor 安装

本文采用的是**gVisor**在[**Github**的安装教程](<https://github.com/google/gvisor>)来安装的。如果**不采用编译源码的方式**，可以按照[官方Doc的方法](https://gvisor.dev/docs/user_guide/docker/)下载安装。

主要的安装步骤如下

#### Download gVisor

```shell
$ git clone https://gvisor.googlesource.com/gvisor gvisor
$ cd gvisor
```

#### Install bazel

具体参照[Bazel官方文档](<https://docs.bazel.build/versions/master/install-ubuntu.html>)

#### Build gVisor

```shell
$ bazel build runsc
$ sudo cp ./bazel-bin/runsc/linux_amd64_pure_stripped/runsc /usr/local/bin
```

#### 关于Docker的设置

参照[Configuring Docker](https://gvisor.dev/docs/user_guide/docker/#configuring-docker)

将runsc的目录添加到Docker的配置文件当中

#### 学习资料

目前国内关于gVisor的学习资料相对匮乏，所以主要的请参照[gVisor官网文档](<https://gvisor.dev/docs/>)和[gVisor源码](<https://github.com/google/gvisor>)。

#### gVisor设置platform为kvm
安装依赖(Ubuntu)
```shell
$ sudo apt-get install qemu-kvm
```
修改/etc/docker/daemon.json文件，增加参数
```json
{
    "runtimes": {
        "runsc-kvm": {
            "path": "/usr/local/bin/runsc",
            "runtimeArgs": [
                "--platform=kvm"
            ]
       }
    }
}
```
重启docker
```shell
$ sudo systemctl restart docker
```

使用gvisor在docker中运行hello-world

```shell
$ docker run runtime=runsc-kvm hello-world
```

正常打出hello world等一系列信息则说明设置gVisor成功。

#### 设置Debug参数

若要启用调试和系统调用的日志记录，将下面的runtimeArgs添加到Docker配置(`/etc/docker/daemon.json`)

```json
{
    "runtimes": {
        "runsc-kvm": {
            "path": "/usr/local/bin/runsc",
            "runtimeArgs": [
                "--platform=kvm",
                "--debug-log=/tmp/runsc/",
                "--debug",
                "--strace"
            ]
       }
    }
}
```

更多的信息请见官方关于[gVisor Debugging的文档](https://gvisor.dev/docs/user_guide/debugging/)。



## gVisor Overview

> gVisor是一个用Go编写的user-space kernel，它实现了很大一部分的Linux系统调用接口。 它在运行的应用程序和主机操作系统之间提供了额外的隔离层。

> gVisor包含一个名为 `runsc` 的Open Container Initiative（OCI）runtime，可以轻松使用现有的容器工具。`runsc` runtime与Docker和Kubernetes集成，使得运行沙盒化(sandboxed)容器变得简单。

> 与现有的sandbox技术相比，gVisor采用了独特的容器沙盒化(sandboxing)方法，并进行了一系列不同的技术权衡，从而为**容器安全**领域提供了新的工具和思路。

更多介绍参考[官方Doc](https://gvisor.dev/docs/)。还有相关的架构(Architecture)介绍[参考此处](https://gvisor.dev/docs/architecture_guide/)。

### Sandbox

gVisor 当中有许多重要的概念，本文不能一一尽述，就介绍几个相关的最为重要的部分。

[gVisor sandbox](https://gvisor.dev/docs/architecture_guide/overview/) 在运行的时候包含了**许多进程**，这些进程共同组成一个共享环境，在其中可以运行**一个或多个**容器。

每个 sandbox 都有其独立的实例(instance)

* **Sentry**，运行容器的 **user space kernel**，负责直接和 APP 交互，它**拦截并响应**来自于 APP 的系统调用
* **Gofer**，为容器提供文件系统的访问

具体的结构如下图所示

![](Sentry-Gofer.png)

本文之后的内容会重点放在 APP 与 Sentry 之间的 system calls 以及 gVisor sandbox 和 Host Kernel 之间的 system calls 上。

### Sentry

[Sentry](https://gvisor.dev/docs/architecture_guide/overview/) 是 gVisor 的最大的组件。 它可以被认为是一个 user-space OS kernel。 Sentry 实现了不受信任的应用程序(APP)所需的**所有内核功能**。 它实现了**所有**受支持的<u>系统调用</u>，<u>信号传递</u>，<u>内存管理</u>和<u>页面错误逻辑</u>，<u>线程模型</u>等。

当不受信任的应用程序(APP)进行**系统调用**时，当前使用的[平台(Platform)](https://gvisor.dev/docs/architecture_guide/overview/#platforms)会将**调用重定向**到 Sentry，后者将执行必要的工作来为其提供服务。 值得注意的是，Sentry **不会简单地将系统调用传递给主机内核**。 作为用户空间应用程序，Sentry将进行一些主机系统调用以支持其操作，但它不允许应用程序直接控制它所进行的系统调用。事实上，在 kvm platform 下，sentry 不会把未实现的 system call 传递给主机内核，而是[简单的报错](https://groups.google.com/d/msg/gvisor-users/B8mkgwiJFOY/jU5BR-OsBAAJ)。

超出sandbox（不是内部/proc文件，管道等）的文件系统操作将发送到[Gofer](https://gvisor.dev/docs/architecture_guide/overview/#gofer)。



## gVisor system calls

gVisor 的系统调用分成两个部分来讲述，第一部分 System Call Implementation 介绍了 Linux 如何实现系统调用，以及 gVisor 是如何仿照 Linux 来做的。第二部分则利用 gVisor 设计实现系统调用的原理，自己设计并添加了一个新的系统调用，借此来熟悉 gVisor 具体的系统调用流程。

### System Call Implementation

#### Linux Implementation

首先需要搞清楚的一点是，Linux 是怎么实现系统调用的。[Anatomy of a system call, part 1](https://lwn.net/Articles/604287/) 介绍了Linux实现系统调用的具体流程。

系统调用和普通的函数不同，被调用的代码放在内核里面。当 APP 进行系统调用的时候，需要有**特定的指令**，使得处理器进行**ring 0**(特权模式)的转换。同时系统调用不是由地址来标识，而是**由系统调用号来标识**的。

上文提到的那篇文章，以 `read()` 系统调用为例详细介绍了 Linux kernel 的的系统调用流程。这里有个重点需要指出，`system_call` 系统调用处理函数的地址写入寄存器 **MSR_LSTAR (0xc0000082)**，这是用于处理SYSCALL指令的x86_64特定于模型的寄存器，不同的硬件设备是不一样的。

系统调用的简要流程如下

user-space 的用户进程将系统调用号放入寄存器 `RAX`，然后其他参数放到具体的寄存器，之后运行 `SYSCALL` 指令。硬件完成切换 Ring 0，调用 MSR_LSTAR 寄存器指向地址的代码 `system_call`，寄存器值(之前的参数值)放入kernel Stack，根据 `RAX` 获取函数指针，然后最后调用具体的系统调用实现函数。这写过程都是由硬件完成的，而 `system_call` 的地址是在系统调用初始化的时候就放入寄存器当中的。

#### gVisor Implementation

根据 [gVisor 论坛当中的帖子](https://groups.google.com/d/msg/gvisor-users/15FfcCilupo/7ih35uPHBAAJ)，在 KVM 平台下，系统调用的拦截和普通的OS类似。当在 guest mode 运行程序的时候，KVM 把上文提到的 `MSR_LSTAR` 寄存器设置为一个 [**system call handler**](https://github.com/google/gvisor/blob/master/pkg/sentry/platform/ring0/entry_amd64.go#L23-L32)，每次程序发生系统调用或者是 sentry 自己进行系统调用的时候，这个 `sysenter()` 函数都会执行。详细的在下一节当中有介绍。

### Add a System Call To gVisor

以下实验请在 `release-20190806.1` 版本上实践，不同的 gVisor 版本对于 syscall 的实现方式会有偏差。

#### 添加 syscall 的主要步骤

参考[我在gvisor论坛上面的Topic](https://groups.google.com/d/msg/gvisor-users/-_IgtpPHiic/LoqmMtAuBQAJ)，在gVisor中添加一个系统调用主要分三步

- 在[`pkg/sentry/syscalls/linux/linux64.go`](https://github.com/google/gvisor/blob/fb996668e40031671e08d107e1a5307e813215f9/pkg/sentry/syscalls/linux/linux64.go#L48)文件当中的`var AMD64`变量当中添加调用函数名和系统调用号
- 在`pkg/sentry/syscalls/linux`目录下添加`×××.go`文件，在这个文件里定义调用函数
- 在[`pkg/sentry/syscalls/linux/BUILD`](https://github.com/google/gvisor/blob/fb996668e40031671e08d107e1a5307e813215f9/pkg/sentry/syscalls/linux/BUILD#L7)文件当中添加构建项

以上三步完成之后，重新编译gvisor就可以在Application当中进行系统调用(按照系统调用号)，调用我们刚刚定义的函数。

#### gVisor 关于 redirect syscall 的原理

[Anatomy of a system call, part 1](https://lwn.net/Articles/604287/) 介绍了Linux实现系统调用的具体流程。

和Linux实现系统调用类似，gVisor 也存在着这样的机制。

gVisor 在 sentry 当中实现系统调用，具体的架构如下(来自于[申文博老师的runc, gvisor, and kata container](https://wenboshen.org/posts/2018-12-01-sectainer.html))

```
          Application     --> Guest Ring 3
         ----------------
          Sentry (ring0)  --> Guest Ring 0
GUEST
-------------------------------------------
HOST
          Sentry (kvm)    --> Host Ring 3
         -------------
          Host kernel     --> Host Ring 0
```

**Sentry** 在初始化的时候，将系统调用函数 [`sysenter()`](https://github.com/google/gvisor/blob/fb996668e40031671e08d107e1a5307e813215f9/pkg/sentry/platform/ring0/kernel_amd64.go#L243) 放入 **MSR_LSTAR** 寄存器当中，当 **Application** 发生系统调用的时候，机器会直接陷入 trap，然后自动调用存放在 **MSR_LSTAR** 寄存器当中的函数，即 `sysenter()`。

每次 **Application** 发生系统调用或者是 **sentry** 自己进行系统调用的时候，这个 `sysenter()` 函数都会执行。这个函数是用汇编代码来写的，主要做的事情就是在 **context switch** 的过程当中保存寄存器的值，然后根据 **kernel space** 里面的 **syscall table** 来进行跳转到具体的系统调用实现函数中去。

**sentry** 当中的 **kernel space** 其实在 **Host** 看来是 **user space** 当中分配给 **sentry** 的其中一部分。

当添加一个系统调用的时候，需要做的就是编写调用函数，同时将系统调用函数名和调用号都添加到 **syscall table** 当中去。

#### 添加调用函数名和调用号

在[`pkg/sentry/syscalls/linux/linux64.go`](https://github.com/google/gvisor/blob/fb996668e40031671e08d107e1a5307e813215f9/pkg/sentry/syscalls/linux/linux64.go#L48)文件当中的`var AMD64`变量当中添加调用函数名和系统调用号。

上文讲到的 **syscall table** 定义在 `var AMD64` 变量当中的 `Table` 属性下。

```go
var AMD64 = &kernel.SyscallTable{
    // ...
    Table: map[uintptr]kernel.SyscallFn{
        0:  Read,
        1:  Write,
        // ...
        400: Square,
    }
    // ...
}
```

添加新的系统调用的时候，形式应该和上述的调用一样，以`调用号: 调用函数名`的形式，如上述代码中的`400: Square`。

这一步相当于在调用表当中注册了新的调用函数。接下来就是定义这个新添加的调用函数。

#### 定义调用函数

定义调用函数的目录是确定的，在 `pkg/sentry/syscalls/linux`。

在该目录新建一个`.go`文件，这里以之前添加的 `Square` 调用函数为例，在目录下新建一个名为`sys_square.go`的文件

```go
package linux

import (
    "gvisor.googlesource.com/gvisor/pkg/sentry/arch"
    "gvisor.googlesource.com/gvisor/pkg/sentry/kernel"
)

// Square returns the square of arg[0]
func Square(t *kernel.Task, args arch.SyscallArguments) (uintptr, *kernel.SyscallControl, error) {
    num := args[0].Int()
    num = num * num
    return uintptr(num), nil, nil
}
```

这里的`Square()`函数接受一个参数并返回它的平方。

注意这里的 **package** 和 **import** 部分必须一致。函数名要和之前注册的相对应。

#### 添加构建项

打开[`pkg/sentry/syscalls/linux/BUILD`](https://github.com/google/gvisor/blob/fb996668e40031671e08d107e1a5307e813215f9/pkg/sentry/syscalls/linux/BUILD#L7)文件，添加新建的`sys_square.go`文件。

```
package(licenses = ["notice"])
load("//tools/go_stateify:defs.bzl", "go_library")
go_library(
    name = "linux",
    srcs = [
        "error.go",
        "flags.go",
        "linux64.go",
        "timespec.go",
        ...
        "sys_square.go"     <-------------- new
    ],
...
```

重新 **build** gVisor。

#### 测试

参考[这篇文章](https://www.fengbohello.top/archives/go-env-docker)，使用 **docker** 来构建 **go** 语言环境，生成可执行文件。

```go
// test-square.go
package main
import (
    "fmt"
    "os"
    "syscall"
)

const SQUARE = 400 
func main() {
    r1, _, errNo := syscall.RawSyscall(SQUARE, 5, 0, 0)
    if errNo != 0 { 
        fmt.Fprintf(os.Stderr, "ERROR: %d\n", errNo)
        os.Exit(1)
    }
    fmt.Printf("%d\n", r1) 
}      
```

这个文件通过`syscall.RawSyscall(SQUARE, 5, 0, 0)`利用系统调用号来进行系统调用，这里用到的参数仅仅为第一个`5`。

将该程序编译之后得到可执行文件`test-square`，在 **gVisor** 当中运行它，得到如下输出

```shell
root@91071194a4c4:/code# ls
test-square  test-square.go
root@91071194a4c4:/code# ./test-square 
25
```

输出为`5`的平方`25`。



## gVisor & Nginx

这节主要介绍在gVisor上运行[Nginx](https://zh.wikipedia.org/wiki/Nginx)，并研究在运行Nginx的时候，sentry和Nginx，gVisor sandbox和Host之间的system call的调用情况。

主要的步骤如下

* 在本地的 docker上部署 Nginx，使用 gVisor 运行成功
* 寻找 Nginx 向 gVisor 发出的 system calls
* 寻找 gVisor 向 Host Kernel 发出的 system calls



### Deployment

参考[这篇文章](https://www.runoob.com/docker/docker-install-nginx.html), 可以从 Docker Hub下载 Nginx 镜像并用 docker run 命令来运行它。

```
$ docker pull nginx
```

运用 `docker pull` 命令来下载官方镜像

```
$ docker run --name nginx-test -p 8081:80 -d nginx
```

这里有几个参数需要说明：

* **nginx-test** 是容器的名字
* **8081** 是容器对外的端口，用 `-p` 参数将本地的 8081 端口映射到容器内部的 80 端口
* 因为使用了 `-p` 参数，这个容器**运行在后台**

在运行了上述命令之后，打开浏览器，在地址栏处输入“127.0.0.1:8081”，回车后可以看到 Nginx 的”welcome page“，说明成功的在 docker 上运行了 Nginx。

在这之后，新建一些文件来和 Nginx 容器来做一些映射。

```
$ mkdir -p ~/nginx/www ~/nginx/logs ~/nginx/conf
$ docker cp 6dd4380ba708:/etc/nginx/nginx.conf ~/nginx/conf
```

这里的 `6dd4380ba708` 是运行的 Nginx 容器的ID，我们可以用 `docker ps` 来获取。

* `www` 是 Nginx 的文件目录
* `logs` 用来存放日志文件的目录
* 文件 `conf` 用来映射 Nginx 的配置文件

最后添加 `index.html` 文件到 `www` 目录下, 该文件将会是访问该服务器时的主页。

```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>zty's web</title>
</head>
<body>
    <h1>我的第一个标题</h1>
    <p>我的第一个段落。</p>
</body>
</html>
```

运行 `docker run` 命令来启动 Nginx 容器，同时添加 `--runtime=runsc-kvm` 以 gVisor sandbox 来运行容器，并用 `-v` 参数来映射文件。

```shell
docker run --runtime=runsc-kvm -d -p 8082:80 --name nginx-web --rm -v ~/nginx/www:/usr/share/nginx/html -v ~/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v ~/nginx/logs:/var/log/nginx nginx
```

再次访问 `127.0.0.1:8082` 网址，即可浏览到刚刚编写的 `index.html` 网页。



### System calls from Nginx to gvisor

为了统计所有从 Nginx 产生的 System calls，需要先明确具体的系统调用流程。这里的流程指的是源代码当中的函数调用关系。因此先观察日志文件。

上文提到在 docker 配置文件当中添加了 debug 等一系列参数，因此当运行 Nginx 容器的时候，在之前设置的目录下会有一系列的日志文件产生，这些日志文件当中以 `.boot` 结尾的就是 sentry 进程对应的日志文件。这里有关于 [`.create`](https://github.com/hujikoy/gvisor-learning/blob/master/gvisor_learning.md) 和 [`.boot`](https://github.com/hujikoy/gvisor-learning/blob/master/sentry.md#set-up) 的相关研究结果可供参考。

在 `.boot` 文件当中，有所有来自于 Nginx 的系统调用的记录，选取一个如下所示

```
nginx E read(0x3 /lib/x86_64-linux-gnu/libdl-2.28.so, 0x7f2d2c4640d8, 0x340)
```

这里显示 Nginx 调用了 `read()` system call。

在已知 Nginx 会调用 `read()` system call 的条件下，结合之前讲述的系统调用具体实现，可以找到实现 `read()` 系统调用函数的位置，添加一段代码，将运行到该处时的 stack trace 打印出来。

因此在[`pkg/sentry/syscalls/linux/sys_read.go`](https://github.com/google/gvisor/blob/release-20190806.1/pkg/sentry/syscalls/linux/sys_read.go)这个文件里面修改 gvisor 实现的 read 系统调用，增加 print stack trace 代码

```go
func Read(t *kernel.Task, args arch.SyscallArguments) (uintptr, *kernel.SyscallControl, error) {
	// add by zty in oder to watch stack trace
	db.PrintStack()
```

并在文件的开头修改 `import` 信息

```go
import(
    ...
	// added by zty to watch stack trace
	db "runtime/debug"
)
```

重新编译并运行 Nginx 之后，在日志当中的输出如下

```
goroutine 40 [running]:
runtime/debug.Stack(0xd0853a, 0xd, 0x0)
	GOROOT/src/runtime/debug/stack.go:24 +0x9d
runtime/debug.PrintStack()
	GOROOT/src/runtime/debug/stack.go:16 +0x22
gvisor.googlesource.com/gvisor/pkg/sentry/syscalls/linux.Read(...)
	pkg/sentry/syscalls/linux/sys_read.go:49 +0x7e
gvisor.googlesource.com/gvisor/pkg/sentry/kernel.(*Task).executeSyscall(...)
	pkg/sentry/kernel/task_syscall.go:165 +0x10a
gvisor.googlesource.com/gvisor/pkg/sentry/kernel.(*Task).doSyscallInvoke(...)
	pkg/sentry/kernel/task_syscall.go:283 +0x69
gvisor.googlesource.com/gvisor/pkg/sentry/kernel.(*Task).doSyscallEnter(...)
	pkg/sentry/kernel/task_syscall.go:244 +0x96
gvisor.googlesource.com/gvisor/pkg/sentry/kernel.(*Task).doSyscall(...)
	pkg/sentry/kernel/task_syscall.go:219 +0x13d
gvisor.googlesource.com/gvisor/pkg/sentry/kernel.(*runApp).execute(...)
	pkg/sentry/kernel/task_run.go:219 +0xe09
gvisor.googlesource.com/gvisor/pkg/sentry/kernel.(*Task).run(...)
	pkg/sentry/kernel/task_run.go:91 +0x194
created by gvisor.googlesource.com/gvisor/pkg/sentry/kernel.(*Task).Start
	pkg/sentry/kernel/task_start.go:286 +0xfe
```

定位到 `pkg/sentry/kernel/task_syscall.go` 文件，找到了 [`doSyscall()`](https://github.com/google/gvisor/blob/fb996668e40031671e08d107e1a5307e813215f9/pkg/sentry/kernel/task_syscall.go#L184) 函数

```go
// doSyscall is the entry point for an invocation of a system call specified by
// the current state of t's registers.
//
// The syscall path is very hot; avoid defer.
func (t *Task) doSyscall() taskRunState {
	sysno := t.Arch().SyscallNo()
	args := t.Arch().SyscallArgs()
    
	// add by zty in oder to get syscall number
	log.Infof("Syscall %d: called from APP", sysno)
```

在这个函数当中加入一行日志输出的代码，即上述的 `log.Infof(...)`，同时在该文件的 `import` 部分添加 `"gvisor.dev/gvisor/pkg/log"`，保证编译的时候不会出错。

重新编译 gVisor，并运行 Nginx 之后，就可以获得有上述输出的 `.boot` 日志文件。笔者通过如下的 [python 代码](https://github.com/hujikoy/gvisor-learning/blob/master/nginx/pythonfile/collect_syscall.py)获取了文件当中的所有 system call number。

注意文代码当中的 [`path`](https://github.com/hujikoy/gvisor-learning/blob/master/nginx/pythonfile/collect_syscall.py#L5) 变量为运行 Nginx 容器之后，gvisor 输出的 `.boot` 日志文件。

运行该文件之后收集到的系统调用如下

```shell
$ python3 collect_syscall.py 
number of syscalls:  61
[[3, 74], [228, 50], [257, 47], [9, 46], [8, 44], [232, 42], [5, 41], [45, 40], [4, 26], [233, 25], [54, 21], [1, 20], [20, 20], [40, 20], [288, 20], [0, 18], [10, 16], [202, 16], [13, 14], [12, 7], [16, 5], [41, 5], [72, 5], [83, 5], [92, 5], [17, 4], [42, 4], [14, 3], [63, 3], [302, 3], [11, 2], [15, 2], [39, 2], [47, 2], [50, 2], [53, 2], [61, 2], [107, 2], [130, 2], [213, 2], [217, 2], [231, 2], [273, 2], [290, 2], [18, 1], [21, 1], [33, 1], [38, 1], [46, 1], [49, 1], [56, 1], [87, 1], [99, 1], [105, 1], [106, 1], [110, 1], [116, 1], [157, 1], [158, 1], [206, 1], [218, 1]]
```

然而这里的 system calls **总数不是确定的**，笔者试了10次，得到的结果不尽相同，判断可能是因为运行流程当中的一些操作不同，比如未停止 Nginx container 的时候，得到的数目是54，但是关闭 container 之后得到的是61，即在关闭 container 的过程当中会有新的 system calls 被调用。同时，如果在访问服务器地址的时候，用多个浏览器打开，数目也会变化，从 54 上升到 55。因此为了能够得到固定的数值，需要制定一个确定的流程。从**开始->访问->结束**，所有操作都是相同的情况下，再去看这个数值是否固定。

因此笔者编写了[一个脚本](https://github.com/hujikoy/gvisor-learning/blob/master/nginx/app-to-sentry-deploy.sh)，包含运行 Nginx，访问网址，关闭容器，连续多次实践之后得到了**稳定的system calls 统计为 61**。之前在多浏览器访问的情况下出现的 62 个 system calls，经过筛查发现是多了一个 186 号系统调用——`gettid()`。*可能是因为并发访问下的线程数增加，才有了该调用的产生。*

将这些 system calls 可视化的结果如下。



![](syscall_to_sentry_2.png)



### System calls from gvisor to Host

若要获取从 gVisor 发往 Host Kernel 的 system calls，有两个思路

* 一个是从 **gVisor 的系统调用过程**入手，找到 gvisor 向 kernel 发出调用**请求的入口**
* 一个是将 gVisor 视为一个普通的进程，**借用工具观察**该进程的所有 syscall

对于上述两个想法，笔者都进行了探究。

#### gVisor 的系统调用入口

gVisor 往 Host Kernel 发送系统调用请求的代码基本如下

```go
// gvisor/runsc/cmd/cmd.go:	
if _, _, err := syscall.RawSyscall(syscall.SYS_SETGID, uintptr(nobody), 0, 0); err != 0 {

//gvisor/runsc/container/container.go
wpid, err := syscall.Wait4(int(c.GoferPid), nil, syscall.WNOHANG, nil)
```

因此为了统计所有调用的入口，需要运行以下命令

```shell
$ grep -r "syscall\..*(" gvisor/ | wc -l
584
```

有 584 个调用入口，且没有统一通过的路径，不能像上节一样进行代码的修改，在日志文件当中输出系统调用。因此**第一个思路现阶段走不通**。



#### 借用工具观察 System Call

参考 [gVisor User 的回答](https://groups.google.com/forum/#!msg/gvisor-users/B8mkgwiJFOY/pCTwvOTpAwAJ)，可以利用 `strace` 来追踪 gvisor sandbox 具体做了什么系统调用。有了上节的经验，为了避免找到的系统调用不一致，笔者编写了如下 shell code 来规范流程。

```shell
#!/bin/bash
CID=$(docker run --runtime=runsc-kvm -d -p 8082:80 --name nginx-web --rm -v ~/dev/nginx/www:/usr/share/nginx/html -v ~/dev/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v ~/dev/nginx/logs:/var/log/nginx nginx)
PID=$(docker inspect -f='{{.State.Pid}}' $CID)
sleep 5
sudo strace -ff -o syslog/log -p $PID &
for((i=1; i<=100; i++));
do
sleep 0.001
curl -s 127.0.0.1:8082 > /dev/null
done
sleep 5
find syslog/ -name "log.*" > syslog/list
python3 pythonfile/collect_sys_stoh.py >> py_out
docker stop nginx-web
```

简要解释一下上述 code 做的事情，首先在 gVisor 中运行 Nginx 并获取 container ID `CID`，再根据 `CID` 获取 gVisor sandbox 的 `PID`，这里的 `PID` 是在 Host OS 上的。有了 `PID` 就可以用 `strace -p`  命令来追踪进程的系统调用。具体的流程如下

* 等待 5 s，目的是为了让 Nginx 初始化工作完成
* 开始追踪，注意这里使用了 `-ff` 参数，目的是追踪所有的线程并输出文件。
* 循环访问网址，一共 100 次
* 等待 5 s，确保 Nginx 进入稳定状态
* 查询输出的跟踪日志文件，并用[一段 Python 代码](https://github.com/hujikoy/gvisor-learning/blob/master/nginx/pythonfile/collect_sys_stoh.py)去获取 system calls
* 停止容器

在运行20次以上之后，发现 system calls 的数量在 22 - 24 之间波动如下

```
num of syscalls:  23
[202, 35, 1, 228, 47, 16, 15, 281, 271, 299, 46, 20, 24, 318, 158, 17, 18, 14, 219, 186, 131, 56, 9]
num of syscalls:  22
[202, 35, 1, 228, 47, 16, 15, 281, 271, 299, 46, 20, 24, 318, 158, 17, 18, 14, 186, 131, 219, 56]
num of syscalls:  24
[202, 35, 228, 281, 1, 47, 24, 16, 15, 271, 299, 46, 20, 318, 158, 17, 18, 14, 219, 186, 131, 56, 9, 128]
```

其中 24 和 22 都只出现了一次，22 相较于 23，少了 `9` 号系统调用 [`mmap()`](http://man7.org/linux/man-pages/man2/mmap.2.html)，24 相较于 23，多了 `128` 号系统调用 [`re_sigtimewait()`](https://linux.die.net/man/2/rt_sigtimedwait)



