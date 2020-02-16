---
layout: post
title: Map-Reduce 工作流程理解
categories: [Bigdata, Hadoop]
description: Describe workflow of MRv1, Difference for MRv2
keywords: Map-Reduce
---

本文详细的说明了 MapReduce 的工作框架和流程，以及介绍了 MapReduce 2.0的区别。最后在先前搭载的 Hadoop 集群中实现了 Python 词频统计程序。

MapReduce 计算模型实现数据处理时，应用程序开发者只需要负责 Map 函数和 Reduce 函数的实现，而不需要处理分布式和并行编程中的各种复杂问题。如分布式存储、分布式通信、任务调度、容错处理、负载均衡、数据可靠等，这些问题都由 Hadoop MapReduce 框架负责处理，应用开发者只需要负责完成 Map 函数与 Reduce 函数的实现。

## 一、工作框架

MapReduce 的实际处理过程可以分解为 Input、Map、Sort、Combine、Partition、Reduce、Output 等阶段，其中{Sort、Combine、Partition}可以统一为 Shuffle 阶段（但要强调的是，shuffle 是 MR 处理流程的一个过程，它的每一个处理步骤都是分散在各个 map task 和 reduce task 上完成的）。下图是MapReduce做词频统计的完整流程图。

<img src="/images/posts/Map-Reduce/process.png" alt="MapReduce Process" />

* 在 Input 阶段：框架根据数据的存储位置，把数据分成多个**分片**（Split），在多个结点上并行处理。

* 在 Map 阶段：框架调用 Map 函数对输入的每一个 <key, value> 进行处理，也就是完成 Map<K1, V1> → List(<K2, V2>) 的映射操作。注意这里的 k1 是指一行文本，k2 是指一个单词，是不一样的，之后会细说。

  Map 任务通常**运行在数据存储的结点**上，也就是说，框架是根据数据分片的位置来启动 Map 任务的，而不是把数据传输到 Map 任务的位置上。这样，计算和数据就在同一个结点上，从而不需要额外的数据传输开销。

* 在 Sort 阶段：当 Map 任务结束以后，会生成许多 <K2,V2> 形式的中间结果，框架会对这些中间结果**按键值排序**。

* 在 Combine 阶段：框架对于在 Sort 阶段排序之后有相同键的中间结果进行**合并**。合并所使用的函数可以由用户进行定义。这样，在每一个 Map 任务的中间结果中，每一个字母只会出现一次。

* 在 Partition 阶段，框架将 Combine 后的中间结果**按照键的取值范围划分**为 R 份，分别发给 R 个运行 Reduce 任务的结点，并行执行。分发的原则是，首先必须保证**同一个键的所有数据项发送给同一个 Reduce 任务**，尽量保证每个 Reduce 任务所**处理的数据量基本相同**。框架默认使用 Hash 函数进行分发，用户也可以提供自己的分发函数。

* reduce任务并不是在map任务完全结束后才开始的，Map 任务有可能在不同时间结束，所以 reduce 任务没必要等所有 map 任务都结束才开始。事实上，每个 reduce任务有一些 threads 专门负责从 map主机复制 map 输出（默认是5个）。

* 在 Output 阶段，框架把 Reduce 处理的结果按照用户指定的输出数据格式写入 HDFS 中。

<img src="/images/posts/Map-Reduce/architecture.png" alt="architecture"/>

上图是关于词频统计的更加详细的介绍，注意的就是黑框指的就是一台机器，一个 map 或者 reduce 的处理节点，蓝框指的是 shuffle 过程。这张图为我们展现了更加丰富的内容在于：

