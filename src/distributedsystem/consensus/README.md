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
- **[Paxos vs Raft: have we reached consensus on distributed consensus?][paxos-vs-raft]**

#### Further readings

[1] [Disk Paxos](../../assets/pdfs/disk-paxos.pdf) by Eli Gafni & Leslie Lamport, 2002<br>
[2] [The ABCD’s of Paxos](../../assets/pdfs/paxos-abcd.pdf) by Butler W. Lampson, PODC 2001<br>
[3] Paxos for System Builders: [An Overview](../../assets/pdfs/paxos-for-system-builders-an-overview.pdf) and [The Complete Specification](../../assets/pdfs/paxos_for_system_builders.pdf) by Yair Amir and Jonathan Kirsch, 2008<br>
[4] [How to Build a Highly Available System Using Consensus](../../assets/pdfs/how-to-build-a-highly-available-system-using-consensus.pdf) by Butler W. Lampson, 1996<br>
[5] [There is no now](../../assets/pdfs/there_is_no_now.pdf) by Justin Sheehy, 2015

#### References

[1] [Can’t we all just agree?](https://blog.acolyer.org/2015/03/01/cant-we-all-just-agree/)<br>
[2] [Distributed Consensus Reading List 📚](https://github.com/heidihoward/distributed-consensus-reading-list) maintained by Heidi Howard


[paxos-simple]: paxos-made-simple.md
[paxos-live]: paxos-made-live.md
[raft]: raft.md
[flp]: flp.md
[zab]: zab.md
[vr]: vr.md
[vivela]: paxos-vs-vr-vs-zab.md
[paxos-vs-raft]: paxos-vs-raft.md
