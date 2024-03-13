### [Efficient and robust approximate nearest neighbor search using Hierarchical Navigable Small World graphs](/assets/pdfs/hnsw_1603.09320.pdf)

> https://arxiv.org/abs/1603.09320

理解 HNSW 的关键在于理解 **Skip List** 和 **NSW**。NSW 具有小世界网络和可导航的特点，即 short-range link 聚类，long-range link 快速检索，HNSW 利用跳表的思想实现了 long-range link 快速检索的能力。

>  Hierarchical NSW algorithm can be seen as an extension of the probabilistic skip list structure with proximity graphs instead of the linked lists.

算法本身论文描述的非常清晰，了解以下三个参数对理解和使用 HNSW 都非常有帮助:

- M — the number of nearest neighbors that each vertex will connect to. A reasonable range of M is from 5 to 48.
- efSearch — how many entry points will be explored between layers during the search.
- efConstruction — how many entry points will be explored when building the index.

#### Open Source Implementations

- [hnswlib](https://github.com/nmslib/hnswlib) Header-only C++/python library for fast approximate nearest neighbors
- [IndexHNSW from faiss](https://github.com/facebookresearch/faiss/blob/main/faiss/IndexHNSW.cpp)
- [pgvector](https://github.com/pgvector/pgvector)

hnswlib 的内存 layout 如下，可以帮助理解 hnswlib 的代码:

![hnswlib layout](/assets/images/hnswlib_layout.png)

#### References:

- [Graph-Based Approximate Nearest Neighbors (ANN) and HNSW](https://www.youtube.com/watch?v=4PsyNdFlxmk)
- [Hierarchical Navigable Small Worlds (HNSW)](https://www.pinecone.io/learn/series/faiss/hnsw/)
- [HNSW算法原理](https://zhuanlan.zhihu.com/p/441470968)
