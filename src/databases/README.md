## databases

- **[Column-stores vs. row-stores: how different are they really?][column-stores-vs-row-stores]**
- **[Main Memory Database Systems](mmdb/index.html)**
  - **[Main Memory Database Systems: An Overview][mmdb-overview]**
- **[KV databases](kv/index.html)**
  - **[Optimizing Space Amplification in RocksDB][rocksdb-cidr17]**
- **[OLTP](oltp/index.html)**
  - **[OLTP Through the Looking Glass, and What We Found There][lookingglass]**
  - **[Staring into the Abyss: An Evaluation of Concurrency Control with One Thousand Cores][staring-abyss]**
- **[OLAP](olap/index.html)**
  - **[Lakehouse: A New Generation of Open Platforms that Unify DataWarehousing and Advanced Analytics][lakehouse]**
  - **[Delta Lake: HighPerformance ACID Table Storage over Cloud Object Stores][deltalake]**
  - **[The Vertica Analytic Database: C-Store 7 Years Later][vertica]**
- **[HTAP](htap/index.html)**
  - **[Greenplum: A Hybrid Database for Transactional and Analytical Workloads][greenplum]**
- **[Citus: Distributed PostgreSQL for Data-Intensive Applications][citus]**
- **[Optimizer](optimizer/index.html)**
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
