### [Impossibility of Distributed Consensus with One Faulty Process](../../assets/pdfs/flp.pdf)

> Proceedings of the 2nd ACM SIGACT-SIGMOD symposium on Principles of database systems, March 1983
>
> https://dl.acm.org/doi/abs/10.1145/588058.588060

> In this paper, it is shown that every protocol for consensus problem has the possibility of **nontermination**, even with only one faulty process.

在异步环境下，或者真实的系统中，往往会出现进程崩溃、网络分区、消息丢失/乱序/冗余等问题。本文在假设网络可靠（it delivers all messages correctly and exactly once）、non-Byzantine Failure 的异步系统中，即使只有一个进程在不合适的时间停机，也会导致任何分布式提交协议失败。

FLP result 的证明还有如下几点假设:

- make no assumptions about the relative speeds of processes or about the delay time in delivering a message.
- the processes do not have access to synchronized clocks, so algorithms based on time-outs can not be used.
- do not postulate the ability to detect the death of a process

证明过程有点晦涩，可以结合参考中的连接和 paper 一起理解，我只是浏览了一遍，后面需要可能需要继续学习 :(

论文的结论中指出：

FLP result 并非认为 `fault-tolerant cooperative computing` 问题在实际中不可解决，而是指出了如果想要解决这些问题，需要更细化的错误模型，如对通信时间、失败检测的假设。

Paxos 和 Raft 理论上可能永远运行却不能达成共识（即 nontermination），软件工程通常使用 random 超时机制来减少这种可能性。

#### References

[1] [A Brief Tour of FLP Impossibility](https://www.the-paper-trail.org/post/2008-08-13-a-brief-tour-of-flp-impossibility/)<br>
[2] [John Feminella on Impossibility of Distributed Consensus with One Faulty Process](https://www.youtube.com/watch?v=Vmlj-67aymw)
