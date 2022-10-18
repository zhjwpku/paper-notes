## B-Tree family

[B-tree](https://infolab.usc.edu/csci585/Spring2010/den_ar/indexing.pdf) 是由 Rudolf Bayer 和 Edward M. McCreight 在 1970 年发明的、适用于读取较大数据块（如 disk）的一种数据结构，作为索引（index）层广泛应用于数据库和文件系统。当插入或删除记录产生 underflow(due to deletion) 或 overflow(due to insertion) 时，B-tree 通过分裂或合并节点的方式保持树的平衡。在 [B-Tree Visualization](https://www.cs.usfca.edu/~galles/visualization/BTree.html) 可以直观感受一下 B-tree 的结构及算法。

- **[The Bw-Tree: A B-tree for New Hardware Platforms][bw-tree]**

#### Optional readings

- [Building a Bw-Tree Takes More Than Just BuzzWords](../../assets/pdfs/open_bwtree.pdf)
- [Cache Craftiness for Fast Multicore Key-Value Storage](../../assets/pdfs/masstree_eurosys12.pdf) a.k.a Masstree

[bw-tree]: bw-tree.md