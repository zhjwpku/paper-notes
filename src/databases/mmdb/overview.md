### [Main Memory Database Systems: An Overview](../../assets/pdfs/main_memory_database_systems_an_overview.pdf)

> IEEE Transactions on Knowledge and Data Engineering, H. Garcia-Molina, 1992
>
> https://dl.acm.org/doi/10.1109/69.180602

这是一篇 1992 年的 paper，虽然那时的内存容量可能只有几十兆上百兆，但 MMDB 已经开始崭露头角，人们希望通过将数据全量驻留在内存来提供更快的响应时间及更大的事务吞吐。传统数据库（DRDB）可以将磁盘数据缓存在内存中加速查询，MMDB 需要将数据在磁盘备份以防数据丢失。

`MMDB` 和 `DRDB with a very large cache` 有何不同呢？DRDB 在设计的时候假设内存不足以保存所有数据，因此没有充分利用内存的特性。而 MMDB 在设计之初就假设数据可以全量存储在内存，其各模块都存在着一些与 DRDB 不同的点。

#### Concurrency Control

1. 访存远快于读盘，因而事务能够快速结束，同时意味着锁不会长时间持有，可以锁的粒度可以加大
2. 由于数据存储在内存，因此锁机制可以通过数据对象的少量比特位来实现

#### Commit Processing

数据需要通过日志持久化，对于 MMDB，日志的开销占比显得非常突出。解决该问题的办法包括：

1. Stable main memory 可以缓解事务的响应时间但不能消除日志瓶颈
2. Precommit 事务写完日志就可以响应请求
3. Group Commit 将多个事务的日志攒批落盘，可减少 IO

#### Access Method

不再需要 B-Tree，索引可以通过排序的链表来实现。

#### Data Representation

关系元组可以通过一组指针来表示。

#### Query Processing

顺序访问与随机访问的速度对于内存而言差别不大，因此依赖快速顺序访问的查询算法不再具有优势，如 sort-merge join。但 process cost 变得难以预测。

#### Recovery

在 MMDB 中，Checkpoint 和 Recovery 是需要访问磁盘数据的两个唯一原因。因此磁盘访问可以对 Checkpionter 进行优化，比如使用加大的 block size。在恢复的时候可以采用 `on demand` 的方式从磁盘恢复数据或使用磁盘分配或磁盘阵列来并行读取数据以加速恢复。

#### Performance

传统数据库备份对性能的影响要远小于 MMDB，因此 checkpoint 算法在 MMDB 显得至关重要。

#### Application Programming Interface and Protection

1. Once the transaction can access the database directly, they can read or modify unauthorized parts
2. and the system has no way of knowing what has been modified, so it can not log the cahnges

最佳的解决方案是只运行由特定数据库系统编译器编译过的事务。

#### Data Clustering and Migration

Migration and dynamic clustering are components of a MMDB that have no counterpart in conventional database systems.