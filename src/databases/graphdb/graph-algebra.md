### [An Algebra and Equivalences to Transform Graph Patterns in Neo4j](https://ceur-ws.org/Vol-1558/paper24.pdf)

> EDBT/ICDT 2016

论文提出了一种面向图数据库的**图查询代数**（graph algebra）及**等价变换规则**（equivalence rules）。

传统的关系数据库查询优化器（如 SQL 优化器）通过：

- 将查询转换为 relational algebra
- 应用等价重写规则
- 枚举执行计划与成本估计

得到了高效的执行策略。而图数据库也有自己的查询语言 Cypher，但目前主流图优化器：

- 缺乏通用的图代数模型与变换规则
- 许多优化策略依赖具体实现，而非理论等价规则

论文定义了两个主要图代数操作：

- GetNodes：返回图中的所有节点或满足条件的节点
- Expand：基于节点探索其邻居节点（有方向的边）

及一系列代数等价变换规则（见原文 Section 4）。

### 读后感

尽管 `GetNodes` 和 `Expand` 是图数据领域中常用的基础算子，能够处理很多基本的查询模式，但也正如论文中所指出，单纯依赖这两个算子，难以应对更复杂的查询需求，比如 **可变长度模式**、**聚合操作** 以及 **多标签查询** 等问题。因此，当前的图代数框架还需要进一步丰富和扩展，以应对更加复杂和多样化的图查询场景。对此，未来的研究可能需要在这些方面进行更多的探索和优化。

> First, the algebra presented in this paper is restricted to simple pattern matching functionalities. However, in graph query languages such as Cypher, it is possible to search for variable length patterns or to aggregate nodes. Therefore, we plan to integrate these features in our algebra in a next step.
