## What is Ticketing System? - Understanding the problem and defining the scope.
It is an online platform that allows users to browse, book and manage tickets for events such as concerts, sports and travel. It must handle real-time seat availability, secure transactions and high concurrency. The spikes in traffic when a popular concert/ movies is about to go live should also be handled gracefully while it can gracefully shut down some other services temporarily.
### Functional Requirements
Users should be able to:
1. Browse events and view available seats.
2. Book/reserve tickets.
3. Get real-time seat availability.
4. Pay for tickets securely.
5. Receive email/SMS confirmations.

Admins should be able to:
1. Create/manage events and venues.
2. Define seat layouts and pricing.

This has got two different functions unlike before and clear boundaries and permissions must be set in order to do so.

### Non-Functional Requirements
- High Availability (no downtime during peak sales)
- Low Latency (booking response in milliseconds)
- Scalability (Handle flash sales, global traffic)
- Data consistency (Avoid double booking of the same seat)
- Audit logs( For tracking transactions and fraud preventions)
CAP Theorem: Consistency will be affected as Availability will high

### Constraints and Challenges
- 5M total users, 100k concurrent users at peak.
- Global event organisers (multi-region support).
- Handling payment failures (release locked seats quickly).
Ticketing system: seat timeout, payment failures, high concurrency and global usage must be handled carefully.
## Estimating Scale and Identifying Bottlenecks
1. User Load Assumptions:
	1. 1M DAU.
	2. 100k Concurrent Users during peak events.
	3. Each user browses ~10 events/day -> 10M read requests/day.
	   This is a read heavy business.
2. Booking Traffic estimations:
	1. ~500K bookings/day.
	2. Average: ~6 bookings/sec.
	3. Peak Load: up to 2000 bookings/sec.
	   This will have write heavy during peak seasons.

### Identifying System Bottlenecks
1. Concurrency in Seat allocation:
	1. Race conditions while multiple users book the same seat.
2. DB Write Pressure:
	1. Sudden spike in bookings can overwhelm the write DB
3. Payment and External API Latency
	1. Delay or Failures in third-party APIs can block seat availability.
4. Notification Backlogs
	1. Email/SMS confirmation systems can queue up during spikes.

## High Level Design: Services, APIs and Communication
### Core Components
1. Frontend Clients: Web and Mobile App.
2. API Gateways: Unified entry point for routing and authentications.
3. Authentication Service.
4. Admin Portal: Event Creation, venue setup, pricing.
### Backend Services
1. Event Management Service: Manage events, venues and seat layouts.
2. Seat inventory Service: Track available/ locked/ booked seats.
3. Booking Service: Handles bookings, locking seats, confirming payments.
4. Payment Service: Integrates with payment gateway, handles retries.
5. Notification Service: Sends booking confirmations via Email/SMS.
All will be event driven and not highly coupled. Distributed system with Each service having it's own separate work.
### Data and Caching Architecture.
1. Relational DB (PostgreSQL/MySQL): For transactions, bookings, users. As this is  related to each other.
2. NoSQL DB (MongoDB/ DocumentDB): For events and seat layouts. This will be changing dynamically so NoSQL will be prefered.
3. Caching Layer (Redis/ Memcached): Real-time seat availability.
4. Queue System(Kafka/ RabbitMQ): Async handling for:
	1. Emails.
	2. Payment retries.
	3. Audit logs.
### Notable Design Decisions
1. Concurrency Control: Use Optimistic Locking(version check): Here we assume that the double booking is rare hence until booking is done we assume that the seat is not booked, if the version of the seat has changed we can simply tell booking done please try again later or Pessimistic Locking (seat-level) lock, we lock the seats for some time and if booking is not done we unlock those after some time. 
2. Seat Hold Timeout Logic: Redis/ Memcached-backed TTL-based lock -> auto releases after 5 minutes.
3. CQRS Pattern: Split reads (seat availability, listing) from writes (bookings).
4. Idempotency keys for Payments: Prevents duplicates charges and ensures safe retries. Without protection duplicate charges may occur and reduce reliability.

### API Design
1. Event Management Service:
	1. Resource: /events
	2. POST /events: Create a new event.
		1. Request Body: { Details of the event}
		2. Response Body: { Success and one ID generated}
	3. GET /events: Get details of all events:
		1. Response body:{ 200 OK, id:"...", name:"..."}.
	4. GET/events/{eventID} Get details of specific Event.
		1. Request Body: {Details of event}
		2. Response body:{ 200 OK, id:"...", name:"..."}.
2. Booking Service:
	1. Resource: /bookings
	2. POST /bookings: Create a new booking for one event.
	3. GET /bookings: Get a list of all booking for particular person.
	4. GET /bookings/{bookingID}: Get information for a particular bookingID.

## Making Tech and Infra Decisions Strategically
### Strategic Tech and Infra Decisions:
1. API Gateways: Use NGINX for self-managed AWS API gateway for serverless routing and also for rate-limiting.
2. Authentication: Implement OAuth 2.0 with JWT tokens for secure, stateless auth.
3. Booking DB: Choose PostgreSQL for strong consistency and transactional support.
4. Event and Venue Data: Use MongoDB or Elastic search for flexible, schema-less search and filtering.
5. Caching Layer: Integrate Redis for fast access to seat availability and temporary locks.
6. Async Messaging: Adopt Kafka for high-throughput event streaming and async workflows(notification, logging).
7. Payment Gateway: Integrate with stripe or Razorpay, ensuring support for retries and webhooks.
8. Notification: Use AWS SES for email and Twilio for SMS confirmations.
9. Infrastructure: Deploy on Kubernetes with auto-scaling groups for elasticity and resilience.
10. Monitoring: Leverage Prometheus and Grafana for metrics and real-time dashboards.
11. Logging: Use the ELK Stack (Elasticsearch, Logstash, Kibana) for centralised log aggregation and search.
## The Final Design - Ticketing System
![[Pasted image 20260901152318.png]]