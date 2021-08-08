### [C-Store: A Column-oriented DBMS](../assets/pdfs/cstore-vldb05.pdf)

> VLDB 2005 Mike Stonebraker, etc.
>
> https://dl.acm.org/doi/10.5555/1083592.1083658

行存（row store）RDBMS 作为当时的主流，通常将同一行记录的所有属性存储在磁盘的连续空间，以获得卓越的写性能，这种方式也被称为 write-optimized，适用于 OLTP 场景。而数据仓库、CRM 等需要 ad-hoc 查询大量数据的系统应该具有 read-optimized 的特性，此类系统往往通过列存（colunm store）架构来获得更有效的性能（如 Sybase IQ, Addamark, KDB）。

列存架构中，查询操作只需按需读取所需要的列，避免将无关数据读取到内存。CPU 的性能增速要远高于磁盘的带宽增速，在 read-optimized 系统，用 CPU cycles 换取磁盘带宽是一件值得的事情，列存通过 1) code 2) densepack 两种方式达到以 CPU cycles 换取磁盘带宽的目的。

C-Store 是作者提出的一个新的列存数据库，具有如下革新特性：

1. A hybrid architecture with a WS component optimized for frequent insert and update and an RS component optimized for query performance
2. Redundant storage of elements of a table in several overlapping projections in different orders, so that a query can be solved using the most advantageous projection
3. Heavily compressed columns using one of several coding schemes
4. A column-oriented optimizer and executor, with different primitives than in a row-oriented system
5. High availability and improved performance through K-safety using a sufficient number of overlapping projections
6. The use of snapshot isolation to avoid 2PC and locking for queries

### 混合架构

人们对实时数仓数据可见性的延迟要求越来越高（toward 0），C-Store 的 `Writeable Store` 模块支撑频繁地插入和更新，`Read-optimized Store` 模块提供高性能查询，`Tuple Mover` 则负责将数据从 WS merge 到 RS，这是结构非常类似 LSM。

### 数据模型

C-Store 支持标准关系数据库的逻辑数据模型 —— 数据库由表构成，每个表含多个数据列，表的属性可以作为 primary key 或其它表的 foreign key。但 C-Store 的物理存储模型不同于这种逻辑模型，是以 projection 进行存储的。

- 一个 projection 存储了一个逻辑表的一个或多个列属性，它还可以存储其它表的一个或多个属性以表示外键
- 一个 projection 和它的锚表（anchor table）具有的行数相同
- 一个 projection 具有一个 `sort key`，由该 projection 的一个或多个列构成。同一个 projection 中的 K 个属性各自存储于一个数据结构中，且都按照 `sort key` 进行排序
- 每个 projection 都被水平切分（Horizontally partitioned）为一个或多个 segment，切分的依据为 `sort key`，这样每个 segment 都对应一段 key range
- 由于每个 projection 的 `sort key` 不一致，因此需要 `join indexes` 来维持不同 projection 的对应关系

#### Storage key

- 在 RS 上，storage key 是记录在 segement 上的序数，不存储且按需计算获得
- 在 WS 上执行引擎给每一次插入赋予一个唯一的 storage key, 大于 RS 上 storage key 的最大值
- WS 和 RS 按照相同的规则进行水平切分，因此 RS segement 和 WS segent 是 1:1 对应的，因此 (sid, storage_key) 只能存在 RS 或 WS 中的一个
- 由于 WS 在尺寸上相对较小，因此使用 B-tree 直接存储 projection 的每一列，按 storage key 排序，另外（sort key， storage key）以 `sort key` 作为 key 保存为 B-tree。这样就根据 sort key 快速定位到 storage key，然后再根据 storage key 找到对应的列值
- WS 构建在 BerkeleyDB 的 B-tree 结构上

#### Join Indices

> If T1 and T2 are two projections that cover a table T, a join index from the M segments in T1 to the N
> segments in T2 is logically a collection of M tables, one per segment, S, of T1 consisting of rows of the form:
> 
> (s: SID in T2, k: Storage Key in Segment s)

Join Index 按照 T1 的切分方式进行切分，且跟 T1 的 segement 保存在一起。

Join Index 在更新的时候非常难以维护，因此会把每个 column 存储在多个 projection 上，但这对 DBA 设计表结构提出了更高的要求，文中提到他们正在写一个自动化表结构设计工具。

### Coding Schema

projection 中的每一列都紧凑地存储在一个结构体中，基于该列是否是 `sort key` 以及该列不同数据的多少，文中给出了四种编码方式：

1. Self-order, few distinct values
2. Foreign-order, few distinct values
3. Self-order, many distinct values
4. Foreign-order, many distinct values

### Column-oriented optimizer and executor

这块不懂，后面有机会再补充

### High availability

由于不同的列存在与多个 projection，C-Store 基于此可以实现 K-safe 的系统。

### Snapshot Isolation

> Snapshot isolation works by allowing read-only transactions to access the database as
> of some time in the recent past, before which we can guarantee that there are no uncommitted transactions.

对于只读事务，可以根据时间戳来判断记录的可见性，因此可以不用加锁

对于 Read-write 事务，C-Store 遵循严格的 two-phase locking，并使用了 NO-Force, STEAL 策略。

事务相关的内容感觉跟 Mysql 有些相像，但有些内容还没搞明白，这块后面有机会再研究一下。
