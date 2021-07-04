### [File Systems Unfit as Distributed Storage Backends: Lessons from 10 Years of Ceph Evolution](../assets/pdfs/FileSystemsUnfitAsDistributedStorageBackends-lessonsFromTenYearsOfCephEvolution.pdf)

> SOSP 2019
> 
> https://doi.org/10.1145/3341301.3359656

很多分布式文件系统选择将其存储后端构建在本地文件系统之上，如 ext4, XFS，原因在于:

- 将持久化及磁盘空间管理的工作交给了健壮的内核代码
- 本地文件系统提供了 Posix 接口及文件目录抽象
- 可以使用标准工具（ls, find）来查看数据内容

Ceph 也不例外，在其架构演进的进程中前后使用过 Btrfs，XFS 等文件系统作为其 FileStore 的后端存储，但在遵循这种惯例 10 年后，发现以本地文件系统作为存储后端有着昂贵的代价:

- developing a zero-overhead transaction mechanism is challenging
- metadata performance at the local level can significantly affect performance at the distributed level
- supporting emerging storage hardware is painstakingly slow

第一，在现有的文件系统之上实现高效的事务非常困难（性能开销、功能限制、接口复杂度或者实现难度），有三种可做尝试的方法：

1. hooking into a file system’s internal (but limited) transaction mechanism
2. implementing a WAL in user space
3. using a key-value database with transactions as a WAL

Ceph 在尝试上述三种方式时都遇到了一些问题，如使用文件系统内置的事务机制会造成数据部分提交；在用户态实现 WAL 或使用 KV数据库作为 WAL 都会遇到大量 Read-Modify-Write、操作非幂等引起进程重启后数据不一致、数据双写等问题。

第二，元数据的性能低效会扩散到整个集群。Ceph 团队遇到的一个挑战是 Posix 文件系统的 readdir 操作返回结果是乱序的，为了保证排序的高效，需要对目录进行分裂，这就会影响全局性能。

第三，由于本地文件系统通常运行在内核态，当 Ceph 想要支持一个新的硬件（如 SMR、ZNS）的时就需要等待文件系统维护者的排期。

在 2015 年，Ceph 开始设计并实现 BlueStore —— 一个用户态的存储后端，数据直接存储在裸盘之上，元数据存储在 kv 中。它实现了如下目标：

- Fast metadata operations
- No consistency overhead for object writes
- Copy-on-write clone operation
- No journaling double-writes
- Optimized I/O patterns for HDD and SSD

为了达到快速元数据操作，BlueStore 使用 RocksDB 来存取元数据。为了能让数据和元数据共存底层设备，BlueStore 实现了一个能运行 RockesDB 的极简文件系统 BlueFS，BlueFs 具有一下特点：

- 为每个文件维护一个 inode，inode 记录了文件使用的 extent 数组
- 目录只有一层，不存在目录嵌套
- 元数据都在内存(super block、dirmap、filemap)
- super block 存储在 offset = 4096 的磁盘位置，其中保存了 ino = 1 的 journal 文件 inode
- 所有对文件系统的更改操作（如 mkdir, open, delete）都保存在 ino = 1 的文件中
- 通过回放来获取 Dir/File 视图，然后通过 fileMap 来重构 Allocator 的可用块

通过这种方式，journal 和其他文件的 extent 是交错存放在整个磁盘上的，而磁盘的分配由 Space Allocator 统一管理。

对于大块写入，BlueStore 首先将数据写入新分配的 extent 中，一旦数据被持久化，相应的元数据被插入到 RocksDB，如此可以高效地支持克隆操作并能避免日志双写；对于小于 allocation size 的写入，数据和元数据都被插入 RocksDB，然后再异步写入磁盘。

BlueStore 实现为用户态并通过 DirectIO 直接访问裸盘，因此不实用 kernel 提供的 page cache。为了能提供高效的读写，BlueStore 是实现了它自己的 write-through cache，使用 2Q 算法并对数据进行 shard 来支持并发。

由于控制了所有的IO路径，Ceph 能更灵活高效地提供用户需要的功能特性。

论文中在开头部分提到：

- Stonebraker: operating systems offer all things to all people at much higher overhead
- exokernels demonstrated that customizing abstractions to applications results in significantly better performance

Ceph 在其演进过程中体会到了定制抽象层的重要性，对于云计算，byass kernel 也是很多产品的竞争高地。
