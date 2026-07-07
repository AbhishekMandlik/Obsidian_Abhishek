LangGraph supports time travel through [checkpoints](https://docs.langchain.com/oss/python/langgraph/checkpointers#checkpoints):

- **[Replay](https://docs.langchain.com/oss/python/langgraph/use-time-travel#replay)**: Retry from a prior checkpoint.
- **[Fork](https://docs.langchain.com/oss/python/langgraph/use-time-travel#fork)**: Branch from a prior checkpoint with modified state to explore an alternative path.
FORK
Call an update_state on a prior checkpoint to create the fork, then invoke with None to continue execution. New branch with modified state![Fork](https://mintcdn.com/langchain-5e9cc07a/-_xGPoyjhyiDWTPJ/oss/images/checkpoints_full_story.jpg?fit=max&auto=format&n=-_xGPoyjhyiDWTPJ&q=85&s=a52016b2c44b57bd395d6e1eac47aa36)


```
# Find checkpoint before write_joke
history = list(graph.get_state_history(config))
before_joke = next(s for s in history if s.next == ("write_joke",))

# Fork: update state to change the topic
fork_config = graph.update_state(
    before_joke.config,
    values={"topic": "chickens"},
)

# Resume from the fork — write_joke re-executes with the new topic
fork_result = graph.invoke(None, fork_config)
print(fork_result["joke"])  # A joke about chickens, not socks
```

> checkpoint records that the node is getting updated.
> as_node is inferred by default from the checkpoint's version history.

Specify as_node explicitly when:
- Parallel branches: if the langGraph cant determine the last updated state it will throw an Error.(InvalidUpdateError)
- No execution history: Setting up state on a fresh thread (for testing).
- Skipping nodes: We can set as_node to a later node ti make the graph think that the node has already ran.
```
fork_config = graph.update_state(
	# add one more header to this
	...,
	as_node = "generate_topic",
)
```

### Interrupts
Used for human-in the-loop workflows, these are re-triggered during time-travel. 
interrupt( pauses for a new Command(resume="")).

### For Subgraphs
By default the subgraphs inherits the parents checkpointers. Parents trears the entire subgraph as a single super-step.
>If in subgraphs we keep Checkpointers=True then we can have checkpointers for each and every subgraph for each of it's node which is beneficial for us.





This is linked to [[Time Travel]], [[Interrupts]] , [[Checkpointers]] and [[Subgraphs]].





