## kv databases

- **[Optimizing Space Amplification in RocksDB][rocksdb-cidr17], 2017**
- **[Evolution of Development Priorities in Key-value Stores Serving Large-scale Applications: The RocksDB Experience][fast21-dong], 2021 [a slightly more detailed version](/assets/pdfs/rocksdb-evolution-2021.pdf)**
- **[WiscKey: Separating Keys from Values in SSD-Conscious Storage][wisckey], 2016**

### Further readings

- [Pipelined Compaction for the LSM-tree][pcp], 2014
- [HashKV: Enabling Efficient Updates in KV Storage via Hashing](/assets/pdfs/hashkv-atc18.pdf), 2018
- [Dostoevsky: Better Space-Time Trade-Offs for LSM-Tree Based Key-Value Stores via Adaptive Removal of Superfluous Merging][dostoevskykv], 2018, [video](https://www.youtube.com/watch?v=fmXgXripmh0)
- [Accordion: better memory organization for LSM key-value stores][accordion], 2018, HBase Memory Improvement
- [LDC: A Lower-Level Driven Compaction Method to Optimize SSD-Oriented Key-Value Stores][ldc], 2019
- [Characterizing, Modeling, and Benchmarking RocksDB Key-Value Workloads at Facebook][fast20-cao_zhichao], 2020


[rocksdb-cidr17]: optimizing-space-amplification-in-rocksdb.md
[fast21-dong]: /assets/pdfs/fast21-dong.pdf
[wisckey]: wisckey.md
[ldc]: /assets/pdfs/lower-level-driven-compaction.pdf
[pcp]: /assets/pdfs/pcp_pipelined_compaction_for_lsm_tree.pdf
[dostoevskykv]: https://scholar.harvard.edu/files/stratos/files/dostoevskykv.pdf
[accordion]: https://dl.acm.org/doi/abs/10.14778/3229863.3229873
[fast20-cao_zhichao]: https://www.usenix.org/system/files/fast20-cao_zhichao.pdf
