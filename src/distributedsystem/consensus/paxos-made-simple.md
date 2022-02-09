### [Paxos Made Simple](../assets/pdfs/paxos-simple.pdf)

> ACM SIGACT News (Distributed Computing Column) 32, 4 | December 2001, pp. 51-58
>
> https://www.microsoft.com/en-us/research/publication/paxos-made-simple/

Paxos 分布式共识算法是一个系列，Paxos Made Simple 是对其基本算法的描述，称为 Basic Paxos。Paxos 算法运行一次只能决定**一个**值，而非多个，将此牢记于心可以帮助理解 Paxos。

算法包含三种角色: `Proposer`、`Acceptor` 和 `Learner`，同一进程可扮演多种角色。其中 `Acceptor` 的多数派构成 quorum，2m+1 个 Acceptor 可以容忍 m 个节点 fail-stop 错误（非拜占庭失败）。

算法分两个阶段:

- Phase 1
  - A proposer selects a proposal number n and sends a prepare request with number n to a majority of acceptors.
  - If an acceptor receives a prepare request with number n greater than that of any prepare request to which it has already responded, then it responds to the request with a promise not to accept any more proposals numbered less than n and with the highest-numbered proposal (if any) that it has accepted.
- Phase 2
  - If the proposer receives a response to its prepare requests (numbered n) from a majority of acceptors, then it sends an accept request to each of those acceptors for a proposal numbered n with a value v, where v is the value of the highest-numbered proposal among the responses, or is any value if the responses reported no proposals.
  - If an acceptor receives an accept request for a proposal numbered n, it accepts the proposal unless it has already responded to a prepare request having a number greater than n.

以上算法没有描述 Learner 是如何知道共识已经达成，paper 中给出了两种方法:

- For each acceptor, whenever it accepts a proposal, respond to all learners, sending them the proposal
- A less reliable model, but one that reduces communication, is to have one or more nominated ‘distinguished learners’ to which acceptors send their acceptance notifications, and these then broadcast to the rest of the learners.

Paxos 算法中一些要点:

- Acceptor 需要持久存储记录 minProposal、acceptedProposal、acceptedValue 等状态
- Paxos 算法的三个里程碑（详细介绍参考 [3]）
  - When the proposer receives promise(n) messages from a majority of acceptors.
  - When a majority of acceptors all issue accepted(n,val) messages for proposal number n and some value val.
  - When the proposer(s) and learners receive accepted(n,val) messages from a majority of the acceptors.
- 如果需要对一系列值进行决策，需要多次运行 Paxos 算法，这称为 Multi-Paxos，Multi-Paxos 在学术界没有明确的阐述，在实现上需要解决很多问题，[4] 中有相应介绍


#### References:

[1] [The Paxos Algorithm](https://www.youtube.com/watch?v=d7nAGI_NZPk), A Google TechTalk presented by Luis Quesada Torres (Feb 2, 2018)<br>
[2] [Paxos made simple](https://blog.acolyer.org/2015/03/04/paxos-made-simple/) by Adrian Colyer<br>
[3] [Distributed Systems Lecture 15](https://www.youtube.com/watch?v=UCmAzWvrFmo) by Lindsey Kuper<br>
[4] [Paxos lecture (Raft user study)](https://www.youtube.com/watch?v=JEpsBg0AO6o) by John Ousterhout