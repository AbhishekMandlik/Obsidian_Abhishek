Subgraphs are useful for:

- Building [multi-agent systems](https://docs.langchain.com/oss/python/langchain/multi-agent)
- Reusing a set of nodes in multiple graphs
- Distributing development: when you want different teams to work on different parts of the graph independently, you can define each part as a subgraph, and as long as the subgraph interface (the input and output schemas) is respected, the parent graph can be built without knowing any details of the subgraph

|Pattern|When to use|State schemas|
|---|---|---|
|[Call a subgraph inside a node](https://docs.langchain.com/oss/python/langgraph/use-subgraphs#call-a-subgraph-inside-a-node)|Parent and subgraph have **different state schemas** (no shared keys), or you need to transform state between them|You write a wrapper function that maps parent state to subgraph input and subgraph output back to parent state|
|[Add a subgraph as a node](https://docs.langchain.com/oss/python/langgraph/use-subgraphs#add-a-subgraph-as-a-node)|Parent and subgraph **share state keys**—the subgraph reads from and writes to the same channels as the parent|You pass the compiled subgraph directly to `add_node`—no wrapper function needed|

### Call a subgraph inside a node
When a parent graph and a subgraph have different state schemas (no shared keys),
invoke the subgraph inside a node function. This is common when you want to keep a private message history for each agent in a multi-agent system.

### Add a subgraph as a node
When the parent graph and subgraph share state keys, we can pass a complied subgraph directly to add_node. No wrapper function is needed -- The subgraph directly reads and writes into parent's state channels automatically. In a multi-agent system the agent often communicats over a shared messages key.![SQL agent graph](https://mintcdn.com/langchain-5e9cc07a/ybiAaBfoBvFquMDz/oss/images/subgraph.png?fit=max&auto=format&n=ybiAaBfoBvFquMDz&q=85&s=c280df5c968cd4237b0b5d03823d8946)

- Define the subgraph workflow and compile it.
- Pass the compiled subgraph to the add_node method when defining the parent graph workflow.

Subgraph persistence:
When you use a subgraph, you need to decide what happens to its internal data between calls.



This is connected to [[Checkpointers]]