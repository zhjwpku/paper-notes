## Trie

前缀树或字典树，是一种有序树，其发明者 Edward Fredkin 把它读作/ˈtriː/ "tree"，但更多人把它读作/ˈtraɪ/ "try"。朴素的 Trie 由于存储稀疏，存在空间利用率低下的问题。Radix tree (又被称为 radix trie、压缩前缀树、compressed trie) 将作为唯一子节点的每个节点都与其父节点合并，是一种更节省空间的前缀树，[Rax](https://github.com/antirez/rax) 是一个 Radix tree 的 C 语言实现。

- **[The Adaptive Radix Tree: ARTful Indexing for Main-Memory Databases][art]**
- **[HOT: A Height Optimized Trie Index for Main-Memory Database Systems][hot]**

#### Optional readings

- V. Alvarez, et al., [A Comparison of Adaptive Radix Trees and Hash Tables](../../assets/pdfs/comparison-of-art-and-hash-tables-icde2015.pdf), in ICDE, 2015
- Viktor Leis, et al., [The ART of Practical Synchronization](../../assets/pdfs/the-art-of-practical-synchronization-damon2016.pdf), DaMoN '16. 介绍了 ART 实现中使用的两种多线程同步技术 — 1) OPTIMISTIC LOCK COUPLING 2) READ-OPTIMIZED WRITE EXCLUSION (ROWEX)

#### References

- [Trie](https://en.wikipedia.org/wiki/Trie)
- [Radix tree](https://en.wikipedia.org/wiki/Radix_tree)
- [Trees I: Radix trees](https://lwn.net/Articles/175432/) by J. Corbert, 2006

[art]: art.md
[hot]: hot.md