* 整个过程的 shuffle 阶段比较复杂，我们看蓝色的框框，从⑥步开始。 

  首先，每个输入分片会让一个map任务来处理，默认情况下，以HDFS的一个块的大小（默认为64M）为一个分片，当然我们也可以设置块的大小；

  接着，map 输出的 (k, v) 键值会由 OutputCollector 收集并写入到环形缓冲区中，环形缓冲区默认大小为100M。当环形缓冲区里面的数据达到其大小的80%时（由 io.sort.spill.percent 属性控制）就会触发 spill 溢出，会在本地文件系统中创建一个**溢出文件**，将该缓冲区中的数据写入这个文件；

  但是，在写入磁盘前，线程首先根据 reduce 任务的数目将数据划分为相同数目的分区，也就是一个 reduce 任务对应一个**分区**的数据。这样做是为了避免有些 reduce 任务分配到大量数据，而有些 reduce 任务却分到很少数据，甚至没有分到数据的尴尬局面。分区就是对数据进行 hash 的过程，然后对每个分区中的数据按照 key 进行排序；

  注意：如果此时设置了Combiner，将排序后的结果进行 Combine 操作，这样做的目的是**让尽可能少的数据写入到磁盘**；

  如果数据量较大，当 map 任务输出最后一个记录时，可能会有很多的溢出文件，多个文件再被 merge 成大的溢出文件合并的过程中会不断地进行排序和 combine 操作，目的有两个：1.尽量减少每次写入磁盘的数据量，2.尽量减少下一复制阶段网络传输的数据量。最后合并成了一个已分区且已排序的文件。为了减少网络传输的数据量，这里可以将**数据压缩**，只要将 mapred.compress.map.out 设置为true就可以了；

  最后将分区中的**数据拷贝**给相对应的 reduce 任务。有人可能会问：分区中的数据怎么知道它对应的 reduce 是哪个呢？其实 map 任务一直和其父 TaskTracker 保持联系，而 TaskTracker 又一直和 JobTracker 保持心跳。所以 JobTracker 中保存了整个集群中的宏观信息。只要 reduce 任务向 JobTracker 获取对应的 map 输出位置就 ok 了，下一部分介绍。

* reduce 阶段，会调用 **groupingcomparator** 进行分组，之后的 reduce 中会按照这个分组，每次取出一组数据，调用 reduce 中自定义的方法进行处理。

* 在文件被读入的时候调用的是 Inputformat 方法读入的。InputFormat —> RecordReader —> Read(k, v) （行，行内容）  -> map（变成真正的key-value） -> context.Write -> OutputCollector -> shuffle -> reduce -> OutputFormat -> RecordWriter -> Write(k, v)。
  
  

## 二、流程详细

### 1. Input 阶段

Input 是输入文件的存储位置，注意这里不一定是 HDFS 分布式文件系统位置，默认是HDFS文件系统，也可以修改为本机上的文件位置。
<img src="/images/posts/Map-Reduce/input.jpg" alt="Input" />

① 在客户端启动一个作业，向 JobTracker 请求一个 Job ID。JobClient 中的 Run 方法 会让  JobClient  把所有 Hadoop Job 的信息，比如 MapReduce程序打包的 JAR 文件、配置文件和客户端计算所得的**输入划分信息**。这些文件都存放在 JobTracker 专门为该作业创建的文件夹中。文件夹名为该作业的Job ID。JAR文件默认会有10个副本（mapred.submit.replication属性控制）；输入划分信息告诉了JobTracker应该为这个作业启动多少个map任务等信息。如下面的代码所示：

```java
public int run(String[] args) throws Exception {
        
        //create job
        Job job = Job.getInstance(getConf(), this.getClass().getSimpleName());
        
        // set run jar class
        job.setJarByClass(this.getClass());
        
        // set input . output
        FileInputFormat.addInputPath(job, new Path(PropReader.Reader("arg1")));
        FileOutputFormat.setOutputPath(job, new Path(PropReader.Reader("arg2")));
        
        // set map
        job.setMapperClass(HFile2TabMapper.class);
        job.setMapOutputKeyClass(ImmutableBytesWritable.class);
        job.setMapOutputValueClass(Put.class);
        
        // set reduce
        job.setReducerClass(PutSortReducer.class);
        return 0;
    }
```

其中，InputFormat 类去计算如何把 input 文件 分割成一份一份，然后交给 mapper 处理。inputformat.getSplit() 函数返回一个 InputSplit 的 List, 每一个 InputSplit 就是一个 mapper 需要处理的数据。Jobclient 有一个方法叫 setInputFormat()，通过它我们可以告诉 JobTracker 想要使用的 InputFormat 类 是什么。如果我们不设置，Hadoop默认的是 TextInputFormat，它默认为文件在 HDFS上的每一个 Block 生成一个对应的 InputSplit.。所以大家使用 Hadoop 时，也可以编写自己的 input format, 这样可以自由的选择分割 input 的算法，甚至处理存储在 HDFS 之外的数据。

② JobTracker 接收到作业后，将其放在一个作业队列里，等待作业调度器对其进行调度，当作业调度器根据自己的调度算法调度到该作业时，会根据输入划分信息为每个划分创建一个map任务，并将map任务分配给 TaskTracker 执行。对于 map 和 reduce 任务，TaskTracker 根据主机核的数量和内存的大小有固定数量的 map 槽和 reduce 槽。

