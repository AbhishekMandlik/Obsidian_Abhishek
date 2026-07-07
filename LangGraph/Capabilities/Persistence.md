Keep useful information beyond a single graph run. When agent needs to continue a conversation, resume after an interruption, recover from a failure, or remember information across interactions.
Two complementary persistence systems:
1. Checkpointers : tracks current thread.
2. Stores: tracks information across threads.

```
from langgraph.checkpoint.memory import InMemorySaver
from langgraph.store.memory import InMemoryStore
checkpointer= InMemorySaver()
store= InMemoryStore()
graph = builder.compile(checkpointer=checkpointer,store=store)
result= graph.invoke(
	{messages:[{role:user, content:"Hi, my name is Abhishek"}]},
	{configurable: {thread_id: thread_1}}
)
```


![[Pasted image 20260630181530.png]]

## Troubleshooting common issues

### PostgresSaver: thread_id too long
Fix: thread_id<255 characters. Use UUID or hash if you need deterministic IDs.

### `MemorySaver` does not persist between restarts

`MemorySaver` and `InMemorySaver` store checkpoints in RAM. When the process restarts, all checkpoints are lost.**Fix:** Use a persistent checkpointer for production:

- `PostgresSaver`: PostgreSQL with async support
- `SqliteSaver`: Local file-based storage for development

###  Checkpoints growing unboundedly
Over long conversation checkpoints accumulate which increases latency and storage cost.
Prune old checkpoints;
```
from langgraph.checkpoint.postgres import PostgresSaver

checkpointer = PostgresSaver.from_conn_string("postgresql://...")
checkpointer.setup()  # Creates tables with indexes
# Consider adding a cron job to delete checkpoints older than N days
```

### State access from parent graph to subgraph

When a subgraph updates state, the parent graph may not see the changes immediately. This is because each subgraph manages its own checkpoint namespace.**Fix:** Use [shared state via Store](https://docs.langchain.com/oss/python/langgraph/stores) for data that needs to cross graph boundaries, or configure your subgraph to write to the parent checkpoint.