# 论文笔记

This repo is inspired by [gaocegege/papers-notebook](https://github.com/gaocegege/papers-notebook).

I have changed this repo to a mdBook, run `mdbook serve` to read the notes locally.

## 目录(TOC)

* [数据结构](#数据结构data-structure)
    * [B-Tree](#btree)
        * [Organization and Maintenance of Large Ordered Indices](#organization-and-mantenance-of-large-ordered-indices)
        * [The Ubiquitous B-Tree](#the-ubiquitous-b-tree)
    * [LSM-Tree](#lsmtree)
        * [The Log-Structured Merge-Tree](#the-log-structured-merge-tree)
* [分布式(Distributed Systems)](#分布式distributed-systems)
    * [一致性(Consensus)](#一致性consensus)
        * [Paxos](#paxos)
        * [Raft](#raft)
    * [分布式跟踪(Tracing)](#tracing)
        * [Dapper](#dapper)
    * [存储(Storage)](#存储storage)
        * [The Google File System](#the-google-file-system)
        * [Bigtable: A Distributed Storage System for Structured Data](#bigtable-a-distributed-storage-system-for-structured-data)
* [数据库(Database)](#数据库database)
    * [主存数据库(MMDB)](#主存数据库mmdb)
* [Persistent Memory](#persistent-memory)
    * [System Evaluation of the Intel Optane Byte-addressable NVM](#system-evaluation-of-the-intel-optane-byte-addressable-nvm)
    * [An Empirical Guide to the Behavior and Use of Scalable Persistent Memory (FAST ’20)](#an-empirical-guide-to-the-behavior-and-use-of-scalable-persistent-memory)

## 数据结构(Data Structure)

### BTree

#### **[Organization and Maintenance of Large Ordered Indices](https://infolab.usc.edu/csci585/Spring2010/den_ar/indexing.pdf)**

B-tree 是由 Rudolf Bayer 和 Edward M. McCreight 在 1970 年发明的、适用于读取较大数据块（如 disk）的一种数据结构，广泛应用于数据库和文件系统作为索引（index）层。

B-tree 可以在插入或删除的时候一直保持树的平衡，当产生 underflow(due to deletion) 或 overflow(due to insertion) 时，可通过与邻居节点重重排数据来减少合并或分离的次数。[B-Tree Visualization](https://www.cs.usfca.edu/~galles/visualization/BTree.html) 提供了可视化的 B-tree 操作接口，try it。

#### **[The Ubiquitous B-Tree](http://carlosproal.com/ir/papers/p121-comer.pdf)**

> Unfortunately, a B-tree may not do well in a sequential processing environment. While a simple preorder tree walk [KNUT68] extracts all the keys in order, it requires space for at least h = logd(n + 1) nodes in main memory since it stacks the nodes along a path from the root to avoid reading them twice.

> Additionally, processing a next operation may require tracing a path through several nodes before reaching the desired key.

**B\* Tree**

B tree 在插入的时候会产生 overflow，可以通过 redistribution 来减少分裂的次数，但真正分离的时候采取的是一分为二的做法，因此只能保证 1/2 的空间利用率。B* Tree 同样是通过 redistribution 来减少分离次数，更进一步，只在两个邻居节点都满的时候讲两个节点分离成三个节点，从而使得每个节点达到 2/3 full，降低了树的总高，进而优化检索速度。

但实际中可能应用的并不多。

**B+ Tree**

B+ Tree 在保持树平衡的同时，其所有叶子节点通过指针串联在一起(left-to-right)，因此在顺序遍历下一个 key 的时候，最多只需要一次读盘，而且内存中只需要驻留一个 node。所有串联起来的叶子节点被称为 `sequence set`。

B+ Tree 的索引节点和叶子节点通常使用不同的排布格式，因为索引节点不需要 value 信息。

非页节点中的 key 作为 `index`，包含了部分 key。B+ Tree 的插入删除操作可在 [B+ Tree Visualization](https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html) 进行实际操作。

> 另外值得注意的是，在删除数据的时候，可以在 index 中保留删除的 key 以简化操作。

当 B+ Tree 用于数据库这样有多并发的应用事，需要考虑`锁`相关的内容。由于插入和删除都有可能更改上层的节点，并发会导致竟态。

**Prefix B+ Tree**

*注：[BAYE77][baye77] 将 B+ Tree 叫做 B\* Tree*

由于 B+ Tree 不必使用 key 来构建 `index`，据此提出了用其它较短 string 来构建 B+ Tree 索引，从而压缩树的高度、提升检索效率的 Prefix B+ Tree。

### LSMTree

#### **[The Log-Structured Merge-Tree](https://www.cs.umb.edu/~poneil/lsmtree.pdf)**

LSM-tree 在存储领域备受关注，不同于 B-Tree 实时写入带来的性能开销，LSM-tree 在将数据写入 Log（sequential log file） 及 内存之后就返回结果，将下盘操作延后并批量化，从而将每次写操作的磁盘寻道及转动的时延平均，从而提高了写性能。

**论文要点**

- LSM-Tree 适用于写远大于读的场景
- LSM-Tree 由一个内存常驻的 C0 tree 及多个磁盘常驻的 Ck tree 组成
- C0 tree 可以实现为 (2-3) tree 或 AVL tree，而 Ck tree 实现为 B-tree
- C0 tree 在达到一定阈值之后会将一部分数据迁移到 C1 tree，同样，Ck-1 tree 在达到一定阈值之后会迁移到 Ck tree，此过程称为 onging `rolling merge` process
- merge 过程中会用到 Multi-page block（256K bytes == 64 page）IO，从而提高吞吐
- merge 过程中产生的新数据不覆盖旧数据
- 为了能快速 recover，实现会不定时对 C0 tree 做 checkpoint

论文分析了要达到一定 I/O B-tree 和 LSM-Tree 的成本对比，以及不同 Component 大小比例调优的过程，详细内容见原文。

## 分布式(Distributed Systems)

### 一致性(Consensus)

一说 Consensus 翻译成`共识`更恰当，以区分 consistency。

一致性是一组参与者就一个结果达成一致的过程，当参与者或它们之前的通信媒介可能经历失败时，这个问题变得困难。常见的分布式一致性协议有 Paxos、Raft。

#### **[Paxos](https://en.wikipedia.org/wiki/Paxos_(computer_science))**

*There is only one consensus protocol, and that’s Paxos-all other approaches are just broken versions of Paxos. -- Mike Burrows(Google)*

Paxos 是在不可靠的处理器网络中解决一致性的一系列协议。

#### Raft

### Tracing

#### Dapper

### 存储(Storage)

#### **[The Google File System](https://static.googleusercontent.com/media/research.google.com/en//archive/gfs-sosp2003.pdf)**

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

HDFS 是 GFS 的开源实现。GFS 的一致性模型是我认为该文最难懂的地方，需结合 2.7 3.1 和 3.3 节多看几遍。

Google Filesystem: Architecture + Consistency Model Overview [Part 1](https://www.youtube.com/watch?v=64ioICo0YBo) & [Part 2](https://www.youtube.com/watch?v=kVY_3CNPjhk)

#### **[Bigtable: A Distributed Storage System for Structured Data](https://static.googleusercontent.com/media/research.google.com/en//archive/bigtable-osdi06.pdf)**

Bigtable 是 Google 用来存储结构化数据的分布式存储系统，底层使用 GFS 存储数据和日志文件。具有可扩展，高性能，高可用性等特点。Apache HBase 是基于 Bigtable 的一个开源实现。

**设计要点**

- Bitable 没有固定的 schema，存储的数据往往是稀疏的，且是多维有序（multidimentional sorted map）的
- (row:string, column:string, time:int64) --> string
- 在逻辑视图层面，包含 table, row, column, version, cell 等概念，类似 RDMS
- 在物理视图层面由于 Column Familiy 概念的存在，数据的存储是按照 K/V 来存储的，且相同 CF 的数据有序排列在一起
- 由于 table 可以很大，且数据是按照 row key 有序存储的，因此可以将 table 划分为多个 tablet 存储在不同的 tablet server
- Column Family 作为最基本的访问控制单元，在将数据存储到对应 CF 之前，CF 必须已经创建
- 一个表的 CF 不能太多（最多几百），但一个 table 的 column 数量可以无限
- 每个 cell 可以包含多个版本的数据，使用 timestamp 作为索引
- Bigtable 只支持单行事务

**SSTable, MemTable, CommitLog**

Bigtable 的使用 SSTable(Sorted Strings Table) 保存持久化数据，包括数据文件和日志文件。

> An SSTable provides a persistent, ordered immutable map from key to values, where both keys and values are arbitray byte strings.

SSTable 文件保存了一些列连续的块，通常为 64KB，在文件的结尾处保存着用于定位块的索引（block index），该索引通常在 open SSTable 的时候被装载进内存，这样查找数据的时候可以先在内存中对 block index 进行二分查找，然后一次读盘就能找到 key 对应的数据。

MemTable 是存储在内存中的一个有序的 buffer，更新首先以 redo log 被记录在 commit log 中，然后插入到 Memtable 中。

commit log 用于失效回放，MemTable 在达到一定阈值之后会转换为 SSTable，并最终与其它 SSTable 合并为一个 SSTable，该过程称为 Compaction，详细介绍见 5.4 节。

如上三种结构保证了数据能够有效地持久化并不丢失。

**架构**

- Bigtable 是一个典型的主从结构，依赖 chubby 提供的分布式锁服务来保证同一时刻只有一次 active master
- chubby 除了提供上述的分布式锁服务，
    - 还保存了 bigtable 的 schema 信息（每个 table 的 column family 信息）
    - 用于发现 tablet server 及终止 tablet server
- 一主多从
    - master 负责将 tablet 分配给 tablet server, 发现 tablet server 的加入或结束，均衡 tablet server 的负载，GC 以及表信息的更改
    - 每一个 tablet server 管理了一系列 tablet，并处理相应 tablet 的读写，另外 tablet server 还负责 tablet 的分裂
    - 每一个 tablet 存储了一段 key 范围内的所有数据，每个 tablet 通常大小为 100-200MB
- 客户端直接与 tablet server 进行读写请求
- 为了减少读盘次数，为 SSTable 指定 Bloom filter 以提高读数据的效率

Bigtable 中 Column Family 是我认为相对难懂一点的概念，读者可以结合 HBase 的介绍及操作来加速理解。下面是几个 HBase 的相关视频：

1. [Apache HBase - Just the Basics](https://www.youtube.com/watch?v=2Ci_QxJ1kiE)
2. [HBase Tutorial For Beginners](https://www.youtube.com/watch?v=V1fXSCASVDc)
3. [Introductio to HBase Command Line](https://www.youtube.com/watch?v=_T9-Hmp1mEY)

## 数据库(Database)

### 主存数据库(MMDB)

#### **[Main Memory Database Systems: An Overview](https://15721.courses.cs.cmu.edu/spring2016/papers/garciamolina-tkde1992.pdf)**

## Persistent Memory

持久内存作为一种新型存储介质，弥补了内存和SSD介质之间的性能鸿沟，PMem 的不易失（相对DRAM），容量大（相对DRAM），时延小（相对SSD）等特性。

#### **[System Evaluation of the Intel Optane Byte-addressable NVM](https://arxiv.org/pdf/1908.06503.pdf)**

持久内存具有高密度、低功耗、成本低（per bit）的特性，同时也存在一些劣势: 3~20 倍于 DRAM 的访问时延、低带宽、读写性能不对称。作者对 DRAM 和 PMem 在不同配置下的时延、带宽、耗电等特性进行了测试和分析，其结论对持久内存编程的性能分析及优化具有一定启发。

- Using DRAM as a cache to NVM can effectively bridge the performace gap and brings performace close to DRAM
- Coordinating 256B accesses to PMM to to exploit locality may reduce latency and write-amplification
- Explicit data placement that utilizes local PMM could mitigate high cost of accessing DRAM on the remote socket

#### **[An Empirical Guide to the Behavior and Use of Scalable Persistent Memory](https://www.usenix.org/system/files/fast20-yang.pdf)**

在 Optane DC Persistent Memory 出现之前，很多学者使用仿真的方式研究 NVM，本文通过对 Optane DIMM 的实验对先前的经验进行了分析修正，并给出了使用 Optane DIMM 的四个原则:

1. 避免小于 256 B 的随机读写
2. 尽可能使用 ntstore（non-temporal stores）进行大数据的写，并控制 Cache 的淘汰
3. 限制访问 Optane DIMM 的并发线程数
4. 避免NUMA访问（尤其是read-modify-write 操作序列）

[baye77]: https://dl.acm.org/doi/pdf/10.1145/320521.320530


