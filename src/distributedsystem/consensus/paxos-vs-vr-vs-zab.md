### [Vive La Difference: Paxos vs. Viewstamped Replication vs. Zab](../../assets/pdfs/viveLaDifference.pdf)

> IEEE Transactions on Dependable and Secure Computing ( Volume: 12, Issue: 4, July-Aug. 1 2015)
>
> https://doi.org/10.1109/TDSC.2014.2355848

本论文使用 **refinement** 的方法对 **Paxos**、**Viewstamped Replication** 和 **Zab** 三种复制协议进行了对比。

文中给出了通用算法及三种协议不同术语的对比:

![terms](../../assets/images/vive_la_difference_terms.jpg)

论文在 Replication 层将算法分为 Active Replication 和 Passive Replication:

- Active Replication: also known as **state machine replication**, each replica implements a deterministic state machine. All replicas process the same operations in the same order.
- Passive Replication: also known as **primary backup**, a primary replica runs a deterministic state machine, while backups only store states. The primary computes a sequence of new application states by processing operations and forwards these states to each backup in order of generation.

论文将 VR 和 Zab 归类为 Passive Replication，这是我不太理解的地方。本篇论文只是浏览了一遍，之后可能会回过头再次学习。