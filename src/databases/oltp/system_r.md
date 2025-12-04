### [A History and Evaluation of System R](https://dl.acm.org/doi/pdf/10.1145/358769.358784)

> System R：现代关系数据库的奠基石

在 1970s–1980s，关系模型 (relational model) 与 SQL 提出后，数据库领域面临一个重大未知：**通用、高性能的关系数据库系统是否真正可行？**
那时已有诸多基于网络 (network)、层次 (hierarchical) 的数据库系统，但它们往往结构复杂、不通用、难扩展。
于是诞生了 System R，一个实验性的关系数据库系统，它将关系模型、SQL、事务、优化器、索引、恢复机制整合到一个通用引擎中。

#### 原型实现

- 支持 SQL 查询
- 实现基础关系操作 (SELECT, INSERT, DELETE, UPDATE)

#### 优化与物理设计

- 引入代价模型 (cost-based optimizer)
- 支持多种访问路径 (full scan, index)
- 实现 buffer management、locking、transaction、recovery 等机制

#### 主要贡献与启示

- ✅ **关系 + SQL + 通用引擎可行**
- ✅ **查询优化器必不可少** — 后续所有关系数据库系统都实现了类似优化模型
- ✅ **事务、并发控制、恢复机制** 是工业级数据库系统的核心保障

System R 不仅是一篇论文，而是数据库系统史上一次伟大的工程尝试。它证明了关系模型 + SQL + 通用引擎是可行且高效的，也为后世所有主流 RDBMS 奠定了架构基础。