③ JobTracker 尽量把 mapper 安排在离它要处理的数据比较近的机器上，以便 mapper 从本机读取数据，节省网络传输时间。对于每个 map任务, 我们知道它的 split 包含的数据所在的主机位置，我们就把 mapper 安排在那个相应的主机上。既然一个 InputSplit 对应一个 map任务, 那么当 map 任务收到它所处理数据的位置信息，它就可以从 HDFS 读取这些数据了。

③ RecordReader 类，它主要的方法是 next()，作用就是从 InputSplit 读出一条 key-value对。



### 2. Map 阶段

④ TaskTracker 每隔一段时间会给 JobTracker 发送一个心跳，告诉 JobTracker 它依然在运行，同时心跳中还携带着很多的信息，比如当前 map 任务完成的进度等信息。当 JobTracker 收到作业的最后一个任务完成信息时，便把该作业设置成“成功”。当 JobClient 查询状态时，它将得知任务已完成，便显示一条消息给用户。



### 3. Shuffle 阶段

<img src="/images/posts/Map-Reduce/shuffle.png" alt="Shuffle" />

⑤ 在map中，每个 map 函数会输出一组 key/value对, Shuffle 阶段需要从所有 map主机上把相同的 key 的 key value对组合在一起，（也就是这里省去的Combiner阶段）组合后传给 reduce主机, 作为输入进入 reduce函数里。Partitioner 组件 HashPartitioner类，它会把 key 放进一个 hash函数里，然后得到结果。如果两个 key 的哈希值 一样，他们的 key/value对 就被放到同一个 reduce 函数里。我们也把分配到同一个 reduce函数里的 key /value对 叫做一个reduce partition。

⑥ 当 Map 结束时 map 阶段可能会产生多个 spill file，这些 spill file 会被 merge 起来，不是 merge 成一个 file，而是也会按 reduce partition 分成多个。

⑦ 当 Map tasks 成功结束时，他们会通知负责的 tasktracker, 然后消息通过 jobtracker 的 heartbeat 传给 jobtracker. 这样，对于每一个 job, jobtracker 知道 map output 和 map tasks 的关联。Reducer 内部有一个 thread 负责定期向 jobtracker 询问 map output 的位置，直到 reducer 得到所有它需要处理的 map output 的位置。

⑧ Reducer 的另一个 thread 会把拷贝过来的 map output file 合并成更大的 file。 如果 map task 被 configure 成需要对 map output 进行压缩，那 reduce 还要对 map 结果进行解压缩。当一个 reduce task 所有的 map output 都被拷贝到一个它的 host上时，reduce 就要开始对他们排序了。



### 4. Reduce 阶段

<img src="/images/posts/Map-Reduce/reduce.jpg" alt="Reduce" />

⑨ Reduce会接收到不同map任务传来的数据，并且每个map传来的数据都是有序的。如果reduce端接受的数据量相当小，则直接存储在内存中（缓冲区大小由mapred.job.shuffle.input.buffer.percent属性控制，表示用作此用途的堆空间的百分比），如果数据量超过了该缓冲区大小的一定比例（由mapred.job.shuffle.merge.percent决定），则对数据合并后溢写到磁盘中。

⑩ 随着溢写文件的增多，后台线程会将它们合并成一个更大的有序的文件，这样做是为了给后面的合并节省时间。合并的过程中会产生许多的中间文件（写入磁盘了），但MapReduce会让写入磁盘的数据尽可能地少，并且最后一次合并的结果并没有写入磁盘，而是直接输入到reduce函数。



## 三、MapReduce2.0升级

Hadoop框架的自身问题限制了集群的发展。首先是，JobTracker 和 NameNode 的单点问题，严重制约了集群的扩展和可靠性。其次，MapReduce采用了基于slot的资源分配模型，slot是一种粗粒度的资源 划分单位，通常一个任务不会用完槽位对应的资源，且其他任务也无法使用这些空闲资源,同时map的槽位和reduce的槽位是不可以通用的。会导致部分资源紧张，部分资源空闲。

<center class="half">
    <img src="/images/posts/Map-Reduce/1.png" />
    <img src="/images/posts/Map-Reduce/2.png" />
</center>

所以 Yarn框架 应运而生。



### 1. 模块分析

<img src="/images/posts/Map-Reduce/yarn-arch.png" />

