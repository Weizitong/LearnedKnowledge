# Knowledge
## SQL vs NoSQL
### SQL: MySQL, Oracle database, PostgreSQL.
- Transaction (ACID)
- Structured Data (Join)
  
### NoSQL: CouchDB, Neo4j, Cassandra, HBase, Amazon DynamoDB
- Low latency.
- Only need to serialize and deserialize data (JSON, XML, YAML).
- Store a massive amount of data.

## Load balancer
- Multiple servers to handle requests. 
- To eliminate single point failure. 
- Reduce the workload on a server.
- Achieve high availability.

## Database Replication
- Master/Slave
- Multiple masters

### Advantages:
- Better Perf: Read spread to all slave DBs
- Reliability: Fail-over
- HA: Fail-over

## Cache
### Pros:
- Fast response.

### Why cache?
- Read frequently but modified infrequently. 
- Data dont need to keep in persistent data stores

### Challenges:
- Expiration policy
- Consistency.
- Need multiple cache servers.

## CDN
CDN cache static content like images, videos, CSS, JavaScript files, etc. (Dynamic for HTML pages based on request path, query strings, cookies, and request heards)   

## Stateless web tier
Definition: Each web server in the cluster can access state data from databases.   

## Data centers
geo-routed    

### Technical challenges
- Traffic redirection: GeoDNS to route request to correct data center.
- Data Synchronization: A common strategy is to replicate data cross multiple data centers. (Can search how Netflix achieves this).
- Test and deployment: Automated deloyment tools are vital to keep services consistent through all data centesr.

## Message queue
A message queue is a durable component, stored in memorty, that supports asynchronous communication.    

Mainly used to decouple Producer and Consumer. We can scale the 2 componets individually.

## Logging, metrics, automation
Usually necessary for large business server groups.    
- Logging: Monitorring error logs.
- Metrics: Gain and understand: Business insights and Health status of the system. 
  - Host level metrics: CPU, Mem, I/O
  - Aggregated lv metrics: Perf of entire database tier, cache tier, etc.
  - Key business metrics: DAU, retention, revenue, etc.
- Automation: CI/CD. 

## Database scaling
### Vertical scaling
Powerful machine.

### Horizontal scaling
Data sharding: Split data into multiple databases.  

#### Challenges
- Resharding data:
  1. A single share could no longer hold more data due to rapid growth.
  2. Uneven data distribution. (Improve sharding function, moving data around, consistent hashding)
- Celebrity problem: Excessive access to a specific shard (Nee a further data partition).
- Join and de-normalization: Once a database has been sharded across multiple servers, it is hard to perform join operations across database shards. A common workaround is do de-normalize the database so that queries can be performed in a single table.

# Back-of-the-envelop estimation
We should estimate:    
QPS, Required Storage    

## Tips
- Write down assumptions
- Label your units. KB? MB? 
- Estimate QPS, **Peak** QPS, stroage, cache, number of servers.

# A 4-step process for effective system design interview
## Understand the problem and establish design scope (3 - 10 mins)
Here are some example questions you can ask:
- What specific features are we going to build?
- How many users does the product have?
- Host fast does the company anticipate to scale up? What are the anticipated scales in 3 months, 6 months, and a year?
- What is the company's technology stack? What existing services you might to leverage to simplify the design?

## Propose high-level design and get buy-in (10 - 15 mins)
- Come up with an initial blueprint for the design.
- Draw box diagrams with key components on the whiteboard or paper. This might include clients (mobile/web), API, web servers, data stores, cache, CDN, message queue, etc.
- Do back-of-the-envelope calculations to eval if your blueprint fits the scale constraints.

## Design deep dive (10 - 25 mins)
- Agreed on the overall goals and feature scope
- Sketched out a high-level blueprint for the overall design
- Obtained feedback from your interviewer on the high-level design
- Had some initial ideas about areas to focus on in deep dive based on her feedback.

## Wrap up (3 - 5 mins)
- The interviewer might want you to identify the system bottlenecks and discuss potential improvements.
- Useful to give the interviewer a recap of your design.
- Error cases (server failure, network loss, etc)
- Operation issues are worth mentioning. How do your monitor metrics and error logs? How to roll out the system.
- How to handle the next scale curve is also an interesting topic.
- Propose other refiniements you need if you had more time.

