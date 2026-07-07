Two types of API's
- [Use the Graph API](https://docs.langchain.com/oss/python/langgraph/quickstart#use-the-graph-api) if you prefer to define your agent as a graph of nodes and edges.
- [Use the Functional API](https://docs.langchain.com/oss/python/langgraph/quickstart#use-the-functional-api) if you prefer to define your agent as a single function.


## Define a tool:
```
@tool
def multiply(a:int, b:int)-> int:
	"""Docstring which LLM Reads"""
	return a * b

## Docstring should be optimised because it is fed into LLM and it uses a lot of tokens.
```
## Define state:
The graphs state is used to store the messages and the number of LLM calls.
> State in LangGraph persists throughout the agent’s execution.The `Annotated` type with `operator.add` ensures that new messages are appended to the existing list rather than replacing it.

```
from langchain.messages import AnyMessage
from typing_extensions import TypedDict, Annotated
import operator


class MessagesState(TypedDict):
    messages: Annotated[list[AnyMessage], operator.add]
    llm_calls: int
```

## Define Model Node
```
def llm_call (state: dict):
""" LLM decides whether to call a tool or not """
	return {
		 "messages":[
			 llm.invoke(
				 [
					 SystemMessage(
					 content=""
					 )
				 ]
				 + state["messages"]
			 )
		 ],
		 "llm_calls": state.get('llm_calls',0)=1
	}
```
## Define Tool Node
#Calls_Tool
```
from langchain.messages import ToolMessage

def tool_node(state: dict):
    """Performs the tool call"""

    result = []
    for tool_call in state["messages"][-1].tool_calls:
        tool = tools_by_name[tool_call["name"]]
        observation = tool.invoke(tool_call["args"])
        result.append(ToolMessage(content=observation, tool_call_id=tool_call["id"]))
    return {"messages": result}
```

## Define End logic
```
from typing import Literal
from langgraph.graph import StateGraph, START, END


def should_continue(state: MessagesState) -> Literal["tool_node", END]:
    """Decide if we should continue the loop or stop based upon whether the LLM made a tool call"""

    messages = state["messages"]
    last_message = messages[-1]

    # If the LLM makes a tool call, then perform an action
    if last_message.tool_calls:
        return "tool_node"

    # Otherwise, we stop (reply to the user)
    return END
```

##  Build and compile the agent
Build using stateGraph and complied using .compile function:
```
# Build workflow
agent_builder = StateGraph(MessagesState)

# Add nodes
agent_builder.add_node("llm_call", llm_call)
agent_builder.add_node("tool_node", tool_node)

# Add edges to connect nodes
agent_builder.add_edge(START, "llm_call")
agent_builder.add_conditional_edges(
    "llm_call",
    should_continue,
    ["tool_node", END]
)
agent_builder.add_edge("tool_node", "llm_call")

# Compile the agent
agent = agent_builder.compile()

# Show the agent
from IPython.display import Image, display
display(Image(agent.get_graph(xray=True).draw_mermaid_png()))

# Invoke
from langchain.messages import HumanMessage
messages = [HumanMessage(content="Add 3 and 4.")]
messages = agent.invoke({"messages": messages})
for m in messages["messages"]:
    m.pretty_print()
```