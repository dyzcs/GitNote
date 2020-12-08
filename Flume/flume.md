# Flume

## 1. Flume 概述

### 1.1 Flume定义

Flume 是一个高可用的，高可靠的，分布式的**海量日志采集、聚合和传输的系统**。Flume 基于流式架构，灵活简单。

![image-20201208202715280](flume.assets\image-20201208202715280.png)

**Flume 最主要的作用就是，实时读取服务器本地磁盘的数据，将数据写入到 HDFS。**

### 1.2 Flume 基础架构

Flume 组成结构如图所示：

![image-20201208202932593](flume.assets\image-20201208202932593.png)

#### 1.2.1 Agent

Agent 是一个 JVM 进程，它以**事件**的形式将数据从源头送至目的。

Agent 主要有 3 个部分组成，**Source**、**Channel**、**Sink**。

#### 1.2.2 Source

Source 是负责接收数据到 Flume Agent 的组件。Source 组件可以处理各种类型、各种格式的日志数据，包括 **avro**、thrift、**exec**、jms、**spooling directory**、**netcat**、sequence generator、syslog、http、legacy。

#### 1.2.3 Sink

Sink 不断地轮询 Channel 中的事件且批量的移除它们，并将这些事件批量写入到存储或索引系统、或者被发送到另一个 Flume Agent。

Sink 组件目的地包括 **hdfs**、**logger**、**avro**、thrift、ipc、**file**、**HBase**、solr、自定义。

#### 1.2.4 Channel

Channel 是位于 Source 和 Sink 之间的缓冲区。因此，Channel 允许 Source 和 Sink 运作在不同的速率上。Channel 是线程安全的，可以同时处理几个 Source 的写入操作和几个 Sink 的读取操作。

Flume 自带两种 Channel：**Memory  Channel** 和 **File Channel** 以及 **Kafka Channel**。

Memory Channel 是内存中的队列。Memory Channel 在不需要关心数据丢失的情境下使用。如果需要关心数据丢失，那么 Memory Channel 就不应该使用，因为程序死亡、机器宕机或者重启都会导致数据丢失。

File Channel 将所有事件写入到磁盘，因此在程序关闭或机器宕机的情况下不会丢失数据。

#### 1.2.5 Event

传输单元，Flume 数据传输的基本单元，以 Event 的形式将数据从源头送至目的地。Event 由 **Header** 和 **Body** 两部分组成，Header 用来存放该 event 的一些属性，为 K-V 结构，Body 用来存放该条数据，形式为字节数组。

![image-20201208204943007](flume.assets\image-20201208204943007.png)