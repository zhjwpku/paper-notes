### [The Cascades Framework for Query Optimization](https://15721.courses.cs.cmu.edu/spring2016/papers/graefe-ieee1995.pdf)

> Goetz Graefe，IEEE Data Engineering Bulletin 1995

*以下内容由 ChatGPT 生成*

在数据库查询优化领域，**Cascades 框架**是继 *EXODUS* 和 *Volcano* 之后的一种更通用、更灵活的优化器架构，它将规则驱动搜索进行了系统性重构，并引入了更强的扩展性和执行控制机制。本文将从背景动机、核心设计、搜索策略与 Volcano 对比等角度，为你系统梳理这篇开创性论文的精髓。

#### 背景：为什么需要 Cascades？

早期的查询优化器如 *System R*、*EXODUS*、*Volcano* 都提供了强大的代价驱动优化能力，但它们存在一些实际问题：

* **固定的优化阶段划分**：如 Volcano 先生成所有等价逻辑表达式，再执行物理实现选择，会造成预计算开销大。
* **规则机制缺乏灵活控制**：转换规则（transformation rules）和实现规则（implementation rules）在 Volcano 中是分离的，无法针对不同模式或策略进行分组指导。
* **接口抽象不够完备**：早期优化器对接数据库实现者（DBI）接口繁琐，不利于扩展。

为了解决这些短板，Graefe 提出了 **Cascades 优化框架**，它不仅保留 Volcano 的动态规划加 Memorization 机制，还进一步增强了：

* 规则作为对象支持更丰富的约束和分组
* 逻辑和物理算子统一处理
* 支持 schema/query 特定规则
* 支持更灵活的搜索控制与执行跟踪
* Clean C++ 接口，便于扩展和生产系统集成

#### Cascades 框架的核心设计

**1. `Rule` 成为一级对象**

在 Cascades 中，规则（Rule）不再是简单的静态匹配＋替换定义，而是可以被数据库实现者扩展、分组以及带有优先级和指导策略（Guidance）的对象。

这意味着：

* 可以为不同的逻辑结构定义特定规则组
* 可以为不同场景（如物化视图、谓词下推）单独引导规则
* 可以按权重或启发式 promise 排序规则应用

这种设计极大增强了优化器的可配置性与可控搜索策略。

**2. 逻辑与物理算子统一处理**

在 Cascades 中：

* 一个算子可以既是**逻辑算子**，也可作为**物理实现算子**
* 例如：谓词既可看作逻辑上的条件，又可看成物理上可以拆分并下推的算子

旧架构（如 Volcano）中逻辑与物理算子是严格分离的，这对一些转换变换造成了限制。Cascades 的统一处理能更直接表达复杂转换。

**3. 任务（Task）作为核心控制单元**

与 Volcano 的递归调用不同，Cascades 采用了 **任务驱动（Task-based）** 模型：

* 优化过程拆分成多个 **任务对象（Task）**
* 每个任务代表一个优化需求，如：

  * `OptimizeGroup`
  * `OptimizeExpression`
  * `ExploreGroup`
  * `ExploreExpression`
  * `ApplyRule`
  * `OptimizeInputs`

这些任务被组织在一个 **任务结构（通常是栈）** 中管理与调度。

优势包括：

* 支持启发式搜索控制
* 支持并行度扩展
* 任务之间依赖可以更清晰表达
* 更易于实现动态剪枝与提前停止

#### Cascades 中的搜索机制

Cascades 搜索策略的关键创新是：

**按需进行等价表达生成**

在 Volcano 中，优化器会先 **预生成所有等价逻辑表达式**，然后再进入物理优化阶段。Cascades 则采用 **按需探索（Explore）**：

* 当某条规则匹配某个模式时
  * 先探查该 group 是否已有匹配模式
  * 如果没有，则触发 exploration task 生成该模式下的表达式
* group 和 expr 中分别维护已经应用过哪些规则
  * 避免重复探索
* 只有在真正需要时才生成等价表达式

这种模式削减了不必要的枚举开销，使最优执行计划搜索更高效。

**紧密集成实现规则**

Cascades 的规则系统支持：

* Schema-specific 规则，例如物化视图重写
* 插入 “enforcers” 或 “glue operators” 以满足物理属性
* 使用 promise/priority 指导规则调度
* 支持规则参数控制 operator argument 的调整

规则分为两类：

1. **Transformation Rule**

   * 逻辑表达式等价变换
   * 例如连接结合律、谓词下推等

2. **Implementation Rule**

   * 将逻辑算子映射到物理算法
   * 例如 Nested-Loop Join、Sort-Merge Join 等

不同于 Volcano 的先逻辑后物理阶段划分，Cascades **交错应用 transformation 和 implementation rule**，根据当前任务需求动态展开。

**Memoization 与 Duplicate elimination**

为了防止重复计算与存储，Cascades 使用了与 Volcano 类似的 **Memo 数据结构**：

* Group 代表一组等价表达式集合
* 新表达式插入 Memo 时检查重复
* 对已经优化过的 group/prop 组合进行动态编程缓存

但是 Cascades 在此基础上引入了：

* **Pattern Memory**：记录某 group 已经被某个规则模式探索过
* **Rule Memory**：记录某表达式已经被某规则应用过

这两个额外的记忆机制有效防止了重复规则触发和无效展开。

#### Cascades 与 Volcano 的对比

下面是两者在设计哲学和实现细节方面的对比：

| 维度      | Volcano            | Cascades                        |
| ------- | ------------------ | ------------------------------- |
| 搜索策略    | 先生成所有逻辑等价表达式，再物理优化 | 按需探索，逻辑与物理规则交错                  |
| 规则机制    | 静态规则匹配             | 规则作为对象，支持分组与启发式引导               |
| 架构模式    | 递归函数               | 任务驱动（Task）                      |
| 逻辑/物理分离 | 明确分离               | 算子可同时为逻辑和物理                     |
| 重复剪枝    | 基本 Memoization     | 额外 Pattern Memory + Rule Memory |
| 并行扩展    | 不支持                | 设计时考虑并行搜索扩展                     |

总体来看，Cascades 相对 Volcano 提升了：

* 控制粒度
* 扩展能力
* 搜索效率与剪枝效果
* 架构清晰度

Cascades 框架是数据库查询优化器设计史上的重要里程碑，其核心贡献包括：

* **更灵活的规则机制与指导**
* **任务驱动的动态优化搜索引擎**
* **逻辑与物理规则混合应用**
* **更强的 Memoization 与搜索剪枝机制**
* **更易扩展的 C++ 接口和工业级实现基础**

Cascades 不仅解决了 Volcano 的实际问题，还为现代优化器架构提供了实用范式，其思想在当代优化器（Apache Calcite、ORCA、SQL Server 等）中依然可见。
