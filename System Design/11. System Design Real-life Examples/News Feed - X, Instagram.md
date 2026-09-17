## Understanding the Problem and Defining the scope.
### Functional Requirements
1. Post a tweet/update.
2. Follow/unfollows users.
3. View home timeline.
4. Timeline must show updates from the followed users as soon as they are uploaded.
5. Likes, replies, retweets.
6. Media uploads (images, videos)

### Non-Function Requirements
1. Timeline Expectations:
	1. Feeds load instantly.
	2. New posts appear in real-time.
	3. Feed is relevant and fresh.
	4. Posts are not missing or duplicated.
2. High Availability
3. Low Latency
4. Scalability

### Constraints and Challenges
1. Fan-In / Fan-Out Challenge
	1. 1 user -> 1 tweet -> must reach 1 -> followers?
	2. Should we push to all followers? Or pull on demand.
2. Fan-out on write vs. Fan-out on read -- this will shape our entire design. As both of them cause different problems. If we post if directly to all the users then user enjoy a much better timeline, downside is that posting becomes very expensive. Else posting will become cheap but will increase latency for every user as it will pull information when user requests for it.
### Estimating Scale and Identifying Bottlenecks
1. Estimated User Traffic:
	1. Daily Active User(DAU): 10 Million
	2. Monthly (MAU): 300 M.
	3. New Short URLs/day: 1% of DAU (100k).
	4. Redirect Requests/day: 50M(5/user)
2. Memory Requirement:
	1. Cache top 1M most accessed URLs.
	2. Each Mapping: 500B.
	3. Total memory: 500MB.
3. Network Bandwidth:
	1. 50M Redirects/day  x 700B = 35GB/day
	2. Avg. throughput: 0.4MB/sec
	3. Peak throughput: 5 MB/sec
4. Storage Requirement (URL Mapping DB):
	1. 100K new URLs/day x 500B = 50MB/day.
	2. Yearly data = 18GB raw + overhead
	3. Round up to 50 GB/year (indexes, logs, backups)

## Bottleneck Identification:
- Daily Load Estimates:
	- 500M total users | 200M DAU.
	- 1B tweets/day | 300M media uploads.
	- 1B likes, 500M replies, 250M retweets.
	- 2B+ feed requests/day (10 opens/user)
- Read> Write: 80% traffic is read-heavy
- Media Storage and Delivery:
	- 300M media uploads/day = 100-500TB/day
	- Must support:
		- Upload APIs.
		- Object Storage(S3, GCS).
		- CDN-backed delivery.
		- Tweet --> Media linking.
### More Bottlenecks
1. Timeline=Fan-out at Scale
	1. 1 user tweets ->needs to be visible to 1M followers.
	2. Fan-out Models:
		1. on write: Pre-computed timelines(fast-reads, heavy writes).
		2. Fan-out on read: Compose timeline at read (slower reads, lighter writes).
2. Hot User can trigger write storms.
	1. Read and Write amplification
		1. Write-Intensive Actions:
			1. Posting a tweet: May fan-out to 100s or millions of timelines.
			2. Re-tweets, replies, likes: Update multiple engagement counters and visibility.
			3. Media uploads: Chunked writes, storage and metadata association.
	2. Read-Heavy Patterns:
		1. Opening the timeline: Aggregates and sorts the tweets from 100s of followed users.
		2. Viewing a tweet: Triggers fetches for media, replies likes and retweets.
		3. Scrolling: Causes pagination, cache lookups and lazy loads.
One user actions can trigger multiple backend reads/writes across services.
## High- Level Design: Services, APIs and Communication
### Core Services Breakdown
- User Service - Profile, follow/unfollow.
- Tweet Service - Create, retrieve tweets.
- Timeline Service - Compose timelines for users.
- Engagement Service - Likes, replies, re-tweets.
- Media Service - Upload, store, fetch media.
- Notification Service- Fan-out alerts, activity.
- Fanout Worker - Background timeline propagation.

### High-Level Architecture
User Flow:
Mobile App -> API Gateway -> Microservices -> DBs, Queues, Storage
API Gateway for request routing, Auth, rate limiting, Aggregating service calls.

### API Design (High Level) - Sample API Contracts:
- POST /tweet
	-  Create new tweet (text + media references).
- GET /timeline: Building this is the biggest challenge in such systems.
	- Returns latest tweets for a user.
- POST /follow or /unfollow
	- Modify following relationships.
- POST /like. /retweet. /reply
	- Register engagement events.
- POST /media. /upload. 
	- Upload Media, return CDN URL
ALL APIs are stateless; session/auth handled via token headers.

### Timeline Generation Strategy
1. Fan-out on Write:
	1. Pre-compute timeline when a user tweets.
	2. Fast reads, heavy write for hot users.
2. Fan-out Read
	1. Compose timeline on demand from followed users.
	2. Lighter writes slower reads.
3. Hybrid Model = best of both:
	1. Fan-out to regular users.
	2. Read-time fetch for hot-users.

### Sync vs Async Communication
1. Synchronous (API Calls):
	1. Fetch User timeline.
	2. Get tweet details.
	3. Submit engagement.
2. Asynchronous (Event Queues):
	1. New Tweet -> Enqueue fan-out jobs.
	2. Media uploads -> process and link.
	3. Engagement ->push notification trigger.
Improves Throughput because system spends less time handling each user request, and decouples latency-sensitive paths: The Components accepting the request does not need to wait for every downstream task to finish before responding to the user.

## Making Tech and Infra Decisions
1. Guiding Principles:
	1. Optimise for read-heavy load, real-time delivery and massive fan-out.
	2. Prioritise Latency, scalability and operational simplicity.
2. Key Tech Choices
	1. Storage and DBs:
		1. Tweets, users, timeline -> Scalable NoSQL (Cassandra, DynamoDB).
		2. Engagements-> Relational or Key-Value Store (PostgreSQL, Redis).
		3. Media -> Object Storage (S3, GCS) +CDN(Cloudflare, Akamai).
	2. Async Jobs:
		1. Kafka/RabbitMQ for Fan-out jobs engagement events, media pipelines.
		2. Caching
		3. Redis/ Memcached for hot timelines, tweets and user sessions.
	3. Compute and Infra:
		1. Kubernetes/ECS for autoscaling microservices.
		2. API Gateway / Envoy for routing, rate-limiting, auth.
	4. Monitoring and Resilience:
		1. Prometheus + Grafana for metrics + alerts.
		2. Circuit Breakers, retries, queues for graceful degradation.
Every choice is made to handle massive volume, reduce latency and isolate blast radius of failures.
## The Final Design 
![[Pasted image 20260901164134.png]]