### [Staring into the Abyss: An Evaluation of Concurrency Control with One Thousand Cores](../../assets/pdfs/staring_into_the_abyss.pdf)

> PVLDB, November 2014
>
> https://dl.acm.org/doi/10.14778/2735508.2735511

计算机架构不再追求单核的时钟频率，越来越多的 CPU 转向 multi-core on a sinle Chip 的架构，out-of-order、super-scalar 开始逐渐被简单的 in-order、single issue 所代替。

> We are entering the era of the many-core machines that are powered
> by a large number of these smaller, low-power cores on a single chip.

在这种趋势下，单节点、共享内存 OLTP DBMS 的扩展性显得尤为重要，如果现有的技术不能适应这种超多核架构，会出现各种瓶颈并造成计算资源的浪费。

本论文对并发控制的可扩展性进行了研究，通过模拟在 1000 核的 CPU 上运行一个支持插拔 **lock manager** 进而替换不同并发控制算法的 OLTP 内存数据库，表明了现有的并发控制协议都不能随 core 数量的增加而扩展。

> Our analysis shows that all algorithms fail to scale as the number
> of cores increases. In each case, we identify the primary bottlenecks
> that are independent of the DBMS implementation and argue
> that even state-of-the-art systems suffer from these limitations.

#### Concurrency Control Schemas

OLTP 负载中的事务通常具有三个显著特点:

1. they are short-lived
2. they touch a small subset of data using index look-ups (no full table scans or large joins)
3. they are repetitive (i.e., executing the same queries with different inputs)

Concurrency Control 为数据库提供了 ACID 中的 `Atomicity` 和 `Isolation`。所有的 Concurrency Control 都可归为以下两类:

1. Two-Phase Locking (Pessimistic): Assume transactions will conflict so they must acquire locks on database objects before they are allowed to access them.
   - different transactions cannot simultaneously own conflicting locks
   - once a transaction surrenders ownership of a lock, it may never obtain additional locks
2. Timestamp Ordering (Optimistic): Assume that conflicts are rare so transactions do not need to first acquire locks on database objects and instead check for conflicts at commit time.
   - a transaction is assigned a unique, monotonically increasing timestamp before it is executed
   - this timestamp is used by the DBMS to process conflicting operations in the proper order

2PL 大大减少了死锁的概率，但却不能避免死锁的发生，因此不同的变种来解决死锁的问题:

1. 2PL with Deadlock Dection (DL_DETECT): 维护一个 `wait-for graph` 并检查是否有环来判断是否出现了死锁，当检测到有环出现时，系统需要 abort 一个事务来打破这个环
2. 2PL with Non-waiting Deadlock Prevention (NO_WAIT): 当获取不到锁时，直接将自己 abort 掉，避免死锁的出现
3. 2PL with Waiting Deadlock Prevention (WAIT_DIE): 事务允许等待比它新的事务持有的锁，如果要获取的锁被一个较老的事务持有，则将自己 abort 掉，这种算法需要在事务开始时获取一个时间戳，T/O 保证了不会出现死锁

T/O 根据冲突检查的粒度（行级或分区级）和检查的时机（事务运行中或事务结束时）可以分为如下几个变种:

1. Basic T/O (TIMESTAMP): 事务开始时获得时间戳，如果事务的时间戳小于元组最后一次写的时间戳，则拒绝读写；如果事务的时间戳小于元组的最后一次读时间戳，则拒绝写。该变种还需要拷贝元组来保证 `repeatable read`
2. Multi-version Concurrency Control (MVCC): 每个写操作都会创建一个新的版本，新版本标记有穿件它的时间戳。该变种的优点是 **读不阻塞写，写不阻塞读**
3. Optimisitic Concurrency Control (OCC): 事务开始时不分配时间戳，**Read Phase** 将需要读写的元组及写操作都记录在事务私有的工作区，**Validation Phase** 对要写的元组 **加锁** 并检查是否可以有冲突，**Write Phase** 则将没有冲突的事务修改 merge 到数据库并 **释放锁**
4. T/O with Partion-level Locking (H-STORE): 将数据库分为多个 partition，每个 partition 被一把锁保护，事务到来时分配一个时间戳并将事务放到所有与事务相关的目标分区的 `lock acquisition queue` 中，执行引擎根据队列中事务的时间戳大小来执行相应的事务

#### 结论

实验显示不同并发控制算法在 multi-core 架构上的 Scalability 都遇到了瓶颈:

- **Lock Thrashing**: 一个事务的等待会造成另外一个事务更长的等待时间
  - DL_DETECT
  - WAIT_DIE
- **Timestamp Allocation**: 所有需要分配时间戳的算法都会遇到时间戳分配的问题
  - All T/O algorithms
  - WAIT_DIE
  - HSTORE
- **Memory Allocations**: 因为需要拷贝元组需要分配内存
  - OCC
  - MVCC
- **Abort Rate**: 由于冲突造成的事务回滚
  - OCC
  - NO_WAIT

具体的结果数据参见原论文。

#### References

[1] [CMU 15-721 In-Memory Databases](https://www.youtube.com/watch?v=a70jRWLjQFk&list=PLSE8ODhjZXjasmrEd2_Yi1deeE360zv5O&index=3) and [the note](https://15721.courses.cs.cmu.edu/spring2020/notes/02-inmemory.pdf) <br>