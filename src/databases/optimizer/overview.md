### [An Overview of Query Optimization in Relational Systems](https://dl.acm.org/doi/pdf/10.1145/275487.275492)

> Surajit Chaudhuri, PODS 1998

系统地回顾与分析关系数据库中查询优化（query optimization）的关键技术、挑战和实现方法。本文总结了传统关系系统优化器的构造模块、优化流程，以及现实系统中设计取舍。

#### 查询优化器 (Query Optimizer) 的角色

当用户写一条 SQL：

```sql
SELECT a, b FROM R JOIN S ON R.x = S.x WHERE R.y > 100;
```

数据库系统可以有很多种执行方式:

- 扫描 R、扫描 S，然后 Nested-Loops Join
- 用索引扫描 R、再 Hash Join
- 用索引扫描 S、再 Merge Join
- 甚至改变连接顺序、重新排列谓词顺序 (predicate pushdown)

如果系统盲目执行某一种方式，可能会非常慢。**查询优化器**的职责是：

- 把高层 SQL / 关系代数 → 转成逻辑计划 (logical plan)
- 枚举可能的执行计划 (physical plan)
- 估算每个计划代价 (cost)
- 选择一个最优 / 足够优的计划去执行

正是因为优化器，关系数据库才能兼顾 声明式接口 (SQL) 与 高性能执行。

查询优化器负责将用户的 SQL / 关系代数表达转化为高效执行计划 (execution plan)，是连接关系模型与物理实现的桥梁。优化器的存在，使得关系数据库既能保持声明式接口 (SQL)，又能获得高性能。

查询优化划分为若干阶段:

- 解析与逻辑重写 (Parsing & Logical Rewrite)
    - 将 SQL 转为逻辑查询 / 代数表达 (logical plan)，并进行语义、语法检查。
    - 对表达进行逻辑重写 (logical rewrite)：例如谓词下推 (predicate pushdown)、冗余消除、视图展开 (view expansion)、连接 (join) 重写、子查询优化等优化。
- 代价模型 (Cost Model)
    - 对每个可能执行计划 (access path / join order / join algorithm) 估算代价 (估计 I/O、CPU、内存等)，作为优化决策依据。
    - 代价模型是优化器决策质量的基础。假设统计 (statistics) 与估算 (estimation) 的准确性极其关键。
- 访问路径与连接算法 (Access Paths & Join Algorithms)
    - 支持多种访问路径 (seq scan、indexed scan等) 与连接算法 (nested-loop join, hash join, sort-merge join 等)。
    - 优化器需要根据数据特点 (基数、分布、索引、排序状态等) 选择合适路径。
- 计划枚举与搜索 (Plan Enumeration / Search)
    - 因为可能的计划组合很多 (join order, index usage, join method……)，优化器需要枚举 (或以启发式 / 动态编程 /代价 pruning) 搜索空间。
    - SELECT … JOIN … WHERE … GROUP BY … 的复杂查询，其可能计划数目指数级增长，因此枚举与剪枝策略对效率非常关键。
- 统计与估算 (Statistics & Estimation)
    - 优化器对基数 (cardinality)、数据分布 (histogram / distinct counts / null ratios / correlation) 的统计数据进行收集，并据此估算中间结果规模。
    - 估算偏差会严重影响优化结果，导致次优计划。

System R/Starburst 是 bottom-up 类型的优化器，Volcano/Cascades 是 top-down 类型的优化器。
