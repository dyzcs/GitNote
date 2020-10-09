# Hadoop

## 1. Hadoop 介绍

```wiki
The Apache Hadoop project develops open-source software for reliable, scalable, distributed computing.
The Apache Hadoop software library is a framework that allows for the distributed processing of large data sets across clusters of computers using simple programming models. It is designed to scale up from single servers to thousands of machines, each offering local computation and storage. Rather than rely on hardware to deliver high-availability, the library itself is designed to detect and handle failures at the application layer, so delivering a highly-available service on top of a cluster of computers, each of which may be prone to failures.
```

Hadoop 是开源的可靠、可伸缩的分布式计算软件。从单台服务器扩展到数千台机器，每台都能提供本地计算和存储，通过简单的编程模型跨计算机集群对大型数据集进行分布式处理。与依赖硬件来提供高可用不用，Hadoop 本身的设计目的就是在每一台服务器都有可能发生故障的前提下设计完成的。

Hadoop 的开发过程主要经历了 1.x、2.x 和 3.x，现在主流还是使用 2.x 和 3.x 居多，2.x 版本比 1.x 版本多了 Yarn 资源调度框架，3.x 版本在 2.x 版本的基础上天生支持 HA（Hadoop 高可用）。

## 2. Hadoop 集群配置

### 2.1 Hadoop 集群安装

初学者应该多关注 Hadoop Apache 官网，官网上有安装及报错资料。Hadoop 单节点伪分布式安装介绍网址如下：

