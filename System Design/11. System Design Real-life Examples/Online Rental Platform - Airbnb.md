### Understanding the problem and Key Actors:
Core Concepts:
1. Listings.
2. Search/ Filters.
3. Bookings and calendar sync.
4. Payments and guest reviews.
5. Media uploads.

Key Actors:
1.  Guest.
2. Host.
3. Admin.
4. Payment Gateway.
5. External Calendar.

### Functional Requirements:
1. Account Creation and login (Host/ Guest)
2. Host can:
	1. Create and update listing.
	2. Upload Media.
	3. Set pricing and availability.
3. Guest Can:
	1. Search listing with filters.
	2. View listing details and reviews.
	3. Book listing and make payments.
4. Admins can manage users and moderate content.
5. Calendar sync with external platforms.
6. Notification System(email and push).

### Non - functional Requirements
1. High Availability: 24/7 uptime, especially during peak season.
2. Scalability: Handle millions of users and listings.
3. Security: Payment security, personal data privacy.
4. Performance: <300ms response for operations like search.
5. Reliability: Consistent booking logic, no double booking.
6. Localisation: Multiple currencies, time zones and languages.

### Assumptions and Constraints
1. Payments are handled via a 3rd party gateway.
2. Reviews are moderated by the platform.
3. Users must verify email before booking and listing.
4. Media (images/videos) is uploaded to cloud object storage.
5. Users use we and mobile apps( we need REST APIs).
6. Real-time search but bookings can have slight delays( eventual consistency for availability).

### Estimating Scale
- DAU: 5M.
- Concurrent User ~ 100k at peak.
- Listings: 50 M.
- Searches/day: DAU x 10.
- Bookings: 1M.
- Media per listing: 20/ Listing.
- Payment/day: 1M; sync with bookings.
Implications:
1. Read-heavy.
2. Frequent writes.
3. Large media storage and delivery.
4. Real-time sync needed.

### Data Size and Storage Needs
1. Rough Estimations:
	1. Listings DB: 50M listings x 5KB= 250GB
	2. Bookings: 1B records/year x 1KB = 1TB
	3. Media Storage: 10 Images x 1MB x 50M =500 TB
	4. User Profiles + History: 5M DAU x 0.5 KB = 2.5GB/day.
2. Hot Paths:
	1. Search Listings(frequent reads).
	2. Availability/ calendar updates(write-heavy).
	3. Booking confirmation (atomic write + payment + calendar update).
3. Cold Paths:
	1. Reviews history.
	2. User profile edits.
	3. Admin moderation.

### Identifying System Bottlenecks and Challenges
1. Search Service:
	1. Handles very high query volume
	2. Needs fast, filtered, geo based search.
	3. Requires scalable indexing and distributed querying.
2. Availability Calendar:
	1. Frequently updated due to bookings and sync with external calendars.
	2. Requires consistency to avoid double bookings.
	3. Timezone management adds complexity.
3. Booking System:
	1. Requires atomic operation: availability lock + payment + confirmation.
	2. Needs to prevent race conditions in peak traffic.
	3. May benefit from queues or transactions to ensure reliability.
4. Media Storage and Delivery:
	1. Huge storage needs.
	2. High bandwidth and performance requirements for Content Delivery.
	3. Needs CDN caching and object storage.
5. Payment Integration:
	1. High dependency on third party api.
	2. Must be fault-tolerant and secure (PCI compliance).
	3. Needs retry logic, logging and fallbacks.

### High level architecture Overview and Key components
1. Frontend: User facing clients for searching, booking and managing listings.
2. API Gateway: Entry point for all client requests, Handles routing, authentication, rate limiting and request shaping.
3. Core Microservice:
	1. User Service: Handling sign-up, login, profiles, preference.
	2. Listing Service: Manages property listings.
	3. Search Service: Indexes listings for geo-based, filtered search.
	4. Availability Service: Manages booking calendar and date availability.
	5. Booking Service: Handles reservation lifecycle and booking flow.
	6. Payment Service: Integrates with payment providers, tracks payments.
	7. Notification Service: Email, SMS and push notifications.
	8. Review and Ratings Service: User feedback, property ratings.
4. Support Services:
	1. Media Service: Upload and store images/videos (via S3, CDN)
	2. Calendar Sync Services: Syncs with external calendar providers.
	3. Analytics and Logging service: Tracks user events and system health.

### Service Interactions
1. User requests to book a listing.
2. Availability Service checks if dates are open.
3. Booking Service locks availability, creates a reservation draft.
4. Payment Service initiates and verifies payment.
5. Upon success, Booking Service finalises booking.
6. Notification Service sends confirmation email/SMS.

All interactions are async where possible(payment confirmation, notification) using message queues (RabbitMQ, Kafka).

### Communication Patterns an APIs
1. Sync Communication (REST/gRPC):
	1. Used for user interactions, search, listing fetch.
	2. APIs exposed via Gateway.
2. Async Communication (Message Queue/ Event Bus):
	1. Booking events(booking_created, booking_failed).
	2. Notification, calendar syncs, email dispatches.
	3. Payments webhook events.
3. Authentication and Authorisation:
	1. OAuth2/ JWT-based tokens.
	2. RBAC: guest, host, admin.

### Data Storage and Indexing Strategy
1. Primary Datastores:
	1. Relational DB (PostgreSQL/ MySQL) for transaction, user and listing data.
	2. NoSQL (MongoDB/ DynamoDB) for availability snapshots, reviews.
2. Search Index:
	1. Elastic-search for full-text + geo search on listings.
3. Caching Layers:
	1. Redis for hot data (recent searches, popular listings).
	2. CDN(Cloudflare/ Akamai) for images and static content.

### Sample DB Schema for Online Rental Platform
- Users
- Listing
- Availability
- Bookings
- Payments
- Notification
- Media

### Strategic Tech and Infra Decisions
- Scalability: Use Azure for flexible scaling and horizontal scaling across multiple servers
- Redundancy: Leverage load balancers and multi-region DB replication for redundancy and availability.
-  Data Storage: Store transactional data in SQL and media assets in Azure Blob Storage for durability and cost-effectiveness.
- Asynchronous Communication: Use RabbitMQ of Kafka for async tasks like payments and notification to decouple services.
- Caching: Integrate Redis for caching frequently accessed data to improve response  time and reduce load.
- Micro-service Architecture: Adopt micro-services to scale and maintain services(booking, payment, availability) independently.
- Monitoring and Logging: Use Azure Monitor for health tracking and centralised logging (ElasticSearch) for real-time diagnostics.

### Final Design 
![[Pasted image 20260904142957.png]]