Two types:
1. Dense : Semantic, Vector Search.
	1. Convert text into vectors. and has advantages like
		1. Semantic Understanding.
		2. Synonyms
		3. Better for natural language.
2. Sparse: Use Exact words.
	1. BM25 (Term Frequency, Document Length, Inverse Document frequency)
	2. TF-IDF
Dense Retrieval Pipeline:
```
Document->Chunk->Embedding Model-> Vector-> Vector Database.....
USER
Query->Embedding->Vector Search-> Nearest Neighbours-> Top-k
```
### Similarity Metrics
1. Cosine Similarity: Measures angles between vectors, not magnitude.
2. Euclidean Distance: Straight line distance.
3. Dot Product: Higher value -> More relevant.
4. k-NN: Retriever finds the nearest neighbours and return top k neighbours.
5. A-NN approximate: Comparing against every vector is vey slow if the database is too big. Popular algorithms are:
	1. HNSW:
	2. IVF:
	3. PQ:
	4. ScaNN:
   ANN trades small accuraccy for a huge speed improvement.
	- HNSW:
	  Instead of checking every vector build a graph. Search navigates the graph towards the closest vectors instead of scanning everything.
	  Advantages:
		- Very fast.
		- High recall.
		- Used by many vector databases.
	- Metadata filtering:
	  Where year = 2025 etc;
	- Hybrid Retrieval:
	  One of the most common production approaches:
	  Combine BM25 + vector search. Both the searches run in parallel. This balances exact keyword matching with semantic understanding.
	- Multi-Query Retrieval:
	  Rewrite query
	- Self-Query Retriever:
	  LLM interprets the user's request and generates both:
		- Semantic query
		- metadata filters
		Useful only when metadata is rich and users don't specify filters explicitly.
	- Parent-Child retriever:
	  Suppose we split all the documents hierarchically:
	  parent given if we want more context and child only if we want accuracy.
	- Contextual Compression Retriever:
	  Sometime the retrieved chunk is much larger than necessary. 
	  Compression step trims irrelevant content before passing it to the LLM, reducing token usage and improving focus.
	- Ensemble Retriever:
	  Merge and re-rank: dense, sparse, knowledge graph.
	- Reranking (Cross Encoder)
	  ### Stage 1: Fast retrieval
	  Return the top 20–50 candidate chunks using BM25 or vector search.
	  ### Stage 2: Re-ranking
	  A more expensive model evaluates each **query–document pair** and assigns a more accurate relevance score.
	- Choosing k (Top - k):
	  or we can do top - p as well.
	  
	  


# Putting it all together

A modern production RAG pipeline often looks like this:
```
                  User Query
                       │
                       ▼
              Query Expansion (optional)
                       │
                       ▼
             Metadata Filter Generation
                       │
                       ▼
        Hybrid Retrieval (BM25 + Vector Search)
                       │
                       ▼
          Retrieve Top 20–50 Candidate Chunks
                       │
                       ▼
         Cross-Encoder Re-ranking
                       │
                       ▼
         Keep Best 5–10 Chunks
                       │
                       ▼
     Contextual Compression (optional)
                       │
                       ▼
              Prompt Construction
                       │
                       ▼
                     LLM

```

### Interview perspective
If you're asked, **"How would you design retrieval for an enterprise RAG system?"**, a strong answer is:
> "I'd start with a hybrid retriever combining dense vector search and BM25 because they complement each other—dense retrieval captures semantic similarity, while BM25 handles exact keywords and identifiers. I'd apply metadata filtering where available, retrieve around 20–50 candidates, then use a cross-encoder reranker to select the best 5–10 chunks. For large documents, I'd use parent–child retrieval and, if needed, contextual compression before sending the final context to the LLM. This design balances recall, precision, latency, and token efficiency."