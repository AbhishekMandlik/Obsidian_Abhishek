### What are we building?
Key User workflows:
1. Upload video: Chunked upload -> Processing -> storage -> playback.
2. Watch a video: CDN support.
3. Search/ Browse.
4. Engage: Like, comments, shares.

### Functional Requirements
1. User Registration and Authentication
2. Upload video content
3. Download video content
4. Video streaming.
5. Counting likes, shares, comments
6. Search by keywords.
7. Personalisation of home feed.
8. Video metadata like title, description, tags etc.

### Non - Functional Requirements
1. Low Latency in video streaming (Cache CDN).
2. High Availability of videos/ data.
3. Scalability to Billion videos.
4. Efficient storage and cost optimisation.
5. Security and abuse protection.

### Assumptions
1. User will upload short content.
2. 95% of content will be static.
3. CDN will be used for global delivery.
4. Multiple video quality should be supported.
5. Metadata is small and queryable ( title, tags, timestamps).

### Constraints and Challenges.
1. High storage volume.
2. Processing pipelines need to scale.
3. Playback must support adaptive bitrate streaming.
4. Abuse prevention: spam, copyright, explicit content.
5. Consistency of metadata vs video availability.
6. Cost management at scale ( storage + CDN egress).

### Scale Assumptions:
1. 100M users.
2. 10M uploads/day.
3. 500M daily video views.
4. 100M comments, likes and shares per day.
5. Average video length: 10mins.
6. Video metadata: ~1KB; Engagement events: 500B per action.

### Estimating storage needs
1. Raw video storage:
	1. average upload: 10min @5MB/min -> 50MB video.
	2. 10M uploads/day -> 500TB / day.
	3. With 30-day retention (for processing): ~15PB/month.
2. Encoded Versions:
	1. Assume 4 variants: 240p,720p etc.
	2. Storage multiplies: ~3x: ~45PB/month.

### Estimating Bandwidth Needs
1. Video Streaming Bandwidth:
	1. 3 videos/day for every user ->300M hrs/day.
	2. 1Mbps streaming = ~0.45GB/hr
	3. Total egress/day = ~135 pB/day
	4. CDN for concurrent users.
2. Peak Load Estimate:
	1. 10M @1Mbps -> ~ 10TBps egress.
10TB of outbound traffic per second. This is the reason CDNs are such foundational and fundamental part of video.

### Metadata and Engagement Scale:
1. Video metadata: 10M new rows/day.
2. Likes/comment: 100M new events/day.
3. Search index updates in real-time.
4. Hot content = read heavy patterns.

### Implication of scale assumptions:
1. Storage:
	1. Requires multi-tiered storage (hot, warm, cold).
	2. Frequent writes -> distributed blob storage (s3, GCS).
	3. Cold storage and deletion policies to control cost.
2. Processing:
	1. Encoding pipeline needs autoscaling and GPU support
	2. Parallel processing jobs per resolution.
3. Global Distribution:
	1. CDN.
	2. Region aware content routing.
4. Engagement Data:
	1. Write-heavy -> eventual consistency and event queues.
	2. Aggregation for views/likes should be async and sharded.
5. Search and Discovery:
	1. Need real-time indexing at scale.
	2. Distributed search infra(Elasticsearch, Meilisearch).

### Core Components
1. API Gateway.
2. Uploads and Ingestion: Triggers encoding jobs via a queue + video file uploads, generates video IDs, stores temporarily.
3. Encoding and processing: Transcode videos to multiple resolution, generates thumbnails, prepares HLS/DASH formats and stores in final storage.
4. Video Storage and CDN.
5. Metadata Service: mapping between videoID and location.
6. User Service: Manages user accounts, auth, channel subscription and user preferences.
7. Engagement Service: Track view, subscription, user preferences.
8. Search and Discovery: Powers real-time search for videos/channels using tags, titles and trends, supports indexing of new content.
9. Recommendation Engine: Personalises video feed using behavioural data, embeddings and collaborative filtering.

### Communication and API Design.
Communication: 
1. Sync (HTTP, gRPC) for metadata fetch, user info, search.
2. Async: Uploads, encoding, engagement events.

API Design:
1. Client-> API Gateway (REST):
	1. Security
	2. Endpoints: POST /upload, GET/videos/videoID, POST /like
2. API Gateway -> Internal Services (REST):
	1. Security: API Gateway handles authentication, rate-limiting and routing.
	2. Services: Metadata, Encoding, Engagement, User.
3. Video Upload -> Encoding Service.
	1. Async processing: Event Bus (Kafka/SQS).
	2. Security: Secure access to file storage via signed URLs.

### Storage and Caching Decisions
1. Storage for video Data:
	1. Video Files: Object Storage (S3, GCS, Azure Blob) for scalable, durable storage, Stores video chunks, manifests files and thumbnails.
	2. CDN: Cloud-Flare or Akamai for fast global video streaming, It reduces latency by caching videos at edge locations.
2. Storage for Metadata:
	1. Relational Database: MySQL for structured metadata (video titles, tags, user data). Provides fast querying and indexing for search.
	2. NoSQL Database: MongoDB for flexible data (user preferences, video recommendations)
3. Caching and Performance:
	1. In-Memory Cache: Redis/ Memcached for frequently accessed data (video metadata, user sessions).
4. Backup & Durability:
	1. Regular backups of databases and video files stored in geographically distributed regions.

### Cursory DB Schema
1. User Table: user_id(PK), email, username, password_hash, join_date, dp.
2. Videos Table: video_id(PK), user_id(FK), title, description, upload_date, status, thumbnail_url.
3. Likes Table: like_id(PK), user_id(FK), video_id(FK), timestamp.
4. Comments Table: same as Likes table with comment_text.
5. Watch History Table: history_id(PK), user_id(FK), video_id(FK), watch_time
6. Video Analytics Table: video_id(FK), views, likes, shares, dislikes, comments_count etc.

### Strategic Tech and Infra Decisions
1. Frontend Framework: React.js/ Vue.js for responsive, component based UI
2. Backend Framework: Node.js with Express for scalable, async API handling.
3. Database: 
	1. PostgreSQL for structured metadata storage.
	2. MongoDB for flexible scalable data (e.g. user preferences).
4. Video Storage and CDN:
	1. AWS S3/ GCS for scalable, durable video storage.
	2. Cloudflare/ AWS CloudFront for low-latency video delivery.
5. Authentication: OAuth2/ JWT for secure, token-based authentication.
6. Event Processing: Kafka/ SQS for async task processing.
7. Infrastructure:
	1. AWS/ GCP for scalable cloud infra.
	2. Kubernetes for efficient microservice orchestration/


### Final Design
![[Pasted image 20260913005931.png]]