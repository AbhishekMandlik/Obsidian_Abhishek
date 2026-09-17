## Understanding the problem and Defining the Scope
Convert Long URLs into short, unique links for easy tracking and sharing.
Why is it useful?
- Improved UX: Easy to share, clean and readable.
- Click Tracking: Analytics for link performance.
- Custom Branding.
- Cross Channel Friendly.
- Compact links for platforms like Twitter and SMS.
It works when you submit long url the system generates a unique short key, stores mapping in DB and redirects short URL to original.

### Functional Requirements for TinyURL
1. Shorten URL: Accept a valid long URL and return a shortened URL.
2. Redirect to Original URL: When accessing the short URL is submitted, redirect to the original long URL.
3. Prevent Duplicate Short URLs: If the same long URL is submitted, return the same short URL or handle according to configuration(unless custom alias is used).
4. User Authentication: Allow users to register/login to manage URLs, view analytics and set expiration.
Questions like how we generate unique keys, store mappings efficiently or handle millions of redirect requests are architectural decisions that we will explore in the upcoming slides

### Non-Functional Requirements for TinyURL.
- High Availability: System must be available 24/7 with 99.9% uptime.
- Performance and low latency: URL redirection should occur in milliseconds. Shortening URLs should be near instantaneous.
- Scalability: System must handle millions or billions of URLs supporting high read volume (redirects) and moderate write volume (URL shortening).
- Reliability: Ensure data persistence and no data loss even during failures using durable storage and backups.

### Unique URL Generation Strategies:
1. Random string Generation: Creates a fixes-length from random characters:
	1. Unpredictable, no obvious pattern.
	2. Risk of collisions.
	3. Collisions add complexity.
2. UUID (Universally Unique Identifier: 128-bit globally uniques identifier (e.g. 123e4-..)
	1. Guaranteed uniqueness, no central co-ordination.
	2. Very Long, not user friendly.
	3. Not ideal for TinyURL due to length.
3. Hashing with Salt: Hashes the original URL(e.g., SHA-256 + salt)
	1. Unique, secure, hard to reverse.
	2. May not be short, collision possible, needs mapping storage.
	3. Useful for security-focussed cases, but not optimal for shortening.
4. Base62 Encoding: Converts incrementing ID to Base62(0-9,a-z,A-Z)
	1. Short, Compact, Deterministic, easy to implement.
	2. Needs counter management to avoid collisions.
	3. Recommended for TinyURL (fast, scalable).


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
- High read volume -> Focus more on cache and fast DB reads: As we thought earlier our redirect requests outnumber the new creation requests vastly.
- Write throughput is moderate -> Ensure consistency. Even though new creation is very less we still need mapping between the short URLs and the bigger ones and it must remain accurate. As system grows we should ensure that the written URLs are valid and reliable.
- Latency sensitivity in redirects -> Low-Latency infra needed. Redirection should be very fast. Even additional millli-second increases the perceived delay.
- Plan for burst traffic with auto scaling and CDN support. Occasionally some links may go viral and attract millions of users for very short period of time. We will rely on autoscaling to add capacity during periods of high demand and use a CDN to efficiently serve requests from locations that are closer to the user.
None of these bottle necks are independent. High Rate traffic leads towards caching, latency requirements and influence our infrastructure. choices. Burst traffic pushes us towards elastic scaling.

## High- Level Design: Services, APIs and Communication
### API Design - Create Short URL
- Accepts a long URL and returns a shortened URL
- Endpoint(Follows Rest principles): POST /api/shorten
- Request (JSON)
  ```
	  {
		"long_url":"https://www.example.com/..."
		"Name":"Abhishek Mandlik"
		"Date":"26/08/26"
	  }
  ```
- Response (JSON):
  ```
	  {
		  "short_url":"https://tinyurl.com/my-alias"
	  }
  ```

### API Design - Redirect
- Redirect to Original URL
- Endpoint: GET/:short_key
- Behaviour:
	- Looks up the original long URL using the short key.
	- Returns a 302 HTTP Redirect to the long URL.
>Here Re-direction is a design choice. Although this endpoint is simple this will be executed millions of time everyday. So low-latency is critical.

### API Design - Delete
- Delete a Short URL (Optional)
- Endpoint: Delete /api/url/:short_key
- Behaviour:
	- Deleted the mapping if user is authenticated and owns the URL.
	- Requires a bearer token authorisation.
	- We can perform a soft-delete feature as well. So instead of Deleting many URLs at once We can have A Cron job set in the back that will clear the URLs as and when the Cron API endpoint is hit or after a certain time.
### User Authentication APIs
- User Registration
- Endpoint: POST /api/auth/register.
	- We will Request email, username, password.
	- Response will be message "Registration Complete".
- User Login
- Endpoint: POST /api/auth/login
	- We will request username, password
	- We will return in response: access_token, token_type, expiry date. Which on the webpage we can store in HTTP bearer which the user will be able to access without him knowing the token
- Secure Endpoints with Bearer Token

### High-Level System Design - Overview
- API- Gateway: Entry point for all clients; handles routing, rate limiting, auth.
- URL Shortener Service: Contains logic for key generation, duplicate checking, alias validation.
- Redirect Service: High- Performance resolver for short keys -> long URLs. Optimised for extremely fast look-ups and minimal latency.
- Database: Persistent store for all mappings, users, metadata.
- Cache Layer: Redis/ Memcached for top N frequently accessed URLs.
- Auth Service: Manages user login, JWT tokens and sessions.

### Collision Handling in Distributed URL Generation - Hello Zookeeper
1. Why Collisions happen?
	1. Multiple Service instance generating IDs independently -> Risk of duplicates.
	2. No Global co-ordination service by Apache.
2. What is Zoo-Keeper?
	1. Distributed co-ordination service by Apache.
	2. Ensures synchronisation across nodes in a distributed system.
3. Zookeeper as a solution?
	1. Atomic ID generation using Zookeeper-managed global counter.
	2. Guarantees each instance gets a unique ID.
	3. Uses znodes to store and manage counters.
	4. Supports distributed locking to serialise ID generation.
4. Flow
	1. Service requests next ID from Zookeeper.
	2. Zookeeper increments global counter atomically.
	3. ID is Base62 encoded and used as TinyURL.
	4. Mapping stored in DB

## Making Tech and Infra Decisions
1. Database:
	1. SQL(e.g. PostgreSQL with auto-increment IDs).
	2. NoSQL (Redis for caching, DynamoDB for scalability).
	3. Cache: Redis or Memcached for high-speed lookup
2. Scalability and Performance:
	1. Horizontal Scaling for URL generation services.
3. High Availability
	1. Load Balancer to distribute incoming traffic across service instances.
	2. Replication in DB to avoid single point of failure.
	3. Failover-ready infrastructure using cloud-managed DBs or services.

## The Final Design - URL Shortener
![[Pasted image 20260826155142.png]]