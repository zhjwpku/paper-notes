### [The Streaming Batch Model for Efficient and Fault-Tolerant Heterogeneous Execution](https://arxiv.org/pdf/2501.12407)

> Frank Sifei Luan, Ron Yifeng Wang, Yile Gu, et al., arXiv:2501.12407v5, 2025

现代 ML pipeline 不只有 GPU 上的训练或推理，还包含读取、解码、检索、编码和结果写回等 CPU、I/O 乃至其他 GPU 操作。不同算子的资源需求和处理速度差异很大，CPU 预处理经常反而成为 GPU 的瓶颈。

传统批处理系统和流处理系统分别只解决了问题的一半：

- 批处理系统把数据切成可在任意 executor 上运行的 partition，便于动态负载均衡、扩缩容和基于 lineage 的细粒度恢复，但 stage barrier 阻止了异构算子间的流水线执行，并要求物化大量中间结果；
- 流处理系统让固定 executor 直接交换动态大小的数据批次，能通过 pipelining 和 backpressure 控制中间数据量，但 operator、数据范围和资源被长期绑定，重新负载均衡、扩缩容和故障恢复更昂贵。

论文提出 **Streaming Batch Model**，并在 Ray Data 中实现：仍以 partition 作为可调度和可恢复的执行单位，但 partition 的数量和大小可以在运行时根据真实内存占用动态决定；每个新生成的 partition 又可立刻触发下游 task，从而同时获得批处理的弹性与流处理的流水线能力。

## 一句话总结

Ray Data 把 **动态生成的 partition** 作为执行、流水线、内存控制和 lineage 恢复的统一粒度：executor 按真实字节数流式地产生 partition，集中式调度器在 partition 边界重新分配 CPU、GPU 等资源，并以在线 profile 估计 pipeline 的排空速度，在不突破内存限制的前提下尽量提前启动上游任务。

## 问题背景：异构数据流的两组矛盾

典型的 batch inference pipeline 是：

```text
对象存储 ──► load / decode (CPU) ──► inference (GPU)
                                      │
                                      ▼
                              encode / upload (CPU)
```

Stable Diffusion 一类多模态训练甚至会同时包含 CPU 数据加载、较便宜 GPU 上的 frozen encoder，
以及高端 GPU 上的训练。要保持所有设备繁忙，系统必须并行运行这些阶段，但这引出两组矛盾。

### Pipelining 与内存

上游产生的中间数据太少，下游 GPU 会饿死；产生太多，又会 OOM 或把数据 spill 到磁盘。
更麻烦的是记录大小可能在运行前不可知，例如一个视频解码后产生多少帧取决于视频长度，
按输入记录数静态分区不能可靠地估计中间 partition 的内存大小。

### 弹性与状态

不同记录的处理时间不同，静态 round-robin 容易产生 straggler；各算子的最佳并行度也随实际输入、资源和瓶颈变化。系统需要在运行时把资源从一个 operator 转给另一个 operator，并在节点增减或失败时继续执行。可是，越是把 executor、operator 和数据范围绑定起来以实现高效流式处理，迁移与恢复这些状态的代价就越高。

## Batch、Stream 与 Streaming Batch

| 维度 | Batch | Stream | Streaming Batch（Ray Data） |
| --- | --- | --- | --- |
| 中间数据传递 | stage 间完整物化 | executor 间持续发送 | task 流式产生 partition |
| Partition | 执行前静态确定 | executor 本地动态 batching | 运行时按真实字节数动态生成 |
| 资源分配 | 每个 task 动态分配 | operator/executor 通常静态绑定 | 每个 partition 动态分配 |
| 内存控制 | stage barrier、spill | backpressure | 全局内存预算、backpressure、spill |
| 负载均衡 | partition 粒度 | 受静态数据分片限制 | partition 粒度 |
| 容错 | lineage 重算 | logging 或 global checkpoint | 确定性动态分区 + lineage 重算 |
| 最小回滚范围 | partition | record 或 epoch | partition |

这里的 “streaming batch” 不是 Spark Streaming 的 micro-batch。后者把无限输入切成一个个独立 batch job，每个 job 内部仍是同步 stage，并且分区计划仍然是静态的；本文是在**单个有界 job 内部**让 task 持续产生可立即消费的新 partition。

## 执行模型

