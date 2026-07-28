AI Agent Memory is the ability of an AI agent to store, recall and use information from past interactions to make better decisions in the present and future. Without memory, an agent treats every interaction as if it is the first interaction

1. Short-Term Memory:
   AI-agent's temporary notepad. It holds recent information. In slingshot we were storing context based memory in Milvus.
   Overlapping/similar context is collapsed using Jaccard/cosine/diff similarity
2. Long Term Memory: Across different sessions

| Memory Type       | What it Stores                       | Example                                                             |
| ----------------- | ------------------------------------ | ------------------------------------------------------------------- |
| Short Term Memory | Context of current session.          | Conversation in a session.                                          |
| Long Term Memory  | Knowledge/ Data over time.           | User Bday across different sessions.                                |
| Episodic Memory   | Specific events and experiences.     | How some task is done. Same thing can be applied to different task. |
| Semantic Memory   | Facts and world knowledge.           | Random Knowledge: Abhishek Mandlik is the Best.                     |
| Procedural Memory | Rule based data for immediate tasks. | Numbers held while solving a math problem.                          |
### Storage methods
**Knowledge Graph:** Complex to build and maintain. But it is good for reasoning and inference.
**Neural Turing Machine:** Integrates with deep learning models. Computationally intensive.
**VectorDB:** Handles fuzzy matching and is scalable. Requires embedding generation.
**RDBMS:** Structured long-term memory.
**Simple Buffer (FIFO)**:Short term context.