## databases

- **[Cloud Native Database Systems](cloudnative/index.html)**
  - **[Amazon Aurora: Design Considerations for High Throughput Cloud-Native Relational Databases][aurora]**
  - **[Taurus Database: How to be Fast, Available, and Frugal in the Cloud][taurus]**
- **[Column-stores vs. row-stores: how different are they really?][column-stores-vs-row-stores]**
- **[Main Memory Database Systems](mmdb/index.html)**
  - **[Main Memory Database Systems: An Overview][mmdb-overview]**
- **[KV databases](kv/index.html)**
  - **[Optimizing Space Amplification in RocksDB][rocksdb-cidr17]**
  - **[WiscKey: Separating Keys from Values in SSD-Conscious Storage][wisckey]**
- **[OLTP](oltp/index.html)**
  - **[OLTP Through the Looking Glass, and What We Found There][lookingglass]**
  - **[Staring into the Abyss: An Evaluation of Concurrency Control with One Thousand Cores][staring-abyss]**
  - **[A History and Evaluation of System R][system-r]**
- **[OLAP](olap/index.html)**
  - **[Lakehouse: A New Generation of Open Platforms that Unify DataWarehousing and Advanced Analytics][lakehouse]**
  - **[Delta Lake: HighPerformance ACID Table Storage over Cloud Object Stores][deltalake]**
  - **[The Vertica Analytic Database: C-Store 7 Years Later][vertica]**
  - **[Data Management for Data Science Towards Embedded Analytics][duckdb]**
- **[HTAP](htap/index.html)**
  - **[Greenplum: A Hybrid Database for Transactional and Analytical Workloads][greenplum]**
- **[Vector DB](vectordb/index.html)**
  - **[Hierarchical NSW][hnsw]**
  - **[IVF-HNSW][ivf-hnsw]**
  - **[DiskANN][diskann]**
  - **[Product Quantization][pq]**
- **[Graph Database](graphdb/index.html)**
  - **[Graph Pattern Matching in GQL and SQL/PGQ][gpml]**
  - **[KÙZU Graph Database Management System][kuzu]**
- **[Citus: Distributed PostgreSQL for Data-Intensive Applications][citus]**
- **[Optimizer](optimizer/index.html)**
  - **[An Overview of Query Optimization in Relational Systems][overview]**
  - **[Access path selection in a relational database management system][system_r]**
  - **[The Volcano Optimizer Generator: Extensibility and Efficient Search][opt-volcano]**
- **[Executor](executor/index.html)**
  - **[Volcano — An Extensible and Parallel Query Evaluation System][volcano]**
- **[Concurrency Control](concurrencycontrol/index.html)**
  - **[An Empirical Evaluation of In-Memory Multi-Version Concurrency Control][empirical-evaluation-of-mvcc]**
- **[Change Data Capture](cdc/index.html)**
  - **[DBLog: AWatermark Based Change-Data-Capture Framework][dblog]**
- **[Designing Access Methods: The RUM Conjecture][rum]**

#### Optional readings

- M. Stonebraker, et al., [What Goes Around Comes Around](./../assets/pdfs/whatgoesaround-stonebraker.pdf)
- A. Pavlo, et al., [What’s Really New with NewSQL?](../assets/pdfs/pavlo-newsql-sigmodrec2016.pdf)
- Yihe Huang, et al., [Opportunities for Optimism in Contended MainMemory Multicore Transactions](../assets/pdfs/opportunities_for_optimism_in_contented_main-memory_multicore_transactions.pdf)

[column-stores-vs-row-stores]: colum-stores-vs-row-stores.md
[mmdb-overview]: mmdb/overview.md
[lakehouse]: olap/lakehouse.md
[dblog]: cdc/dblog.md
[rum]: rum.md
[greenplum]: htap/greenplum-htap.md
[deltalake]: olap/delta-lake.md
[vertica]: olap/vertica.md
[rocksdb-cidr17]: kv/optimizing-space-amplification-in-rocksdb.md
[lookingglass]: oltp/oltp-through-the-looking-glass.md
[staring-abyss]: oltp/staring-into-the-abyss.md
[empirical-evaluation-of-mvcc]: concurrencycontrol/empirical-evaluation-of-mvcc.md
[volcano]: executor/volcano.md
[citus]: citus.md
[hnsw]: vectordb/hnsw.md
[pq]: vectordb/pq.md
[ivf-hnsw]: vectordb/ivf-hnsw.md
[wisckey]: kv/wisckey.md
[diskann]: vectordb/diskann.md
[duckdb]: olap/duckdb.md
[aurora]: cloudnative/aurora.md
[taurus]: cloudnative/taurus.md
[gpml]: https://arxiv.org/pdf/2112.06217
[kuzu]: graphdb/kuzu.md
[system-r]: oltp/system_r.md
[overview]: optimizer/overview.md
[system_r]: optimizer/system_r.md
[opt-volcano]: optimizer/volcano.md
