### [Distributed Snapshots: Determining Global States of Distributed Systems](../assets/pdfs/chandy.pdf)

> ACM Transactions on Computer Systems, Volume 3, Issue 1 ∙ Feb. 1985 ∙ pp 63–75
>
> https://doi.org/10.1145/214451.214456

本文提出了一个在分布式系统中生成全局逻辑一致性快照的算法，该算法依附于底层的分布式计算（即不需要暂停计算任务）。分布式系统中的进程通过发送和接收消息进行通信，进程可以记录它自己的状态和它发送和接收的消息，不记录任何其它内容。线程之间不共享时钟或内存。

Chandy-Lamport 假设进程之间的拓扑是强连接（既任何两个进程之间直接或间接可以通信），在讲述该算法时，很多例子会用全连接的拓扑进行描述。

强连接:

![strongly connected](https://raw.githubusercontent.com/ChrisWhealy/DistributedSystemNotes/master/img/L9%20Connected%20Graph.png)

全连接:

![complete graph](https://raw.githubusercontent.com/ChrisWhealy/DistributedSystemNotes/master/img/L9%20Total%20Graph.png)

该算法正确工作的前提:

- FIFO Delivery
- Reliable Delivery
- Processes Don't Crash

每个进程负责记录:

- Its own internal state
- The state of all messages on its incoming channels

算法的核心依赖 Marker 消息在拓扑图中的传递，完整的全局状态检测算法由 `Marker Sending Rule` 和 `Marker Receiving Rule` 获取:

- The Marker Sending Rule for a process p: for each channel c incident on, and directed away from p, p sends one marker along c after p records its state and before p sends any further messages along c.
- The Marker Receiving Rule for a process q: on receiving a marker along a channel c, if q has not yet recorded its state then it records its state, and records the state of c as empty. However, if q has already recorded its state, then the state of c is simply recorded as the sequence of messages received along c in between q recording its state and receiving the marker on c.

[CSE138 (Distributed Systems) L8: Chandy-Lamport snapshot algorithm](https://www.youtube.com/watch?v=x1BCZ351dJk) 描述了全连接拓扑时的 Chandy-Lamport 算法:

*The Initiator Process*

- Records its own state
- Sends a marker message out on all its outgoing channels
- Starts recording messages arriving on all incoming channels

*Processes Receiving a Marker Message* (含 Initiator 进程)

当进程第一次看见 Marker 消息时:

- Records its own state
- Flags the channel on which the marker message was received as **empty**
- Sends out a marker message on each of its outgoing channels
- Starts recording incoming messages on all channels **except** the one on which it received the original marker message (now flagged as empty)

如果不是第一次看见，则意味着该进程已经发出自己的 Marker 消息之后又接收到其它的 Marker 消息:

- Stops recording incoming messages on that channel
- Sets that channel's final state to be the sequence of all messages received whilst recording was active

简单理解就是，对于一条链路，接收端在接收到 marker 消息之前的消息都会被接收端的状态记录，发送端发送 marker 消息之后的消息都会被接收端忽略。

算法记录的全局状态虽然可能对应不上系统所处的任何状态，但它提供了一个逻辑一致的快照状态，保证初始系统状态到最终系统状态是可达的。

另外，管理系统快照需要外部协调进程来处理:

- 接收所有进程记录的 local state 和 ingest channel state
- 整理这些状态来形成全局系统快照

全局快照可用于解决如下问题:

- Checkpointing
- Deadlock detection
- Stable Property Detection

#### References:

[1] [Distributed Snapshots: Determining Global States of Distributed Systems](https://blog.acolyer.org/2015/04/22/distributed-snapshots-determining-global-states-of-distributed-systems/) by Adrian Colyer<br>
[2] [Distributed Systems Lecture 8 Notes](https://github.com/ChrisWhealy/DistributedSystemNotes/blob/master/Lecture%2008.md)