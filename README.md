# 论文笔记

This repo is inspired by [gaocegege/papers-notebook](https://github.com/gaocegege/papers-notebook).

记录阅读论文时的收获。

## 目录(TOC)

* [分布式(Distributed Systems)](#分布式distributed-systems)
    * [一致性(Consensus)](#一致性consensus)
        * [Paxos](#paxos)
        * [Raft](#raft)
    * [分布式跟踪(Tracing)](#tracing)
        * [Dapper](#dapper)
    * [存储(Storage)](#存储storage)
        * [The Google File System](#the-google-file-system)
* [数据库(Database)](#数据库database)
    * [主存数据库(MMDB)](#主存数据库mmdb)

## 分布式(Distributed Systems)

### 一致性(Consensus)

一说 Consensus 翻译成`共识`更恰当，以区分 consistency。

一致性是一组参与者就一个结果达成一致的过程，当参与者或它们之前的通信媒介可能经历失败时，这个问题变得困难。常见的分布式一致性协议有 Paxos、Raft。

#### [Paxos](https://en.wikipedia.org/wiki/Paxos_(computer_science))

*There is only one consensus protocol, and that’s Paxos-all other approaches are just broken versions of Paxos. -- Mike Burrows(Google)*

Paxos 是在不可靠的处理器网络中解决一致性的一系列协议。

#### Raft

### Tracing

#### Dapper

### 存储(Storage)

#### [The Google File System](https://static.googleusercontent.com/media/research.google.com/en//archive/gfs-sosp2003.pdf)

GFS 是基于 Google 的业务负载设计的一个分布式文件系统，因此具有一些独特的设计要点。

**与其他DFS相同的设计目标**

- performance
- scalability
- reliability
- availability

**设计假设**

- 由于 GFS 是构建在很多廉价的 commodity hardware 上，因此认为`故障`是正常现象而非异常
- 文件通常都是大文件（>100MB）
- 文件修改大都是 `append` 而非 `overwrite`

**设计要点及架构**

- 一个 master (HDFS namenode), 维护文件系统所有的元数据（文件和chunk的命名空间、文件到chunk的映射、chunk到chunkserver的映射）
- namespaces 和 file-to-chunk mapping 持久化到 operation log, 而 chunk 到 chunkserver 的映射关系是 master 向 chunkserver 索要的
- 多个 chunkserver (HDFS datanode), 文件数据被分为固定大小的 chunk(64MB) 保存在 chunkserver
- 三副本，分布在多个机器上并位于不同 rack
- master 和 chunckserver 之间通过心跳消息来传递指令并收集状态
- client 和 master 之间通信获取元数据，但数据的存取是直接和 chunkserver 进行的
- master 可以通过 operation log 来恢复命名空间，为了减少恢复时间，要求 operation log 不能过大，通过使用 checkpoint（compact B-tree like form） 来达到此目的
- 一致性模型，GFS 通过使用 `atomic record append` 来达到一个比较松弛（relaxed）的一致性模型，record append 使用的是 GFS 选择的 offset 而非应用指定的 offset
- GFS 使用租约（lease）来保证多副本间一致的更改顺序。master 授权其中一个 chunk 为 primary, 由它来确定多个更改的顺序
- 如果 record append 的数据超过了 chunk 的范围，会将每个 replica padding 到结尾。record append 的大小被限制为 16MB，以避免过多的空间浪费。
- 应用写的数据带 checksum，因而能验证读取数的有效性

HDFS 是 GFS 的开源实现。GFS 的一致性模型是我认为该文最难懂的地方，需结合 2.7 3.1 和 3.3 节多看几遍。

## 数据库(Database)

### 主存数据库(MMDB)

* [Main Memory Database Systems: An Overview](https://15721.courses.cs.cmu.edu/spring2016/papers/garciamolina-tkde1992.pdf)


