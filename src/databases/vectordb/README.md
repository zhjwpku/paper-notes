## vector db

> Consider a query q, and suppose the algorithm outputs a set X of k candidate near neighbors, and suppose G is
> the ground-truth set of the k closest neighbors to q from among the points of the base dataset. Then, we define the k-recall@k
> of this set X to be **|X∩G| / k**.

> The goal of an ANN algorithm then is to **maximize recall** while retrieving the results as quickly as possible, which results in the **recall-vs-latency tradeoff**.

### Graph Index

- [Approximate nearest neighbor algorithm based on navigable small world graphs](/assets/pdfs/Approximatenearest_neighbor_algorithm_based_on_navigable_small_world_graphs.pdf), 2014
- [Efficient and robust approximate nearest neighbor search using Hierarchical Navigable Small World graphs][hnsw], 2016

### Quantization

- [A Survey of Quantization Methods for Efficient Neural Network Inference](/assets/pdfs/A_Survey_of_Quantization_Methods_for_Efficient_Neural_Network_Inference.pdf), 2021

### Misc

- [THE FAISS LIBRARY](/assets/pdfs/The_FAISS_LIBRARY_2401.08281.pdf), 2024
- [Faiss: The Missing Manual](https://www.pinecone.io/learn/series/faiss/)


[hnsw]: hnsw.md
