Types of Chunking
1. Fixed-size chunking:
   Simply split every N characters or tokens.
   When to use: It can be good for clean documents where you know the length of each chunks, These can be used:
	1. Logs
	2. OCR Texts
	3. Rag baseline
2. Fixed-size with overlap:
   Better retrieval but storage increases as now we might capture information that is present on the borderline in two chunks which will help in retrieval.
3. Sentence-based Chunking:
   Never cut a sentence, more natural, but has a disadvantage that sentence length can vary hence every chunk will be of different length.
4. Paragraph-based chunking:
   Each paragraph becomes a chunk
5. Recursive Chunking: It tries splitting with increasingly finer boundaries until chunk fits the desired size.
   Splits in hierarchy :
   ```Document->Section->Paragraph->Sentence->Word->Charachter```
6. Semantic Chunking:
   Use Embeddings to decide where topics change, Embed each sentence and then check where the meaning of the sentences changes.
7. Structure-aware chunking:
   Use the document's structure:
	1. Headings
	2. Subheadings
	3. Tables
	4. Lists
	5. Code blocks
8. Sliding Window chunking: Similar to overlapping chunks.
9. Hierarchical Chunking:
   Splitting document into multiple levels.
   ```
   Entire Chapter-> Parent Chunks:
   ->child1
   ->child2
   ```
   Return smaller chunks for precision and larger for context.
10. Document specific chunking:
	1. Source Code: split by class, function or method.
	2. Legal contracts: split by clause or section.