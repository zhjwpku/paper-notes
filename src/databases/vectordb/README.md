## vector db

> Consider a query q, and suppose the algorithm outputs a set X of k candidate near neighbors, and suppose G is
> the ground-truth set of the k closest neighbors to q from among the points of the base dataset. Then, we define the k-recall@k
> of this set X to be **|X∩G| / k**.

> The goal of an ANN algorithm then is to **maximize recall** while retrieving the results as quickly as possible, which results in the **recall-vs-latency tradeoff**.

[Awesome Vector Database](https://github.com/dangkhoasdc/awesome-vector-database) [![Awesome](https://cdn.jsdelivr.net/gh/sindresorhus/awesome@d7305f38d29fed78fa85652e3a63e154dd8e8829/media/badge.svg)](https://github.com/sindresorhus/awesome)

A curated list of awesome works related to high dimensional structure/vector search &amp; database 

### Vector databases

- [Milvus: A Purpose-Built Vector Data Management System](/assets/pdfs/SIGMOD21_Milvus.pdf), SIGMOD '21
- [Manu: A Cloud Native Vector Database Management System](/assets/pdfs/manu_2206.13843.pdf), 2022
- [VBASE: Unifying Online Vector Similarity Search and Relational Queries via Relaxed Monotonicity](/assets/pdfs/vbase-osdi23.pdf), OSDI 2023 [codebase](https://github.com/microsoft/MSVBASE)

### Graph Index

- [Approximate nearest neighbor algorithm based on navigable small world graphs](/assets/pdfs/Approximatenearest_neighbor_algorithm_based_on_navigable_small_world_graphs.pdf), 2014
- [Efficient and robust approximate nearest neighbor search using Hierarchical Navigable Small World graphs][hnsw], 2016
- [Revisiting the Inverted Indices for Billion-Scale Approximate Nearest Neighbors][ivf-hnsw], 2018
- [FINGER: Fast Inference for Graph-based Approximate Nearest Neighbor Search][hnsw-finger], 2022 [HNSW-FINGER Explained!](https://www.youtube.com/watch?v=OsxZG2XfcZA)
- [DiskANN: Fast Accurate Billion-point Nearest Neighbor Search on a Single Node][diskann], 2019

### Quantization

- [Product Quantization for Nearest Neighbor Search][pq], 2010
- [A Survey of Quantization Methods for Efficient Neural Network Inference](/assets/pdfs/A_Survey_of_Quantization_Methods_for_Efficient_Neural_Network_Inference.pdf), 2021

### Misc

- [THE FAISS LIBRARY](/assets/pdfs/The_FAISS_LIBRARY_2401.08281.pdf), 2024
- [Faiss: The Missing Manual](https://www.pinecone.io/learn/series/faiss/)
- [Cosine similarity = dot product for normalized vectors](https://zhang-yang.medium.com/cosine-similarity-dot-product-for-normalized-vectors-c07bdb61c9d1)
- [USearch: Smaller and faster single-file vector search engine](https://news.ycombinator.com/item?id=36942993)
- [Annoy(Approximate Nearest Neighbors Oh Yeah)](https://github.com/spotify/annoy)
- [RAPIDS RAFT library](https://github.com/rapidsai/raft)
- [Accelerating Vector Search: Using GPU-Powered Indexes with RAPIDS RAFT](https://developer.nvidia.com/blog/accelerating-vector-search-using-gpu-powered-indexes-with-rapids-raft/)
- [Accelerating Vector Search: Fine-Tuning GPU Index Algorithms](https://developer.nvidia.com/blog/accelerating-vector-search-fine-tuning-gpu-index-algorithms/)
- [Accelerated Vector Search: Approximating with RAPIDS RAFT IVF-Flat](https://developer.nvidia.com/blog/accelerated-vector-search-approximating-with-rapids-raft-ivf-flat/)
- [Semantic Search: Measuring Meaning From Jaccard to Bert](https://www.pinecone.io/learn/semantic-search/)
- [SPLADE for Sparse Vector Search Explain](https://www.pinecone.io/learn/splade/)
- [Sparse embedding or BM25?](https://medium.com/@infiniflowai/sparse-embedding-or-bm25-84c942b3eda7)
- [Unlock Advanced Recommendation Engines with Milvus' New Range Search](https://zilliz.com/blog/unlock-advanced-recommendation-engines-with-milvus-new-range-search)
- [Billion-scale vector search with Vespa - part one](https://blog.vespa.ai/billion-scale-knn/)
- [Billion-scale vector search with Vespa - part two](https://blog.vespa.ai/billion-scale-knn-part-two/)
- [Billion-scale vector search using hybrid HNSW-IF](https://blog.vespa.ai/vespa-hybrid-billion-scale-vector-search/)
- [Weaviate: An Architectural Deep Dive (Etienne Dilocker)](https://www.youtube.com/watch?v=4sLJapXEPd4&list=PLSE8ODhjZXjYVdJKka5g3xTKfPBITrxOu) with HNSW and PQ detailed
- [Milvus: A Purpose-Built Vector Data Management System! (Li Liu)](https://www.youtube.com/watch?v=kIj-KKnC-PA&list=PLSE8ODhjZXjYVdJKka5g3xTKfPBITrxOu) Milvus 2.0 architecture explained

[hnsw]: hnsw.md
[pq]: pq.md
[ivf-hnsw]: ivf-hnsw.md
[hnsw-finger]: https://arxiv.org/abs/2206.11408
[diskann]: diskann.md