* Resource Manager

  RM 是一个全局的资源管理器，负责整个系统的资源管理和分配，包括 Resource scheduler、Application Manager、Node Manager。YARN 提供了多种直接可用的调度器， Fair Scheduler 和 Capacity Scheduler 等。调度器仅根据各个应用程序的资源需求进行资源分配，分配的基本单位是Container，而容器里面是将内存，CPU，网络，磁盘封装到一起。

* Application Master

  AM 的工作包括应用程序提交、与调度器协商资源以启动 ApplicationMaster、监控 ApplicationMaster 运行状态并在失败时重新启动它等。用户提交的每个应用程序均包含一个 ApplicationMaster, ApplicationMaster 可以与RM协商获取资源，也可以将得到的任务进行再分配，与Node Manager 通信，同时可以监控所有的任务状态。

* Node Manager

  NM 是每个节点上的资源和任务管理器，一方面，它会定时地向 RM 汇报本节点上的 资源使用情况和各个 Container 的运行状态；另一方面，它接收并处理来自 AM 的 Container 启动 / 停止等各种请求。

* Container

  Container 是 YARN 中的资源抽象，它封装了某个节点上的多维度资源，如内存、 CPU、磁盘、网络等，当 AM 向 RM 申请资源时，RM 为 AM 返回的资源便是用 Container 表示的。YARN 会为每个任务分配一个 Container，且该任务只能使用该 Container 中描述的 资源。容器是一个动态划分资源。

* Jobhistory 机制

  在 MRv1 中，JobHistroy server 是嵌入在 Jobtracker 中的，当有大量的查询 时，对 Jobtracker 造成很大的压力。Yarn中实现一套单独的 JobHistroy server 服务。



### 2. Yarn 工作流程

<img src="/images/posts/Map-Reduce/yarn-workflow.png" />

① 用户向 YARN 中提交应用程序，其中包括 ApplicationMaster 程序、启动 ApplicationMaster 的命令、用户程序等。

② ResourceManager 为该应用程序分配第一个 Container，并与对应的 NodeManager 通信，要求它在这个 Container 中启动应用程序的 ApplicationMaster。

③ ApplicationMaster 首先向 ResourceManager 注册，这样用户可以直接通过 ResourceManage 查看应用程序的运行状态，然后它将为各个任务申请资源，并监控它的运 行状态，直到运行结束，即重复步骤 4~7。

④ ApplicationMaster 采用轮询的方式通过 RPC 协议向 ResourceManager 申请和 领取资源。 步骤

⑤ 一旦 ApplicationMaster 申请到资源后，便与对应的 NodeManager 通信，要求 它启动任务。

⑥ NodeManager 为任务设置好运行环境（包括环境变量、JAR 包、二进制程序 等）后，将任务启动命令写到一个脚本中，并通过运行该脚本启动任务。

⑦ 各个任务通过某个 RPC 协议向 ApplicationMaster 汇报自己的状态和进度，以 让 ApplicationMaster 随时掌握各个任务的运行状态，从而可以在任务失败时重新启动任务。 在应用程序运行过程中，用户可随时通过 RPC 向 ApplicationMaster 查询应用程序的当 前运行状态。

⑧ 应用程序运行完成后， ApplicationMaster 向 ResourceManager 注销并关闭自己。



但是并不意味着MapReduce1.0被淘汰，在Yarn中的MRYarnClild模块中基本上是是采用MapReduce1.0的解决思路，MRv2 具有与 MRv1 相同的编程模型和数据处理引擎，唯一不同的是运行时环境。MRv2 是在 MRv1 基础上经加工之后，运行于资源管理框架 YARN 之上的计算框架 MapReduce。 它的运行时环境不再由 JobTracker 和 TaskTracker 等服务组成，而是变为通用资源管理 系统 YARN 和作业控制进程 ApplicationMaster，其中，YARN 负责资源管理和调度，而 ApplicationMaster 仅负责一个作业的管理。简言之，MRv1 仅是一个独立的离线计算框架， 而 MRv2 则是运行于 YARN 之上的 MapReduce。



## 四、词频统计例子





参考资料：

1. 《[MapReduce中各个阶段的分析](https://blog.csdn.net/wyqwilliam/article/details/84669579)》
2. 《[Hadoop MapReduce工作流程](http://c.biancheng.net/view/3626.html)》
3. 《[MapReduce工作流程最详细解释](https://www.jianshu.com/p/461f86936972)》
4. 《[Yarn框架深入理解](https://www.jianshu.com/p/bac54467ac3a)》
5. 《[]()》



