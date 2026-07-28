**Traditional LLM**
```
Question
   ↓
LLM
   ↓
Answer
```
**Agent**
```
Goal
 ↓
Reason
 ↓
Use Tools
 ↓
Observe Results
 ↓
Reason Again
 ↓
Repeat Until Goal Achieved
```


1. All are ReAct Agents: To reduce Halluncinations
```
Thought
 ↓
Action
 ↓
Observation
 ↓
Thought
 ↓
Action
 ↓
Observation
 ↓
Final Answer
```
2. Agents become useful because of tools.
```
search_web()
query_database()
send_email()
read_pdf()
run_sql()
create_jira_ticket()
```
3. Memory Store short-term memory like conversation history. We were storing past 3 conversations in Redis for conversation history retrieval in slingshot and retrieving them while we were ingesting data in Rag and keeping conversation of past 10 chats with slingshot.
4. Planning Agents - Task Decomposition: Agent breaks tasks into subtasks
5. Reflection agent:

```
Generate answer 
	  ↓
Critique answer 
	  ↓
Improve answer
```
This can be sorted through prompts like giving specific instructions to it for specific tasks.
6. Multi-agents: Use specialised agents with specialised features for doing a particular task.
7. Supervisor Architecture: Delegate Tasks.
8. Agents + MCP (User -> Agent -> MCP Client -> MCP Servers)
9. Agents + RAG (User -> Agent -> RAG ->  Knowledge -> Tool Calls -> Answer)