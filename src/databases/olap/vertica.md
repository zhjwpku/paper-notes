### [The Vertica Analytic Database: C-Store 7 Years Later](../../assets/pdfs/vertica-cstore-7-years-later.pdf)

> Proceedings of the VLDB Endowment, 2012
>
> https://dl.acm.org/doi/10.14778/2367502.2367518

Vertica 是 [C-Store](../../datalayout/c-store.md) 学术原型的一个商业化分析型数据库，采用分布式 share-nothing 的架构，通过对 C-Store 设计要点的合理取舍，实现了一个为支持 ACID 且能高效处理 PB 级数据的分析型数据库。

### 数据模型

**Projection**

一张表的各个属性被拆分成多个投影，每个投影按照自己包含的属性自行排序，通过 `position index` 记录其物理位置和其在原始表位置的对应关系。包含所有属性的投影被称为 `super projection`，其它投影叫作 `non super projection`。通常情况下，一张表由一个 `super projection` 和 `0-3 个 non super projection` 构成。

Vertica 没有实现 C-Store 的 `join index`，原因在于实现起来较为复杂，且在分布式执行器重构元组是的消耗远大于其带来的收益。

> Vertica does not implement join indices at all, instead requiring at least
> one super projection containing every column of the anchoring table.

**Prejoin Projection**

Vertica 允许将维表中的数据和事实表中的 N:1 的 join 关系保存在 `prejoin projection` 中，由于列式存储具有良好的压缩比例，这种 denormalization 并不会消耗过多的物理存储。该特性来自 C-Store，但在 Vertica 中并没有被用户频繁使用，原因在于 Vertica 的执行引擎对于小维表的 join 处理地很出色（using highly optimized hash and merge
join algorithms）。

**Encoding and Compression**

列存格式增加了编码和压缩的可能，Vertica 支持如下编码压缩方式:

- Auto: 当列上的使用样例不足时，自动选取最优的压缩算法
- RLE: best for low cardinality columns that are sorted
- Delta Value: best used for many-valued, unsorted integer or integer-based columns
- Block Dictionary: best for few-valued, unsorted columns such as stock prices
- Compressed Delta Range: ideal for many-valued float columns that are either sorted or confined to a range
- Compressed Common Delta: best for sorted data with predictable sequences and occasional sequence breaks

**Partitioning & Segmentation**

Partitioning 是节点内（intra-node）数据切分，通常使用时间作为切分条件；Segmentation 在有些 OLAP 数据库中被称为 Distribution，是节点间（inter-node）的数据切分。

Partitioning 的作用一是分区裁剪增加查询效率，二是提供了快速 bulk deletion，通过直接删除文件系统上的文件立马回收空间。Segmentation 的作用一是提高并行度，增加查询效率，二是增大存储空间以保存更多的数据。

**Read and Write Optimized Stores**

ROS 和 WOS 的概念也是来自 C-Store，WOS 保存在内存不下盘，为行存（Vertica 在实现的时候由行存改为列存，后来又改回行存）。

ROS 中的数据物理上保存在多个 ROS Container 中:

> Each ROS container logically contains some number of complete tuples sorted by the projection’s
> sort order, stored as a pair of files per column

Vertica 为纯列存存储，一列数据在 ROS Container 中存储为数据和 `position index` 两个文件。

> Data is identified within each ROS container by a position which is simply its ordinal position
> within the file. Positions are implicit and are never stored explicitly.

这里对 `position index` 的描述有点模糊，文中说索引结构没有使用 B-tree（因为文件一旦写入不会修改），又说了索引只存储每个 disk block 的起始位置以及 min/max 等元数据。其实现参照了 **[Small Materialized Aggregates: A Light Weight Index Structure for data warehousing](https://dl.acm.org/doi/10.5555/645924.671173)**，后面有时间学习一下。

Vertica 也支持将多列保存一个文件中，但明确表示了这种存储格式性能较差:

> This hybrid row-column storage mode is very rarely used in practice because of the
> performance and compression penalty it exacts.

这可能是因为 Vertica 并没有想出类似于 ORC 的数据布局。

数据的修改或删除通过 Delete Vector 来实现，Delete Vector 和用户数据一样，先写入内存中的 DVWOS，然后由 Tuper Mover 移动到 DVROS Container 中的磁盘结构中。

### Tuper Mover

WOS 通过缓存一些写，然后批量将 WOS 转换为 ROS 中的列存，ROS 的数据也会合并为更大的文件来解决磁盘及提高查询性能。Tuper Mover 是负责该工作的模块:

- Moveout: asynchronously moves data from the WOS to the ROS
- Mergeout: merges multiple ROS files together into larger ones

类似 LSM-tree 的工作方式，不赘述。

事务及查询优化相关的描述感觉也比较晦涩，暂且忽略相关的笔记。

其中提到了一个 Database Designer (DBD) 的工具，可根据 shema 和少量数据分析出如何拆分 Projection 能节省存储空间或提升查询效率。个人感觉这种工具增加了用户的使用负担 ; (