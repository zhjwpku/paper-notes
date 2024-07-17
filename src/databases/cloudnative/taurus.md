### [Taurus Database: How to be Fast, Available, and Frugal in the Cloud]()

> SIGMOD 2020
>
> https://dl.acm.org/doi/10.1145/3318464.3386129

本文介绍了一种名为 Taurus 的新型多云租户数据库系统的设计。Taurus 通过类似 Amazon Aurora 和 Microsoft Socrates 的方式分离计算和存储层，并提供类似的优势，如读副本支持、低网络利用率、硬件共享和可扩展性。然而，Taurus 架构具有几个独特的优势：

- 改进的可用性和可扩展性：与传统的本地部署相比，使用云数据库服务（DBaaS）越来越普遍，其主要优势包括提高可用性和可扩展性，并且成本更低。
- 异步步骤对延迟的影响：在 Taurus 的设计中，所有步骤都是异步的，只有前两个步骤位于前台路径中，可能会对延迟产生影响。
- 独特的复制和恢复算法：Taurus 提供了新颖的复制和恢复算法，使用相同数量或更少的副本，提供更好的可用性。

#### 架构

Taurus 包含 4 个主要组件：

- 数据库前端: 基于 MySQL 的修改版，包含 Master 实例（提供读写服务）和若干个 RO 实例（提供只读服务）。Master 实例生成 redo log，这些日志描述了事务对数据页的修改。Master 将 redo log 发送给 LogStore 和 PageStore，并周期性地通知 RO 实例最新的 LSN，以便 RO 实例可以读取到最新的 log
- SAL(Storage Abstraction Layer): 嵌入到计算层的库，隔离了远程存储、数据分片、故障恢复、主从复制等存储层的复杂性。主要负责计算引擎与存储层的 LogStore/PageStore 的交互，管理 PageStore 的 Slices，以及页面在 Slices 上的映射关系
- Log Store: 负责持久化日志（写入 PLog），事务的所有日志一旦持久化，计算层即可通知客户端事务提交完成。同时，为 RO 实例提供日志内容，RO 实例通过回放 redo log 来更新其缓冲区中的页面
- Page Store: 负责存储数据库的数据页，主节点更新 Page 的时候，通过 LSN 来表示这个 Page 的版本。PageID+LSN 可以唯一标识一个 page version，Page Store 根据 Master 或 RO 的读请求可以构造任意版本的 page version

Page Store 提供了几个主要的 API 供 SAL 调用:

```
(1) WriteLogs is used to ship a buffer with log records 
(2) ReadPage is used to read a specific version of a page 
(3) SetRecycleLSN is used to specify the oldest LSN of pages belonging to the same database that the front end might request (recycle LSN)
(4) GetPersistentLSN returns the highest LSN that the Page Store can serve
```


### Further readings

[Taurus MM: bringing multi-master to the cloud](https://www.vldb.org/pvldb/vol16/p3488-depoutovitch.pdf)
