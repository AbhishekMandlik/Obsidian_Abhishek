Introduction to LangChain:
>LangChain is an open-source framework that simplifies building applications using large language models. It helps developers connect LLMs with external data, tools and workflows and is available in both Python and JavaScript.

![[Pasted image 20260624152455.png]]

**Key Components:**
Chains Sequence of steps where each step can use LLM.
Prompt Management: Helps design and manage prompts using prompt templates.
Agents: LLM driven components that decide which action to take.
Vector Database: Stores data as vectors to enable similarity search.
Models: Supports multiple LLM models.
Memory Management: Maintains context from past interactions.
## Working of LangChain

LangChain enables [Retrieval-Augmented Generation (RAG)](https://www.geeksforgeeks.org/nlp/what-is-retrieval-augmented-generation-rag/) by combining document processing, vector storage and LLMs to generate accurate, context aware responses. It connects embeddings, vector databases and models into a smooth workflow.
All things related to vector ingestion and retrieval is present in vectorstore in langchain.

Once a query is received, LangChain converts it into vectors using embedding to capture the semantic meaning of the query. 

parser = StrOutputParser(): Cleans up and ensures the LLM's response is returned as a string.
```chain = prompt_template | llm | parser```

LangChain’s modularity enables:

- Flexible prompt creation and chaining.
- Integration with various LLMs (OpenAI, Anthropic, Gemini etc).
- Parsing and structuring LLM outputs.
- Retrieval-augmented generation (RAG) for grounding responses in external data.
- Semantic search using [embeddings](https://www.geeksforgeeks.org/nlp/word-embeddings-in-nlp/) and [vector databases](https://www.geeksforgeeks.org/dbms/what-is-a-vector-database/).
- Agentic workflows for multi-step reasoning and tool use.


## Prompt Module
Structure and format user input into prompts that LLMs can interpret. Support both PromptTemplate and ChatPromptTemplate.
```
from langchain_core.prompts import PromptTemplate, ChatPromptTemplate

# String-based prompt
prompt = PromptTemplate.from_template("What is the capital of {country}?")
filled_prompt = prompt.format(country="Germany")
print(filled_prompt)  

# Chat-based prompt
chat_prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    ("human", "{input}")
])
chat_filled = chat_prompt.invoke({"input": "Translate 'hello' to French."})
print(chat_filled.to_messages())
```
## Chat Model and LLM Module
Interface with LLMs for text or chat completion.
```
from langchain_google_genai import ChatGoogleGenerativeAI
llm = ChatGoogleGenerativeAI(model="gemini-2.0-flash", temperature=0)
response = llm.invoke("What is the capital of Germany?")
print(response.content)
```
## Output Parser Module
Parse and structure the raw output from LLMs into usable formats
```
from langchain_core.output_parsers import StrOutputParser

parser = StrOutputParser()
parsed_output = parser.parse(response.content)
print(parsed_output)
```
## Retrieval-Augmented Generation (RAG) Module
![[Pasted image 20260624160208.png|389]]
Augment LLM outputs by retrieving relevant data from external sources (HTML, DOC, S3, web buckets, etc.) and injecting it into prompts
Components:
1. Document Loaders: Data Ingestion
2. Embeddings: Convert Documents into Semantic vectors
3. Vector database: Store Vectors for similarity search
4. Retriever: Finds relevant docs for query.
5. LLM Integration: Supplies retrieved content as context.
```
from langchain_core.documents import Document
from langchain_google_genai import GoogleGenerativeAIEmbeddings
from langchain.vectorstores import FAISS
from langchain.chains import RetrievalQA

docs = [
    Document(page_content="Berlin is the capital of Germany."),
    Document(page_content="Paris is the capital of France."),
]
embeddings = GoogleGenerativeAIEmbeddings(model="models/embedding-001")
vectorstore = FAISS.from_documents(docs, embeddings)
retriever = vectorstore.as_retriever()
qa_chain = RetrievalQA.from_chain_type(llm=llm, retriever=retriever)
rag_result = qa_chain.run("What is the capital of Germany?")
print(rag_result)
```
## Embedding and Vector Database Module
Embedding to capture semantic meaning of text for similarity search, Vector DB to store and index embedding for fast retrieval.
## Data Storage
Store and manage unstructured data for retrieval. LangChain supports loading documents from HTML, DOC, S3, and web buckets, which are then embedded and stored in a vector DB(database) for RAG.
```
from langchain.document_loaders import S3DirectoryLoader

loader = S3DirectoryLoader(bucket="your-bucket", prefix="docs/")
docs = loader.load()
```
## Agent Module
Enable autonomous, multi-step reasoning by chaining LLM calls with tool use (calculators, web search, database queries, etc).
![[Pasted image 20260624203829.png]]

```
from langchain.agents import initialize_agent, Tool

def calc_tool(input_str):
    try:
        return str(eval(input_str))
    except Exception as e:
        return str(e)

tools = [Tool(name="Calculator", func=calc_tool, description="Performs calculations.")]
agent = initialize_agent(tools, llm, agent="zero-shot-react-description", verbose=True)
agent_response = agent.run("What is 5 * 7?")
print(agent_response)
```

## Runnable
A **LangChain Runnable** is a standardised, composable building block that processes inputs and produces outputs. It forms the core of the LangChain Expression Language (LCEL). Runnables standardize AI components—like prompts, models, and tools—so they can be easily combined into pipelines. They inherently support synchronous, asynchronous, batch, and streaming execution. 
Key Execution Methods

Every LangChain Runnable exposes a set of core methods to handle different types of workflows:

- **`.invoke()` / `.ainvoke()`:** Transforms a single input into a single output.
- **`.stream()` / `.astream()`:** Streams output dynamically as it is generated by the LLM or component.
- **`.batch()` / `.abatch()`:** Efficiently processes a list of inputs in parallel using a thread pool. 