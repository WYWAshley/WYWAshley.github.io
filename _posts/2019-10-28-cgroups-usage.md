---
layout: post
title: cgroup usage
categories: [cgroup, linux kernel]
description: a translation of linux cgroup man page
keywords: cgroup, linux
---

# cgroup man

written by Tianyu

这篇 blog 是关于 linux cgroup man 的翻译，也加上了我的个人理解，具体的细节请结合官方的 [cgroup man page](http://man7.org/linux/man-pages/man7/cgroups.7.html) 来理解，当前我研究的 cgroup 一共有 2 个版本，v1 和 v2，这两个版本的 cgroup 在架构和功能上有较大的不同，具体内容会在下面的文章当中说明，**当前版本的 kernel 是 5.3.8**，具体研究的时候请参照最新的[说明文档](https://elixir.bootlin.com/linux/latest/source/Documentation/admin-guide/cgroup-v2.rst)来看。



## Name

cgroups - Linux control groups



## Description

cgroups，全名是 control groups，是 linux 用来监控和限制进程资源的 feature。linux kernel 的 cgroup interface 是名为 cgroupfs 的伪文件系统，类似 proc。之所以叫做 control groups，是因为将每个进程分到不同的 group 中去，按照 group 来对资源进行控制。每个资源（cpu、memory等）都是由单独的控制器 controller 来控制的。



### Terminology

cgroup 是一组通过 cgroup filesystem 来定义的，具有特定资源限制的进程 group。

subsystem 也叫做 resource controllers，是用来控制特定 cgroup 中进程资源使用的 kernel 组件。 

cgroup 的资源控制是通过 hierarchy 的方式组织的。后续也会详细说明，cgroup 的 hierarchy 具体表现为文件目录树，方才也提到 cgroup 在 linux 当中的交互是通过一个虚拟文件系统来实现的，此处的 hierarchy 就很像文件系统的目录层次结构。

我们可以创建、删除和重命名 cgroup 目录下的各个文件目录，而这些文件目录的名称，正是 cgroup 的名称。通过设置这些文件目录的当中文件的值，来控制之前提到的 controller 的行为。另外，cgroup 的资源访问的等级是自上而下的，意思就是说，任何 cgroup 的资源访问区域都不能够大于他们的祖先节点，也就是祖先文件目录。

为了较为形象的来描述这个 hierarchy 在 linux 当中的表现形式，可以使用 `tree -d -L 2 /sys/fs/cgroup/` 来查看 cgroup 的目录结构，此处为了观察方便，我们省略了 2 层以上的子目录。

```shell
/sys/fs/cgroup/
├── blkio
│   ├── docker
│   ├── system.slice
│   └── user.slice
├── cpuset
│   └── docker
├── ...
└── unified
    ├── init.scope
    ├── system.slice
    └── user.slice
```

从上面可以看到，`/sys/fs/cgroup` 目录下面由很多文件，其中除了 `systemd` 和 `unified` 文件夹之外，都是用于 cgroup v1 的 controller 目录。



### Cgroups v1 and v2

一开始 linux kernel 当中仅存在 cgroup v1，随着后续的开发，不断的有新的 controller 引入，因而整个 cgroup 体系变得非常的复杂，因为一开始的设计存在缺陷，所以在 linux 3.10 开始开发人员就着手设计新版的 cgroup，[cgroup v2](https://elixir.bootlin.com/linux/latest/source/Documentation/admin-guide/cgroup-v2.rst) 在 linux 4.5 正式上线，接下来的文章里面会有详细的关于这两个版本的分析。

在这里仅简要的介绍一下这两个版本在 hierarchy 上的区别。

以 cpuset 为例，下列是 v1 的设置方法

```
                     "Top cpuset"    <-----  /sys/fs/cgroup/cpuset/
                       /       \
               CPUSet1         CPUSet2
                  |               |
               (Professors)    (Students)
```

在 `cpuset/` 目录下新建两个目录，这两个目录分别表示 2 个不同的 cgroup，在 cgroup v1 当中，针对不同的 controller 都要有不同的 cgroup，而当我们要限制进程的 cpuset 时，只需要把进程和特定的 cgroup 绑定即可，在这里，professor 进程绑定了 CPUSet1，而 Students 进程绑定了 CPUSet2。

而对于 v2 而言，并没有针对每个 controller 设定不同的 cgroup，而是统一设立 cgroup，对每个 cgroup 设定不同的 controller limitation 即可。

```
                        "class"
                       /       \
               cg_Professors  cg_Students
                  |               |
               cpuset: 0       cpuset: 1
```

如上所示，cg_Professors 和 cg_Students 为不同的 cgroup，针对这两个 cgroup 设定不同的 controller 值，然后再把具体的 process 绑定到这两个 cgroup 上即可。

> Currently, cgroups v2 implements only a subset of the controllers available in cgroups v1.

当前 kernel 当中两个版本的 cgroup 是并存的，而且出于 compile 的考虑，并不会将 cgroup v1 移除。在当前系统当中对于 cgroup 的应用也是并存的，但对于那些被 v2 implements 的 controller，只能选择应用 v1 或者 v2，而不能够两个版本同时使用。



## Croups Version 1

cgroup v1 每个 controller 对应一个 cgroup，也就是说，对于不同的 controller 可以有不同的 cgroup 去控制，当然，也可以为 多个 controller 设置同一个 cgroup。在上文当中，有介绍可以使用 `tree -d -L 2 /sys/fs/cgroup/` 来查看 cgroup 的目录结构，这里再次用它作为例子

```shell
$ tree -d -L 3 /sys/fs/cgroup/
/sys/fs/cgroup/
├── blkio
│   ├── docker
│   ├── system.slice
│   │   ├── accounts-daemon.service
│   │   └── ...
│   └── user.slice
├── cpu -> cpu,cpuacct
├── cpuacct -> cpu,cpuacct
├── cpu,cpuacct
│   ├── docker
│   ├── system.slice
│   │   ├── accounts-daemon.service
│   │   └── ...
│   └── user.slice
└── ....
```

为了方便观察，我设置只显示了 3 层文件目录。在上图中可以看到，`cpu/`、`cpuacct/` 都是link，它们 link 到了同一个目录 `cpu,cpuacct/`，这就是多个 controller 共同对应一个 cgroup 的例子。前文中也说过，cgroup 提供了虚拟文件系统，这些文件目录就是 cgroup 在内核中 hierarchy 的镜像。每个文件夹对应一个特定的 cgroup，它们的子文件夹就是 child cgroup，父文件夹就是 parent cgroup。



### Tasks (threads) vs processes

在 cgroup v1 当中，进程 processes 和线程 threads 的 cgroup 是可以区分对待的。一个 process 可以有多个 threads，在 cgroup v1 当中，我们可以独立控制每个 thread 的 cgroup。

然而在这个模型当中，对每个 thread 进行 cgroup 分配会出现一些问题，比如对于 memory 来讲，同一个 process 的不同 thread 是共享内存地址空间的， 对这些进程分别应用不同的 memory controllers 是毫无意义的。

因此在 cgroup v2 当中，独立操控 thread 级别的 cgroup 功能被移除了，相对的，为了精准控制多线程的情况，cgroup v2 引入了 *thread model*，在后续会进一步详细叙述。



### Mounting v1 controllers

在 Linux kernel 当中启用 cgroup 需要在 build 内核的过程中设置 `CONFIG_CGROUPS` 参数，我参考了 Linux 5.3 的 `/.config` 文件，发现关于 cgroup 的设定如下

```shell
CONFIG_CGROUPS=y
CONFIG_CGROUP_SCHED=y
# CONFIG_CGROUP_PIDS is not set
# CONFIG_CGROUP_RDMA is not set
CONFIG_CGROUP_FREEZER=y
# CONFIG_CGROUP_HUGETLB is not set
CONFIG_CPUSETS=y
# CONFIG_CGROUP_DEVICE is not set
CONFIG_CGROUP_CPUACCT=y
# CONFIG_CGROUP_PERF is not set
# CONFIG_CGROUP_DEBUG is not set
```

从上列设置中可以看到，*sched*、*freezer*、*cpusets*、*cpuacct* 这四个 controller 默认被启用了，而其余的并没有启用。

我们可以通过 mount 指令来使用 cgroup

```shell
$ mount -t cgroup -o cpu none /sys/fs/cgroup/cpu
```

当然同时 mount 多个 controller 也是可以的，就像前文所说，可以指定多个 controller 共用同一个 cgroup。

```shell
$ mount -t cgroup -o cpu,cpuacct none /sys/fs/cgroup/cpu,cpuacct
```

实际上在 *Linux 4.15.0-66-generic* 内核版本下，controller cpu 和 cpuacct 就是一同 mount 的。

```shell
$ ll /sys/fs/cgroup/
lrwxrwxrwx  1 root root  11 Oct 29 02:36 cpu -> cpu,cpuacct/
lrwxrwxrwx  1 root root  11 Oct 29 02:36 cpuacct -> cpu,cpuacct/
dr-xr-xr-x  5 root root   0 Oct 30 06:00 cpu,cpuacct/
```

对于一个特定的 process，它可以分 controller 对应不同的 cgroup，如 cpuset 对应 cg1，cpu 和 cpuacct 对应 cg2。然而同一个 controller 只能对应一个 cgroup，不可以对应多个 cgroup。

**注意：**许多 controller 都已经在 system boot 阶段就已经 mount 到了 `/sys/fs/cgourp/` 目录下，`systemd` 亦然。



### Unmounting v1 controllers

一个已经挂载的 cgroup 文件系统，可以被卸载，如现在要卸载 pids controller 对应的 cgroup file system

```shell
$ unmount /sys/fs/cgroup/pids
```

**注意：**只有在一个 cgroup 没有 child cgroups 的时候，才能够 unmount 它，即它没有其它子目录。当 remove 子目录（child cgroup）的时候，必须保证它控制的 process 都已经转移到 root cgroup 或者清除，否则也不能 remove。



### Cgroups v1 controllers

每一个 cgroup v1 当中的 controller 在使用之前必须在 build kernel 的时候显式的设定 configuration option（后续列出），并且在这之上必须要设定 `CONFIG_CGROUPS` 参数。

* *cpu* (`CONFIG_CGROUP_SCHED`)
  * 设定一个在 cpu busy 时分到 cpu 资源的权重，权重越高，在竞争 cpu 资源时得到的越多
  * 当 enable `CONFIG_CFS_BANDWIDTH` 时，可以通过设定每个 scheduling period 的上限来限制 cpu 使用时间，即使 cpu 并没有 busy，被该 controller 控制的 process 也必须服从这一限制
* *cpuacct* (`CONFIG_CGROUP_CPUACCT`)
  * 提供了对于一组 processes **cpu 使用量的计算**
* *cpuset* (`CONFIG_CUPSETS`)
  * 用来将 process 绑定到特定的 cpu 和 [NUMA](https://whatis.techtarget.com/definition/NUMA-non-uniform-memory-access) 节点上
* *memory* (`CONFIG_MEMCG`)
  * 记录 report 和限制 limit 具体 cgroup 的 process memory，kernel memory 还有 swap 的使用
* *devices* (`devices`)
  * 控制 process 对于设备的创建（`mknod`）、读和写
  * 通过 whitelist 或者是 blacklist 的形式来限制
  * 由于其 hierarchy 的结构，child cgroup 的设定必须不能和 ancestor cgroup 冲突
* *freezer* (`CONFIG_CGROUP_FREEZER`)
  * 用来挂起和回复所有该 cgroup 控制的 processes，freeze 一个 cgroup 也会使它的 child cgroup frozen，即这个操作是递归的
* *net_cls* (`CONFIG_CGROUP_NET_CLASSID`)
  * 该 controller 创建了一个 classid，这个 classid 和 cgroup 绑定，会在网络 packet 创建的时候用来匹配防火墙的规则
  * 它仅对从 cgroup 发出的 packet 做限制，而不限制外来的 packet
* *blkio* (`CONFIG_BLK_CGROUP`)
  * 通过对存储层次结构中的叶节点和中间节点应用 I/O 控制（节流和上限）来控制和限制对指定块设备的访问
  * 两种策略，一种是设定权重，基于权重进行磁盘时间的划分；另一种是设定 I/O rate 上限
* *net_prio* (`CONFIG_CGROUP_NET_PRIO`)
  * 设定网络的优先级
* *hugetlb* (`CONFIG_CGROUP_HUGETLB`)
  * 限制 huge pages 的使用
* *pids* (`CONFIG_CGROUP_PIDS`)
  * 限制单个 cgroup 创建进程的个数
* *rdma* (`CONFIG_CGROUP_RDMA`)
  * 限制 RDMA/IB-specific 资源的使用



### Creating cgroups and moving processes

创建 cgroup 的方法很简单，类似新建目录

```shell
 $ mkdir /sys/fs/cgroup/cpu/cg1
```

上述语句创建了一个新的空的 cgroup。

当然，对于 cgroup v1 来说，每个 cgroup 都有所属的 controller，因此当创建具体 controller 的 cgroup 时，根目录就必须为该 controller 在 `/sys/fs/cgroup/` 下的目录。

通过将 PID 写入 `cgroup.procs` 可以将该进程加入特定的 cgroup

```shell
$ echo $$ > /sys/fs/cgroup/cpu/croup.procs
```

一次只能往该文件写入一个 PID。若写入的值是 `0` 那么将会把写入进程的 PID 添加到该文件当中。

当写入进程已有对应的 cgroup 时，它原先所属的 cgroup 的 `cgroup.procs` 文件当中的记录将被移除，同时，所有属于该进程的 threads 的 TID 都会被添加到 `task` 文件当中。可以手动的将 TID 赋值给其它 cgroup 的 `task` 文件，此时该线程将会属于其他的 cgroup。



### Removing cgroups

如果要 remove 一个 cgroup，必须保证它没有 child cgroups 且没有任何 processes (nozombie) 才能够成功的 remove。

**注意：**cgroup 目录下的文件不能也不需要被手动的移除。



### Cgroups v1 release notification

在 cgroup 层次目录下面有两个文件是用来处理当某个 cgroup 为 empty 时的情况。当一个 cgroup 没有 child cgroup 且没有 member processes 的时候，它就成为了一个 empty cgroup。

在每个 cgroup hierarchy 的根目录，比如上文中的 `/sys/fs/cgroup/cpu/`，有一个名为 `release_agent` 的文件，该文件初始值为空，可以在其中添加**处理程序的路径**。这里的处理程序是自己编写的，专门在 cgroup becomes empty 时使用的，当特定的 cgroup 变为 empty 时，release_agent 会自动执行路径当中的程序，并将 cgroup 对应的文件挂载点作为参数传递给该程序。

`release_agent` 文件的内容可以通过以下方式来声明

```shell
$ mount -o release_agent=pathname ...
```

若要在每次有 cgroup empty 出现的时候就触发该功能，需要再设定一个文件的值——`notify_on_release`，这个文件位于每一个 cgroup 的目录下，对应 `release_agent` 在该 cgroup 变为 empty 时的行为。当文件的值为 **1** 时， `release_agent` 文件才会启动，默认的值是 **0**。

### Cgroup v1 named hierarchies

cgroup 允许创建不和 controller 对应的 cgroup hierarchy，每个这样的 hierarchy 都要有唯一的名字。

```shell
$ mount -t cgroup -o none,name=somename none /some/mount/point
```

像这样的 hierarchy 可以创建多个。cgroup 提供这种机制的原因是需要有 track process 的功能，比如 `name=systemd` cgroup hierarchy 就是用来 track services 和 user sessions 的。

从 Linux 5.0 开始，可以通过设定 `cgroup_no_v1 ` 这个 kernel boot option 来禁用 cgroup v1 的相关功能，若要禁止 named hierarchy，声明 `cgroup_no_v1=named` 即可。



## Cgroups Version 2

与 cgroup v1 不同的是，cgroup v2 的所有 controller 都位于同一个 hierarchy 中，在虚拟文件系统表示为同一个文件夹下。考虑到存在 cgroup v1 和 v2 共同使用的情况，不允许一个 controller 即属于 cgroup v1 又属于 v2。

cgroup v2 的新特性如下所示，部分需要重点描述的将会在后续当中详细说明

1. Cgroup v2 为所有 mounted controllers 提供了一个共同的 hierarchy。
2. **Internal** 进程不允许存在，简单的来说，除了 root cgroup 之外，所有的进程都只能绑定在 leaf nodes 上，所有既有 parent cgroup 又有 child cgroup 的 cgroup，都不能够有绑定的 process。具体的细节将会在后续给出。
3. 有效的 cgroups 必须通过 *cgroup.controllers* 和 *cgroup.subtrss_control* 来描述。
4. 原来存在于 cgroup 文件目录下的 *task* 文件被移除了，同时，为 cpuset controller 服务的 *cgroup.clone_children* 也被移除了。
5. 之前提到的 Release Notification 机制，现在通过 *cgroup.events* 文件来实现。

更多关于 cgroup v2 的具体信息可以参考 Linux source code 中的 [`cgroup-v2.rst`]( https://elixir.bootlin.com/linux/latest/source/Documentation/admin-guide/cgroup-v2.rst ) 文件。

另外在 Linux 4.14 引入了 ***thread mode***，也会在后续当中详细说明。

