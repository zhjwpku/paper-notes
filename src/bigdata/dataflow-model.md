### [The Dataflow Model: A Practical Approach to Balancing Correctness, Latency, and Cost in MassiveScale, Unbounded, OutofOrder Data Processing](../pdfs/../../assets/pdfs/the_dataflow_model.pdf)

> Proceedings of the VLDB EndowmentVolume 8Issue 12August 2015 pp 1792–1803
>
> https://doi.org/10.14778/2824032.2824076

**The future of data processing is unbounded data**，尽管 bounded data 依然有用武之地，它在语义上可以被归类为 ubounded data。`Dataflow Model` 就是这样一个提供无界数据和有界数据统一处理框架的模型。

论文介绍了 Windowing、Time Domains 相关的各种概念，附加了很多图例，Apache Flink 在一定程度上借鉴了该论文的理念。

阅读该论文是因为最近在学习 Flink，并总结了一些学习资源: [Awesome Flink Learning Resources](https://zhjwpku.com/2021/11/01/awesome-flink.html)。