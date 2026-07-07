It takes a resource(repo/files, documents, confluence/Jira content etc) parses into texts, splits into chunks and then embeds and stores them into the configured vectorDB(Milvus or pgvector).
Chunking:
##  Text/ Doc Chunking
We use langChain's RecursiveCharacterTextSplitter with:
1. Chunk size: 3000
2. Chunk_overlap: 600
3. Seperator's:
	1. For most files: {"\n\n","\n", " ", ""}
	2. For JSON (when parser.enable_json_parser=true) tries RecursiveJsonSplitter; if JSON parsing fails it falls back to text splitting with JSON friendly seperators like{",", "..."} etc/
## Code Chunking
RecursiveCharacterTextSplitter.from_language(...)