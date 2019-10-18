---
layout: post
title: sgroup - Houdini's escape
categories: [cgoup, Linux container, Dokcer]
description: a recurrent of the cgroup escaping paper
keywords: cgroup, Docker, Linux container
---

## Linux Control Groups Escaping - Houdini

### Introduction

Linux Control Group，也称位 cgroups，是用于监控、限制 process 资源的一种 Linux kernel feature[^note1]。它同时也时操作系统级别容器化的重要组成模块。它将进程划分到多个分层的组，同时为这些组提供资源控制器，从而管理 CPU，内存还有块设备的输入输出。当创建子进程的时候，该子进程会自动从他的创建者那里拷贝 cgroup 属性，从而强制实行资源控制。然而，在创建进程的时候从父进程继承 cgroup 约束并不是万无一失的，有时候一致性和资源合理分配并不能很好的保持。

本文介绍的就是一种 cgroup escaping 的方法，被作者称之为 Houdini’s Escape，[这篇论文](https://gzs715.github.io/pubs/HOUDINI_CCS19.pdf)[^note2] 被 [CCS 2019](https://www.sigsac.org/ccs/CCS2019/) 会议[^ note3] 录用。

论文主要针对之前提到的子进程 cgroup 进行攻击，使得该子进程脱离父进程的 cgroup 类别，从而进行 ou-of-band 攻击，使得同一物理机上的其他 container 的运行受到影响，同时还可以获得更多的资源（超出它本应该获得的范围）。我撰写该 blog 的目的是在于复现该论文的 esacape 方法，同时总结这些方法的特性和推广的价值。











[^note1]:  cgroup man page: http://man7.org/linux/man-pages/man7/cgroups.7.html.

[^note2]: Gao, Xing, et al. "Houdini’s Escape: Breaking the Resource Rein of Linux Control Groups." (2019).
[^note3]: The 26th ACM Conference on Computer and Communications Security in **London, United Kingdom from November 11 to November 15, 2019**.