1. UV vs Pip?
   UV is a modern Python package manager written in Rust. It provides significantly faster dependency resolution and installation compared to pip, supports lock files, virtual environment management, and reproducible builds. Pip remains the standard package installer, but UV is increasingly adopted for production and CI/CD workflows because of its speed and dependency management capabilities.
2. Transport Layer of MCP
   MCP transport layers commonly include STDIO for local execution, HTTP for request-response communication, and WebSockets for persistent bidirectional communication.
3. MCP vs A2A protocol?
   MCP standardises communication between LLMs and tools, whereas A2A protocols standardise communication between multiple autonomous agents. MCP solves tool interoperability, while A2A solves agent collaboration.
4. Chain of Thought vs Chain of Density?
   Chain of Thought improves reasoning by generating intermediate steps, while Chain of Density improves summarisation by progressively increasing information density within a fixed-length summary.
5. Spec-Driven Development.
   Spec-driven development starts with formal specifications such as API contracts, schemas, or requirements documents before implementation. Development and testing are driven directly from the specification.
6. I would build a RAG system that chunks and indexes the file. A planner agent would decompose the multi-question prompt into subqueries, perform retrieval for each subquery, and then synthesise a consolidated response from all retrieved evidence.
7. Can Rag be implemented without VectorDB?
   RAG can be implemented without vector databases using lexical retrieval methods such as BM25, Elasticsearch, SQL-based filtering, document hierarchies, or knowledge graphs. Vector databases are one retrieval mechanism, not a requirement for RAG.
8. Design a Multi query RAG pipeline.
   I would use an agentic RAG architecture where a planner decomposes the query into multiple subqueries, retrieves evidence for each subquery independently, reranks the results, and synthesizes a final answer using the LLM.
9. 
   