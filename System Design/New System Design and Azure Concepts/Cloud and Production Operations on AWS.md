### AWS GenAI Architecture
```
Client
-> Amazon CloudFront or direct HTTPS
-> Amazon API Gateway or Application Load Balancer
-> Auth layer: Cognito, custom JWT authorizer, or API keys
-> Compute: Lambda for simple event-driven logic, ECS/EKS for long-running services
-> Model inference: Amazon Bedrock, SageMaker endpoint, or self-hosted model on ECS/EKS/EC2
-> Retrieval: S3 documents, vector database, Bedrock Knowledge Bases, OpenSearch,
Aurora, or DynamoDB
-> Storage: S3 for objects, database for conversations and metadata
-> Observability: CloudWatch logs, metrics, alarms, traces, dashboards
-> Security: IAM, KMS, Secrets Manager, VPC controls, WAF, Guardrails
-> CI/CD: CodePipeline/GitHub Actions, CodeBuild, CDK/Terraform, model registry, canary
deployment
```
Bedrock is a managed way to access foundation models without operating model infrastructure.
SageMaker gives more control over custom training, model registry and endpoints and monitoring.
ECS/EKS give container orchestration if you need custom services, self hosted models, or complex workloads.
Lambda is excellent for event driven glue and lightweight APIs.

### SageMaker
Amazon SageMaker is AWS’s managed platform for machine learning development, training, deployment, and operations. In interviews, think of SageMaker as the service you use when you own the ML model lifecycle:
- data processing
- training jobs
- model artefacts 
- model registry 
- deployment to endpoints 
- batch transform
- pipelines
- monitoring
- governance.

We provide training data, algorithm, container image, hyper-parameters, o/p location.
It runs training saves artefacts to S3 and then tears down the infrastructure.

SageMaker Model Registry catalogs models for production, manages model versions, stores metadata and approval status and helps track lineage.
This helps us identify which version of model is deployed and how to role it back as and when required. These Models can be monitored and quality issues and drifts can be detected . Drift means change in model behaviour compared to baseline used during training or validation.

BedRock is when you want to manage foundation models It is similar to azure AI foundry.
```
Use Bedrock when you want managed foundation models, fast GenAI application development, model choice from supported providers, guardrails, agents, knowledge bases, and no direct model hosting operations.
```


### AWS Lambda
Serverless compute.
Excellent for short lived event-driven tasks like API handlers,S3 upload processing, lightweight orchestration, webhooks, schedule jobs(CRON), log processing and glue between services.
Strengths:
- Low operational head
- Automatic scaling
- Pay-per-use
- Deep AWS integration
Weakness:
- Runtime limits
- Cold starts
- Reduced control over infrastructure

For high volume streaming 
- Containerised service behind Application Load Balancer or API Gateway if it gives more control over connection lifecycle and backpressure.


### ECS and EKS
Container orchestration service:
Containers package application code, dependencies, and runtime into a portable unit. Orchestration means scheduling containers, restarting failed containers, scaling services, connecting networking, load balancing, and managing deployments.

ECS is AWS-native container orchestration. It is simpler if you are fully on AWS and want tight integration with IAM etc..
Fargate lets you run containers without managing EC2 instances.

EKS is managed Kubernetes. Kubernetes is powerful and portable across environments, but more complex.
Use EKS if your organisation already uses Kubernetes, needs Kubernetes ecosystem tooling, service mesh, custom controllers, portability, or standardised deployment patterns across clouds.

For LLM APIs, ECS can run a FastAPI, Node, or Java service that maintains streaming responses, handles connection state, and calls Bedrock or SageMaker. EKS can do the same but adds Kubernetes control.
For self-hosting run EKS with GPU's.


### Amazon API Gateway
Amazon API Gateway is a managed entry point for APIs. It can route requests to Lambda, HTTP services, VPC links, and other backends. It supports REST APIs, HTTP APIs, and WebSocket APIs. It can help with authentication, authorisation, request validation, throttling, usage plans, API keys, deployment stages, and logging. AWS documentation notes that throttling and quotas can protect APIs from being overwhelmed. Throttling or overwhelmed APIs give "**429 HTTP Error**."

### Amazon S3
Amazon S3 is object storage. It stores objects in buckets with keys. It is not a filesystem, although people often organise keys like paths. S3 is used for datasets, logs, model artefacts, documents, uploads, exports, backups, static assets, and data lakes.

> Important S3 concepts: Versioning keeps multiple variants of an object, helping recover from accidental deletion or overwrite. For ML, versioning data and artefacts is essential for reproducibility.

Server-side encryption protects objects at rest. S3 applies server-side encryption with S3-managed keys as a base level for new uploads, and you can choose SSE-KMS for tighter key control.


### Amazon Bedrock
Amazon Bedrock is a fully managed service for building generative AI applications using foundation models from Amazon and third-party providers. It is highly relevant for GenAI interviews because it lets teams use LLMs without operating model infrastructure directly.
Core Bedrock concepts include model invocation, model selection, agents, knowledge bases, guardrails, model customisation, provisioned throughput, and prompt caching.


### CI/CD
```
Git commit
-> unit tests and static checks
-> build training/inference container
-> data validation
-> preprocessing job
-> training or fine-tuning job
-> evaluation on holdout set
-> bias/safety/security checks if applicable
-> register model version in Model Registry
-> manual or automated approval gate
-> deploy to staging endpoint
-> integration tests and load tests
-> canary or blue/green production deployment
-> monitor metrics and rollback if needed
```
Types of Deployment:
- Canary- 1% then 5 % then 100 %.
- Blue/Green- shift to green if any flaws rollback to blue.
- Shadow- run both and check the one in shadows.


### Monitoring and Observability
- Metrics: For APIs, track request rate, error rate, latency percentiles, saturation, and availability. Percentiles matter because averages hide tail latency.
- Traces: Distributed tracing connects the path of a request across services.
- Drift Detection:Model drift happens when production input distribution or output quality changes.
- Logs: Logs should be structured JSON, not random strings. Include request ID, user or tenant ID if allowed, endpoint, status, latency, model name, token counts, error code, and trace ID.


### Disaster Recovery and Multi-Region Basics
Disaster recovery is planning for events that prevent a workload from meeting business objectives in its primary location. RTO is how quickly the system must recover. RPO is how much data loss is acceptable.
1. Backup Strategy:
Back up databases, S3 buckets, model artefacts, prompt templates, container images, infrastructure definitions, and evaluation datasets. Test restores. A backup that has never been restored is only a hope.
2. Failover:
Failover can be manual or automatic. Automatic failover is faster but can create split-brain or cascading failure if not designed carefully. Manual failover may be acceptable for lower criticality systems. Use Route 53 health checks or other routing mechanisms where appropriate.


### Cost Optimisation for LLM Inference
1. Model Routing: Not every request needs the largest model. Route simple classification, formatting, or extraction tasks to cheaper smaller models.
2. Prompt and Context Optimisation: Input tokens cost money and increase latency. Keep system prompts concise.
3. Response Length Control: Set reasonable max output tokens.
4. Caching: Cache deterministic or common outputs when acceptable.
5. Batching and Async Processing: Batching improves throughput for embedding generation and offline summarisation.
6. Autoscaling and Capacity: For self-hosted models, GPU utilisation matters