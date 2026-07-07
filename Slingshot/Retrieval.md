## End to End flow
![[Pasted image 20260707174606.png]]


### Step 1: Parse request context
It reads rag_context from request:
- Validates the user if he is authorised for the requested projects/resources.
- Splits context by resource type: code, text, jira, confluence, uploaded documents, user-added context, rag_metadata.
- Entry has source, collection_name, context, filters, embedding_model_name and is_bm25_search_enabled etc.

### Step 2: Build retrievers
`create_ensemble_retriever()` loops over `context_details` and creates a retriever per source.
```
retriever = vectordb.get_retriever(
    account_name=retriever_details["collection_name"],
    authorization_key=key,
    filter_expression=expression,
    enable_bm25=retriever_details.get("is_bm25_search_enabled"),
    embedding_model_name=retriever_details.get("embedding_model_name"),
    k=get_config().getint("retriever_params", "k"),
    fetch_k=RetrieverFactory.get_config().getint("retriever_params", "mmr_fetch_k"),
    lambda_mult=RetrieverFactory.get_config().getfloat("retriever_params", "mmr_lambda_mult"),
    ...
)
```
#### RAG metadata (code-specific)
Two-phase retrieval for code:
1. Metadata filtering — LLM or BM25 narrows to relevant class/function/file names.
2. Vector search with a tighter Milvus expression on those names.


### Step 3: Vector search (the actual DB lookup)
1. Embedding the query:- The same embedding model to embed the query via enterprise embedding service. We will do query transformation and Hyde which is basically give the basic answer to the question and then retrieve the documents based on the expected_answer.
2. Search mode:
	1. k=50
	2. mmr_fetch_k=100
	3. search_type=similarity
	4. metric_type=L2
	5. enable_bm25= true
   Dense (semantic embeddings) +Sparse BM25 run together. Results are fused with the weights: semantic_weights=0.3, keyword_weights=0.7
3. Code Enrichment
   results can be enriched with imports and global statements via CustomImportsRetriever.

### Step 4: Query rewriting before search
1. Paraphrase + metadata extraction(process_user_query).
   LLM rewrite the user question into a standalone query and extracts the metadata for filtering. This uses structured output based on metadata_type.
2. History-aware retriever (create_history_aware_retriever)
    another LLM call rewrites the question into a search friendly query using conversation context.
3. Context processing: It searches separate context-processing collection for similar past Q&A from the same session, and inject that as additional context.

### Step 5: Ensemble Fusion (EnsembleRetriever)
When multiple sources are active, Code, jira, uploaded docs, each retriever returns its own top-k chunks. EnsembleRetriever combines them using Weighted Reciprocal Rank Fusion(RRF)
Version v1.1:
- Metadata retriever gets 50% budget
- Remaining split across code, jira, confluence etc.
- All candidates go through weighted RRF to produce a final ranked list.
- Capped at overall_retrieved_k =75
#### Optional: LLM chunk relevancy

When graph-based context expansion is enabled, an LLM picks the top 4 most relevant chunks from semantic/metadata results before graph expansion (`get_top_relevant_document_indices`).

#### Optional: Knowledge graph expansion

If `use_graph` is enabled in the request, retrieved code chunks can be expanded via Neo4j to pull related entities.

### Step 6: Feed chunks to LLM
After retrieval, the LangChain chain runs:
```
1. history_aware_retriever = create_history_aware_retriever(...)
2. question_answer_chain = create_stuff_documents_chain(llm_model, prompt=prompt, ...)
3. qa_chain = create_retrieval_chain(history_aware_retriever, question_answer_chain)
```

1. History-aware retriever runs the (possibly rewritten) query → returns fused `Document` list
2. Stuff documents chain concatenates all retrieved chunks into the prompt’s `{context}` variable
3. LLM generates the answer using that context + chat history + system prompt



### Other information

>`POST /get-context` (via `retrieve_context_data()`) runs the same ensemble retrieval without calling the LLM for an answer — it just returns the retrieved chunks. Useful for debugging or UI preview.

