Crawl and index billions of webpages, Serve keyword-based queries, return relevant ranked results in real-time and update the index periodically for freshness.
Key Users: General users performing web searches and Internal analytics team querying the data.

### Functional Requirements:
1. Web crawling: Discover and fetch web pages at scale.
2. Indexing: Extract text, normalise and structure for fast querying.
3. Keyword Search: Accept queries and return relevant documents.
4. Webpage Ranking: Score and sort results.
5. Content Refreshing are re-indexing: Periodic re-crawl for fresh content.

### Non- Functional Requirements:
1. Low-Latency: <200ms query response time.
2. High Availability: Update index with recent changes in hours.
3. Scalability: Index billions of documents, handle 50K QPS.
4. Fault Tolerance: No single point of failure, retry on fetch failure.
5. Storage Efficiency: Compress and deduplicate massive data volumes.

### Key System Design Challenges:
1. Crawling the web: How to avoid duplication and rate limits.
2. Indexing:  How to store and search documents fast.
3. Relevance/ Scoring: Scoring algorithms.
4. Scaling: Scale search infra as usage and index size grow.
5. Freshness: How often to re-crawl. 

### Assumptions and Constraints
1. Publicly accessible websites only.
2. Crawlers will respect robot.txt and politeness policies.
3. Query language is limited to simple keyword searches.
4. Ranking will be based on basic relevance models (PageRank, TF-IDF).
5. User Experience: <200ms for top N results.

### Constraints:
1. Limited Bandwidth for crawling -- need scheduling and de duplication.
2. Storage and indexing must handle PB scale data.
3. Real-time indexing not required -- periodic batches acceptable.
4. No personalisation or user profiling in this phase.
5. Distributed systems complexity -- must handle node failures, retries and horizontal scaling.

### Estimating Web Scale:
1. Totale indexed pages: ~100B 
2. Average page size: ~100KB
3. Storage Volume: ~10PB (raw), 2-3PB (compressed).
4. Daily searches: 5 - 10 B(global search engines).
5. Target for MVP: 100M pages, 1M QPS capacity, 1PB storage budget.

### Traffic and Query Load Estimation.
- Assumption
	- 10M DAU.
	- 5 queries/day = 50M queries/day.
	- Peak QPS = ~2000 QPS
- Query Pattern
	- 95% Read.
	- 5% Write.
We need to optimise it for low latency read and indexing and uploading website can be processed asynchronously is background without affecting the search experience.

### Index Size and Storage Estimation
1. Raw HTML content: 100M pages x 100KB = ~10 TB
2. During indexing we extract useful text, tokenise it, normalise it and discard info not required so, Processed and tokenised content: ~3-5 TB.
3. Inverted index (terms -> doc IDs): 500-800 GB..Highly optimised
4. Forward index( docID -> content): 5TB
5. Metadata (titles, links, ranks):100-200GB

### Crawling Throughput Estimation
1. Target: Crawl pages in 7 day:
	1. Pages/day: 14.3M.
	2. Pages/hour: 600 k.
	3. Pages/sec: 170.
2. With 500 Crawler workers -> 0.34 pages/sec per worker
Feasible with politeness and back off strategies.

### Query Latency Expectation
1. Query parsing: <5ms
2. Index lookup: <20ms
3. Ranking and scoring: <50ms
4. Result formatting: <10ms
5. Total SLA: <200ms end-to-end

### Identifying Key Bottlenecks
1. Crawling Layer:
	1. Bandwidth limitation and site rate limits.
	2. Risk of duplicate content.
	3. Distributed Crawling, de-duplication filters, crawl scheduling, robots.txt.
2. Indexing Layer:
	1. High memory & disk I/O usage when processing large datasets.
	2. Segment-based indexing, compression(delta encoding, front-coding)
3. Query Layer:
	1. High QPS can cause latency spikes.
	2. Sharded inverted indexes, result caching (popular queries), fast in-memory lookups.
4. Storage Layer: 
	1. PB -scale storage requirement(raw + indexes data).
	2. Distributed file systems,(HDFS, S3), columnar formats, cold/hot storage separation.
5. Freshness Challenge:
	1. Web content changes frequently -- need efficient re-crawling.
	2. Prioritise high change rate sites, adaptive re-crawling interval, content diffing.

