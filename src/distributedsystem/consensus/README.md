## consensus

共识算法通常伴随着复制状态机（replicated state machines）问题，它是分布式系统能够容错的基础，多台服务器就一系列值达成一致，一旦它们就某个值做出决定，该决定就不能被改变。

> 任何具有 TAV 特性的算法都可以被认为在解决共识问题
>
> - termination: all non-faulty processes make a decision
> - agreement: all deciding processes make the same decision
> - validity: some process proposed the decision

- **[Impossibility of Distributed Consensus with One Faulty Process][flp]**
- **[Paxos Made Simple][paxos-simple]**
- **[Paxos Made Live][paxos-live]**
- **[Viewstamped Replication][vr]**
- **[Zab: High-performance broadcast for primary-backup systems][zab]**
- **[Vive La Difference: Paxos vs. Viewstamped Replication vs. Zab][vivela]**
- **[In Search of an Understandable Consensus Algorithm][raft]**

#### References

[1] [Can’t we all just agree?](https://blog.acolyer.org/2015/03/01/cant-we-all-just-agree/)


[paxos-simple]: paxos-made-simple.md
[paxos-live]: paxos-made-live.md
[raft]: raft.md
[flp]: flp.md
[zab]: zab.md
[vr]: vr.md
[vivela]: paxos-vs-vr-vs-zab.md