Ray Data 的 `Dataset` 是惰性构建的逻辑 DAG。`read`、`map`、`flat_map`、`filter` 和
`map_batches` 等 transformation 不会立即运行，`write`、`iter`、`iter_split` 或 `cache`
等 consumption API 才触发执行。每个 transform 可以声明 `{CPU: n}`、`{GPU: n}` 或自定义
资源需求。

一次执行大致经过：

```text
Dataset logical DAG
        │
        ▼
Query Planner ── operator fusion、初始 read partition
        │
        ▼
Physical operator DAG
        │
        ▼
Ray Data Scheduler ── partition 元数据、资源与内存决策
        │  提交 task / 接收 output reference
        ▼
Ray workers / actors ── 读取对象、执行 UDF、动态切分输出
        │
        ▼
Ray distributed object store ── partition 数据面
```

连续且资源需求相同的 operator 可以 fusion。Planner 会让初始 partition 数足以占满 execution
slot，同时尽量把已知大小的输入切为 1–128 MB；这个初值不必非常准确，因为执行时还会再次分区。

普通无状态 UDF 由 Ray task 执行。模型加载等初始化成本较高的只读有状态 UDF 使用 actor pool：
模型状态常驻 actor，但 partition、队列等**系统状态不进入 actor**，因此任意输入 task 仍可交给池中的任意 actor，保留负载均衡能力。

## 核心机制一：动态分区与流水线

Ray Data 扩展 Ray，引入可以产生未知数量输出的 **generator task**。一个 task 以 iterator
方式处理输入并累积输出记录；当缓冲区超过目标大小时，将其作为新 partition 写入 Ray object
store，并立即把 object reference 返回给调度器。默认目标上限是 128 MB。

调度器收到一个 reference 后，不必等待上游 task 完成，就可把该 partition 放入下游 operator
的输入队列并启动下游 task。数据本身不经过调度器，因此这是一个集中控制、分布式数据面的设计：

```text
upstream task:  [produce B1] [produce B2] [produce B3]
                        │            │            │
downstream:             [consume B1] [consume B2] [consume B3]
```

这种机制解决了静态 partition 的两个问题：

- `flat_map`、视频解码等数据膨胀算子可以按实际输出字节数拆分，避免单个 partition 过大；
- 下游可以尽早消费并释放 B1，而不是等待整个上游 partition 全部完成，降低峰值内存。

若过滤等算子使输出 partition 太小，调度器会把多个小 partition 合并后交给一个下游 task，
直到目标大小。`map_batches` 还把用户关心的 GPU batch size 与系统的 partition size 分开：
executor 在 partition 内切 batch，调度器也可合并过小 partition 来凑够 batch。

## 核心机制二：让动态分区仍可用 lineage 恢复

普通 Ray task 在提交时就知道输出数量，而 generator task 的输出数量只有执行后才知道。
Ray Data 利用纯函数的确定性把两者兼容起来：

1. 相同输入、相同目标 partition size 和无副作用的确定性 transform，必须产生相同的
   partition 序列；
2. 第一次成功执行 generator task 时，caller 记录实际输出数量；
3. 任一输出丢失时，重新执行整个 task；
4. 重放得到的输出数量若不同，则直接报错，说明 UDF 或分区过程不满足确定性假设。

因此动态分区计划不必预先写入 lineage，而是可以在首次执行时确定下来。恢复仍然只需重算受影响的 partition，不必像 global checkpoint 那样让所有 operator 回滚。

这个保证有明确前提：driver/集中式调度器仍然存活，task 的参数和已知输出不可变，用户 UDF
确定且无副作用。若调度器失败，论文版本中的 Ray Data 会让整个 job 从头执行；可以通过额外
checkpoint 缩小回滚范围。

## 核心机制三：内存感知的动态资源调度

调度器保存每个 materialized partition 的行数、字节数、节点位置，以及每个 operator 的输入队列、运行中 task 和资源需求。每当有 partition 产生或 task 结束，它都重新做一次决策：

1. 把新输出 reference 放入下游队列，task 全部输出完毕后释放其资源；
2. 从“有输入、资源可用、输出缓冲区有空间”的 operator 中选择新 task；
3. 优先选择已缓冲输出最少的 operator，避免慢算子上游继续堆积；
4. 在 partition 边界把空闲 CPU、GPU 或自定义资源重新分配给当前最需要的 operator。

