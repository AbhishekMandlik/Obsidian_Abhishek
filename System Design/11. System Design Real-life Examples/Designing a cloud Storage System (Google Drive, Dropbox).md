### Understanding the problem
- Allows users to upload, store and manage files of any type.
- Keeps files synchronised across devices(desktop, mobile, web).
- Enables secure sharing and collaboration with others.
- Handles massive scale(millions of users, petabytes of data).
- Maintains version history and supports file recovery.

### Functional Requirements
1. File Upload and Download: Users can upload files of any type and retrieve them on demand.
2. Multi-delete Sync: Automatically sync files across web, mobile and desktop clients.
3. File Organisation: Support folders, nested directories and tagging.
4. Sharing and Collaboration: Public/ Private links, editable access, shared folders.
5. File versioning: Maintain previous versions and allow rollback.
6. Soft Delete and Restore

### Non- Functional Requirements
1. Scalability: Must support millions of users and petabytes of data.
2. High Availability and Durability: 99.999999999% durability (aka eleven 9s) for files.
3. Low Latency: Fast upload/ downloads for small and large files.
4. Security: End-to-end encryption, secure sharing, access control
5. Cost Efficiency: Optimise storage and bandwidth usage.
6. Observability and Monitoring: Track usage, errors, sync delays, API failures.

### Assumptions and Constraints
1. Key Assumptions:
	1. Files can be large -> requires chunked uploads.
	2. Users access from multiple devices -> real-time sync necessary.
	3. Cloud object storage will be used -> separation of metadata and file content
	4. Authentication and user identity is handled externally -> we focus only on storage layer.
2. Core Constraints:
	1. Uploads must be resumable-> chunk tracking, session management.
	2. File sync latency < 5 seconds -> event driven updates, push mechanisms.
	3. Storage is cost efficient-> de duplication and lifecycle policies.
	4. Fine grained access control management. Permission model and ACL support.

### Estimating Scale - Users, Listing and booking
Key Metrics:
1. Users = 10M.
2. Average files/user = 500.
3. Total stored files: 5B
4. Average File size: 2MB
5. Total data stored: 10PB
6. Upload Rate: 2K uploads/sec (peak)
7. Sync events 10K updates/sec (peak)

Implication:
Scale requires object storage for durability, distributed metadata service and horizontal scalability in sync and notification pipelines. Sync and notification pipelines must also be horizontally scalable.

### Understanding Access Patterns
1. Write - Heavy:
	1. Uploads = large payloads (chunked).
	2. Frequent sync updates.
	3. Versioning = multiple writes per file.
2. Read- Heavy:
	1. Downloads from multiple devices
	2. Folders listings and metadata reads.
	3. Shared link previews
3. Implication: Must be optimised both file write flow(chunking session management) and metadata read flow( low-latency access and caching)

### Identifying System bottlenecks and Challenges.
1. Single metadata DB -> becomes a hotspot.
2. Large file uploads -> risk of failures, timeouts.
3. Real-time sync -> must avoid stale states and race condition.
4. Permission check -> slow file/folder access.
5. High volume shared links -> unauthenticated access traffic.
6. Implication: Push toward partitioned metadata, chunked resumable uploads, pub-sub sync caching for shared/public content.

Partitioned metadata service helps eliminate db hotspot, chunked and resumable uploads make large file transfers reliable, A published subscribe approach enables scalable real-time sync. Caching helps us reduce load on DB therefore reducing cost of serving frequently access metadata and shared content.

### Major Components in our cloud storage system
1. Upload Service: Handles chunked/ resumable uploads.
2. Metadata service: stores file structure, permissions, ownership.
3. Auth Service: Validates permissions for all operations.
4. Sync Service: Pushes changes to connected devices in near-real time.
5. Storage Service: Interfaces with cloud object storage.
6. Deduplication Service: Eliminates redundant file chunks.
7. Versioning Service: Manages previous versions and deleted files.

