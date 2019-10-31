---
layout: post
title: cgroup usage
categories: [cgroup, linux kernel]
description: a translation of linux cgroup man page
keywords: cgroup, linux
---

## cgroup man

written by Tianyu

这篇 blog 是关于 linux cgroup man 的翻译，也加上了我的个人理解，具体的细节请结合官方的 [cgroup man page](http://man7.org/linux/man-pages/man7/cgroups.7.html) 来理解，当前我研究的 cgroup 一共有 2 个版本，v1 和 v2，这两个版本的 cgroup 在架构和功能上有较大的不同，具体内容会在下面的文章当中说明，**当前版本的 kernel 是 5.3.8**，具体研究的时候请参照最新的[说明文档](https://elixir.bootlin.com/linux/latest/source/Documentation/admin-guide/cgroup-v2.rst)来看。

### Name

cgroups - Linux control groups

### Description

cgroups，全名是 control groups，是 linux 用来监控和限制进程资源的 feature。linux kernel 的 cgroup interface 是名为 cgroupfs 的伪文件系统，类似 proc。之所以叫做 control groups，是因为将每个进程分到不同的 group 中去，按照 group 来对资源进行控制。每个资源（cpu、memory等）都是由单独的控制器 controller 来控制的。

#### Terminology

cgroup 是一组通过 cgroup filesystem 来定义的，具有特定资源限制的进程 group。

subsystem 也叫做 resource controllers，是用来控制特定 cgroup 中进程资源使用的 kernel 组件。 

```
       The cgroups for a controller are arranged in a hierarchy.  This
       hierarchy is defined by creating, removing, and renaming
       subdirectories within the cgroup filesystem.  At each level of the
       hierarchy, attributes (e.g., limits) can be defined.  The limits,
       control, and accounting provided by cgroups generally have effect
       throughout the subhierarchy underneath the cgroup where the
       attributes are defined.  Thus, for example, the limits placed on a
       cgroup at a higher level in the hierarchy c
```

cgroup 的资源控制是通过 hierarchy 的方式组织的。后续也会详细说明，cgroup 的 hierarchy 具体表现为文件目录树，方才也提到 cgroup 在 linux 当中的交互是通过一个虚拟文件系统来实现的，此处的 hierarchy 就很像文件系统的目录层次结构。我们可以创建、删除和重命名 cgroup 目录下的各个文件目录，而这些文件目录的名称，正是 cgroup 的名称。通过设置这些文件目录的当中文件的值，来控制之前提到的 controller 的行为。另外，cgroup 的资源访问的等级是自上而下的，意思就是说，任何 cgroup 的资源访问区域都不能够大于他们的祖先节点，也就是祖先文件目录。

为了较为形象的来描述这个 hierarchy 在 linux 当中的表现形式，可以使用 ``



