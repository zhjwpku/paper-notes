### [Amazon Aurora: Design Considerations for High Throughput Cloud-Native Relational Databases](../../assets/pdfs/aurora-sigmod-17.pdf)

> SIGMOD 2017
>
> https://www.amazon.science/publications/amazon-aurora-design-considerations-for-high-throughput-cloud-native-relational-databases

本文描述了 Aurora 的架构以及设计考虑，高吞吐量数据处理的主要瓶颈已从计算和存储转移到网络，Aurora 通过将 redo log 处理推送到一个专用的多租户可扩展的存储服务，减少了网络流量，同时允许快速崩溃恢复、无数据丢失的副本故障切换。

在分布式云服务中，通过计算与存储分离（存算分离）和多节点存储复制，增加了云服务的弹性和可扩展性。在这种环境中，传统数据库系统面临的 I/O 瓶颈发生变化。由于 I/O 可以分散到多节点和多磁盘的集群中，单个磁盘和节点不再是热点。瓶颈转移到请求 I/O 的数据库层与执行 I/O 的存储层之间的网络。最慢的存储节点、磁盘或网络路径的性能会主导响应时间。

数据库中的大多数操作可以重叠进行，但某些情况需要同步操作，导致停滞和上下文切换。例如，缓存未命中的磁盘读取会导致读线程等待，可能还需要缓存替换和刷脏缓存页。checkpoint 和脏页写入等后台处理可以减少这种问题，但也会引起停滞、上下文切换和资源争用；事务提交是另一个干扰源，提交停滞会阻碍其它事务。两阶段提交（2PC）在云分布式系统中的处理也具有挑战性，因为这些协议不容忍故障且延迟高。

Aurora 通过更激进地使用 redo log 来解决上述问题，每个实例仍包含传统数据库内核的大部分组件（查询处理器、事务、锁定、缓冲缓存、访问方法和 undo 管理），但一些功能（redo log、持久存储、崩溃恢复和备份/恢复）被转移到存储服务中处理。

Aurora 的架构相比传统数据库的三大优势:

- 通过在多个数据中心构建独立的容错和自愈存储服务，保护数据库免受网络或存储层的性能波动和临时或永久故障的影响
- 仅将 redo log 记录写入存储服务，将网络 IOPS 降低了一个数量级
- 将一些最复杂和最关键的功能（备份和重做恢复）从数据库引擎中的一次性昂贵操作转变为跨大规模分布式集群的连续异步操作，实现了无需 checkpoint 的近乎即时崩溃恢复以及不影响前台处理的低成本备份

#### DURABILITY AT SCALE

Aurora 使用了 6 副本的复制机制，每个数据在三个 AZ 中分别复制两次，这样能够容忍单个 AZ + 一个额外节点的丢失（AZ + 1）而不丢失数据。

- write quorum 4/6 (Vw = 4)
- read quorum 3/6 (Vr = 3)

三副本则不能解决这样的问题，因为在分布式系统中，磁盘损坏被认为是正常的现象，如果同时有一个 AZ 因为自然灾害丢失，那么会失去两个副本，无法确定第三个副本是否是最新的。

Aurora 的单个 AZ 放置两个副本对于常见的磁盘故障恢复能够更快从本 AZ 恢复数据。

为了进一步减少 MTTR (Mean Time to Repair)，Aurora 将数据库卷分割成小的固定大小的段（10GB，故障和修复的基本单位），这些段以 6 副本的方式分布到 Protection Groups (PG) 中，每个 PG 由六个 10GB 段组成，分布在三个 AZ 中，每个 AZ 管理单个 PG 的两个段。通过这样的方式，Aurora 能够在一个副本出现故障时快速修复（在 10Gbps 的网络链接上，修复一个 10GB 的段需要10秒），提供高可用性。

如果在同一个 10 秒窗口内看到两个这样的故障，再加上一个不包含这两个独立故障的 AZ 故障，才会导致数据库的不可用，但根据 AWS 对客户大量数据库的观察，这种情况极不可能发生。

#### THE LOG IS THE DATABASE

![Network IO in mirrored MySQL](/assets/images/aurora_network_io_in_mirrored_mysql.png)

在云上，传统的数据库会有严重的写放大问题，比如上图是一个同步主备的 MySQL 配置，AZ1 中有一个活跃的 MySQL 实例，数据写入到 EBS 存储上（networked storage）；同时，在 AZ2 中有一个备用的 MySQL 实例，也使用 EBS 进行存储。重做日志、binary log（为了支持时间点恢复，归档到Amazon S3）、修改后的数据页、Double write（双写，防止页面撕裂）以及元数据（FRM）文件都通过网络进行写入，由于很多写入是顺序的，延迟会累加。

在 Aurora 中，跨网络传输的唯一写入是 redo log。数据库层不写入页面，无论是后台写入、检查点还是缓存页换出时刷脏页。相反，log applicator 下沉到存储层，在那里它可以用于后台或按需生成数据库页面。

