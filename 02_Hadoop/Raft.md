# Raft

## 1 Raft是什么

```wiki
Raft is a consensus algorithm designed as an alternative to the Paxos family of algorithms. It was meant to be more understandable than Paxos by means of separation of logic, but it is also formally proven safe and offers some additional features. Raft offers a generic way to distribute a state machine across a cluster of computing systems, ensuring that each node in the cluster agrees upon the same series of state transitions. It has a number of open-source reference implementations, with full-specification implementations in Go, C++, Java, and Scala. It is named after Reliable, Replicated, Redundant, And Fault-Tolerant.
```

Raft 是一种共识算法，旨在替代 Paxos 系列算法。 通过逻辑分离，它比 Paxos 更易于理解，但是它也被证明是安全的，并提供了一些附加功能。Raft 提供了一种在计算系统集群中分布状态机的通用方法，可确保集群中的每个节点都同意相同的一系列状态转换。它具有许多开源参考实现，并具有 Go，C ++，Java 和 Scala 的完整规范实现。 Raft（Reliable, Replicated, Redundant, And Fault-Tolerant）是这四个单词的首字母缩写。

**主要参考信息**：http://thesecretlivesofdata.com/raft/ *Raft可视化显示*

## 2 实现细节

### 2.1 