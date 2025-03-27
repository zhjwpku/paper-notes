### [KÙZU Graph Database Management System](https://www.cidrdb.org/cidr2023/papers/p48-jin.pdf)

> CIDR 2023

Kùzu 的目标是成为一个功能完备、用户友好、可扩展的 GDBMS。它基于磁盘存储，支持事务处理，并且优化了图数据的存储和查询处理。其设计重点在于优化m-n连接的中间结果表示（通过因式分解（Factorization））和避免全列扫描或连接索引扫描，以提高查询性能。

三个核心技术：

- 因式分解：Kùzu 使用因式化向量来表示中间结果，避免了传统块处理中数据冗余的问题。因式化向量通过将中间结果表示为笛卡尔积的形式，减少了数据的存储和处理开销。
- ASP-Join 算法：Kùzu 的 ASP-Join 算法是一种改进的哈希连接算法，包含三个阶段：积累探针侧的因式化元组、构建哈希表、探针哈希表。ASP-Join 能够同时实现因式化结构和顺序扫描，解决了传统 INLJ（索引嵌套循环连接）算法在非顺序扫描时的性能问题。
- 多路 WCO（最坏情况最优）连接算法：Kùzu 还实现了基于 ASP-Join 的多路 WCO 连接算法，用于处理循环连接查询。这种算法通过交集多个邻接列表来扩展前缀元组，避免了全表扫描，并且能够利用因式化结构提高性能。

### More readings

- [What every competent GDBMS should do (a.k.a. the goals and vision of Kuzu)](https://blog.kuzudb.com/post/what-every-gdbms-should-do-and-vision/)
- [Factorization and great ideas from database theory](https://blog.kuzudb.com/post/factorization/)
- [Why (Graph) DBMSs Need New Join Algorithms (The Story of Worst-case Optimal Join Algorithms)](https://blog.kuzudb.com/post/wcoj/)
- [KÙZU Graph Database Management System (CIDR 2023) presentation record](https://www.youtube.com/watch?v=4XCHUJC81KU)