显然，从时间开始的所有修改链生成每个页面是极其昂贵的。因此，后台会不断地物化数据库页面，以避免每次按需生成它们。需要注意的是，从正确性的角度来看，后台物化是可选的：对于引擎而言，**日志就是数据库**，存储系统物化的任何页面只是日志应用的缓存。还要注意的是，与检查点不同，只有具有长修改链的页面需要重新物化。检查点由整个重做日志链的长度决定，而 Aurora 页面物化则由给定页面的链长度决定。

![Network IO in Amazon Aurora](/assets/images/aurora_network_io_in_aurora.png)

Aurora 的方法尽管增加了复制所需的写入量，但显著降低了网络负载，上图展示了一个 Aurora 集群，其中包含一个主实例和多个跨多 AZ 部署的副本实例。主实例仅将日志记录写入存储服务，并将这些日志记录以及元数据更新流式传输到副本实例。I/O 流程基于共同的目的地（一个逻辑段，即一个PG）对完全有序的日志记录进行批处理，并将每个批次交付给所有 6 个副本，在那里日志被持久化到磁盘上，数据库引擎等待 6 个副本中的 4 个确认，以满足 write quorum，并认为发送的日志记录是持久的。副本使用重做日志记录对其缓存进行更改。

上面的处理下移至 storage service 还能够最小化故障恢复的时间，同时消除了 checkpoint、后台数据页写入、备份等操作引起的抖动。在传统数据库中，崩溃后系统必须从最近的检查点开始，重放日志以确保所有持久化的 redo log 都被应用。在 Aurora 中，redo log 的回放发生在存储层，这是连续的、异步的，并分散在整个集群中。任何对数据页的读取请求如果页面不是最新的，可能需要应用一些重做记录。因此，崩溃恢复的过程分散在所有正常的前台处理中。在数据库启动时无需进行任何操作。

Aurora 存储服务的一个核心设计原则是最小化前台写请求的延迟。大部分存储处理被移至后台。鉴于前台请求与存储层的平均请求之间自然存在的变化性，存储层有充足的时间在前台路径之外执行这些任务，并有机会用 CPU 换取磁盘性能。例如，在存储节点忙于处理前台写请求时，除非磁盘接近容量，否则不必运行旧页面版本的垃圾收集（GC）。在 Aurora 中，后台处理与前台处理呈负相关。这与传统数据库不同，在传统数据库中，页面的后台写入和检查点与系统上的前台负载呈正相关。

![IO Traffic in Aurora Storage Nodes](/assets/images/aurora_traffic_in_storage_node.png)

- (1) receive log record and add to an in-memory queue
- (2) persist record on disk and acknowledge
- (3) organize records and identify gaps in the log since some batches may be lost
- (4) gossip with peers to fill in gaps
- (5) coalesce log records into new data pages
- (6) periodically stage log and new pages to S3
- (7) periodically garbage collect old versions
- (8) periodically validate CRC codes on pages.

上述的步骤中只有（1）和（2）位于可能影响延迟的前台路径中

#### THE LOG MARCHES FORWARD

Aurora 通过将重做日志记录传输至存储层，简化了日志生成、持久状态、运行时状态和副本状态的一致性维护，避免了昂贵的两阶段提交协议（2PC）。它通过异步处理方式，在收到存储请求的确认后，维护并推进一致性点和持久性点。

数据库事务被分解为多个迷你事务（MTR），每个 MTR 由多个连续的日志记录组成，最后一个日志记录被标记为 CPL（Consistency Point LSN），确保一致性。

在崩溃恢复中，Aurora 依赖存储服务来恢复日志记录，确保数据库看到一致的存储视图。存储服务确定 VCL（Volume Complete LSN）和 VDL（Volume Durable LSN），并在恢复过程中截断高于 VDL 的日志记录。数据库在重启时，通过读取每个保护组（PG）的一致性段，重新计算 VDL 并生成截断范围。此外，数据库需要执行撤销恢复，来逆转崩溃时未完成的事务操作。这些操作在数据库上线后进行，确保系统恢复过程中不中断用户操作。

在正常操作中，Aurora 持续与存储服务交互，保持状态以建立仲裁、推进持久性并登记提交的事务。写操作中，随着每批日志记录的写入仲裁确认，当前 VDL 不断推进。提交操作异步完成，处理提交请求的线程会将事务记录在等待提交的列表中，然后继续处理其它任务。读操作中，Aurora 保证缓存中的页面总是最新版本，通过从存储节点获取最新的页面版本，避免了传统数据库中页面被替换前需要写入磁盘的问题。副本操作中，最多可支持15个读副本，它们通过消费写入器生成的日志流保持同步。副本异步处理日志记录，并确保只应用 VDL 以下的日志记录，同时保证单个迷你事务内的日志记录原子性地应用到缓存中。

#### PUTTING IT ALL TOGETHER

![Aurora Architecture: A Bird's Eye View](/assets/images/aurora_architecture_bird_eye_view.png)


### Further readings

- [MIT 6.824 Lecture 10: Cloud Replicated DB, Aurora](https://www.youtube.com/watch?v=jJSh54J1s5o)
- [AWS re:Invent 2022 - Deep dive into Amazon Aurora and its innovations](https://www.youtube.com/watch?v=pzZydB78Eyc)
- [AWS re:Invent 2023 - Deep dive into Amazon Aurora and its innovations](https://www.youtube.com/watch?v=je6GCOZ22lI)
