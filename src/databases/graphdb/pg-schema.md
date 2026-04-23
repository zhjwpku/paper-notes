### [PG-Schema: Schemas for Property Graphs](https://arxiv.org/abs/2211.10962)

> Proc. ACM Manag. Data, 2023
>
> DOI: [10.1145/3589778](https://doi.org/10.1145/3589778)

这篇论文讨论的不是图查询优化或执行引擎，而是 property graph 更基础的一层：**如何为 property graph 设计一个正式、可验证、可被工具使用的 schema language**。

作者认为，property graph 已经在数据库系统和应用中广泛使用，GQL 标准也在推进，但现有系统对 schema 的支持仍然比较弱。很多系统允许用户直接使用 label 和 property 建图，这带来了很高的灵活性，但也导致数据约束、查询辅助、图探索工具、静态检查和数据验证都缺少统一基础。

PG-Schema 的核心可以概括为：

> PG-Schema = PG-Types + PG-Keys

其中 **PG-Types** 描述节点、边和图的局部结构；**PG-Keys** 描述 key、foreign key、participation constraint 等全局约束。论文刻意把 type 和 constraint 分开：type 更偏局部性质，可以更容易做静态检查和赋型；constraint 更偏全局性质，需要对图实例进行验证。

### Motivation

Property graph 的一个核心矛盾是：系统通常很灵活，但用户在真实场景里又需要 schema。论文特别强调三类需求：

- **Schema-first**：先定义图结构和约束，再写入数据。
- **Flexible schema**：允许图结构持续演进，不能强行要求所有数据都完全固定。
- **Partial schema**：只对稳定或关键部分施加 schema，其余部分继续保持开放。

schema 不只是用于拒绝非法数据。它还可以服务于 tooling：例如图浏览器可以基于 schema 自动生成搜索表单，限制属性输入类型，提示可能的连接路径，辅助构造更可靠的探索查询。

### Design Requirements

论文把 property graph schema 的基本需求拆成几类：

- **Node types**：定义节点类型，说明节点应具有哪些 label 和 properties。
- **Edge types**：定义边类型，除了边自己的 label 和 properties，还要说明边连接哪些类型的源点和终点。
- **Content types**：属性值需要有实用的数据类型系统。
- **Key constraints**：支持节点或边上的 key。
- **Participation constraints**：表达某类节点必须或至多参与某类关系。

这个目标并不是试图一次性覆盖所有业务规则，而是先为 property graph 补上最通用、最基础、也最适合标准化的 schema 能力。

### PG-Types

PG-Types 是 PG-Schema 的类型部分，包含三层：

- **Node type**：描述节点允许的 label 组合和属性内容。
- **Edge type**：描述边允许的 label、属性，以及源点/终点类型。
- **Graph type**：把一组 node types 和 edge types 组织成图级 schema。

论文给出了一套接近 DDL 的语法，例如 `CREATE NODE TYPE`、`CREATE EDGE TYPE`、`CREATE GRAPH TYPE`。

边类型中的 endpoint type 很重要。它把 property graph 中“关系连接哪些实体”正式纳入 schema，而不只是检查边本身有没有某些属性。

PG-Types 支持几个很有 property graph 特征的设计：

- **Multi-inheritance**：一个 type 可以继承多个 type，适合表达多角色、多 label 的实体。
- **Label expressions**：label 规格可以使用并、交、可选等组合，适配 multi-label graph。
- **Open / closed types**：closed type 禁止未声明内容，open type 允许额外内容。
- **Strict / loose graph types**：strict schema 要求图元素被 schema 覆盖；loose schema 支持 partial schema 和渐进式数据治理。

这些设计说明 PG-Schema 并不想把 property graph 强行变成关系数据库的表结构，而是在保留图数据灵活性的前提下增加正式约束。

### Formal Semantics

论文不是只提出一套语法，而是给出正式语义。它先把 schema 编译或解释为 formal graph type，再定义节点、边、图相对于类型的 conformance。

formal graph type 大致包含：

- 节点类型名集合；
- 边类型名集合；
- 节点类型到 formal base types 的映射；
- 边类型到三元组的映射，三元组描述源节点类型、边自身类型和目标节点类型。

在此基础上，论文定义了：

- node conforms to a node type；
- edge conforms to an edge type；
- graph conforms to a formal graph type；
- graph 相对于 schema 的 typing。

这里一个关键点是 **conformance** 和 **typing** 的区分。特别是在 loose schema 或 partial schema 场景中，图里可能存在 schema 没覆盖的元素；系统仍然可以给被 schema 命中的部分赋型，并对这部分进行验证。

### PG-Keys

PG-Keys 是 PG-Schema 的约束部分。虽然名字里是 keys，但它的表达能力不只 key。论文基于早期 PG-Keys 工作做扩展，使约束不仅能引用 label，也能引用 type name。

它可以表达：

- primary key / unique key；
- foreign key；
- participation constraint；
- 类 SQL 的 check constraint；
- denial constraint。

PG-Types 和 PG-Keys 的分工很清晰：前者描述“元素允许长什么样”，后者描述“整个图必须满足什么一致性条件”。例如，`personType.id` 可以声明为 key；`customerType.id` 可以要求引用 `personType.id`，这就类似 foreign key；还可以表达某个人至多有一个 best friend 之类的 cardinality/participation 约束。

### Validation

论文指出，图相对于 graph type 的 typing 可以高效计算；得到 typing 后，再按 PG-Keys 的语义验证约束。

更有意思的是 **partial validation**。在真实图系统中，数据常常不是从一开始就完全符合 schema，而是逐步清洗和治理。因此 schema validation 不应只有“全量通过”和“全量失败”两种结果。PG-Schema 允许系统识别：

- 哪些元素成功赋型；
- 哪些部分不符合 schema；
- 哪些约束在已赋型部分上成立或失败。

这使它更适合已有图数据集、半结构化图和演进中的图应用。

### Comparison

论文将 PG-Schema 与多类 schema 或建模方式比较，包括 ER 模型、RDF graph schema、XML/JSON schema 以及现有 graph database systems。

作者认为 PG-Schema 的优势主要来自两方面：

- PG-Types 支持组合性、抽象类型、类型层次、多继承、open/closed types、edge properties 等 property graph 需要的类型能力。
- PG-Keys 支持更丰富的 key、foreign key 和 participation constraints。

这种组合让它既能描述 property graph 的结构，又能表达较强的数据完整性约束。

### Takeaways

这篇论文最重要的价值在于给 property graph 补上关系数据库长期拥有、但图数据库生态长期较弱的一层：正式 schema。

几个值得记住的点：

- property graph schema 不应只描述节点属性，还应描述 edge endpoint types。
- schema 不只是数据校验，也能支撑图工具、交互式探索、查询构造和静态检查。
- strict / loose graph types 让 schema 同时支持强约束建模和渐进式治理。
- type 与 constraint 的分层非常关键：局部结构交给 PG-Types，全局一致性交给 PG-Keys。
- 对查询优化或图 IR 来说，schema 可以提供静态推理信息，例如某些 pattern 是否类型上不可能、某些 expand 是否可为空、属性访问是否类型正确。

### Limitations

PG-Schema 更像一个 formal DDL proposal，而不是完整系统实现论文。它主要解决 schema language 的表达能力和语义问题，不讨论查询执行、优化器设计、schema inference 或大规模 benchmark。

如果关注图查询优化，可以把这篇论文视为 schema 层的基础设施：它本身不是 optimizer paper，但它定义的类型和约束信息未来可以被 planner、rewriter、validator 和图探索工具使用。
