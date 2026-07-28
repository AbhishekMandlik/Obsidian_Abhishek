Used for 
- Building AI applications.
- Deploying foundation models.
- Fine-tuning models.
- Evaluating AI systems.
- Monitoring AI applications.
- Managing agents and tools.

1. AI Foundry Hub: Top-level resource, It contains:-
   >Organisation->Hub->Projects
	1. Projects
	2. Models
	3. Deployments
	4. Connections
	5. Monitoring
	
2. Project: Our working environment:
   It contains:-
	1. Models
	2. Endpoints
	3. Datasets
	4. Evaluations
	5. Agents
3. Model Catalog:
   Contains models from: All the currents llm's. eg(Anthropic etc.)

## Model Deployment Flow
> Choose Models -> Configure Compute -> Deploy -> Endpoint Created -> Consume via API.

Types of Deployments:
1. Server-less: Azure hosts everything
2. Managed Compute Deployment: We select VM resources
	1. Better Control
	2. Lower cost
	3. but we have to do infrastructure management

### Deploying a Model
1. Go to model Catalog.
2. Choose model.
3. Click on deploy.
4. Choose Server-less/ Managed Compute.
5. Create Deployment
   Azure Creates 
	1. Endpoint URL
	2. API Key
	3. Deployment Name
6. Call the endpoint:
   ```
   from openai import OpenAI

	client = OpenAI(
	    api_key="KEY",
	    base_url="ENDPOINT"
	)
	
	response = client.chat.completions.create(
	    model="llama-3",
	    messages=[
	        {"role":"user","content":"Hello"}
	    ]
	)
   ```
7. Internally:
   User Query->Azure Endpoint->Model Container->GPU->Inference->Response
8. Fine- Tuning in AI Foundry
	1. GPT -> Company Dataset -> Fine Tune -> Company GPT
9. Prompt Flow: Azure's Orchestration framework. (User Query->Prompt->Retriever->....)
10. RAG in AI Foundry  Uses Azure's AI Search if asked.
11. Azure AI Search:- Provides
	1. Vector Search
	2. Hybrid Search
	3. Semantic search
12. Model Evaluation: - Metrics for evaluation:
	1. Grounded-ness.
	2. Relevance.
	3. Coherence.
	4. Safety.
	5. Similarity.
	These are similar metrics Like we used in Deep-Eval framework for slingshot:
	Like Answer Relevancy, Faithfulness, Rag Completeness etc.
13. Agents: (LLM + Tools + Memory)
	1. API's.
	2. Search.
	3. Databases.
	4. Function's.
14. Monitoring: Requires monitoring things like 
	1. Latency.
	2. Token Usage.
	3. Cost.
	4. Errors.
	5. User Feedback.
15. Security: Supported features
	1. Managed Identity.
	2. RBAC.
	3. Private Endpoints.
	4. VNET Integration.
	5. Key Vault.