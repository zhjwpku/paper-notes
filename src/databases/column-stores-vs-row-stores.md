### [ColumnStores vs. RowStores: How Different Are They Really?](../assets/pdfs/Column-Stores%20vs.%20Row-Stores-%20How%20Different%20Are%20They%20Really.pdf)

> SIGMOD 2008 Daniel J. Abadi, etc.
>
> https://dl.acm.org/doi/10.1145/1376616.1376712

列存数据库在数仓、决策支持、BI 等分析型应用中被证明比传统行存数据库的表现要好出一个量级以上，其原因也显而易见：列存数据库对于只读查询按需访问所需列，因而更 `IO efficient`。这种直观的感受不禁让人猜想：如果用行存数据库模拟列存数据库，使得每个列单独访问，是否可以获得列存数据库的性能收益？

这引出了论文要回答的第一个问题:

- Are these performance gains due
to something fundamental about the way column-oriented DBMSs
are internally architected, or would such gains also be possible in
a conventional system that used a more column-oriented physical
design?

作者使用三种技术在行存数据库中模拟列存，其查询性能都非常差。那么是什么造就了列存数据库的优越性能呢？

这引出了论文要回答的第二个问题:

- Which of the many column-database specific optimizations proposed
in the literature are most responsible for the significant performance
advantage of column-stores over row-stores on warehouse
workloads?

### ROWORIENTED EXECUTION

作者在 System X (一款商业行存数据库) 上使用了三种方法来模拟列存数据库。通过在 SSBM 上的实验发现这些方法都不能取得较好的性能。

#### 1. Vertical Partitioning

将表进行垂直切分是最直观的方式，但这需要将切分后的数据进行关联。行存不具有列存将每列数据都按照相同的顺序进行存储的特性，因而一个逻辑表垂直切分出的每一个物理表，都保函两列数据 —— 该列的原始数据和 "position" column（通常是 primary key），查询需要被改写为基于 "position" column 的 join，不管是 Hash join 还是 index join，性能都较差。

这种方式由于在每个列对应的表中都存储了 "position" column，浪费了存储空间和磁盘带宽。另外行存数据库的每一行数据都会保存一个相对较大的 header，这进一步浪费了存储空间。

#### 2. Index-only plans

在基础行存表的之外对每一列存储一个（value, record-id）的非聚簇索引，这样可以不存储重复数据，且没有 tuple header 的存储空间浪费。这种方式对没有谓词的返回列需要进行全表扫描。

#### 3. Materialized Views

对查询的中的每个表所需的列生成物化视图，使用这种方式减少查询需要读取的数据量，以期望该种方法取得优于另外两种方法的性能。

### COLUMNORIENTED EXECUTION

上述行存模拟列存不能取得较好的性能收益在于，在 SSBM 场景下，C-Store 特有的 `Compression`、`Late Materialization`、`Block Iteration` 和 `Invisible Join` 等特性对查询性能具有极大的帮助，论文中将 C-Store 进行逐一特性阉割，将其`退化`为一个行存数据库，得出各因素对于性能的影响:

- Compression improves performance by almost a factor of two on average
- Late materialization results in almost a factor of three performance improvement
- Block-processing can improve performance anywhere from a factor of only 5% to 50% depending on whether compression has already been removed
- The invisible join improves performance by 50-75%

**Compression** 除了通过节省 I/O 来提升查询性能，其 `operate directly on compressed data` 的特性进一步提高了性能。

**Late Materialization** 和 **Block Iteration** 合在一起被称为 `Vectorvectorized query processing`。

**Invisible Join** 是一种 `Late materialized join` 技术，但减少了需要读取的数据，具体细节论文中的例子做了简明的解释。

### 思考

基于 Btree 的行存数据库中的一些特性是为了解决某些实际问题（写优化），比如论文中提到的:

- 每行数据都有 header => 为了实现 MVCC 而存在
- 数据并非物理有序 => 有些存储引擎使用 directory slots 的方式存储数据以避免数据插入时过多的数据移动

所以能够直观地判断出在基于 Btree 的行存数据库中模拟列存不会有好的性能收益。如果是 LSM Tree 呢？虽然可以实现 Late Materialization，但 Compression 和 Block iteration 可能不如在 Column Store 中的收益明显。