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

## PDF/DOCX/PPTX
functions routed through specialised splitter: 
1. get_pdf_splitter
2. get_pptx_splitter
3. get_docx_splitter

## Jira/Confluence
Content converted into a document Then split using the same recursive CharacterTextSplitter (chunk_size, chunk_overlap) via splitter.split_documents(...)

## Which Embeddings are used:
Embeddings come from ai-common via EmbeddingModel(...)
We use CustomOpenAIEmbeddings instance that calls your enterprise embedding service.

We have:
```filter_oversized_chunks=true```
What it does is that it may drop or truncate chunks that exceed internal thresholds.



> We use text-embedding-3-small model for embeddings.
> This is the latest affordable embedding model from OpenAI. This was chosen when benchmarking was done for cost vs context retrieved and hence the decision was taken regarding the same.



### What do these get_pdf_splitter, get_pptx_splitter or get_docs_splitter do?
1. Extract structured content: text, tables, images, charts, notes, etc.
2. Chunk by token/word limit using `chunking.chunk_size` (default `3000`), not character-based LangChain splitting.
3. Return dict chunks, not plain strings:
    - `{"chunk": "...", "slide_number": ..., "slide_title": ...}` for PPTX
    - `{"chunk": "...", "page_number": ...}` for PDF
    - `{"chunk": "...", "section_number": ..., "heading": ...}` for DOCX
4. Need `authorization` because image/chart extraction can call an LLM vision model (`ImageDataExtractor`) or OCR (`OCREngine`).

Individually:
1. `get_pdf_splitter`:
	Per page it extracts:
	- Text via PDF text layer
	- Tables via `pdfplumber` → markdown tables
	- Images via LLM vision (`gpt-4.1` by default) or Tesseract OCR
2. get_pptx_splitter:
	1. Slide title
	2. Text from shapes
	3. Tables(PrettyTable format)
	4. Charts -> chart title, type, and data from embedded Excel.
	5. Images -> LLM vision or OCR
	6. Speaker notes
	Chunking Logic:
	- Slide Number + Slide title
	- Sections like slide text, slide table etc.
	- If slide is large split into multiple chunks while preserving slide metadata.
3. get_docx_splitter:
   Parse word in original document order, then chunk by section:
	- Paragraphs, tables, inline images, textboxes (`w:txbxContent`)
	-  Headers/footers (optional)
	- Groups content under Heading styles into sections
	- Merges adjacent text blocks to reduce tiny fragments before chunking
	1. text
	2. table
	3. textbook
	4. image
## Why these exist instead of generic chunking

Plain `RecursiveCharacterTextSplitter` would:

- Treat PDF/PPTX/DOCX as raw bytes or poorly extracted text
- Miss tables, charts, images, speaker notes
- Lose page/slide/section structure

These specialised splitters are extract + structure + chunk pipelines designed for RAG quality on office documents.

### 