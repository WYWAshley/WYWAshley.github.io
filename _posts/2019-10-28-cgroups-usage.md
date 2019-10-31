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