### API Design Overview
Upload and File Management:
- POST /upload/initiate:
- PUT /upload/{id}/chunk:
- POST /upload/{id}/complete:
File Retrieval and Metadata:
- GET /files/{field} -> Downloads the file with access control checks
- GET /files/{field}/metadata -> fetch file metadata
Sharing and Collaboration:
- POST .files/{field}/share -> create shareable link.
- GET /files/shared/{token} ->Access file via shared link.
Sync and Change Tracking
- GET /sync/updates -> streams or polls for real-time file/folder changes.

All APIs are secured, support idempotency and follow RESTful principles. Uploads are resumable. Metadata is separate from file content.

### How Service Communicate
1. Patterns used:
	1. Rest + gRPC APIs for synchronous communication:
	2. Pub/sub or message queues for (Asynchronous: Uploading files):
		1. Change events (for sync).
		2. Post-upload processing.
2. Event. Sourcing for sync versioning triggers.
	1. Real-time needs -> WebSockets or long polling to push file updates.

### Handling Chunking for Large Files
1. Why chunking?
	1. Efficient upload for large files
	2. Resumable uploads to prevent data loss.
	3. Parallel uploads for faster transfers.
2. How we handle it:
	1. Split files into manageable chunks.
	2. Each chunk is independently uploaded.
	3. Chunk metadata is tracked in the Metadata service(upload progress, chunk ID).
	4. After all chunks are uploaded, they are assembled into single file in the storage service.
3. Chunk Management:
	1. Retries if a chunk fails during uploads.
	2. Error handlingL If upload is interrupted, resume from last successful chunk.


### Versioning For File updates:
1. Why versioning?
	1. Preserve Historical file states for rollback.
	2. Allow users to track changes and restore previous versions.
2. How we handle versioning?
	1. File metadata stores version information (timestamps, versions).
	2. Each file update trigger a new version entry in the Metadata service.
	3. Version IDs for easy access to specific versions.
	4. Users can restore any versions via a simple API call (GET/files/{field}/version/{versionId}).
3. Versioning Workflow:
	1. A new file upload will generate a new version if file content differs from previous, else not.
	2. Change detection is a must or else there will be unnecessary storage issue and will also affect the user if the versioning is same.

### Storage Strategy:
1. Object Storage:
	1. Stores files as chunks for efficient uploads, parallelism and resilience.
	2. Ensure data replication for redundancy and availability.
2. SQL vs NoSQL for Metadata:
	1. SQL for structured metadata like user permissions, file details etc.
	2. No SQL for flexible or dynamic metadata (logs, dynamic file attributes).
3. Scalability and Fault tolerance:
	1. Object storage offers horizontal scalability.
	2. NoSQL for fast scaling of unstructured data.

### Caching Strategy
1. Metadata Caching:
	1. Use in-memory Caches for fast access to metadata.
2. File Content Caching:
	1. Cache files at edge locations via CDN for quick retrieval.
3. Sync and Consistency:
	1. Use event systems( queues) for real-time sync and cache invalidation.
Cache data should not become stale as versioning might change. So a new challenge of consistency is introduced in here. So we need event system for real-time syncing.

### Database Schema
1. User Management:
	1. SQL for structured user data and relations.
2. File Metadata:
	1. SQL for structured metadata, NoSQL for flexible or dynamic attributes.
3. Chunk Tracking:
	1. NoSQL for chunk metadata tracking.
4. File Versions: 
	1. SQL for relational versioning, No SQL for scalable versioning.
5. Permissions:
	1. SQL for IAM.
6. Audit logging
	1. Structured -> SQL.
	2. Unstructured -> NoSQL.

### Strategic Tech and Infra Decisions.
	1. Architecture: Microservices, containers
	2. Storage:Object storage(s3)
	3. Database:
		1. SQL -> PostGres
		2. NoSQL (MongoDB) for scalable metadata
	4. Caching: In-memory (Redis), CDN for content delivery.
	5. API Gateway:For request routing and security
	6. Auto-Scaling: Dynamic scaling based on load

### Final Design
![[Pasted image 20260911155108.png]]