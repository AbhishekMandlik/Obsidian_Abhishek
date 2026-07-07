It saves snapshot of graph state at each super-step, organised into threads. Compile it with a graph to enable Human-In-The-Loop, time travel debugging, fault-tolerant execution and conversational memory.

Each sequential node is a separate super-step, parallel have same super-step. State and relevant metadata packaged at every super-step.
Thread=collection of checkpoints.

It is used for the following features:
- **Human-in-the-loop**: Checkpointers facilitate [human-in-the-loop workflows](https://docs.langchain.com/oss/python/langgraph/interrupts) by allowing humans to inspect, interrupt, and approve graph steps. Checkpointers are needed for these workflows as the person has to be able to view the state of a graph at any point in time, and the graph has to be able to resume execution after the person has made any updates to the state. See [Interrupts](https://docs.langchain.com/oss/python/langgraph/interrupts) for examples.
- **Memory**: Checkpointers allow for [“memory”](https://docs.langchain.com/oss/python/concepts/memory) between interactions. In the case of repeated human interactions (like conversations) any follow up messages can be sent to that thread, which will retain its memory of previous ones. See [Add memory](https://docs.langchain.com/oss/python/langgraph/add-memory) for information on how to add and manage conversation memory using checkpointers.
- **Time travel**: Checkpointers allow for [“time travel”](https://docs.langchain.com/oss/python/langgraph/use-time-travel), allowing users to replay prior graph executions to review and / or debug specific graph steps. In addition, checkpointers make it possible to fork the graph state at arbitrary checkpoints to explore alternative trajectories.
- **Fault-tolerance**: Checkpointing provides fault-tolerance and error recovery: if one or more nodes fail at a given superstep, you can restart your graph from the last successful step.
- **Pending writes**: When a graph node fails mid-execution at a given [super-step](https://docs.langchain.com/oss/python/langgraph/checkpointers#super-steps), LangGraph stores pending checkpoint writes from any other nodes that completed successfully at that super-step. When you resume graph execution from that super-step you don’t re-run the successful nodes.


## Core Concepts
### Threads
It is a unique ID or a thread identifier assigned to each checkpoint, saved by a checkpointer. It contains the accumulated state. State->Thread.
While invoking the graph:
```
{"configurable":{"thread_id":"1"}}
```
Checkpointer requires #thread_id as primary key for storing and retrieving checkpoints. 
Threads current and historic state can be retrieved. Thread must be created before execution. LangSmith provides several endpoints for managing threads.
### Checkpoints
it is a snapshot of graph saved at each super-step. Represented by a StateSnapshot.

### Super-steps
As each node in super-step finishes it's outputs are written to the checkpointers #checkpoint_writes table as task linked to the main thing.

Can be used to restore the state of a thread at a later time.

### Checkpoint namespace
#checkpoint_ns field that identifies which graph or subgraph it belongs to:
- **`""`** (empty string): The checkpoint belongs to the parent (root) graph.
- **`"node_name:uuid"`**: The checkpoint belongs to a subgraph invoked as the given node. For nested subgraphs, namespaces are joined with `|` separators (e.g., `"outer_node:uuid|inner_node:uuid"`).

## Get and Update state
### Get
You can view the _latest_ state of the graph by calling `graph.get_state(config)`. This will return a `StateSnapshot` object that corresponds to the latest checkpoint associated with the thread ID provided in the config or a checkpoint associated with a checkpoint ID for the thread, if provided.
```
# get_state will look like:
StateSnapshot(
	valuvalues={},
	next=(),
	config={'configurable':{thread_id:, checkpoint_ns:, checkpoint_id:}}
	metadata={source;, writes{node_b:, 'step':last_step}}
	created_at=
	parent_config={configurable},tasks()
)
```
![[Pasted image 20260701114508.png]]

to get full state history call function **"graph.get_state_history(config)"**
in config #thread_id should be present
Filtering for a specific checkpoint
```
history = list(graph.get_state_history(config))

# Find the checkpoint before a specific node executed
before_node_b = next(s for s in history if s.next == ("node_b",))

# Find a checkpoint by step number
step_2 = next(s for s in history if s.metadata["step"] == 2)

# Find checkpoints created by update_state
forks = [s for s in history if s.metadata["source"] == "update"]

# Find the checkpoint where an interrupt occurred
interrupted = next(
    s for s in history
    if s.tasks and any(t.interrupts for t in s.tasks)
)
```
#### Replay
It re-executes steps from a prior chekpoint.Invoke it with a prior #checkpoint_id to re-run nodes after that check point.
### Update
We can edit the graph state using update_state. This creates a new checkpoint with the updated values -- It does not modify the original checkpoint. It is treated same as a node update: values are passed through reducer functions so the values keep updating rather than overwriting them.
We can specify #as_node to control which node the update is treated as coming from, which affects which node executes next.


### Durability modes
```
graph.stream(
	{"input": "test"},
	durability="sync"
)
```
least durable : #exit
- "exit": LangGraphs persists changes only when graph execution exits - with error or due to human-in-the-loop interrupt. Intermediate state us not saved, so can't recover from system failures mid-execution.
- "async": LangGraph persists changes asynchronously while the next step executes.
  risk:- It does  not right checkpoints if the process crashes during execution.
- "sync": Most durable. Synchronous changes. High durability but  has a little performance overhead.


### Optimise checkpoint storage
Delta channel stores only incremental deltas instead of full accumulated value.



Checkpointer Libraries
1. langgraph-checkpoint :The base interface for checkpointer savers (BaseCheckpointSaver) and serialization and deserialisation interface (SerializeerProtocol) and inclludes in-memory checkpointer implementation (InMemorySaver).
2. langgraph-checkpoint-sqlite :SqlliteSaver/ AsyncSqlliteSaver.
3. langgraph-checkpoint-postgress: PostgresSaver / AsyncPostgresSaver.
4. langchain-azure-cosmosdb: for Azure's NoSQL (CosmosDBSaverSynce/CosmosDBSaver).


### Checkpointer interface

Each checkpointer conforms to [`BaseCheckpointSaver`](https://reference.langchain.com/python/langgraph/checkpoints/#langgraph.checkpoint.base.BaseCheckpointSaver) interface and implements the following methods:

- `.put` - Store a checkpoint with its configuration and metadata.
- `.put_writes` - Store intermediate writes linked to a checkpoint (i.e. [pending writes](https://docs.langchain.com/oss/python/langgraph/checkpointers#pending-writes)).
- `.get_tuple` - Fetch a checkpoint tuple using for a given configuration (`thread_id` and `checkpoint_id`). This is used to populate `StateSnapshot` in `graph.get_state()`.
- `.list` - List checkpoints that match a given configuration and filter criteria. This is used to populate state history in `graph.get_state_history()`

### Serializer
When checkpointers save the graph state, they need to serialize the channel values in the state. This is done using serializer objects.

### Encryption
Checkpointers can optionally encrypt all persisted state. To enable this, pass an instance of [`EncryptedSerializer`](https://reference.langchain.com/python/langgraph/checkpoints/#langgraph.checkpoint.serde.encrypted.EncryptedSerializer) to the `serde` argument of any [`BaseCheckpointSaver`](https://reference.langchain.com/python/langgraph/checkpoints/#langgraph.checkpoint.base.BaseCheckpointSaver) implementation. The easiest way to create an encrypted serializer is via [`from_pycryptodome_aes`](https://reference.langchain.com/python/langgraph/checkpoints/#langgraph.checkpoint.serde.encrypted.EncryptedSerializer.from_pycryptodome_aes), which reads the AES key from the `LANGGRAPH_AES_KEY` environment variable (or accepts a `key` argument):
```
from langgraph.checkpoint.serde.encrypted import EncryptedSerializer
from langgraph.checkpoint.postgres import PostgresSaver

serde = EncryptedSerializer.from_pycryptodome_aes()
checkpointer = PostgresSaver.from_conn_string("postgresql://...", serde=serde)
checkpointer.setup()
```

 Other encryption schems can be used by implementing CipherProtocol and supplying it to EncryptedSerializer.


### Custom Checkpointer
LangGraph’s persistence layer is built on two storage abstractions:

- **Checkpoints table** — one row per superstep; stores the serialized graph state (`channel_values`, `channel_versions`, `versions_seen`) and links to its parent checkpoint.
- **Writes table** — one row per node output within a superstep; stores `(task_id, channel, value)` tuples linked to a checkpoint.

Your checkpointer manages both tables. `put` writes a checkpoint row; `put_writes` writes node-output rows; `get_tuple` reads both back into a `CheckpointTuple`.

### Base contract

Subclass `BaseCheckpointSaver` and implement these five methods. All are required — a missing base method raises `NotImplementedError` at runtime.
```
from collections.abc import AsyncIterator, Iterator, Sequence
from typing import Any
from langchain_core.runnables import RunnableConfig
from langgraph.checkpoint.base import (
    BaseCheckpointSaver,
    ChannelVersions,
    Checkpoint,
    CheckpointMetadata,
    CheckpointTuple,
)

class MyCheckpointer(BaseCheckpointSaver):
    async def aput(
        self,
        config: RunnableConfig,
        checkpoint: Checkpoint,
        metadata: CheckpointMetadata,
        new_versions: ChannelVersions,
    ) -> RunnableConfig:
        ...

    async def aput_writes(
        self,
        config: RunnableConfig,
        writes: Sequence[tuple[str, Any]],
        task_id: str,
        task_path: str = "",
    ) -> None:
        ...

    async def aget_tuple(self, config: RunnableConfig) -> CheckpointTuple | None:
        ...

    async def alist(
        self,
        config: RunnableConfig | None,
        *,
        filter: dict[str, Any] | None = None,
        before: RunnableConfig | None = None,
        limit: int | None = None,
    ) -> AsyncIterator[CheckpointTuple]:
        ...
        yield  # make this an async generator

    async def adelete_thread(self, thread_id: str) -> None:
        ...
```

### put/ aput
Store one checkpoint row. Return an update config with the stored #checkpoint_id 

Key requirements:
- Serialise the checkpoint using self.serde.dumps_typed(checkpoint)
- Store metadata in full - do not strip unknown keys.
- Store config["configurable"].get( #checkpoint_id) as parent checkpoint ID so as get_tuple can populate parent_config.

### put_writes / aput_writes

Store node-output rows for a single task within the current superstep. These rows are linked to the checkpoint by `(thread_id, checkpoint_ns, checkpoint_id)`.

### get/aget tuple
This checkpoint may contain
- No #checkpoint_id  --return the lates checkpoint for the thread + namespace/
- A specific #checkpoint_id  -- return that exact checkpoint.

### list / alist

Return checkpoints for a thread, newest first. Respect `before` (return only checkpoints older than that config’s `checkpoint_id`) and `limit`.

### delete_thread / adelete_thread

Delete all checkpoints and writes for a thread. Both checkpoint rows and write rows must be deleted.

> #checkpoint_id  is a ULID , it sorts lexicographically, So to get it is quite easy as to get the latest we just have to query  like ORDER BY #checkpoint_id  DESC LIMIT  1 ;

For non_SQK=L stores the same principle applies. Direct lookup should be present which would take O(1) or a very small constant time. Indexing should be done properly for this thing to happen. Indexing should be for #checkpoint_id , #thread_id and #checkpoint_ns.


### Serialiser
In **LangGraph**, a **serialiser** is ==a utility used by the **persistence and checkpointing layer**==. It converts complex Python objects (like your agent's current state, conversation history, or LLM tool calls) into a storable format (like JSON or bytes) and reconstructs them later. 
Use self.serde (Inherited from BaseCheckpointSaver)

### Extended capabilities
These methods are optional but unlock additional Agent Server features. Implement them if your storage backend can support them efficiently.

| Method                       | What it enables                                          |
| ---------------------------- | -------------------------------------------------------- |
| `adelete_for_runs`           | Rollback multitask strategy                              |
| `acopy_thread`               | Efficient thread forking                                 |
| `aprune`                     | Thread history pruning                                   |
| `aget_delta_channel_history` | Efficient delta channel state reconstruction (see below) |
Agent Server auto-detects which capabilities your checkpointer implements at startup and activates the corresponding features.

### Delta Channel support
It is only a reducer that stores sentinal(missing) values in the checkpoint blobs. Which helps to reduce the memory management by a lot. State us reconstructed by replaying ancestor writed through a reducer. This makes checkpoint blobs O(1) instead of O(N) for channels like messages.

#### What runtime needs 
When loading a checkpoint whose delta values are missing from channel values. 
LangGraph calls `saver.get_delta_channel_history(config=config, channels=[...])`. This returns, for each channel:

- **`writes`** — all writes to that channel in the ancestor chain, oldest first, up to the nearest snapshot.
- **`seed`** (optional) — the stored `_DeltaSnapshot` blob at the nearest ancestor that has one; absent if the walk reaches the root without finding a snapshot.

The runtime then calls `channel.from_checkpoint(seed)` and `channel.replay_writes(writes)` to reconstruct the live value.
```
Base CheckpointSaver provides a default get_delta_channel_history that works with any correct get_tuple implementation.
```

**The critical dependency:** `get_tuple(cursor)` is always called with a specific `checkpoint_id` (the parent’s id). If that lookup returns `None`, the walk stops immediately and every delta channel reconstructs as empty — silently, with no error. This is why the specific-id path in `get_tuple` must be correct.

#### Performance override

The default walk issues one `get_tuple` call per ancestor checkpoint. For backends with good query support, override `get_delta_channel_history` (and its async twin) to retrieve the ancestor chain and writes in two queries:

#### Pruning with delta channels

`DeltaChannel` state is not self-contained in a single checkpoint — it depends on the ancestor write chain back to the nearest `_DeltaSnapshot`. If you implement `prune` or `delete_for_runs`, you must not delete write rows that a surviving checkpoint’s delta channels depend on.Safe options:

1. **Walk before pruning** — for each checkpoint you intend to keep, walk its ancestor chain and mark all write rows up to the nearest `_DeltaSnapshot` as non-deletable.
2. **Force a snapshot before pruning** — rewrite `channel_values[ch] = _DeltaSnapshot(reconstructed_value)` on the checkpoint you are keeping, then delete ancestors freely.
3. **Skip pruning for delta-channel threads** — the safest short-term option if you do not yet need pruning.

### Conformance suite
Used to validate your implementation against full contract, including delta channel history.
``` langgraph-checkpoint-conformance```









