---
layout: post
title: Hadoop Streaming 实践
categories: [Hadoop]
description: Hadoop Streaming try
keywords: Hadoop
---

&emsp;&emsp;尝试运行 Hadoop Streaming 的”hello world“小程序，记录自己遇到过的坑。

## 一、进行管道模拟

&emsp;&emsp;首先查看机子是否已经安装了 Python，一般 Ubuntu 是自带的，可以用```Python3 version```检查一下。

&emsp;&emsp;接下来正式进入管道模拟，在主机上尝试，目的是为了测试代码的正确性，运行下述代码：

```powershell
$ sudo echo "foo foo quux labs foo bar quux" | ~/HadoopFiles/mapper.py
$ echo "foo foo quux labs foo bar quux" | ~/HadoopFiles/mapper.py | sort -k1,1 | ~/HadoopFiles/reducer.py
```

&emsp;&emsp;可能会遇到文件夹权限的问题不能运行，需要更改权限```$ sudo chmod -R 777 ~/HadoopFiles```

<img src="/images/posts/Hadoop-Streaming/mapper-test.png" alt="mapper test"/>

<img src="/images/posts/Hadoop-Streaming/reducer-test.png" alt="reducer test"/>

<br/>

## 二、找到 Streaming 包地址

&emsp;&emsp;因为版本问题，Hadoop3.2.1中 Hadoop-Streaming 的 jar 包位于：```/usr/local/hadoop/share/hadoop/tools/lib/hadoop-streaming-3.2.1.jar```

<br/>

## 三、拷贝 txt 文件到 HDFS

&emsp;&emsp;注意要先启动HDFS，是否启动成功可以通过查看端口；```$ bash $HADOOP_HOME/sbin start-all.sh```

<img src="/images/posts/Hadoop-Streaming/netstat-nltp.png" alt="netstat-nltp"/>

&emsp;&emsp;下载txt到指定目录：```$ wget -P ~/HadoopFiles/ http://www.gutenberg.org/cache/epub/20417/pg20417.txt```

&emsp;&emsp;拷贝文件到HDFS

&emsp;&emsp;创建文件夹：```$ hadoop fs -mkdir /user```       

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;```$ hadoop fs -mkdir /user/HadoopFiles ```
&emsp;&emsp;拷贝：```$ bin/hadoop dfs -copyFromLocal ~/HadoopFiles/tmp /user/HadoopFiles```

<img src="/images/posts/Hadoop-Streaming/file.png" alt="file"/>

<br/>

## 四、运行

```$ bin/hadoop jar share/hadoop/tools/lib/hadoop-streaming-3.2.1.jar -mapper /user/HadoopFiles/mapper.py -reducer /user/HadoopFiles/reducer.py -input /user/HadoopFiles/tmp/* -output /user/HadoopFiles/gutenberg-output```

&emsp;&emsp;开始报错：

1. 没有成功连接 ResourceManager，如下，ResourceManager没有开始运行。[上网](https://stackoverflow.com/questions/20586920/hadoop-connecting-to-resourcemanager-failed)查了是因为 jdk 版本问题，于是我决定把原本的 jdk11改为 jdk8；

```powershell
xiaolongbao@master:/usr/local/hadoop$ jps
5105 SecondaryNameNode
7346 Jps
4664 NameNode
4829 DataNode
```

&emsp;&emsp;首先卸载原先的 openjdk：```$ sudo apt-get autoremove openjdk-8-jre-headless```

&emsp;&emsp;在官网下载 jdk1.8，这里分享一个 Oracle 账号：2696671285@qq.com    Oracle123

```sudo tar -zxvf jdk-8u241-linux-x64.tar.gz -C /usr/lib/jvm```

更改环境变量，并且立即生成

```export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_241```

```$ sudo source /etc/profile```

&emsp;&emsp;然后就可以看到启动成功了：

<img src="/images/posts/Hadoop-Streaming/gateway.png" alt="gateway"/>

<br/>

2. 编辑hadoop-env.sh
   ```export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_241```

   <br/>

3. 虚拟机重启之后 ip 地址改变了

&emsp;&emsp;可以在设置里把网络设置修改为静态ip

<br/>

4. 报错，找不到 resource-types.xml

   查看[教程](https://hadoop.apache.org/docs/r3.0.0/hadoop-yarn/hadoop-yarn-site/ResourceModel.html)