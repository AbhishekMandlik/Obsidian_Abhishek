
```
Good prompt Contains
Role
Task
Context
Output Format
```


**Types of Prompting**
1. Zero-Shot Prompting: No examples Provided, model does it based on its own knowledge.
2. One -shot Prompting: One Example provided to teach pattern.
3. Few-Shot Prompting: Provide Multiple Examples. (In- Context Learning)
4. Chain of Thought(CoT): Ask it to do the work step by step just like using plan and track method.
5. Self- Consistency: Instead of generating one reasoning path, generate many and then do majority vote.
6. ReAct (Used in Langchain) Thought, Action, Observation loop and then the Final Answer. Repeat the TAO loop untill you get the output.
7. Tree of Thoughts: Instead of one reasoning Question -> Reasoning Path -> Answer. Evaluate branches and continue with the promising ones. 
	1. ```Question

			 ├─ Path A
			 │   └─ Answer
			 │
			 ├─ Path B
			 │   └─ Answer
			 │
			 └─ Path C
			     └─ Answer
	   ```
8. Role Prompting: Think like a data acientist
9. Structured Output prompting.
10. Prompt Chaining: Instead of direct prompt tell it steps as well it should follow to get to the final output.
11. RAG Prompting:
	1. System Prompt
	2. Retrieved Context
	3. User Query
	4. Output format
12. Hallucination Reduction Prompt
13. System>User>Assistant
	1. System
	2. User
	3. Assistant is the one with some other things like previous context etc.
   


This reminds me of [[Transformers]], [[Production Rag]], [[MCP]]


## Types of Prompt Templates

Various types of Prompt Templates are:

1. ****StringPromptTemplate:**** Simple text based templates with placeholders for variables.
2. ****ChatPromptTemplate:**** Designed for chat style interactions, supporting multiple roles and messages.
3. ****FewShotPromptTemplate:**** Includes example inputs and outputs to guide the LLM’s behavior for better accuracy.