同一个物理 slot 因而可以随时间在多个 operator 之间切换，甚至实现静态 executor 数无法表达的 fractional parallelism。例如两个阶段耗时比为 1:2 时，8 个 CPU 理想并行度为
2.67:5.33；Ray Data 可以通过 time-slicing 在一段时间内逼近 1:2 的资源比例。

### 悲观策略

悲观策略类似流系统的 backpressure：shared memory 达到硬限制后，正在执行的上游 task 停止向 object store flush，新 task 也要等下游释放空间。它安全且不依赖预测，但可能让本来能与下游同时完成的上游 task 启动得太晚，留下空闲资源。

### 乐观策略

乐观策略用在线 profile 估计每个 operator 的 task 时长、输入输出大小比和当前可用 slot，
先估算处理一个 source partition 所需的总时间：

```text
α₀ = 1
αᵢ = αᵢ₋₁ × Oᵢ / Iᵢ
Pᵢ = (Tᵢ / Eᵢ) × αᵢ₋₁
P  = Σ Pᵢ
```

其中 `Tᵢ` 是 operator `i` 的平均 task 时长，`Eᵢ` 是可用 execution slot，`αᵢ` 是从 source
到该阶段的累计数据膨胀率。`P` 近似表示 pipeline 每处理一个 source partition 所需的时间。

调度器维护一个可让新数据进入 pipeline 的 memory budget。每启动一个 source task，就扣除它的预计输出大小；每秒按 `source_partition_size / P` 补充预算，也就是按估计的排空速率放行新输入。理想 profile 下可得到最优 schedule；估计过于乐观时会短暂 backpressure 或
spill，但它具有负反馈：过多上游 task 占用 slot 会降低下游并行度，从而降低下一轮预算补充
速度。

这一设计的本质是联合调度**计算 slot 和中间数据内存**。只看空闲 CPU/GPU 会让上游过度生产，
只做 backpressure 又会保守地浪费可用计算资源。

## 实现

Ray Data 作为 Ray 上层的 Python library 实现，而不是修改成一个新的 monolithic engine。
它复用 Ray 的动态 task、actor、自动数据移动、distributed object store、lineage recovery
和 disk spilling，只对 Ray core 增加 generator task 及相应 recovery 支持。

论文报告 Ray Data 约有 5.1 万行 Python，其中 query planner、scheduler 和 map executor 的
核心逻辑分别约为 1000、1000 和 2000 行。集中式 scheduler 只处理 partition reference 和
元数据，partition 内容在 worker 与 object store 之间直接移动。

## 实验结论

实验比较 Spark 3.5.1、Flink 1.19.0、tf.data、PyTorch DataLoader 和基于 Ray 2.40.0 的
Ray Data。为减少不同系统实现造成的干扰，作者还构造了两个消融版本：

- `Ray Data-staged`：各 stage 顺序物化，模拟 batch；
- `Ray Data-static`：静态 operator 并行度加 round-robin，模拟 stream；
- 完整版本也称 `Ray Data-dynamic`。

| 场景 | 主要结果 | 说明 |
| --- | --- | --- |
| RAG，10 万 prompts | 1 GPU 比顺序 baseline 快 1.32×；2/4/8 GPU 相对 1 GPU 为 1.88/3.58/6.44× | 8 GPU 时 CPU encode/retrieve 成为瓶颈 |
| VideoMAE，4 个 A10G 节点 | 达到理论最优 GPU 时间的 88.4%；比 Flink 快 2.5×，比静态 Ray Data 快 1.25× | batch 版本因完整物化和 spill，到 31/53 分钟才首次产出 |
| 故障与节点变化 | executor 故障几乎无吞吐下降；CPU 节点移除、加入时平滑改变吞吐 | global checkpoint 版本每次变化都停机并回滚 |
| ResNet-50，本地盘 | tf.data 吞吐比 Ray Data 低 19% | tf.data 为避免 OOM 需要降低 batch size |
| ResNet-50，S3 | Ray Data 加 CPU-only 节点后达到 GPU 峰值的 93% | 单节点 tf.data 因本地 CPU 不足，比峰值低 88% |
| Stable Diffusion，一个 epoch | 111.3 h 降至 76.8 h，缩短 31%；成本降低 11% | 704 CPU、32 个 A100 训练 GPU，另加 40 个 A10G 做 encoder |
| 受限内存 microbenchmark | 除最低内存外均约为理论最优时间的 1.3× | 去掉动态分区会产生过大 partition；去掉乐观策略慢 10–88% |
| 扩展性，最多 32 个 worker | 大集群上线性扩展，吞吐最高比原生 Ray task/generator 高 1.8× | 1–2 节点时 query planning 的 warmup 反而更慢 |

