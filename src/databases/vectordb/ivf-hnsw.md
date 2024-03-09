### [Revisiting the Inverted Indices for Billion-Scale Approximate Nearest Neighbors](/assets/pdfs/hnsw_1603.09320.pdf)

> https://arxiv.org/abs/1802.02422

IVF 在较大码本（codebook, K > 2^17）的情况下具有更好的搜索效果，但查找相近 Centroid 会相对比较费时。

> The only practical reason, preventing their usage, is the expensive procedure of query assignment that takes O(K) operations.

使用 HNSW 对 Centroid set 构建一个索引能够获得不错的效果（之前有研究用 kd-tree 来查找最近 centroids 效果并不理想）。

> HNSW allows obtaining a small top of the closest centroids with almost perfect accuracy in a submillisecond time.

<p align="center">
<img src="/assets/images/ivf-hnsw.png" alt="ivf-hnsw" width="600"/>
</p>

文中提到的 `Grouping and pruning` 能够对子分区更好地划分以提高查询精度，可用于大数据的数据压缩。

#### Open Source Implementations

- [ivf-hnsw](https://github.com/dbaranchuk/ivf-hnsw)

#### References:

- [Facebook AI and the Index Factory](https://www.pinecone.io/learn/series/faiss/composite-indexes/)
