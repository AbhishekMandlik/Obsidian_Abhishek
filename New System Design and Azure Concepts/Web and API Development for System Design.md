### What a Web API Actually Is
A web API is a contract between a client and a server.
The server exposes operations over a network, usually HTTP. The contract includes the URL, method, request format, authentication method, response format, error format, versioning rules, rate limits, and behaviour under failure.

For LLM chat APIs, there is an extra complication: inference is often slow and incremental. A normal JSON response may take many seconds if the model generates hundreds or thousands of tokens. That is why streaming protocols such as Server-Sent Events or WebSockets matter. Instead of waiting for the full answer, the server sends chunks as they are generated, improving perceived latency and user experience.

Bad REST URLs often turn operations into verbs without a clear resource model:
### HTTP Methods, Safety, and Idempotency
The most important HTTP methods are GET, POST, PUT, PATCH, and DELETE
1. GET: retrieves a resource and does not change server state
2. POST: creates a new resource or triggers a non-idempotent operation
3. PUT: replaces an entire resource at some known URL. PUT is Idempotent that means if you put the same resource twice at the same url it should remain the same.
4. PATCH: partially updates a resource for eg. increment a counter.
5. DELETE: removes a resource.... repeating it often returns 404 error.
Idempotency is crucial in production because the clients and infrastructure retry requests. The system may store two identical user messages and generate two model responses. Robust idempotency-key header is crucial.

>API's that return list must support pagination. It uses limi and offset.
### REST in LLM Systems
API has two styles:
1. Stateless: every message includes full history and server call the model.
   ```
   POST /v1/chat/completions
   ```
   Easy to scale, as server does not need to store conversation storage. But downside to this is larger request payloads and more responsibility on client.
2. Stateful: Server stores conversations and messages and clients send only new messages.
   ```
   POST /v1/conversations/{conversation_id}/messages
   ```
   Provides better product feature like showing history but requires storage, access control and retention policies.


### GraphQL API Design from 
GraphQL is a query language and runtime for APIs. The core idea is that the server exposes a strongly typed schema, and the client asks for exactly the fields it needs. Instead of many REST endpoints, GraphQL usually has one endpoint, such as "/graphql", and the query describes the data shape.
GraphQL is useful when frontend teams need flexible data fetching. Suppose a chat UI needs user profile, workspace settings, conversation metadata, last messages, attachments, and model options. With REST, the frontend may call many endpoints. With GraphQL, the client can ask for one nested shape. GraphQL also helps schema evolution because adding a new field does not break old clients.

Example shape:
```
# Example GraphQL query:
query GetConversation($id: ID!) {
	conversation(id: $id) {
		id
		title
		messages(limit: 20) {
			id
			role
			content
			createdAt
		}
	}
}
```
Response JSON
```
{
	"data": {
		"conversation": {
			"id": "conv_123",
			"title": "OAuth discussion"
			"messages": [... ]
		}
	}
} 
```

GraphQL clients can ask for deeply nested queries.
N+1 Problem happens when resolving a list causes many databases calls.
>Suppose a query asks for 50 conversations and each conversation’s owner. A naive resolver might first fetch 50 conversations, then fetch owner data one by one, producing 51 database calls. At scale, this is terrible. The usual fix is batching and caching within a request, often with a DataLoader pattern: collect all owner IDs, fetch all users in one query, then map results back.

GraphQL authorisation must be applied at the object and field level.
GraphQL queries are request-response.
Chat generation is incremental so use REST for LLMs as the LLM streaming APIs use SSE or WebSockets directly.

### API Versioning and Backward Compatibility
The golden rule is: additive changes are usually safe, breaking changes require a migration plan. Adding a new optional response field is safe because old clients ignore it. Adding a new required request field is breaking because old clients do not send it. Renaming a field is breaking. Changing units from seconds to milliseconds is breaking. Changing sort order is breaking. Changing error codes can be breaking if clients
depend on them.
For LLM APIs, versioning also applies to model behaviour. If model: default silently changes from a small model to a larger model, cost and latency can change. If safety filters change, outputs can change. Therefore enterprise APIs often pin model names or model aliases and expose model version metadata.

###  Authentication and Authorisation
401 vs 403
API Keys simple secrets used to identify and authenticate a application.
1. Easy to issue.
2. Revoke
3. Rotate
4. Rate-limit