### Scale Driven Design Decisions:
1. Shard architecture for inverted index.
2. Use forward and inverted index for spped and relevance.
3. Crawler partitioning by domain hash
4. Replication for fault tolerance.
5. Introduce caching for frequent queries.

### Core Components:
1. Web Crawler: Continuously fetches and downloads webpages from internet, respects the crawling policy and rate limiting.
2. URL frontier and Scheduler: Manages the queue of URLs to be crawled, prioritises based on freshness, domain policies and crawl history.
3. Content Extractor and Parser: Cleans HTML, extracts metadata and outgoing links.
4. Indexer Service: Tokenises, stems and builds both forward and inverted indexes for fast lookup.
5. Document Store: Stores raw and parsed data versions of the crawled web pages along with the metadata.
6. Inverted Index Store: Maps search terms to list of document(docID) which will be used later for fast lookup.
7. Query Service: Parses incoming search queries, looks up inverted index and fetches the candidate documents.
8. Ranking Engine: TF-IDF, PageRank algorithm for freshness and relevance.
9. Search API: Exposes a user facing interface (or internal API) to handle search requests and return ranked relevance.
10. Cache and Frontend: Stores hot query results in-memory (Redis) and provides the UI or API gateway to the users.

### Crawler Co-ordination Architecture
1. Crawler Flow
	1. URL Frontier: Queue of URLs to visit( distributed, prioritised).
	2. Crawler Workers:
		1. Fetch Page
		2. Store raw content in doc store.
		3. Push URLs to fetch back to frontier.
	3. Duplicate Detection: Use fingerprints to avoid re-crawling.(SimHash)
	4. Scheduler: Controls Crawl frequency and site politeness.
   This helps to keep the index fresh without having load on external websites.
2. Co-ordination Strategy:
	1. Distributed worker pulling from sharded queues.
	2. Hash-based partitioning by domain.
	3. Retry and failover handling.

### Indexing Workflow
1. Content Normalisation: Strip HTML, Apply stemming and remove stop words.
2. Generate Forward Index: Stores docID -> tokens +metadata
3. Inverted Index Generation: For each token, store list of docIDs with positions and frequencies (TF-IDF).
4. Rank Feature Extraction: PageRank, TF-IDF and freshness score.
Indexes are partitioned and replicated across nodes.

### Search Query flow
User Query -> Search API -> Query Parser -> Inverted Index Lookup -> Ranking Engine -> Result are returned
Results may hit cache, Top N results formatted with snippet generation.

### Sample DB Structure:
Crawler DB:
	Table: URL queue
Search Index DB:
	Table: Documents (forward)
	Table: Inverted Index	

### Communication Between Components:
1. Message queues for:
	1. Crawler -> Parser -> Indexer
	2. Indexer -> index store
2. HTTP/GRPC APIs for:
	1. Search API -> Query store
3. Caching Layer:
	1. Redis or Memcached 
4. Monitoring:
	1. Metrics pipeline for crawl status, indexing rate and search latency.

### Strategic Tech and Infra Decisions
1. Crawler Infra: 
	1. Distributed Crawling (kafka, celery) to scale the crawling process and handle large volumes of URLs in parallel.
	2. Cloud providers for scalability.
2. Storage Strategy:
	1. Relational vs NoSQL: Use NoSQL for Documents and inverted index table for flexible schema and fast lookup.
	2. Object storage for Raw Data.
3. Indexing Engine:
	1. Custom vs Pre-built: Build a custom indexing engine or integrate with elastic search for efficient full text ranking.
	2. Real-time indexing: Ensure the indexing process is continuous, processing document as they are crawled for near real-time search results.
4. Ranking Algorithm:
	1. TF-IDF and Page Rank: Implement TF-IDF for basic relevance and PageRank for link- based relevance scoring.
	2. Freshness Consideration: Introduce freshness ranking for new content to have higher priority.
5. API and Query Optimisation:
	1. Load Balancing:
	2. Cache Results:
6. Scalability and Fault Tolerance
	1. Horizontal Scaling:
	2. Failover & Recovery: Implement replication and automatic failover strategies to ensure high availability (using tools like kafka or AWS RDS).

### Final Design
![[Pasted image 20260913144720.png]]