[Hadoop 伪分布式安装](https://hadoop.apache.org/docs/r2.10.1/hadoop-project-dist/hadoop-common/SingleCluster.html#Pseudo-Distributed_Operation)

初学者使用虚拟机或者云服务器主机进行 Hadoop 伪分布式安装即可。

### 2.2 Hadoop 配置

在完成 Hadoop 安装之后，默认配置即可，但是有些参数依旧重要，我们要了解知道，为以后 Hadoop 集群配置调优打下基础。

1. 日志

    Hadoop 使用 Apache log4j 来记录日志，编辑 conf/log4j.properties 文件可以改变 Hadoop 守护进程的日志配置。

2. 机架感知

    Hadoop 组件可识别机架。Hadoop 会将 HDFS 块利用机架感知放置在不同的机架上，以此来进行容灾和容错。如果网络交换机集群中发生故障，但是集群存在分区，这将提供数据可用性。

    默认使用机架标识符和主机 IP 来进行标识。同样，如果我们有特殊需求可以进行定制化机架感知。

    [Hadoop 官网关于机架感知介绍](https://hadoop.apache.org/docs/r2.10.1/hadoop-project-dist/hadoop-common/RackAwareness.html)

### 2.3 运行进程

Hadoop 安装启动后使用 jps 命令可以看到有哪些启动进程。

1. NameNode

    是 HDFS 的主服务器，管理文件系统的目录树以及对集群中存储文件的访问，保存有 metadate，不断读取记录集群中 DataNode 主机状况和工作状态。

2. SecondaryNameNode

    NameNode 的冷备，负责周期性的合并 esimage 以及 editslog ，减少 NameNode 的工作量。

3. DataNode

    负责管理各个存储节点，每个存储数据的节点都有一个 DataNode 守护进程。

4. ResourceManager

    负责调度 DataManager 上的资源，每个 DataNode 都有一个 NodeManager （TaskTracker）来执行实际工作。

5. NodeManager

    管理 slave 节点的资源。

## 3 Hadoop 架构设计

### 3.1 HDFS的设计

Hadoop 以流式数据访问模式来存储超大文件，运行于商业硬件集群上。

![HDFS 架构](https://dyz-picbed.obs.cn-south-1.myhuaweicloud.com/02_hadoop/01_hdfsarchitecture.png)

1. **超大文件**

    "超大文件"在这里指具有几百 MB，几百 GB 甚至几百 TB 大小的文件。

2. **流式数据访问**

    HDFS 的构建思路是这样的：一次写入、多次读取是最高效的访问模式，数据集通常由数据源生成或从数据源复制而来，接着长时间在此数据集上进行各种分析。每次分析都将涉及该数据集的大部分甚至全部，因此读取整个数据集的时间延迟比读取第一条记录的时间延迟更重要。

3. **商用硬件**

    Hadoop 并不需要运行在昂贵且高可靠的硬件上，它是设计运行在商用硬件的集群上，因此至少对于庞大的集群来说，节点故障的概率还是非常高的。HDFS 遇到上述故障时，被设计成能够继续运行且不让用户察觉到明显的中断。

4. **低时间延迟的数据访问**

    要求低时间延迟数据访问的应用，例如几十毫秒范围，不适合在 HDFS 上运行，HDFS 是为高数据吞吐量应用优化的，这可能会以提高时间延迟为代价，对于低延迟的访问需求，HBase 是更好的选择。

5. **大量的小文件**

    由于 namenode 将文件系统的元数据存储在内存中，因此该文件系统所能存储的文件总数受限于 namenode 的内存总量。根据经验，每个文件、目录和数据块的存储信息大约占 150 字节。因此，距离来说，如果有一百万个文件，且每个文件占一个数据块，那至少需要 300 MB 的内存。尽管存储上百万个文件是可行的，但是存储数十亿个文件就超出了当前硬件的能力。

6. **多用户写入，任意修改文件**

    HDFS 中的文件写入只支持单个写入者，而且写操作总是以"只添加"方式在文件末尾写数据。它不支持多个写入者的操作，也不支持在文件的任意位置进行修改

### 3.2 HDFS 的概念

#### 3.2.1 数据块

和普通的文件系统相同，HDFS 同样也有（block）的概念，默认为 128 MB。HDFS 上的文件也被划分为块大小的多个分块（chunk），作为独立的存储单元。HDFS 中小于一个块大小的文件不会占据整个块的空间。

**HDFS 中块为什么默认 128 MB**

HDFS 中的块默认为 128 MB，是为了最小化寻址开销。如果块足够大，从磁盘传输数据的时间会明显大于定位这个块开始位置所需的时间。因此传输一个由多个快组成的大文件的时间取决于磁盘传输速率。

但是这个块大小的参数也不能设置的过大，MapReduce 中的 map 任务通常一次只处理一个块中的数据，因此如果任务数太少（少于集群中的节点数量），作业的运行速度就会比较慢。

**文件分块的好处**

1. 设计成分块，那么一个文件的大小可以大于网络中任意一个磁盘的容量。文件中的所有块并不需要存储在同一个磁盘上，分块后可以利用集群中任意一个机器进行存储。
2. 使用抽象块而非整个文件作为存储单元，简化了存储子系统的设计，简化了存储管理，同时也消除了对元数据的顾虑（块只要存储的大块文件，而文件的元数据，如权限信息，并不需要与块一起存储，这样的话其他系统就可以单独管理这些元数据）。
3. 块非常适用于数据备份进而提供数据容错能力和提高可用性。HDFS 中每个块的默认副本数为 3，使用机架感知，将这些块分布在不同的服务器上，既能减少集群中单个机器的读取负载，又能进行容错容灾。

#### 3.2.2 块缓存

通常 datanode 从磁盘读取块，但是对于访问频繁的文件，其对应的块可能被显式的缓存在 datanode 的内存中，以堆外内存（off-heap block cache）的形式存在。作业调度器（用于MapReduce、Spark和其他框架的）通过在缓存块的 datanode 上运行任务，可以利用块缓存的优势提高读操作的性能。例如，连接（join）操作中使用的一小的查询表就是块缓存的很好的候选。

### 3.3 HDFS 文件读取

![HDFS 文件读取](https://dyz-picbed.obs.cn-south-1.myhuaweicloud.com/02_hadoop/02_hdfsreadfile.png)

1. 客户端向 NameNode 发出写文件请求。
2. 检查是否已存在文件、检查权限。若通过检查，**直接先将操作写入EditLog**，并返回输出流对象。
    （注：WAL，write ahead log，先写 Log，再写内存，因为 EditLog 记录的是最新的HDFS客户端执行所有的写操作。如果后续真实写操作失败了，由于在真实写操作之前，操作就被写入 EditLog 中了，故 EditLog 中仍会有记录，我们不用担心后续 client 读不到相应的数据块，因为在第 5 步中 DataNode 收到块后会有一返回确认信息，若没写成功，发送端没收到确认信息，会一直重试，直到成功）
3. client 端按 **128 MB** 的块切分文件。
4. client 将 NameNode 返回的分配的可写的 **DataNode列表** 和 **Data数据** 一同发送给最近的第一个 DataNode 节点，此后 client 端和 NameNode 分配的多个 DataNode 构成 pipeline 管道，client 端向输出流对象中写数据。client 每向第一个 DataNode 写入一个packet，这个packet便会直接在 pipeline 里传给第二个、第三个…DataNode。
    （注：并不是写好一个块或一整个文件后才向后分发）
5. 每个 DataNode 写完一个块后，会返回**确认信息**。
    （注：并不是每写完一个 packet 后就返回确认信息，个人觉得因为 packet 中的每个 chunk 都携带校验信息，没必要每写一个就汇报一下，这样效率太慢。正确的做法是写完一个 block 块后，对校验信息进行汇总分析，就能得出是否有块写错的情况发生）
6. 写完数据，关闭输输出流。
7. 发送完成信号给 NameNode。
    （注：发送完成信号的时机取决于集群是强一致性还是最终一致性，强一致性则需要所有 DataNode 写完后才向 NameNode 汇报。最终一致性则其中任意一个 DataNode 写完后就能单独向 NameNode 汇报，HDFS 一般情况下都是强调强一致性）

### 3.4 HDFS 文件写入

![HDFS 文件写入](https://dyz-picbed.obs.cn-south-1.myhuaweicloud.com/02_hadoop/03_hdfswritefile.png)

1. client 访问 NameNode ，查询元数据信息，获得这个文件的数据块位置列表，返回输入流对象。
2. 就近挑选一台 datanode 服务器，请求建立输入流 。
3. DataNode 向输入流中中写数据，以 packet 为单位来校验。
4. 关闭输入流

### 3.5 读写过程中保证数据完整性

HDFS 通过校验和保证数据完整性。因为每个 chunk 中都有一个校验位，一个个 chunk 构成 packet ，一个个 packet 最终形成 block ，故可在 block 上求校验和。HDFS 的 client 端即实现了对 HDFS 文件内容的校验和（checksum）检查。当客户端创建一个新的 HDFS 文件时候，分块后会计算这个文件每个数据块的校验和，此校验和会以一个隐藏文件形式保存在同一个 HDFS 命名空间下。当 client 端从HDFS 中读取文件内容后，它会检查分块时候计算出的校验和（隐藏文件里）和读取到的文件块中校验和是否匹配，如果不匹配，客户端可以选择从其他 DataNode 获取该数据块的副本。