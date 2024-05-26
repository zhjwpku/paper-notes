### [Data Management for Data Science Towards Embedded Analytics](https://duckdb.org/pdf/CIDR2020-raasveldt-muehleisen-duckdb.pdf)

> CIDR 2020

该论文探讨了数据科学领域对数据管理解决方案的需求，并提出了一类新的数据管理系统：嵌入式分析系统（embedded analytical systems）。

数据科学崛起导致数据分析任务复杂性增加，数据科学家通常使用脚本语言（如 Python 或 R）在个人电脑上进行中等规模的分析，传统的 RDBMS 不能满足数据科学家的需求，不适合本地数据分析用例。

嵌入式分析系统为了填补了这一空白，要解决如下一些问题:

- Combined OLAP & ETL Workloads: 高效率地支持 OLAP 工作负载以及批量追加和更新
- Transfer Efficiency: 数据库和应用程序在同一进程和地址空间中运行，需要有效利用这一机会实现高效的数据共享
- Resilience: 嵌入式数据库需要能够检测硬件问题并防止数据损坏
- Cooperation: 系统需要能够适应资源竞争，与主机应用程序共享硬件资源

**DuckDB**

DuckDB 是一个为嵌入式分析设计的新型关系型数据库管理系统。

- OLAP & ETL：使用矢量化解释执行引擎，优化了 OLAP 查询
- 弹性：DuckDB 计算并存储持久存储块的校验和，并在读取时验证，以保护数据完整性
- 协作：DuckDB 允许用户手动设置内存和 CPU 核心使用率的硬限制，并计划实现自适应资源使用方案
- 传输效率：DuckDB 实现了高效的客户端 API，允许客户端应用程序直接成为物理查询处理计划的根算子

### Code

[DuckDB is an in-process SQL OLAP Database Management System](https://github.com/duckdb/duckdb)

### Further readings

- [DuckDB: an Embeddable Analytical Database](https://duckdb.org/pdf/SIGMOD2019-demo-duckdb.pdf)
