### [The Volcano Optimizer Generator: Extensibility and Efficient Search](https://15799.courses.cs.cmu.edu/spring2025/papers/04-volcano/graefe-icde1993.pdf)

> Goetz Graefe, 1993

在数据库系统中提到 Volcano，很多人第一反应往往是它著名的 `Iterator Tree` 执行框架（火山模型执行器）。不过，Volcano 实际上是一个更完整的研究项目，它同时包含了 `执行框架` 和 `优化框架`。在这两者之中，真正具有长期影响力、并且成为后续系统设计基础的是 `Volcano Optimizer Framework`。

Volcano 优化框架的核心目标不是提供一个具体的优化器实现，而是提出一套可以`生成优化器`的方法论——一种完全可扩展的查询优化器生成器（Optimizer Generator）。基于这一框架，系统设计者可以按照规则体系、物理算子、代价模型、物理属性等组件的接口，自行插入符合需求的实现，从而构建适配自身系统的优化器。

传统查询优化器（例如 System R）虽然强大，但它们通常固定于关系模型、具体实现环境或特定操作集。相比之下，Volcano 的目标是构建一个可扩展的、模块化的、支持多种数据模型和搜索策略的优化器生成器。也就是说，它不是一个单一的优化器，而是一个框架：数据库系统实现者可以用它快速**生成**一个能处理特定逻辑/物理操作及搜索策略的查询优化器。

Volcano 的设计把查询优化看作一个搜索问题。给定一个查询表达式，它需要在可能的逻辑表达式等价形式和物理执行策略之间进行搜索，并挑选出代价最低的计划。为了支持这种搜索，它采用了以下几种关键机制：

> 逻辑与物理代数分离

- 逻辑代数（Logical Algebra）：描述查询表达式结构，例如选择、投影、连接等运算。
- 物理代数（Physical Algebra）：描述执行具体算法的版本，如使用哈希连接、排序合并连接、索引扫描等。
- 两者的分离让优化器能够先构建出逻辑等价表达式，然后再映射到多种可行的物理计划，并通过搜索比较优劣。

> 优化规则驱动搜索

实现 Volcano 时，开发者为优化器生成器提供：

- 逻辑转换规则（Transformation rules）：定义逻辑等价的重写，如连接顺序变换、选择下推等
- 实现规则（Implementation rules）：定义从逻辑表达式到物理操作的映射
- 代价函数（Cost functions）：用于评估计划代价并比较不同方案
- 适用性条件（Applicability functions）：定义算法是否可以用于某个逻辑表达式，以及它对输入输出的物理属性要求（例如排序、分区等）

这些规则构成了优化器搜索空间和约束，是驱动 Volcano 产生最优计划的核心`知识库`。

#### Search Algorithm

摘自: https://zhuanlan.zhihu.com/p/364619893

```
/*
LogExpr，即当前所面对的Logical Expression，对应一个Logical等价类
PhysProp，对其所应具有的物理输出属性的要求
CostLimit，计划中剩余的cost配额
Return 在当前LogExpr内，满足该PhysProp的代价最低的物理operator
*/
FindBestPlan (LogExpr, PhysProp, CostLimit) returns Plan,Cost
/* step 1
使用(logical expression + property)作为key,在hash lookup table中查找是否已有最优计划
如果已有：
   -> 满足cost limit，返回给上层
   -> 不满足cost limit，返回查找失败，表示这个子节点无法选出满足要求的局部最优解
*/

1. If the pair LogExpr and PhysProp is in the look-up table
     if its state is “in progress” , then ignore and return

else if the cost in the look-up table < CostLimit , then
             return Plan and Cost

else , return failure

/* step 2 - 3
hash lookup table没找到，则做优化，允许3种move
   -> 在transformation rules中，尝试apply，转换为新的logical expression，
      并基于新的logical expression，在本层递归调用FindBestPlan
   -> 在implementation rules中，找满足prop要求的algorithm，根据cost formula + cardinality
      计算cost，并使用applicability function确定input operator的prop要求，
      递归到下层调用FindBestPlan，累计其返回值得到本层总的代价，过程中可以用Cost Limit做pruning
   -> 找可以满足prop要求的enforcer，计算cost，根据enforcer改变对下层logic expr的prop要求，
      递归到下层调用FindBestPlan计算cost
*/
2. Create the set of possible "moves" from:
   - applicable transformations
   - algorithms that give the required PhysProp
   - enforcers for required PhysProp
3. Order the set of moves by promise; and for the most promising moves:

if the move uses a transformation

Apply the transformation creating NewLogExpr

Call FindBestPlan (NewLogExpr, PhysProp, CostLimit)
   else if the move uses an algorithm
      For each input i while TotalCost < CostLimit

          Determine required physical properties PP for i
          Cost = FindBestPlan (i, PP, CostLimit − cost of algorithm)
   else /* move uses an enforcer */
       Modify PhysProp for enforced property

Call FindBestPlan for LogExpr with new PhysProp (cost less enforcing)

/* step 5 - 6
将获取到的LogExpr和局部最优plan记录在hash lookup table中，后续递归回来可能再访问到相同的logical expression,
直接获取局部最优解即可
*/

5. If LogExpr is not in the look-up table, then insert LogExpr into the look-up table
6. Insert PhysProp and best plan found into look-up table

/* step 7
在本次递归中，返回局部最优解和该最优解的cost
*/

7. Return best Plan and Cost
```

#### Volcano 的影响与历史地位

这篇论文及其提出的架构被认为是现代查询优化器设计的一个里程碑，它强调基于规则的代价搜索和灵活的优化生成方式，它也是之后更先进优化框架（如 Cascades 架构）的直接先驱。Volcano/Cascades 之后，该框架被广泛采用并成为许多商业优化器的设计基础（如 Microsoft SQL Server 的优化器等）。