**OAuth2**
OAuth2 is an authorisation framework for delegated access . It is used when a client application needs access to a resource server on behalf of a user or itself. OAuth2 separates roles:
- Resource owner: usually the user.
- Client: application requesting access.
- Authorisation server: issues tokens after authentication and consent.
- Resource server: API that accepts tokens and serves protected data.
The most important modern flow for user login is Authorisation Code with PKCE. The user is redirected to the authorisation server, authenticates there, and the client receives an authorisation code. The client exchanges the code for tokens. PKCE protects public clients, such as mobile apps and SPAs, from authorisation code interception.
It contains scopes but they are not replacement for resource level authorisation.

**JWT**
A JSON Web Token is a compact format for representing claims between parties. A JWT has three parts:header, payload, and signature.
```
base64url(header).base64url(payload).base64url(signature)
```
JWTs are often stateless: the API can validate them without looking in a database. This improves performance but makes immediate revocation harder. If a JWT is valid for one hour, it may remain usable until expiration unless you maintain a denylist or use short expiry. Therefore many systems use short-lived access tokens and refresh-token rotation.


### Authorisation Patterns
Role-Based Access Control assigns permissions through roles, such as admin.
Attribute-Based Access Control uses attributes about the user, resource, and context.
A common bug is Broken Object Level Authorization, where a user changes an ID in the
URL and accesses another user’s object.

###  Streaming Responses: SSE, WebSockets, and LLM Chat APIs
1. Server-Sent Events allow a server to push events to a browser over an HTTP connection. The browser uses the EventSource interface for GET-based SSE, and many API clients can read text/event-stream from POST responses as well. OpenAI’s streaming responses guide describes HTTP streaming over SSE for incremental model output.
   It is not truly bidirectional only good for server to client traffic.

2. WebSockets provide bidirectional communication over a persistent connection. After an HTTP upgrade handshake, both client and server can send messages anytime. This is useful for collaborative editing, multiplayer interactions, real-time dashboards, voice agents, and chat systems where the client may send incremental audio or commands while receiving output. It is complex though as you must manage connection state,heartbeats, reconnection, message ordering, authentication at connection time etc.

 Proxies may buffer responses. If buffering is enabled, the user will not see tokens until the buffer fills. For SSE, you often need headers such as Content-Type: text/event-stream, Cache-Control: no-cache, and proxy settings to disable buffering.


### Rate Limiting, Throttling, and Quotas
1. Rate limiting protects the system from overload, abuse, accidental client bugs, and unfair usage.
2. Throttling is the act of slowing or rejecting requests when limits are exceeded. 
3. Quotas are broader usage budgets, such as requests per day or tokens per month.


Common Algorithms
1. Fixed window: request in fixed intervals
2. Sliding window
3. Token Bucket for APIs: Each request consumes tokens. If enough tokens exist, the request passes. The bucket capacity allows bursts while the refill rate controls sustained traffic. AWS API Gateway throttling uses token bucket concepts for burst and steady-state behaviour
4. A leaky bucket processes requests at a fixed rate and queues or drops excess. It is useful when you want smooth downstream traffic.

### Request Validation and API Safety
Request validation means rejecting bad inputs before they reach business logic. It protects correctness, security, cost, and reliability.
Validation includes checking method, path parameters, query parameters, headers, content type, body schema, field types, enum values, maximum lengths, numeric ranges, file size, allowed MIME types, and semantic constraints. For example, a chat API should validate that messages is an array, roles are allowed values, content length is within limits, model name is permitted for the account, temperature is within range, and file IDs belong to the caller.


### Web/API Reference Architecture for an LLM Chat API
```
Browser or mobile app
-> HTTPS
API Gateway or Load Balancer
-> JWT/API key auth
-> rate limiting
-> request validation
Chat service
-> tenant/resource authorization
-> conversation/message database
-> prompt assembly service
-> retrieval service if RAG is enabled
-> model router
-> Bedrock or SageMaker endpoint
-> streaming response over SSE
Observability
-> structured logs, metrics, traces, token usage, cost metrics
Security controls
-> PII redaction, prompt-injection checks, guardrails, secrets manager, KMS
```
Request flow:
1. The client sends POST /v1/conversations/{id}/messages:stream with a JWT and idempotency key.
2. API Gateway validates basic shape and applies coarse throttling.
3. The chat service verifies the JWT, extracts user and tenant, and checks that the conversation belongs to that tenant.
4. The service validates message length, model permissions, attachment ownership, and quota.
5. It stores the user message with status accepted.
6. It assembles model context from recent messages, summaries, and retrieved documents.
7. It calls Bedrock or SageMaker and streams chunks back to the client as SSE events.
8. It stores the assistant response incrementally or after completion, depending on product needs.
9. It emits metrics: latency to first token, total latency, input/output tokens, model errors, cancellations, cost-estimate, and safety filter actions.


