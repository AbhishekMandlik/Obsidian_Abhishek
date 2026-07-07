
>MCP provides a standard way for AI models to connect to tools, databases, APIs, and applications.
>You just return the output in a specific format you will be able to use it with LLM
>MCP uses **JSON-RPC 2.0** for communication

**Request**
```
{  
	"jsonrpc": "2.0",  
	"method": "tools/list",  
	"id": 1  
}
```
**Response**
```
{
  "jsonrpc": "2.0",
  "result": {
    "tools": []
  },
  "id": 1
}
```
**Error**
```
{
  "jsonrpc": "2.0",
  "error": {
    "code": -32601,
    "message": "Method not found"
  },
  "id": 1
}
```

Advantages of Using MCP are:
1. Reusable integration
2. agent development made easier
3. Standard tool definition

```
User
 ↓
Host
 ↓
MCP Client
 ↓
MCP Server
 ↓
Tool/API
```
**Minimal Setup for MCP server**
```
mcp=FastMCP(
	name="",
	host="",
	port="",
)
```
host and name are required for having server connect through http.

Adding a tool in mcp
```
@mcp.tool()
def add(a:int,b:int->int:
	""" Add two Numbers """
	
	return a+b
```

then in the main function just write
```
mcp.run(transport=sse/stdio)
```

You can inspect MCP server through: MCP inspector
just write the following in terminal
```
mcp dev server.py
```

To run as stdio
```
server_params=StdioServerParameteres(
	command="python".
	args=[server.py],
)

and

transport=stdio
```

> Our Atlassian Mcps just call the Atlassian rest apis and do the following things.
> Not doing anything crazy


### MCP's with LLM's
Server Side:
```
@mcp.tool
def get_knowledge_base():
	"""Docstring"""
	function(Use langchain retriever);
	return	
```
Client Side:
```
async def connect_to_server(self, server_script_path: str:"server.py"):
	"""Connect to MCP server"""
	server_params= StdioServerParameters(
	command="python",
	arg=[server_script_path],
	#Then connect to the server#
	)
	
async def main():
	client=MCPOpenAIClient()
	await client.connect_to_server("server.py")
	#Use http or stdio#
	
async def get_mcp_tools():
	await tool_result=session.list_tools()
	#then return in openai's specified format
	return[
		"type":"function",
		"function":{
			"name": tool.name,
			"description": tool.description,
			"parameters": tool.inputSchema,
		}
		for tool in tool_result..tools
	]
Client Side also has a process query function where you can pass your query
```

So we need to return the tools in OpenAI's format if we are using the OpenAI client.
Transport Method
# 1. stdio
standard input and output
### Advantages
- Very fast
- Simple
- Local machine only
- No networking
Used for:
- Filesystem MCP servers
- Git MCP servers
- Local database tools
# HTTP

Traditional request-response model.
POST /mcp
Advantages
- Simple
- Easy deployment
- Works through firewalls
Good for:
- Remote MCP servers
- Cloud-hosted tools

# SSE
Advantages
- Streaming output
- Simpler than WebSockets
- Efficient for long-running tasks
Interview answer:

> SSE allows the server to push streaming updates to the client over a persistent HTTP connection.

# New method (/mcp endpoint)
Slingshot uses Streamable HTTP
It's a transport mechanism for MCP that uses **standard HTTP** as the communication channel while supporting both:

1. Normal request-response interactions
2. Streaming responses when needed

## Why was it introduced?
Earlier MCP remote servers often used:
```HTTP + SSE```
where:
- One endpoint for requests
- Another SSE connection for streaming
This was more complex.
Streamable HTTP simplifies this into a **single MCP endpoint** (typically `/mcp`) and can return either:
- JSON response
- Streaming event stream
depending on the operation.