## In a summary
**Dos**
- Always ask for clarification.
- Understand the requirements of the problem.
- There is neither the right answer nor the best answer. Ensure we understand the requirements.
- Let the interviewer know what you are thinking.
- Suggest multiple approaches if possible.
- Once you agree with your interviewer on the blue print, go into detail on each component. Design the most critical component first.
- Bounce ideas off teh interviewer.

**Don'ts**
- Don't be unprepared for typical interview questions.
- Don't jump into a solution without clarifying the requirements and assumptions.
- Don't go into too much detail on a single component in the beginning. Give the high-level design first then drills down.
- Ask hints
- Don't think in silence.
- Ask feedbacks until interviewer think the design is done.

# API Rate Limiter
## Some questions, you should ask
1. Is this a server-side limiter or client side?
2. Are we under a distributed envrionment? (If it is server side)
3. How large company are we? (Million of users or just hundres)
4. Where should we place our limiter? Integrated with servers or use as a middleware (Microservice)
5. Is throttled request visable to user?
6. User-based? Or IP address based limiting?

## Algorithm
- Token bucket
- Leaking bucket
- Fixed window counter
- Sliding window log
- Sliding window counter

## Design
- 1 or more limiters (Synchronized issue)
- Use Redis to store user or ip based request call
- Configure the rules
- How to handle exception? MQ or drop

## Improvements
- Perf improve
- Monitoring

# Design a key-value store
## System components
### CAP Theory
Need to know how to leverage between CP and CA system

### Data partition
- Consistent hashing
- Virtual Replca
  - Better server capacity -> More virtual nodes

### Data Replication
After finding the position on the hash ring, replicate data to the next N servers.

### Consistency
N: The number of replica
W: A write quorum of size W. For a write operation to be considered as successful, write operation must be ack from W replicas
R: A read quorum of size W. For a read operation to be considered as successful, read operation must be ack from W replicas.    

- R = 1, W = N: Fast read
- W = 1, R = N: Fast write
- W + R > N, strong consistency is guaranteed.
- W + R <= N, strong consistency is not guaranteed.

### Inconsistency resolution: versioning
#### Scenarios
The following graph is an example:    
![Inconsistency](pics/Inconsistency.png)    

#### Vector clock
A vector clock is a [server, version] pair associated with a data item. It can be used to check if one version precedes, succeeds, or in conflict with others.

The following graph is the example of handling inconsistency by vector clock.    
![version clock](pics/vectorclock.png)     
![VC_Text](pics/text_vc.png)    

**No conflict**
S0 = D([S0, 1][S1, 1])    
S1 = D([S0, 2][S1, 1])    
S0 is the ancestor of S1, because every entry of S0 is <= corresponding entry in S1    

**Conflict** 
S0 = D([S0, 1][S1, 2])    
S1 = D([S0, 2][S1, 1])    
Conflict is detected between S0 and S1. On S0, it believes node S1 has the newer version, but S1 believes S0 has the version. Then the conflict happens. Then we need to let client decide which version shall we choose.     
    
### Handling failures
#### Failure detection: Gossip protocol
- Each node maintains a node membership list, which contains member IDs and heartbeat counters.
- Each node periodically increaments its heartbeat counter.
- Each node periodically sends heartbeats to a set of random nodes, which in turn propagate to another set of nodes.
- Once nodes receive heartbeats, membership list is updated to the latest info.
- If the heartbeat has not increased for more tham predefined periods, the member is considered as offline.

#### Handling temporary failures: Sloppy quorum & Hinted handoff
- Instead of enforcing the quorum requirement, the system choose the first W healthy servers for writes and first R healthy servers for reads on the hash ring. Offline servers are ignored.
- If a server is unavailable due to network of server failures, another server will process requests temporarily.

