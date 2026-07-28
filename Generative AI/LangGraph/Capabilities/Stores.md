Stores let agents persist information across threads, including user preferences, accumulated knowledge and facts that should survive beyond a single conversation. It holds arbitrary key-value data accessible.![Model of shared state](https://mintcdn.com/langchain-5e9cc07a/dL5Sn6Cmy9pwtY0V/oss/images/shared_state.png?fit=max&auto=format&n=dL5Sn6Cmy9pwtY0V&q=85&s=354526fb48c5eb11b4b2684a2df40d6c)


Memories are namespaced by a tuple (<user_id>,"memories")
Use the store.put method to save memories to the namespace in the store.

use store.put method to save memories to the namespace in the store like below:
```
memory_id=str(uuid.uuid4())
memory ={"food_preference": "I like pizza"}
store.put(namespace_for_memory,memory_id,memory)
```
Read out memories from your namespace using the store.search method.
upto certain limit argument ( default = 10 );
InMemoryStore, items are returned in insertion order, so the most recent memory is last in the list; diff for diff backend DB's.
```
memories = store.search(namespace_for_memory)
memories[-1].dict()
{'value': {'food_preference': 'I like pizza'},
 'key': '07e0caf4-1631-47b7-b15f-65515d4c1843',
 'namespace': ['1', 'memories'],
 'created_at': '2024-10-02T17:22:31.590602+00:00',
 'updated_at': '2024-10-02T17:22:31.590605+00:00'}
```

Each memory type is a Python class (Item)  with the above attributes.


### Listing items in a namespace
store.earch or async store.search return the items stored under namespace_prefix up to limit.
Three behaviors to keep in mind:

- **`namespace_prefix` matches by prefix, not exactly.** `("alice",)` also returns items under `("alice", "memories")`, `("alice", "preferences")`, and so on. To restrict to a single level, pass the full namespace or filter the returned items client-side on `item.namespace`.
- **Results past `limit` are silently truncated.** There is no overflow signal—set `limit` above your expected maximum, or paginate with `offset`.
- **Default ordering depends on the store backend.** `PostgresStore` and `AsyncPostgresStore` return results ordered by `updated_at` descending (most recently updated first). `InMemoryStore` returns results in insertion order (most recently inserted last). Do not rely on a specific order across implementations; sort client-side on `item.updated_at` if order matters.


### Semantic Search
The store also supports semantic search, allowing you to find memories based on meaning rather than exact matches. To enable this configure the store with an embedding model:
```
from langchain.embeddings import init_embeddings

store = InMemoryStore(
    index={
        "embed": init_embeddings("openai:text-embedding-3-small"),  # Embedding provider
        "dims": 1536,                              # Embedding dimensions
        "fields": ["food_preference", "$"]              # Fields to embed
    }
)
```
And while searching we can use natural language queries to find relevant memories:
```
# Find memories about food preferences
# (This can be done after putting memories into the store)
memories = store.search(
    namespace_for_memory,
    query="What does the user like to eat?",
    limit=3  # Return top 3 matches
)
```

You can access the store and the `user_id` from _any node_ by using the `Runtime` object. The `Runtime` is automatically injected by LangGraph when you add it as a parameter to your node function.

We can access the store from any node and use the store.search method to get memories. Memories are returned as a list of objects that can be converted to a dictionary.
```
# Search the memory based on some info:
memories = await runtime.store.asearch( 
	namespace, 
	query=state["messages"][-1].content,
	limit=3 
)
# Use the memories in model call.
info = "\n".join([d.value["memory"] for d in memories])
```

## Build a custom store
### Base contract
All five async methods are required. Sync counterparts (`put`, `get`, `delete`, `search`, `list_namespaces`) are optional but recommended for compatibility with sync graph execution.

| Method                                                                               | Description                                                         |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------- |
| `aput(namespace, key, value, index=None)`                                            | Store or overwrite a single item                                    |
| `aget(namespace, key)`                                                               | Retrieve a single item by key; return `None` if missing             |
| `adelete(namespace, key)`                                                            | Delete a single item                                                |
| `asearch(namespace_prefix, *, query=None, filter=None, limit=10, offset=0)`          | Search items under a namespace prefix; optionally by semantic query |
| `alist_namespaces(*, prefix=None, suffix=None, max_depth=None, limit=100, offset=0)` | List namespaces matching a prefix/suffix pattern                    |

#### Namespace design
Prefix matching and exact key lookup must be present and it must be done in O(1) or close time to it.
#### Semantic search support
If your backend supports vector search, implement the `query` parameter on `asearch`:
- Accept a `query: str | None` argument.
- When `query` is not `None`, embed it and rank results by cosine similarity.
- Results should include a `score` field on each `Item` when `query` is provided.
If your backend does not support vector search, raise `NotImplementedError` when `query` is passed.

