1. Request Entry:
   Endpoint: POST /v2/chat
   It Requires:
	1. setup_with_validate_v2 -- parses request from field into ExecutorRequest, validates JWT, sets context["request"] and context["token_data"].
	2. get_auth_token from HTTP Bearer.
	3. get_session_manager -- Redis session from request.id + user_id.
	4. File Validator: If the uploaded file are correct or not.
2. Pre-pipeline checks:
	1. IDE version check
	2. Document upload
		1. Calls ai-ingest(ingest_svc_url + upload suffix)
		2. Sets additional_inference_data.uploaded_documents_context on the request(context_id =session id)
	3. Legacy repo key check (check_for_key_repositories)
	4. Atlassian credential check -- decides later agent vs RAG fallback
	5. Character limit validation on request.message
	6. Saves request.raw_user_message
3. Invoke pipeline:
   Branches on request.options.streaming:
    | `streaming=true` (default) | `invoke_pipeline()` → SSE                        |
	| `streaming=false`               | `invoke_pipeline_nonstream()` → JSON |
4. Session + intent setup (invoke_pipeline-> ConversationNode)
	1. Redis Session: Load last_run_pipeline and store System prompt
	2. Create extractors: 
		1. IntentExtractor - LLM classifies intent
		2. CodeInfoExtractor - language/framework from query
		3. Prompt Extractor - calls ai-prompt-lib
	3. Conversation state branches:
		1. prompt_selection User picks child prompt(1, 2, 3...); restores prior query.
		2. template_variable Collects missing template params or cancel.
		3. Normal: get_intent_and_prompt("chat_llm")
5. Intent and prompt resolution
   (ConversationNode.get_intent_and_prompt)
   Priority Order:
	1. pipeline_to_invoke == "rag".
	2. prompt_info -> fetch prompt from ai-prompt-lib retriever.
	3. Else -> _find_intent_and_prompt
		1. is_basic_chat
		2. intent=DOJO
		3. rag_context.account present then default to rag
			1. optionally parallel code info extraction + prompt v3 search.
6. Request Enrichment:
	1. If a single match prompt exists, inject template into the request
	2. Guideline prompts are added
	3. Child prompt chains, if multi step then run sequentially
7. Route to ai-exec:
	1. intent is DOJO executor endpoint=/agent endpoint
	2. /rag endpoint if DOJO = no atlassian credentials
	3. HTTP call:
		1. streaming: send_http_request() via SSE (aconnect_sse)
		2. Forwards auth
		3. stream data
		4. Ends with [Done]\n\n
8. For the common path:
	1. RAG:
		1. Parse `rag_context` → build retrievers
		2. Optional query rewrite + ensemble retrieval from Milvus
		3. LLM answer generation (streamed back through orchestrator)
		   For uploaded docs in v2: executor also retrieves from `uploaded_documents` collection filtered by `context_id` (session id).
	2. DOJO:
		1. Calls async dojo tools.
		2. Reads Docstrings and then gives output.
		3. Different tools for azure, jira and confluence.
9.  Post-response
	- `session_manager.store_processing_time()`
	- `log_chat_request()` — metrics/logging


### RAG
```
IDE → ai-orc /v2/chat
    → (optional) ai-ingest document upload
    → ai-prompt-lib prompt/guideline lookup
    → ai-exec /rag
        → embed query → Milvus search → LLM answer (SSE stream)
    → IDE
```
### DOJO
```
IDE (with Atlassian creds in headers)
    → ai-orc detects DOJO intent
    → ai-exec /agent
        → LangChain agent + tools (Jira, Confluence, Azure)
        → may also use RAG + MCP
    → IDE
```