#### Handling permanent failures: Anti-entropy protocal & Merkle tree
- Anti-entropy protocal: Compare each piece of data on replicas and updating each replica to the newest version.
- Merkle tree: Inconsistency detection and minimizing the amount of data transferred.
Bucketize the data on each node.
![Bucketized_data](pics/bucketized_data.png)
Hash each bucket, and the parent of 2 bucket, and 2 parents... until we have 1 hash value for a node.    
![Merkle tree](pics/merkle_tree.png)    

#### Handling data center outage
Replicate data cross the data centers

## System architecture diagram
![kv_arch](pics/kv_arch.png)

### Write Path
![WritePath](pics/WritePath.png)
1. Use SSTable on disk to store data (Data durability) (Flush from Mem while is full or beyond the threshold).
2. Write down every commit in log.
3. Data is saved in memory.

### Read Path
**Read data from memory firstly**
![ReadPath1](pics/ReadPath1.png)
**Check SSTable if cannot find**
![ReadPath1](pics/ReadPath2.png)
1. Check memorty firstly.
2. Use bloom filter to find data in SSTable if memory does not have the key.

# Design a unique ID generator in distributed systems
## Multi-master replication
Multi master write. For the server ith (0 based), the generated ID will be new_id = i + (ithCall * N), where N is the total number of servers in the clusters.
**Pros:**
- Easy to implement
**Cons:**
- Hard to scale out
- IDs do not go up with time across multiple servers

## Universally unique identifier
A 128-bit length string generator. Few chance to have a conflict. Therefore, we just put this ID into different servers.

## Ticket server
Centralized ticket server generate ID for each API server. However, ticket server is a SPF and if we use multi-ticket servers design, it will introduce the challenge to do synchronization among each ticket servers.

## Twitter snowflake approach
![Snowflakes](pics/snowflake.png)

# Web Crawler
## Clarify questions
1. Scalability: Highly parallelization.
2. Robustness: Bad HTML, unresponsive servers, crashes, malicious link, etc.
3. Politeness: Dont send too many requests to a website within a short time inverval.
4. Extensibility: The system is flexible so that minimal changes are needed to support new content types.

## Dive Deep on questions
### BFS issue
1. Most links from the same page are linked back to the same host (May cause an impolite visit).
2. Standard BFS does not take the priority of a URL into considerations.

### URL frontier
1. Address politeness: 
![polite](pics/web_crawler_polite.png)
2. Use PageRank to assign weight to each url. Different weights will flow to different queues.
![frontier](pics/frontier.png)
3. Fressness: Recrawl based on history or importaness.
4. Storage: Cache + disk

### HTML Downloader
#### Robots.txt
Robots Exclusion Protocal. It specifies what pages crawlers are allowed to download.

#### Performance optimization
1. Distributed crawl
2. Cache DNS Resolver (Cache Map<Url, IP_Addr>)
3. Locality (Crawl closer server)
4. Short timeout

### Robustness
- Consistent hashing
- Save crawl states and data: A disrupted crawl can be restarted easily by loading saved states and data.
- Exception handling: Dont crash the system.
- Data validation*

### Detect and avoid problematic content
1. Redundant content: Hashes or checksums help to detect duplication.
2. Spider traps: Set a length for maximal length for URLs.
3. Data noise: Exclude useless data, such as ads, spam, etc.

### Moreover techniques
1. Server-side rendering
2. Filter out unwanted pages
3. DB replication and sharding
4. Horizontal scaling
5. Availability, consistency, and reliability.
6. Analytics.

# Design a notification system
## Scope and requirements questions:
1. What type of devices?
2. Is it a real-time system (do we accept delay)?
3. What types of notifications does the system support?
4. What triggers notification?
5. Will users be able to opt-out?
6. How many notifications are sent out each day? (QPS)

## Different types of notifications
1. iOS use APN (Apple push notification)
2. Android push notification - Firebase Cloud Messaging (FCM)
3. SMS message (Twilio, Nexmo)
4. Email (Sendgrid, Mailchimp)

## Contact info gathering flow
**Workflow**
![Contact gather flow](pics/contact_gather.png)

**Schema**
![User info](pics/userinfo_table.png)

**High level design**
![PN design](pics/PN_design.png)

