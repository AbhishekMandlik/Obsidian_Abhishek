- Workflows have predetermined code paths and are designed to operate in a certain order.
- Agents are dynamic and define their own processes and tool usage.
 
![[Pasted image 20260630151002.png]]
Workflow and agentic systems are based in LLMs and the various augmentations you add to them. Tool calling, structured outputs, and short term memory are a few options for tailoring LLMs to your needs.
## Prompt chaining
Prompt chaining is when each LLM call processes the output of the previous call. It’s often used for performing well-defined tasks that can be broken down into smaller, verifiable steps. Some examples include:
## Parallelisation
LLMs work simultaneously on a task.
- Split up subtask and run them in parallel, which increases speed.
- Run tasks multiple times to check for different output, which increases confidence.

![parallelization.png](https://mintcdn.com/langchain-5e9cc07a/dL5Sn6Cmy9pwtY0V/oss/images/parallelization.png?fit=max&auto=format&n=dL5Sn6Cmy9pwtY0V&q=85&s=8afe3c427d8cede6fed1e4b2a5107b71)

## Routing
Routing workflows process inputs and then directs them to context-specific tasks. This allows you to define specialised flows for complex tasks.
A workflow built to answer product related questions might process the type of question first, and then route the request to specific processes for pricing, refunds, returns, etc.![routing.png](https://mintcdn.com/langchain-5e9cc07a/dL5Sn6Cmy9pwtY0V/oss/images/routing.png?fit=max&auto=format&n=dL5Sn6Cmy9pwtY0V&q=85&s=272e0e9b681b89cd7d35d5c812c50ee6)

## Orchestrator-worker
In an orchestrator-worker configuration, the orchestrator:
- Breaks down tasks into subtasks.
- Delegates subtasks to workers.
- Synthesises worker outputs into a final result.

![worker.png](https://mintcdn.com/langchain-5e9cc07a/ybiAaBfoBvFquMDz/oss/images/worker.png?fit=max&auto=format&n=ybiAaBfoBvFquMDz&q=85&s=2e423c67cd4f12e049cea9c169ff0676)

Orchestrator-worker workflows are common and LangGraph has built-in support for them. The `Send` API lets you dynamically create worker nodes and send them specific inputs. Each worker has its own state, and all worker outputs are written to a shared state key that is accessible to the orchestrator graph. This gives the orchestrator access to all worker output and allows it to synthesise them into a final output.

## Evaluator-optimiser
In evaluator-optimiser workflows, one LLM call creates a response and the other evaluates that response. If the evaluator or a [human-in-the-loop](https://docs.langchain.com/oss/python/langgraph/interrupts) determines the response needs refinement, feedback is provided and the response is recreated. This loop continues until an acceptable response is generated.![evaluator_optimizer.png](https://mintcdn.com/langchain-5e9cc07a/-_xGPoyjhyiDWTPJ/oss/images/evaluator_optimizer.png?w=1650&fit=max&auto=format&n=-_xGPoyjhyiDWTPJ&q=85&s=d31717ae3e76243dd975a53f46e8c1f6)
