When a node fails—from a slow external API, a transient network error, or an unhandled exception—LangGraph gives you three composable mechanisms to respond:

- [**Retries**](https://docs.langchain.com/oss/python/langgraph/fault-tolerance#retries) — automatically re-run failed attempts based on exception type and backoff settings
- [**Timeouts**](https://docs.langchain.com/oss/python/langgraph/fault-tolerance#timeouts) — cap how long a single attempt may run
- [**Error handling**](https://docs.langchain.com/oss/python/langgraph/fault-tolerance#error-handling) — run a recovery function after all retries are exhausted
