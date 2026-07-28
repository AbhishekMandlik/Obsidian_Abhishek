**Query Transformation** rewrites or expands the query before retrieval to improve search quality.
1. Query Rewriting: Rewrite the user's question into a more search-friendly form.
2. Query Expansion: Add related terms and synonyms
   Useful for keyword and BM25 retrieval
3. Multi-Query Retrieval: Generate multiple versions of the same question.
4. HyDE (Hypothetical Document Embeddings)
	1. Step 1: LLM generates a hypothetical answer.
	   ``` Question:What causes battery degradation?```
	   Generated document:```Battery degradation occurs due to charge cycles,high temperatures, and aging.```
	2. Step 2: Embed the generated document.
	3. Step 3: Search using that embedding.
	   Reason:```Question embedding vs Answer-like embedding```
	   Answer-like embeddings often retrieve.
	4. Step - Back Prompting: Convert a specific question into a broader one.
5. Decomposition: Break a complex query into smaller queries.
6. Conversational Query Reformulation: Adding context to prompts so that agents can infer from past data.
7. Query Transformation Pipeline
   ```
		   User Query
		      ↓
		Query Transformation
		      ↓
			Retriever
		      ↓
			Documents
		      ↓
			 LLM
		      ↓
			Answer
   ```
