---
layout: post
title: cgroup - Houdini's escape
categories: [cgroup, linux container, docker]
description: a recurrent of the cgroup escaping paper
keywords: cgroup, Docker, Linux container
---

## Linux Control Groups Escaping - Houdini

### Introduction

Linux Control Group，也称位 cgroups，是用于监控、限制 process 资源的一种 Linux kernel feature[^note1]。它同时也时操作系统级别容器化的重要组成模块。它将进程划分到多个分层的组，同时为这些组提供资源控制器，从而管理 CPU，内存还有块设备的输入输出。当创建子进程的时候，该子进程会自动从他的创建者那里拷贝 cgroup 属性，从而强制实行资源控制。然而，在创建进程的时候从父进程继承 cgroup 约束并不是万无一失的，有时候一致性和资源合理分配并不能很好的保持。

本文介绍的就是一种 cgroup escaping 的方法，被作者称之为 Houdini’s Escape，[这篇论文](https://gzs715.github.io/pubs/HOUDINI_CCS19.pdf)[^note2] 被 [CCS 2019](https://www.sigsac.org/ccs/CCS2019/) 会议[^note3] 录用。

论文主要针对之前提到的子进程 cgroup 进行攻击，使得该子进程脱离父进程的 cgroup 类别，从而进行 ou-of-band 攻击，使得同一物理机上的其他 container 的运行受到影响，同时还可以获得更多的资源（超出它本应该获得的范围）。我撰写该 blog 的目的是在于复现该论文的 esacape 方法，同时总结这些方法的特性和推广的价值。

### Cgroups Hierarchy and Controllers

> In Linux, cgroups are organized in a hierarchical structure where a set of cgroups are arranged in a tree. Each task (e.g., a thread) can only be associated with exactly one cgroup in one hierarchy, but can be a member of multiple cgroups in different hierarchies. Each hierarchy then has one or more subsystems attached to it, so that a resource controller can apply per-cgroup limits on specific system resources. With the hierarchical structure, the cgroups mechanism is able to limit the total amount of resources for a group of processes(e.g., a container).

上述是关于 cgroup 的架构介绍，重要的有以下几点

* 一个 cgroup 对应有一种 hierarchy
* 一个 task 在同类 hierarchy 中只能对应一个 cgroup
* 一个 task 可以有多个 cgroups
* resource controller 按照 cgroup 来进行具体资源的管控



Cgroup 相关的 resource controller 一共有四种

* **cpu controller**：在多个 cgroup 竞争 cpu 资源的时候，按照 cpu share 的值来按比例分配 cpu 资源，也可以通过设定 *quota* 和 *period* 来限制在固定周期内的 cpu 使用量
* **cpusets controller**：将 task 限制在具体的 cpu core 和 memory node 上
* **blkio controller**：控制和限制对于块设备的访问，可以通过设定 *blkio.weight* 来划分占用比例，也可以通过设置具体的上限
* **pid controller**：为 container 设置 task number 上限，上限存放在 *pids.max* 中，当前的 task 数目统计放在 *pids.current* 中，一旦达到上线，所有的 fork 和 clone 操作都会被禁止



### Cgroups Inheritance

> One important feature of cgroups is that child processes inherit cgroups attributes from their parent processes. 

子进程被创建时，会调用 fork() 或是 clone() 函数，一开始新创建的进程会 attach 到 root cgroup 上，在完成寄存器还有其他的进程环境拷贝之后，将会调用 cgroup 拷贝函数，将这个新创建的进程 attach 到创建它的父进程所属的 cgroups。

> Particularly, the function attaches the task to its parent cgroups by recursively going through all cgroup subsystems. As a result, after the copying procedure, the child task inherits memberships to the exact same cgroups as its parent task.

这个 cgroup 拷贝函数会把所有 cgroup subsystem 递归遍历一遍，最终实现将子进程的 cgroup 设置为和父进程完全一致。

举例来说，如果 cpusets 资源控制器将父进程指定为 2 号 cpu core，那么创建的子进程也会被指定为只能在 2 号 cpu core 上面运行。同时，如果 cpu 资源控制器对父进程设置了 *quota* 和 *period*，那么新创建的子进程将会和父进程共享这些资源，即子进程和父进程的 cpu 使用量加起来不能超过 *quota*。



### Exploiting Strategies



### Cases Reccurent

> We use the Docker container to set the configuration of cgroups through the provided interfaces. Besides, Docker also ensures that containers are isolated through namespaces bydefault. 
>
> Especially, with the USER namespace enabled, the root user in a container is mapped to a non-privileged user on the host. Thus, the privileged operations within containers cannot affect the host kernel. Our case studies are conducted in such de-privileged containers.



#### case 1: Exception Handling

constrains

* cpu core: 1
* cpu share: 100%, 10%, 5%
* pid limitation: None, 100, 50





#### case 2

#### case 3

#### case 4

#### case 5



### References

[^note1]:  cgroup man page: http://man7.org/linux/man-pages/man7/cgroups.7.html.

[^note2]: Gao, Xing, et al. "Houdini’s Escape: Breaking the Resource Rein of Linux Control Groups." (2019).
[^note3]: The 26th ACM Conference on Computer and Communications Security in **London, United Kingdom from November 11 to November 15, 2019**.

