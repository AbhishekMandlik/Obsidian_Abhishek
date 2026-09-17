### Functional Requirements
Real-time chat application:
1. 1-1 and Group to group messaging.
2. Typing Indicators and online Status.
3. Media Support(Images, videos, documents).
4. Delivery and read receipts.
5. Multi-device support with sync.

### Non Functional Requirements
1. Sub-second message delivery
2. High availability and fault tolerance.
3. End to end encryption.
4. Scalable to millions of users.
5. Smooth UX even on poor networks.

### Constraints and Challenges
1. Message Ordering and Delivery Guarantees.
	1. Ensure No duplicates no lost messages and correct order even under retries or reconnections
2. Unreliable Networks
	1. Mobile users may frequently disconnect; we need robust retry, buffering and sync mechanisms,
3. Security and Privacy
	1. End to end encryption must be seamless, with no access to message content by servers.
4. Latency and Expectations:
	1. Sub-second message delivery is expected, regardless of load or geography.
5. Presence and sync:
	1. Track who's online and sync message state across all devices in near real-time.
6. High Scale:
	1. Millions of concurrent users, chatrooms, and messages per second; we must scale horizontally and efficiently.


### Estimating Scale
1. DAU: 100M
2. Average messages/user.day:50
3. Messages/day: 5B/day
4. Peak Traffic multiplier:3x during events.
5. Concurrent: 20-30M online at peak.

### Identifying System Bottlenecks and Challenges
1. Message Ingestion and Fanout
	1. Fanout to multiple recipients or devices causes exponential delivery load.
	2. Must queue, buffer and batch smartly.
2. Presence Updates
	1. Real-time online/offline tracking is chatty and high frequency.
	2. Use Pub/Sub patterns and efficient TTL-based caching
3. Delivery Acknowledgements
	1. Each sent.read receipt adds more writes -- can overload DBs.
	2. Needs fast, write optimised storage like Cassandra on DynamoDB.
4. Sync Across Devices
	1. Every message must sync across all logged-in devices.
	2. Requires a device registry and smart deduplication.
5. Storage and Retrieval
	1. Billions of messages stored with search capability.
	2. Must partition smartly and use append-friendly stores.
6. Encryption and Security
	1. End-to-end encryption adds CPU cost and complexity.
	2. Key exchange and secure metadata handling must scale too.

### Key Bottleneck: Real-Time Delivery (WebSockets)
Real-time delivery is fundamental to chat system -- users expect messages to appear instantly. This means:
- Persistent connections need to be maintained for millions of users using websockets.
- Load Balancers and application servers must handle long-lived connections.
- Need for connection management service: track connected users, devices and routing.
- Risk: connection churn, network drops, or mobile limitation(background app states).
  This means when you switch from wifi to mobile data, closing the app or keeping it on standby etc.
- It is bottleneck because
	- Handling millions of concurrent WebSockets connections requires optimised infra, horizontal scaling and careful connection lifecycle management.

### High-Level Architecture Overview and Key components
1. Connection Manager (WebSocket Service):
	1. To manage persistent WebSocket connections, delivers real-time messages and tracks user/device sessions.
2. Chat Service:
	1. Core Business logic for sending receiving, storing messages and managing delivery status and history.
3. Present Service:
	1. Tracks online/offline status, typing indicators and syncs across user devices using pub-sub or Redis.
4. Notification Service:
	1. Handles fallback notification when recipients are offline or WebSocket delivery fails.(e.g. Push, SMS)
5. Media Service:
	1. Manages upload and retrieval of Media files like images, videos and attachments using object storage.
6. Auth and User Service
7. Storage Layer:
	1. Backup
	2. High write throughput
8. Group Service:
	1. Group Management
	2. Membership management.

### Real-Time Connection Manager (Websocket Service)
- Maintains long-lived WebSockets connections per user and device.
- Handles message routing between senders and recipients.
- Integrates with Presence and Chat services for sync and state management.
- Supports scale-out using sticky sessions, Redis pub-sub, or message queues.
- Ensures delivery acknowledgements, retries and device-level delivery tracking.

### DB Schema for Direct Chat
1. message_id: int
2. sender_id: int
3. message_content: string
4. timestamp: datetime
5. status: index
6. receiver_id: int
7. message_type: string
8. indexing done on sender, receiver, timestamp
9. In group chat add group_id: int and
10. group_name: string
11. group_members: list of integers

### How Chat and Group Chat Services Use Connections Manager for Real-Time Messaging?
1. Direct Chat
	1. Sender Sends a message to the Chat service with recipients user ID.
	2. The Chat Service then sends the message to connection Manger.
	3. Connection Manager looks up the recipients active WebSocket connection and forwards the message to the corresponding WebSocket server.
	4. Receiver receives the message in real-time.
2. Group Chat
	1. Sender sends a message to the group chat service with groupID, message.
	2. GC service queries connection manager, to look at all WebSocket connections for users, via grp table.
	3. It forwards message to all group members active WebSocket connections
	4. They receive message simultaneously

### Sequence Diagram
1. User request a WebSocket connection to API Gateway.
2. The Request is routed to the Connection Manager, It creates and maintains a persistent connection for that user.
3. From that point Device can send and receive messages in real-time.
4. Now it reaches the Chat service which is responsible for core messaging workflow.
5. This service validates the request, parses the message in storage layer and prepares it for delivery.
6. Present service determines whether those recipients are currently online or not.
7. This will determine the most appropriate delivery path.
8. If WebSocket connection is active then chat works normally, else a push notification is send which tell user a message is waiting for him.
9. Media and Videos are transferred through upload and download method.
10. Together these services create a specialised chat service.

### API Design - WebSocket + Rest
1. WebSocket communication (Real-time)
	1. Connect: Initiates persistent connection with access token and user metadata.
	2. send_message: JSON payload with recipient ID, message content, timestamp, type.
	3. message_ack:Acknowledgement from client.
	4. typing indicator: Optional message indicating typing state.
2. REST APIs (Non- Real time needs)
	1. GET/ messages?userID=&conversationId: Fetch historical messages.
	2. POST /media/upload_image or video or audio
	3. GET/presence/{userId}: online status for a user.
	4. POST /feedback: Submit crash report or feedback {optional}.
3. Security:
	1. All requests {WebSocket handshake +REST} require JWT auth.
	2. Role-based access for user vs. admin endpoints.

### Strategic Tech and Infra Decisions
- Tech Stack Choices:
	- Message Broker:Kafka (high-throughput) or AWS SQS(managed).
	- Real-time Communication: Web Sockets for instant messaging, leveraging SignalR (for.NET) or similar framework.
	- Notification System: Firebase Cloud Messaging (FCM) for push notifications.
	- DB: NoSQL DBs (e.g. MongoDB) for fast reads and flexible schema, Postgress for relational DB like Payments that is integrated.
- Infrastructure & Deployment:
	- Deployment: Microservice on AKS for scalability and management.
	- Scaling: Horizontal scaling for WebSocket connections; KEDA(Kubernetes Event-Driven Autoscaling).
	- Storage: Redis for caching active user sessions, PostgreSQL for transactional data. (user settings)
- Security and Observability:
	- Authentication
	- Logging and Monitoring: Prometheus, Grafana. Cloud-Watch for log alerts.
	- High availability: Multi-region Deployment for failover; load balancing across Websocket servers

### Final Design
![[Pasted image 20260903165811.png]]

