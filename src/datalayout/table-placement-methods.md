### [Understanding Insights into the Basic Structure and Essential Issues of Table Placement Methods in Clusters](../assets/pdfs/p1750-huai.pdf)

> Proceedings of the VLDB Endowment, September 2013
>
> https://dl.acm.org/doi/10.14778/2556549.2556559

基于 Hadoop 生态的多种表放置方法（table placement methods）被提出并实现，CFile、Column Format、RCFile、ORC、Parquet、Segment-Level Column Group Store(SLC-Store)、Trevini、Trojan Data Layouts(TDL) 等项目都是独立完成，且缺少对它们全面系统性的研究。

三个未解决的关键问题:

1. The basic structure of a table placement method has not been defined. This structure should abstract the core operations to organize and to store data values of a table in the underlying cluster. It also serves as a foundation to design and implement a table placement method.
2. A fair comparison among different table placement methods is almost impossible due to heavy influences of diverse system implementations, various auxiliary data and optimization techniques, and different workload configurations.
3. There is no general guideline on how to adjust table placement methods to achieve optimized I/O read performance.

本文的三个主要贡献:

1. 定义了 table placement method 的基本结构（basic structure），用于指导 table placement method 的设计、抽象已有 table placement method 的实现、表征现有 table placement method 之间的差异
2. 设计并实现了一个基准测试工具（micro-benchmarks）来研究不同 table placement method 的设计要素
3. 全面研究了 table placement method 相关的数据读取问题，并提供了如何通过调整 table placement method 来达到优化 I/O 读取的指南

一个 table placement method 的基本结构包含三个部分:

```txt
1. a row reordering procedure       fRR
2. a table partitioning procedure   fTP
3. a data packing procedure         fDP
```

#### MICRO-BENCHMARK

从表中读取数据影响 I/O 性能的 6 个因素:

1. Table organizations
2. Table placement methods
3. Storage device type
4. Access patterns of the application
5. Processing operations of the filesystem at the application level
6. Processing operations of the filesystem inside the local OS

文章设计的实验主要关注以上因素 2、3、4，通过实验给出了以下建议:

1. using a sufficiently large row group size
2. if a sufficiently large row group size is used and columns in a row group can be accessed in a column-oriented way, it is not necessary to group multiple columns to a column group
3. if a sufficiently large row group size is used, it is not necessary to pack column groups (or columns) intomultiple physical blocks.

#### MACRO-BENCHMARK

这部分评估了从 Star Schema Benchmark、TPC-H 和 SS-DB 选取的查询。

Row Group Size 对性能的影响:

1. From 4 MiB row group size to 64 MiB row group size, the elapsed times of the Map phase decrease significantly.
2. when further increase the row group size from 64 MiB to 128 MiB, the change on the elapsed times of the Map phase is not significant.

Grouping Columns 对性能的影响:

1. When the row group size is small, grouping columns can provide benefits in two aspects:
    - because needed columns are stored together, a buffer read operation can load a less amount of unnecessary data from disks
    - a less number of disk seeks is needed to read needed columns
2. when the row group size is large enough, grouping columns cannot provide significant performance benefits


论文中得出来的另一个结论跟并发相关：

> An optimal row group size is determined by a balanced trade-off between
> the data reading efficiency and the degree of parallelism.