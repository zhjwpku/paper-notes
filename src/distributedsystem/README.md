## distributed system

在 [A Brief Introduction to Distributed Systems](../assets/pdfs/A%20brief%20introduction%20to%20distributed%20systems.pdf) 的介绍中，分布式系统被定义为:

> A distributed system is a **collection of autonomous computing elements** that appears to its users as a **single coherent system**.

图灵奖获得者 Leslie Lamport 将分布式系统描述为:

> one in which the failure of a computer you did not even know existed can render your own computer unusable.

- **[consensus](./consensus/index.html)**
  - **[Impossibility of Distributed Consensus with One Faulty Process][flp]**
  - **[Paxos Made Simple][paxos-simple]**
  - **[Paxos Made Live][paxos-live]**
  - **[Viewstamped Replication][vr]**
  - **[Zab: High-performance broadcast for primary-backup systems][zab]**
  - **[Vive La Difference: Paxos vs. Viewstamped Replication vs. Zab][vivala]**
  - **[In Search of an Understandable Consensus Algorithm][raft]**
  - **[Paxos vs Raft: have we reached consensus on distributed consensus?][paxos-vs-raft]**
- **[A principle for resilient sharing of distributed resources][primary-backup]**
- **[Chain Replication for Supporting High Throughput and Availability][chain-replication]**
- **[Paxos Replicated State Machines as the Basis of a High-Performance Data Store][bolosky]**
- **[Detecting Causal Relationships in Distributed Computations: In Search of the Holy Grail][holygrail]**
- **[Distributed Snapshots: Determining Global States of Distributed Systems][chandy]**
- **[Lightweight Asynchronous Snapshots for Distributed Dataflows][abs]**
- **[ZooKeeper: wait-free coordination for internet-scale systems][zookeeper]**
- **[scheduler](./scheduler/index.html)**
  - **[Large-scale cluster management at Google with Borg][borg]**

[primary-backup]: primary-backup.md
[chain-replication]: chain-replication.md
[holygrail]: holygrail.md
[chandy]: chandy.md
[abs]: abs.md
[zookeeper]: zookeeper.md
[flp]: consensus/flp.md
[paxos-simple]: consensus/paxos-made-simple.md
[paxos-live]: consensus/paxos-made-live.md
[zab]: consensus/zab.md
[raft]: consensus/raft.md
[vr]: consensus/vr.md
[vivala]: consensus/paxos-vs-vr-vs-zab.md
[paxos-vs-raft]: consensus/paxos-vs-raft.md
[borg]: scheduler/borg.md
[bolosky]: bolosky.md