### What is it?
It enables uses to list items and bid in real-time within a defined time window. Auction promote competitive pricing and urgency -- highest bid wins. 
The platform handles item listings, bid tracking, real-time updates, auctions lifecycle and payment processing. Key objective is to deliver a secure fair and scalable real-time bidding experience from listing to payment.
### Functional  Requirements
1. User Registration and authentication.
2. Item listing with action params.
3. Real-time bid placement and updates.
4. Auctions state transitions: scheduled -> active -> ended.
5. Payment processing after auction ends.
6. Notification: outbids, auction won, payment pending/complete.

### Key actors and Use Cases
1. Actors:
	1. Seller: List items, sets pricing rules.
	2. Bidder: Places bid, get real-time notifications.
	3. System: handles auction logic , enforces timing, manages payments.
	4. Admin: Monitor platform activity and handles fraud or disputes.
2. Use Cases:
	1. System updates UI and everything in real-time.
	2. Winner bidder completes payment -> seller is notified

### Non-Functional Requirements:
1. Performance:
	1. Handle bids submitted within milliseconds of closing -- resolve ties and order events accurately.
2. Scalability:
	1. Support thousands of concurrent auctions and real-time users.
3. Security:
	1. Secure user authentication, payment data protection anti bot protections.
4. Availability:
	1. Ensure high availability during peak auction time and payment flow.
5. Observability
	1. Logging of auction and payment events, real-time monitoring dashboards.

### Constraints and Challenges
1. Payments Workflow:
	1. Payment failures, timeouts, retries and fraud detection must be handled.
2. Fairness and Trust:
	1. Prevent bots/ snipping and ensure fair auction rules are enforced transparently.
3. Real-Time Pressure:
	1. Handle bids submitted within ms of closing - resolve ties and order events accurately.
4. Concurrency Conflicts: 
	1. Simultaneous bid updates must be conflict-free and idempotent.
5. Live Updates:
	1. Efficient, scalable real -time delivery to all watchers of an auction( WebSocket or pub/sub)

### Estimating Scale:
- Assumed Metrics:
	- 500k DAU.
	- Active Listing: 1M.
	- Bids: Average 10 bids/auction -> 10M/day.
	- Peak Activity: 10K concurrent users bidding/viewing one popular auctions.
	- Payments 100K payment transaction/day.

### Traffic Patterns and Real-Time Pressure Points
1. Read- Heavy Operation:
	1. Viewing auction listing and item details.
	2. Real-time bid updates to watchers.
	3. Browsing user profiles, search filters.
	4. Scale: Millions of read per minute at peak.
2. Write Sensitive Operations:
	1. Bid placement(low latency, strong consistency).
	2. Creating new auction listings.
	3. Finalising auctions and triggering payments.
	4. Write volume is lower but time-critical.
3. Real-Time Pressure Zones:
	1. Last-minute bidding: thousands in final seconds.
	2. Real-time fan out: pushing updates to thousands of watchers.
	3. Closing auctions precisely.
	4. Payment triggers: time sensitive + 3rd party API dependent.

System must scale for read but remain fast and correct for writes under pressure.

### Identifying System Bottlenecks and Challenges:
1. Estimate per-service load to prevent surprise bottlenecks.
2. Isolate real-time components for better scalability.
3. Use async processing where consistency can be relaxed.
4. Used horizontal scaling + partitioning for high volume data.
5. Plan for hot auctions -- design for skewed load.


### High - Level Architecture Overview and Key Components
1. API Gateway - Entry Point, rate limit, routing.
2. User Service- Auth, profile, roles (buyer/seller).
3. Auction Service - Manages auction lifecycle, bid validation.
4. Bid Service - Real-time bid management concurrency control.
5. Payment Service- Triggers post-auction payments, payment status tracking.
6. Notification services.
7. Schedular Service.
8. Analytics and logging for monitoring.

### API Design - Key Endpoints
1. User APIs: 
	1. POST /signup
	2. POST /login
	3. GET /user/profile
2. Auction APIs:
	1. POST /auctions:
	2. GET /auctions/{id}:
	3. POST /auctions/{id}/bids:
	4. GET /auctions/{id}/bids:
	5. GET /auctions/active.
3. Payment APIs:
	1. POST /payments/initiate.
	2. GET /payments/{id}/status.
4. Notification Triggers(Internal):
	1. Auction ending -> notify winner.
	2. New highest bid -> notify previous top bidder.

### Service-to-Service Communication
1. Patterns Used:
	1. Sync (REST /gRPC):
		1. User auth, listing fetch.
		2. Auction -> Bid -> Payment trigger.
	2. Async (Pub/Sub or Event Bus):
		1. New bid placed -> broadcast to watchers.
		2. Auction ended -> notify winner + trigger payment
		3. failed payment -> retry/ alert
2. Sample Event Topics:
	1. auction.ended.
	2. bid.placed.
	3. payment.failed.
	4. user.registered.
Benefits: Loosely coupled systems, retries, better failure handling.

### Real-time Bid Delivery (WebSockets)
1. Why Web-Sockets?
	1. Low- Latency, full duplex communication.
	2. Push updated to all watchers instantly.
2. How those work:
	1. Client subscribes to auction channel: auction:{id}
	2. Bid service validated and accepts bid -> publishes event
	3. WebSocket server fans out update to all connected clients.
	4. Key Concern: Horizontal scale under high concurrency.

We will need multiple WebSocket server for this which will have to work together while delivering the same event to all the clients.
This will be more of an event driven architecture and every event will trigger the new function, like notification, payment, ending or registration.
### Data Model
1. User: Purpose: Stories information about user (buyers/ sellers)
2. Listing: Purpose: Contains details about the items being auctioned.
3. Auction: Represents an auction for a specific listing, including its start and end.
4. Bid: Stores bids placed on auction by users.
5. Payment: Tracks payments after auction completion.

### Handling Auction Timers and Closures - Scheduled Job System for Auction Lifecycle
1. Responsibilities:
	1. Start auction at start_time.
	2. End at end_time.
	3. Determine winner, notify parties, trigger payment.
2. Design Options:
	1. Dedicated Schedular service(cron, queue).
	2. User Delayed Jobs via message queue (SQS delay, kafka, BullMQ).
	3. Store time in Redis + polling/expiration-based mechanism.
3. Reliability Measures:
	1. Retry failed closures.
	2. Idempotent end-of auction logic.
	3. Logging + alerts on missed timers.

### Strategic Tech and Infra Decisions:
1. Technology Stack for Scalability:
	1. FrontendL React or Vue for Responsive, real-time UI.
	2. Backend: Node.js or Java with Spring Boot for handling API requests efficiently.
	3. Database: PostgreSQL for relational data, Redis for caching high demand data like auction info.
2. Ensuring High Availability:
	1. Load Balancing; (WebSockets)
	2. Real-time Updates: (WebSockets)
	3. Caching: Redis:
3. Security Considerations:
	1. Authentication: (OAuth2)
	2. Data Encryption: SSL/TLS for encrypting data in transit, ensuring secure transactions.
4. Cost Optimisation:
	1. Cloud Services: Optimise infra costs.
	2. Autoscaling: Autoscaling to manage traffic surges.


### Final Design - Auction Platform
![[Pasted image 20260904124410.png]]