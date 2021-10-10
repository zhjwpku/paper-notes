### [Lakehouse: A New Generation of Open Platforms that Unify DataWarehousing and Advanced Analytics](../../assets/pdfs/lakehouse.pdf)

> CIDR 2021. Matei Zaharia, Ali Ghodsi, Reynold Xin, Michael Armbrust
>
> https://dblp.org/rec/conf/cidr/Zaharia0XA21.html

数据分析平台发展历程经历了如下两代:

- 第一代: `schema-on-write`。通过将 [Operational database](https://en.wikipedia.org/wiki/Operational_database) 的数据收集到数仓以支持 Decision Support 及 Business Intelligent。Figure 1-a。
- 第二代: `schema-on-read`。将开放文件格式（如 Apache Parquet 和 ORC）的数据离线存储在成本较低 Data Lake 中（如 HDFS），通过 ETL 操作将一部分数据抽取到数仓后提供 DS 及 BI 的能力。这种架构在提供低成本存储各种数据类型的同时牺牲了数据质量及治理能力。从 2015 年开始，Cloud Data Lake（S3，GCS 等）由于具有极低的成本及优秀的持久性开始替代传统的 Data Lake，逐渐形成了 data lake + data warehouse 的两层架构。Figure 1-b。

![two-tier model and lakehouse model](./../../assets/images/lakehouse-two-tier-model.jpg)

两层架构由于存 (如 S3) 算 (如 Redshift) 看起来很便宜，但从用户角度却非常复杂，数据首先需要从 OLTP 数据库 ETL 到 Data Lake，然后再从 Data Lake ETL 到数仓。这种架构主要存在四个问题：

- Reliablity: 多次 ETL 降低了数据可靠性
- Data Staleness: 相比第一代架构数据直接 ETL 到数仓，两层架构反而需要更多的等待
- Limited support for advanced analytics: 不同于 BI 计算模型只需抽取少量数据，机器学习需要处理大量的 dataset，通过 ODBC/JDBC 读取这些数据非常低效
- Totol cost of ownship: 数仓中的数据由于在 Data Lake 中也保存了一份，因而增加了存储成本

*两层架构通过增加对 Parquet 和 ORC 外表的支持，可以使用相同的 SQL 引擎查询 Data Lake 上的数据，但这通常具有较差的性能；一些研究将 SQL 引擎直接运行在 data lake 之上，但这也不能解决 data lake 缺少 ACID 事务的问题。*

由于这些问题的存在，本文讨论的主题围绕一个技术问题展开:

> Is it possible to turn data lakes based on standard open data formats,
> such as Parquet and ORC, into high-performance systems that can
> provide both the performance and management features of data
> warehouses and fast, direct I/O from advanced analytics workloads?

作者认为这种 Lakehouse（Data `Lake` + Data Ware`house`）架构的时代即将来临，鉴于它解决了以下关键问题：

- Reliable data management on data lakes: 支持事务、回滚及零拷贝克隆（如 Delta Lake、Apache Iceberg）
- Support for machine learning and data science: 通过提供 DataFrame API 来支持 ML 等计算模型
- SQL performance: 通过维护一些辅助数据及优化数据布局（Z-order/Hilbert Curves）来提供优异的查询性能

Lakehouse 架构作为一个数据管理系统，具有如下特性:

- 基于低成本直接访问存储
- 提供传统分析型 DBMS 管理功能
- 支持事务、数据版本、审计、索引、缓存
- 支持查询优化

Databricks 通过 Delta Lake、Delta Engine 和 Databricks ML Runtime 三个项目构建了一个 Lakehouse 数据分析平台。

![two-tier model and lakehouse model](./../../assets/images/lakehouse-metadata-layer.jpg)

图中展示的 metadata layer，既 Delta Lake 提供了 ACID 事务及版本管理；Caching 及 Indexing Layer 使得优秀的查询性能成为可能；在如上模块的基础上提供的 DataFrame API 增加了对 ML 和数据科学的支持。文中第三节对 Figure 2 中的各模块进行了详细的描述分析，并给出了对应模块的未来研究方向，这里不再赘述。
