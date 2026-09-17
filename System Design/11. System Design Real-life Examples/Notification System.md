## Understanding the Problem and Defining the scope.
We need to build a system that delivers messages to users across multiple channels in reliable, scalable and user-respectful way.
The system should support: Event-triggered notifications, Multiple Delivery channels, user level notification preferences, Template-based, translatable messages. Retry logic and delivery guarantees.
### Functional Requirements:
1. Accept event from multiple upstream sources(order service, authentication service).
2. Trigger one or more notifications per event.
3. Support for delivery channels: Emails, SMS, Push, In-App.
4. Store and enforce user preferences for notification types and channels.
5. Template-based message generation with localisation support.
6. Implement retry logic and dead-letter handling for failed messages.
7. Expose APIs for sending custom notifications and managing preferences.

### Non-Function Requirements:
1. Scalability: Handle millions of notifications/dat, burst traffic at peak.
2. Reliability: Guarantee at-least-once delivery, avoid duplicates.
3. Low Latency: Notification should be near real-time (<few seconds).
4. Security: Encrypt sensitive data, secure APIs, role-based access.
5. Extensibility: Add new channels and templates with minimal changes.
6. Observability: Logs, metrics, traceability for audits and debugging.
7. Idempotency for safely retrying failed deliveries.
They are about making product dependable in the Productions.

### Constraints and Challenges
1. Channel Limitations:
	1. SMS/email providers have rate limits and SLAs.
	2. Push delivery can be unreliable (app not installed, permissions off).
2. Event Burst Handling:
	1. Spikes during flash sales, releases or system-wide events.
	2. Queue hours and regional compliance(GDPR, DND).
3. Latency Expectation:
	1. Users expect instant feedback -- latency must be low but not at the cost of reliability.
4. Retry and Idempotency:
	1. Retries can cause duplicate messages if not handled idempotently.
	2. Failed external provider calls must not block the whole system.
5. Security and Privacy:
	1. PII like emails and phone numbers must be securely stored and transmitted.
	2. Must log activity without leaking sensitive content.

### Estimating Scale and Identifying Bottlenecks

## Bottleneck Identification:
- Daily Load Estimates:
	- 25M total users | 10M DAU.
	- Average events/user/day:5 
- Notification fan-out per event: 2 channels(Email+push).
- Total notifications=100M/day
- Peak traffic multiplier: 3x (flash sales, incidents).
### More Bottlenecks
1. Event Ingestion: High volume of incoming events -> need rate-limiting, buffering(Kafka, SQS etc)
2. Template Rendering: CPU-heavy if synchronous; use caching or pre-rendering.
3. External Provider APIs: Latency and rate-limited --risk of throttling and timeouts.
4. User Preference Lookup: High QPS reads; might need caching layer(e.g. Redis).
5. Monitoring and Logging: High cardinality data -> risk of overwhelming observability stack.
6. Delivery Channel Characteristics:
	1. SMS is most expensive and regulated.
	2. SMS cost spike drastically with volume.
	3. Push depends on mobile infra, Firebase, APNs.
	4. In-App can be fast but assumes user is online in the app.
	5. Cloud provider API call (SES, Twilio, Firebase).
	6. High-availability infra(queues, load balancers, workers)
## High- Level Design: Services, APIs and Communication

### Key Components:
1. Event Ingestor: 
	1. Captures events from upstream
	2. User rate-limiting and buffering(Kafka, SQS) to handle high volume.
2. Notification Orchestrator:
	1. Decides which notifications to trigger based on event and user preferences.
	2. Co-ordinates with preference service and Template Service
3. Preference Service:
	1. Stores user notification preferences(channels, event types, quiet hours).
	2. Uses Caching for fast access.
4. Template Service:
	1. Generates localised messages based on event data.
	2. Caches templates for performance.
5. Channel Workers(Email, SMS, Push, In-App):
	1. Handles delivery via dedicated workers for each channel.
	2. Manages retries and failures.
6. Delivery Tracker/ Dead-letter Queue:
	1. Tracks delivery status and handles failed messages.
	2. Uses dead-letter queue for. undelivered messages after retries.

### Communication Flow(Diagram)
```
Event Source -> Event Ingestor -> Orchestrator -> Preference service + Template Service -> Channel Queues -> Channel Workers -> Provider -> Delivery Tracker.
```

### API Design (High Level) - Sample API Contracts:
- User APIs:
	- GET /notification?userId={id}
	- POST /notifications/read.
- Admin APIs:
	- POST /notification
	- GET /delivery-report?eventId={id}
- Security and Access Control:
	- All APIs secured with JWT or OAuth2.
	- Admin APIs require elevated role-based access.
	- Rate limiting and API gateway integration for protection.

## Making Tech and Infra Decisions
1. Tech Stack Choices
	1. Message Broker: Kafka (hight-throughput) or Amazon SQS (managed service)
	2. Template Rendering: Handlebars or Liquid templating engine.
	3. Notification Channels: Sendgrid (Email), Twilio(SMS), Firebase FCM (Push).
2. Infra and Deployment
	1. Deployment: Microservices on Kubernetes or AWS Lambda(serverless)
	2. Scaling: Horizontal Pod Autoscaling or Keda for scaling workers.
	3. Persistence: PostgreSQL for preferences, S3 for template storage.
3. Security and Observability
	1. Security: JWT auth, RBAC for admin APIs.
	2. Observability: Prometheus + Grafana for monitoring, Cloudwatch for logs.

## The Final Design 
![[Pasted image 20260902135511.png]]