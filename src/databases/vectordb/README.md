## vector db

> Consider a query q, and suppose the algorithm outputs a set X of k candidate near neighbors, and suppose G is
> the ground-truth set of the k closest neighbors to q from among the points of the base dataset. Then, we define the k-recall@k
> of this set X to be **|X∩G| / k**.

> The goal of an ANN algorithm then is to **maximize recall** while retrieving the results as quickly as possible, which results in the **recall-vs-latency tradeoff**.

### Vector databases

- [Milvus: A Purpose-Built Vector Data Management System](/assets/pdfs/SIGMOD21_Milvus.pdf), SIGMOD '21
- [Manu: A Cloud Native Vector Database Management System](/assets/pdfs/manu_2206.13843.pdf), 2022
- [VBASE: Unifying Online Vector Similarity Search and Relational Queries via Relaxed Monotonicity](/assets/pdfs/vbase-osdi23.pdf), OSDI 2023 [codebase](https://github.com/microsoft/MSVBASE)

### Graph Index

- [Approximate nearest neighbor algorithm based on navigable small world graphs](/assets/pdfs/Approximatenearest_neighbor_algorithm_based_on_navigable_small_world_graphs.pdf), 2014
- [Efficient and robust approximate nearest neighbor search using Hierarchical Navigable Small World graphs][hnsw], 2016
- [Revisiting the Inverted Indices for Billion-Scale Approximate Nearest Neighbors][ivf-hnsw], 2018
- [FINGER: Fast Inference for Graph-based Approximate Nearest Neighbor Search][hnsw-finger], 2022 [HNSW-FINGER Explained!](https://www.youtube.com/watch?v=OsxZG2XfcZA)

### Quantization

- [Product Quantization for Nearest Neighbor Search][pq], 2010
- [A Survey of Quantization Methods for Efficient Neural Network Inference](/assets/pdfs/A_Survey_of_Quantization_Methods_for_Efficient_Neural_Network_Inference.pdf), 2021

### Misc

- [THE FAISS LIBRARY](/assets/pdfs/The_FAISS_LIBRARY_2401.08281.pdf), 2024
- [Faiss: The Missing Manual](https://www.pinecone.io/learn/series/faiss/)


[hnsw]: hnsw.md
[pq]: pq.md
[ivf-hnsw]: ivf-hnsw.md
[hnsw-finger]: https://arxiv.org/abs/2206.11408
