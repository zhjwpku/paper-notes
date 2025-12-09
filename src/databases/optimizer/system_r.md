### [Access path selection in a relational database management system](https://courses.cs.duke.edu/spring03/cps216/papers/selinger-etal-1979.pdf)

> Selinger, SIGMOD '79
>
> https://dl.acm.org/doi/10.1145/582095.582099

1970s–80s，当数据库还多用层次 (hierarchical) 或网络 (network) 模型时，关系模型 (relational model) 的出现提出了一个大胆设想：
> 不用告诉系统 **怎么查**，只告诉它 **查什么** —— SQL 这样的声明式语言

#### SQL 语句是如何被执行的？四个阶段：

1. Parsing — 语法 & 语义检查 & 生成 parse tree
2. Optimization — 关键：决定访问路径 (access path) 与 join order / join method
3. Code generation — 把优化器选出的 plan 转成可执行机器码 (或解释执行)
4. Execution — 真正访问磁盘 / 索引 / 内存，返回结果

System R 的 optimizer 就在第 2 步，它决定`怎么查`与`怎么连表`。即 Access Path Selection:

#### 单表 (Single relation)

- 如果有索引 (index)，可以选择 index scan 或 full table scan
- 如果查询有 ORDER BY / GROUP BY，可以选择有序输出 (如果 index 支持) —— 避免排序 (sort)
- 基于统计数据 (行数、页数、distinct key count)、查询谓词 (predicate) 的选择性 (selectivity)、访问路径 (segment scan / clustered index / non-clustered index) 等，估算每种访问方式的成本 (cost)
- 选择成本最低 (或满足输出顺序要求 + 成本最低) 的方案

#### 多表 JOIN

- 枚举 join order (哪个表先, 哪个表后)
- 枚举 join 方法 (nested-loop, sort-merge) + 每个表的访问路径
- 对所有可能组合计算成本，选择最优 plan
- 若有 ORDER BY / GROUP BY，还得考虑排序开销 vs. 有序索引输出

虽然这篇论文写于 1979 年，但在 PostgreSQL 中仍能看到它的影子。许多概念在那个年代就已经出现了，比如等价类（Equivalence Class）、相关子查询（Correlation Subquery）等。