Video workload 中，Spark 和 staged Ray Data 的完成时间分别为 116 和 61 分钟，说明异构场景
的主要问题不只是某个引擎实现慢，而是 stage barrier 迫使中间结果全部物化。Flink 能立即
产出，但 Java/Python 间序列化复制和静态 round-robin 限制了吞吐。完整 Ray Data 在两个
Ray 消融版本之间的差距进一步说明，动态分区与动态调度各自都有贡献。

内存消融也很关键：Spark 最好仍需理论时间的 2.35 倍，低内存下达到 4.34 倍甚至 OOM；
Flink 最好为 1.68 倍，但低内存时为控制每个 executor 的线程和内存而减少并行度，最差再慢
约 2 倍；Ray Data 则通过动态分区稳定住单 partition 大小，再由乐观调度减少 backpressure
造成的空泡。

## 论文的边界与不足

1. **面向有界、吞吐优先的 map-style pipeline**。论文不是在替代带 event time、watermark、
   window 和长期 operator state 的实时流处理系统；sort、group-by 等 all-to-all 操作依赖
   另一项 Exoshuffle 工作；
2. **容错依赖函数式假设**。动态分区的 lineage recovery 要求 UDF 确定、纯净且无副作用，
   对外部系统写入、随机增强或非确定 GPU 算子需要额外处理；
3. **集中式 scheduler 不是高可用的**。scheduler/driver 失败仍会让 job 从头重做。论文只在
   最多 32 个 worker 的合成空 workload 中证明扩展性，没有解决 scheduler HA；
4. **仍有静态配置**。输入 partition 数由 planner 预估，初始 cluster shape 由用户指定，
   论文把联合 query planning、autotuning 和 autoscaling 留作未来工作；
5. **部分跨系统比较混合了模型与实现差异**。例如 Flink 的 Java/Python 序列化开销和 Spark
   的具体物化路径并非执行模型的必然成本；Ray 内部的 staged/static 消融更能支撑核心结论；
6. **大规模故障实验有限**。故障实验只有一个 GPU 节点和一个 CPU-only 节点，尚不足以展示
   多节点同时失败、长 lineage 重算或控制面故障时的行为；
7. **Stable Diffusion 的措辞需谨慎**。表中 2811 images/s 到 4075 images/s 实际是约 45%
   吞吐提升，111.3 h 到 76.8 h 才是 31% 时间缩短；论文将后者也表述成 “31% better
   throughput”。此外，Ray Data 增加了 40 个 A10G，故成本只降低 11%，少于时间降幅。

## 我的理解

论文最重要的观察是：异构 ML pipeline 中，**partition 不应只是文件的静态切片，而应成为一个运行时控制对象**。一旦允许 partition 在执行中确定，系统就能在同一粒度上做四件事：

- 以 task + partition 保留 batch 系统的可移动性和 lineage；
- 以逐 partition yield 实现 stream 系统的 pipelining；
- 以实际字节数而不是记录数控制内存；
- 在 partition 边界重新分配异构资源，避免长期绑定 executor。

它也体现了控制面与数据面分离的价值。集中调度器之所以能做 pipeline-aware 的全局决策，
是因为它掌握 DAG、partition 大小、位置和资源状态；它之所以没有立刻成为数据瓶颈，是因为数据仍由 Ray object store 在 worker 间传输。与原始 Ray 论文相比，Ray Data 在通用 task
runtime 之上增加了一层领域语义：Ray 只看到黑盒 task，Ray Data 则知道这些 task 属于哪个
operator、位于 pipeline 何处、会产生多少中间数据，因此能做更好的 admission control 和负载均衡。

最后，Streaming Batch Model 的“统一”不是把 batch API 和 stream API 放进同一个产品，
也不是声称一种模型能覆盖所有场景。它针对的是可重放、吞吐优先的有界异构数据流，用动态
partition 消除 batch 与 stream 在执行机制上的一组特定冲突。这个范围比“流批一体”更窄，
但也更具体、更容易从系统机制和实验中验证。
