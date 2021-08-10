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

## High level design
![PN design](pics/PN_design.png)
1. **Service 1 to N**: A service can be a micro-service, a cron job, or a distributed system that triggers notification sending envets.
2. **Notification servers**:
    - Provide APIs for services to send notifications, Those APIs are only accessible internally or by verified clients to prevent spams.
    - Carry out basic validations to verify emails, phone numbers, etc.
    - Query the database or cache to fetch data needed to render a notification.
    - Put notification data to message queues for parallel processing.
3. **Cache**: User info, device info, notification templates are cached.
4. **DB**: User, notification, setting, etc.
5. **Message queue**: Remove the dependencies between components. Serve as a buffer. 3rd party outage will affect our service
6. **Workers**: Pull request from MQ and send to 3rd party notification services.

## Design deep dive
### Reliability
Use log to record whether a notification send successfully or not. + Retry mechanism.    
To ensure a notification will be sent only once, we can use even_id to track each notification. Filter our duplicate notification requests. (However, it still hard to completely eliminate this issue).
![Log](pics/PNLog.png)

### Additional components and consideration.
#### Notification template
A predefined notification message format. Maintain the information consistency and avoid unforeseem margin edge cases.    
e.g. Your driver [NAME] is on the way.

#### Notification setting
User can decide: Receive or not receive a notification, and decide receive what kind of notifications.       
e.g.    
user_id bigInt
channel varchar
opt_in boolean

#### Rate limiting
We should limit the number of notifications in a period of time. Otherwise, uesrs may opt-out completely.

#### Retry mechanism
Retry in like 10 times, or send alert to developers.

#### Security in PN
Use appKey and appSecret are used to secure PN APIs. Only authed or verified clients can send PN using our APIs.

#### Monitor queued notifications
Telemetry to monitor PN perf, delay for data analysis.

#### Events tracking
Notification metrics, such as open rate, click rate, and engagement. ----> Understand cx's behaviors.
![tracking](pics/PN_eventTrack.png)

### Updated Design
![updated](pics/updatedPN.png)

# Design a news feed system
## High level design
### Newsfeed API
#### Feed publishing API
HTTP POST API:
```
POST /v1/me/feed
Params:
• ContentL Content is the text of the post
• auth_token: Auth use
```

#### Newsfeed retrieval API
HTTP GET API:
```
GET /v1/me/feed
Params:
• auth_token: it is used to auth API requests.
```

### Feed publishing
Workflow:    
![workflow](pics/feeds_sys.png)

### News feed building
Workflow:
![workflow](pics/feed_building.png)

## Dive deep
### Feed publishing deep dive
![workflow](pics/feedpub_deep.png)
#### Web servers
Besides communicating with clients, web servers enforce auth and rate-limiting.
#### **Fanout service**
##### Fanout on write (push model)
News feed is pre-computed during write time. Deliver to friemds immediately once it is published.    
**Pros**    
- The news feed is generated in real-time and can be pushed to friends immediately.
- Fetching news feed is fast because the news feed is pre-computed during write time.
**Cons**     
- If a user has too many friends, fetching the friend list and generating news feed for all of them are slow and time consuming. It is called hotkey problem.
- For inactive users or those rarely log-in, pre-computing news feeds waste computing resouces.
##### Fanout on read (read model)
The news is generated during the read time.
**Pros**    
- For inactive users or those who rarely log in, fanout on read works better because it will not waste computing resouce on them.
- Data in not pushed to friends so there is no hotkey problem
**Cons**     
- Slow on updating feed.    
    
Use hyprid model:    
- For majority users, we use push.
- For the users who have many friends, we use pull model to avoid system overload issue.
![Fanout](pics/fanout.png)

### Newsfeed retrieval deep dive
![workflow](pics/newsretrieval.png)
1. User request feeds
2. LB routes request
3. Web servers call the news feed service
4. News feed service get a list of post IDs from the news feed cache
5. The fetched result is a fully hydrated feed.
6. Send back to client for rendering.

### Cache architecture
![news_feed](pics/NF_cache.png)
- News feed: It stores IDs of news feeds.
- Content: It stores every post data. Popular content is stored in hot cache
- Social Graph: It stores user relationship data
- Action: It stores info about whether a user like a post, replied a post, or took other actions on a post.
- Counters: It stores counters for like, reply, follower, following, etc.

# Design a chat system
- mobile app or web app?
- scale?
- group chat support? Size?
- Important features
- Message size limit
- E2E encryption required?
- How long shall we save the chat history

## High level design
![chatsys](pics/chatsys.png)
From the sender side, we can use long-alive HTTP protocol to send message to the server.    
However, what protocal or mechanism we can use to let server relay the message to the receiver?    

### Server-initiated connection: Polling
![polling](pics/chat_polling.png)
Client periodically poll messages from the server

### Server-initiated connection: Long polling
![longpolling](pics/chat_longPolling.png)
#### Drawback
- Sender and receiver may not connect the same chat server, HTTP based servers and usually stateless. If you use round robin for LB, the server that receives the message might not have a long-polling connection with the client who receives the message.
- A server has no good way to tell if a client is disconnected.
- It is inefficient. If a user does not chat much, long polling still makes periodic connections after timeouts.

### WebSocket
![websocket](pics/WebSocket.png)    
WebSocket is a connectio which support bi-directional communication based on TCP. The WS can be used for both sending and receiving message.

### High-level design
![stateless](pics/Chat_Stateless.png)

#### Stateless services
Used to manage the login, signup, user profile, etc.    
LB routes the request to the correct service. The above service can be micro-serivces or monolithic.    

#### Stateful service
Only for chat service. The service discovery coordinates closely with the chat service to avoid server overloading.

#### 3rd-party integration
This usually is notification system.

#### Scalability
![stateless](pics/updated_chatsys.png)
- Chat servers facilitate message sending/receiving.
- Presence servers manage online/offline status
- API servers handle everything including login, signup, change profile, etc.
- Notification servers send push notifications,
- Finally, the key-value store is used to store chat history. When offline users come back online, she/he will see al her/his previous chat history.

#### Storage
**SQL vs NoSQL**    
2 types of data:    
- Generic data: User profile, setting, user friend list. ----> Save in robust SQL DB. (We shall use sharding and replication to improve the perf)
- Chat history data.
  - The amount of data is enormous for chat system.
  - Only recent chats are accessed frequently.
  - Still need to support random access: Like searching in history.
  - R : W =~ 1 : 1

Here are the reasons for why we choose K-V Store:
- K-V allows easy horizontal scaling.
- K-V stores provide very low latency to access data/
- SQL DB does not handle long tail of data well. When the indexes grow large, random access is expensive.
- K-V stores are adopted by other proven reliable chat app. E.g. both FB messenger and Discored use K-V. FB usese HBase, and Discord uses Cassandra.

### Data models
#### Message table for 1:1 chat
![11msg](pics/11msg.png)
Use message_id to show the order of the message. We cannot rely on created_at, since there might have 2 msgs, created at the same time.
#### Message table for group chat
![group](pics/groupmsg.png)
PK is (channel_id, message_id). channel_id is the partition key because all queries in a group chat operate in a channel.    
2 requirements for message_id:    
1. IDs must be unique.
2. IDs should be sortable by time, meaning new rows have higher IDs than old ones.
To achieve those, we can use a local id generator. (Use Twitter snowflake ID generator to generate the sequence).

## Dive deep
### Service discovery
![servicedisc](pics/servicedisc.png)
The primary role of service discovery is to recommend the best chat server for a client absed on the criteria like geographical location, server capacity, etc. Apache Zookeeper is a popular open-source solution for service discovery.    
There are some other system we can use to provide service discovery like [Consul](https://www.consul.io/docs) and [Etcd](https://etcd.io/).

### Message flows
#### 1 : 1
![1:1](pics/11chatflow.png)
1. User A -> Chat Server 1 (CS-1)
2. CS-1 obtains the message ID from the ID generator
3. CS-1 sends the message to the message sync queue
4. The message is stored in a K-V store.
5. Receiver Cases:    
  a. If B is online, the message is forwared to CS-2 where user B is connected.    
  b. If B is offline, a push notification is sent from PN servers.
6. CS-2 forwards the message to User B. There is a persistent WebSocket connection between User B and CS-2.

#### Message sync across multiple devices
![MsgSync](pics/multiDeviceSync.png)
2 devices from the same user will connect to the same Chat server. Each device maintain a var called `cur_max_message_id`, which keeps track of the latest message ID on the device. While a message matches the following 2 conditions, sync the msg:
1. The recipient ID is equal to the current logged-in ID.
2. Message ID in the KV > `cur_max_message_id`.

#### Small group chat flow
Use a message sync queue to cache the message. Each recipient has a sync queue, which acts like a mail inbox.

##### Sender side
![smallgroupchat](pics/smallgroupchat.png)

##### Recipient side
![smallgroupreceive](pics/smallgroupreceive.png)

### Online Presence
Status indicator also need the websocket. There are many ways to trigger this.

#### User login
After the WS connection is built between the client and real-time service, user A's online status and last_active_at timestamp are saved in KV store. Presence Indicator shows the user is online after she/he logs in.
![cs_userlogin](pics/cs_userlogin.png)

#### User logout
When a user logout, it goes through the user logout flows as shown below. The online status is changed to offline in the K-V store. The indicator shows a user is offline.
![cs_userlogout](pics/cs_userlogout.png)

#### User disconnection
If we simply switch the user to offline while the connection is broken, then if the user on the train and go through the tunnel. The presence indicator will switch the status frequently.    
Therefore, we use heartbeat to detect whether user has disconnected. 
![heartbeat_to_show_online](pics/heartbeat_to_show_online.png)

#### Online status fanout
![OnlineStatusFanout](pics/online_status_fanout.png)
We use a publish-subscribe model, in which each friend pair maintains a channel. When User A's online status changes, it publishes the event to three channels, channel A-B, A-C and A-D. Those three channels are subscribed by User B, C and D, respectively. Thus, it is easy for friends to get online status updates. The communication between clients and servers is through real-time WebSocket. (If we have to do this for million users, we have to ask cx to refresh manually).

# Design a search autocomplete system
## Design scope & questions
- Is the matching only supported at the beginning of a search query or in the middle as well?
- How many auto complete suggestions should the system return?
- Does the system support spell check?
- Are search queries in English?
- Do we allow capitalization and special characters?
- How many users use the product?

### Requirements
- Fast response time.
- Relevant
- Sorted
- Scalable: High traffic
- HA

## High-leve design
The service can be divided into 2 parts which are:
- Data gathering service: It gathers user input queries and aggregates them in real-time. Real-time processing is not practical for large data set.
- Query service: Given a search query or prefix, return 5 most frequently searched terms.

### Data gathering serviec
Use a table to record the frequency of each word. When user input: "twitch". "twitter", "twitter" and "twillo" sequentially, the table will be changed as the following trajectory:    
![countTable](pics/word_freq.png)

### Query service
Assume we have a large table collecting by 1st service. Then, while user types "tw", it will automatically complete the sentence based on the table.    
We can do like:
``` SQL
SELECT * FROM frequency_table 
WHERE query like `prefix%` 
ORDER BY frequency DESC 
LIMIT 5
```
## Deep dive
Optimize the following areas:
- Trie data structure
- Data gathering service
- Query service
- Scale the storage
- Trie operations

### Trie data structure
Sample:
![trie](pics/trie_example.png)
For each node, we can add an integer to indiciate the freq.    
#### Workflow
Terms:
- p: length of prefix
- n: total number of nodes in a trie
- c: number of children of a given node

Steps:
1. Find the prefix. O(p)
2. Traverse subtree to get all valid children. A child is valid if it can form a valid query string. O(c)
3. Sort the children and get top k. O(clogc)
   
#### Optimization
1. Limit the max length of a prefix. O(p) -> O(constant)
2. Cache top search queries at each node, check the below graph.
![trieWithCache](pics/trie_cache.png)

### Data gathering service
The real-time update system significant slow down the perf. And it is unneccessary to update the trie frequently.    

The below graph shows the updated workflow:    
![updated_flow](pics/data_gathering.png)

### Analytics logs
Raw log data. Append only and not idxed.

### Aggregators
The interval between 2 aggregating operation is depending on the importance of the real-time result. Like Twitter it may be 2-3 days, Google may be 1 week ish.

### Aggregated data
Aggregate the raw data together. Like counting the query times, so that we can update the trie tree later.

### Workers
Async workers do trie tree update.

#### Trie cache
Trie cache is distributed cache system that keeps trie in memory for fast read. It takes a weekly snapshot of the DB.

#### Trie DB
2 options are available to store the data:
1. Doc store: Periodically take a snapshot of it, serialize it, and store the serialized data in the database. MongoDB is one of the best choices.
2. K-V store: A trie can be represented in a hash table form by applying the following logic:
   1. Every prefix in the trie is mapped to a key in a hash table.
   2. Data on each trie node is mapped to a value in a hash table. The below graph shows how we K-V store save the trie.
![KVTrie](pics/kvtrie.png)

### Query service
The improved design shows as follow:
![improved_qs](pics/improved_autocomplete_design.png)
1. A search query is sent to the LB.
2. The LB routes the request to API servers.
3. API servers get trie data from Trie Cache and construct autocomplete suggestions for the client.
4. In case the data is not in trie cache, we replenish data back to the cache.

**Query service optimized**    
- AJAX request. For web applications, browsers usually send AJAX requests to fetch autocomplete results. The main benefit of AJAX is that sending/receiving a request/response does not refresh the whole web page.
- Browser cachng.
- Data sampling: We dont need to sample every request which are too many.

## Trie operations
### Create
Created by workers using aggregated data. The source of data is from Analytics Log/DB

### Update
There are 2 options:
1. Update the trie weekly. Once a new trie is created, the new trie replaces the old one.
2. In-place update. We should avoid this if the trie is large.

### Delete
We have to delete those hateful, violent, sexually explicit, or dangerous autocomplete suggestions. We add a filter layer in front of the trie cache to filter out unwanted suggestions. Have a filter layer gives us the flexibility of removing results based on different filter rules. Unwanted suggestions are removed physically from the databse asynchronically so the correct data set will be used to build trie in the next update cycle.
![filterLayer](pics/trie_filter.png)

## Scale the storage
We can split data based on the 1st character. We analyze historical data distribution pattern and apply smarter sharding logic, shown as below. We use a shard map manager maintains a lookup database for identifying where rows should be stored.
![smarter_sharding](pics/smarter_trie_sharding.png)

## Other questions
### Multiple languagse?
Then we should use the unicode.

### What if top queries in one country are different from others?
Different countries with different trie trees. Use CDN to expedite the query speed.

### How can we support the trending (real-time) search queries?
A few tips here (need more reading materials):
- Reduce the working data set by sharding
- Change the ranking model and assign more weight to recent search queries.
- Data may come as streams, so we do not have access to all the data at once. Streaming data means data is generated continuously. Stream processing requires a different set of streams: **Apache Hadoop MapReduce**, **Apache Spark Streaming**, **Apache Storm**, **Apache Kafka**, etc.

# Design YouTube
## Requirements and scope questions
1. What features are important?
2. What clients do we need to support?
3. DAU?
4. Average daily time spent on the product?
5. Do we need to support international users?
6. What are supported video resolutions?
7. Is encryption required?
8. Any file size requirement for videos?
9. Can we leverage some of the existing cloud infra provided by Amazon, Google, or Microsoft?

## High-level design
Overview:
![overview](pics/u2b_overview.png)
**Client**: Web, mobile, TV, etc     
**CDN**: Videos are stored in CDN.    
**API Servers**: Everything else except video streaming goes through API servers. This includes feed recommendation, generating video upload URL, updating metadata database and cache, user signup, etc.     

### Video uploading flow
workflow:
![upload](pics/video_upload.png)
- Users
- LB: Distribute requests
- API servers: Handle user requests
- Metadata DB: Video metadata DB. Sharded and Replicated to meet HA and perf.
- Metadata cache
- Original storage: A blob storage system is used to store original videos.
- Transcoding servers: Video transcoding is also called video encoding. Convert a video format to other formats (MPEG, HLS, etc), which provides the best video streams possible for different devices and bandwidth capabilities.
- Transcoded storage: It is a blob storage that stores transcoded videos files.
- CDN: Video are cached in CDN. When you click plan button, a video is streamed from the CDN.
- Completion queue: It is a MQ that stores info about video transcoding completion events.
- Completion handler: This consists of a list of workers that pull event data from the completion queue and update metadata cache and database.

The workflow can be divided into 2 parallel processes:
#### Upload the actual video
![upload_video](pics/upload_actual_video.png)
1. Videos are uploaded to the original storage
2. Transcoding servers fetch videos from the original storage and start transcoding
3. Once transcoding is complete, the following 2 steps are executed in parallel:
   1. Transcoded videos are sent to transcoded storage.
      1. Transcoded videos are distributed to CDN.
   2. Transcoding completion events are queued in the completion queue.
      1. Completion handler contains a bunch of workers that continuously pull event data from the queue.
      2. Completion handler updates the metadata database and cache when video transcoding is complete.
4. API servers inform the client the video is successfully uploaded and is ready for streaming.
   
#### Update the metadata
Overview:    
![metadata](pics/u2b_update_metadata.png)

### Video streaming flow
**Streaming Protocol**: 
- MPEG-DASH. MPEG stands for "Moving Picture Expets Group" and DASH stands for "Dynamic Adaptive Streaming over HTTP"
- Apple HLS. HLS stands for "HTTP Live Streaming"
- Microsoft Smooth Streaming
- Adobe HTTP Dynamic Streaming (HDS)

## Dive deep
### Video transcoding
Reasons for transcoding:
- Raw video consumes large amount of storage space. An hour-long high definition video recorded at 60 frames per second can take up a few hundred GB of space.
- Many devices and browers only support certain types of video formats. Thus, it is important to encode a video to different format for compatibility reason.
- To ensure users watch high quality videos while maintaining smooth playback, ti is a good idea to deliver higher resolution video to users who have high network bandwidth and lower resolution video to users who have low bandwidth.
- Network conditions can change, especially on mobile devices. We should support auto/manually switch video quality based on the network condition.

2 main parts of encoding formats:
1. Container: This is like a basket that contains the video file, audio, and metadata. You can tell the container format by the file extension, such as .avi, .mov or .mp4.
2. Codecs: These are compression and decompression algo aim to reduce the video size while preserving the video quality. The most used video codecs are H.264, VP9, and HEVC.

### Directed acyclic graph (DAG) model
Different content creators may have different video processing requirements. E.g. some content creators require watermark on top of their videos, some provide thumbnail images, and some upload high definition videos, etc.    
    
We adopt a DAG (Directed acyclic graph) model to achieve flexibiilty and parallelism for different tasks.    
![DAG](pics/DAG.png)    

- Insepection: Make sure videos have good quality and are not malformed.
- Video encodings: Videos are converted to support different resolutions, codec, bitrates, etc.
- Thumbnail.
- Watermark.

![Video encoding](pics/video_encoding.png)

### Video transcoding architecture
The proposed video transcoding architecture that leverages the cloud services, is shown below:
![video transcoding arch](pics/video_transcoding_arch.png)

#### Preprocessor
4 responsibilities:
1. Video splitting: Video stream is split or further split into small Group of Pictures (GOP) alignment. GOP is a group/chunk of frames arranged in a specific order. Each chunk is an independently playable unit, usually a few seconds in length.
2. Some old mobile devices or browsers might not support video splitting. Preprocessor split videos by GOP alignment for old clients.
3. DAG generation: The processor generates DAG based on configuration files client programmers write.
4. Cache data. The preprocessor is a cache for segmented videos. For better reliability, the preprocessor stores GOPs metadata in temporary storage. If video encoding fails, the system could use persisted data for retry operations.

#### DAG scheduler
The DAG scheduler splits a DAG graph into stages of tasks and puts them in the task queue in the resource manager. The below graph is an example:
![DAG_scheduler](pics/DAG_scheduler.png)

#### Resource manager
The resource manager is responsible for managing the effciency of resource allocation. It contains 3 queues and a task scheduler.
![Resource_manager](pics/video_resource_manager.png)

- Task queue: The priority queue that contains tasks to be executed.
- Worker queue: The priority queue that contains worker utilization info.
- Running queue: The queue contains info about the currently running tasks and workers running the tasks.
- Task scheduler: It picks the optimal task/worker, and instructs the chosen task worker to execute the job.

**Workflow**    
1. The task scheduler gets the highest priority task from the task queue.
2. The task scheduler gets the optimal task worker to run the task from the worker queue.
3. The task scheduler instructs the chosen task worker to run the task.
4. The task scheduler binds the task/worker info and puts it in the running queue.
5. The task scheduler removes the job from the running queue once the job is done.

#### Task workers
Runs the tasks which are defined in the DAG. Different task workers may run different tasks:
- Watermark workers stamp watermarker on the video.
- Encoder workers encode the video.
- etc.

#### Temporary storage
Multiple storage types are used here. It depends on what data and types will be stored. E.g.
- The metadata is frequently accessed by workers. Then memcache would be best choice
- Video or audio is too large, so we keep them in blob storage.
All data in temporary storage is freed up once the corresponding video processing is complete.    

#### Encoded video
Encoded video is the final output of the encoding pipeline.

### System optimizations
#### Speed optimization: parallelize video uploading
Uploading a video entirely is inefficient. We should split them into multiple GOPs. The splitting task can be implemented by the client.
![client_splitting](pics/client_split.png)

#### Speed optimization: place upload centers close to users
Use CDN to allow user uploads the video to the closest servers.

#### Speed optimization: parallelism everywhere
Loosely coupled system and enable high parallelism.     
Use message queue everywhere.    
![MQ_encode](pics/mq_encoded.png)

#### Safety optimization: pre-signed upload URL
Check below figure:
![presign](pics/presigned_url.png)
1. The client makes a HTTP request to API servers to fetch the pre-signed url (named in AWS S3. in Azure blob storage, called Shared Access Key).
2. API servers respond with pre-signed URL.
3. The client use the fetched url to upload video.

#### Safety optimization: protect your videos
To protect the copyright, 3 options can be applied:
1. Digital rights management (DRM) systems: Apple FairPlay, Google Widevine, and Microsoft PlayReady.
2. AES encryption: You can encrypt a video and configure an authorization policy. The encrypted video will be decrypted upon playback. This ensures that only authorized users can watch an encrypted video.
3. Visual watermarking.

#### Cost-saving optimization
YouTube video stream follow long-tail distribution. Many videos are rarely watched. So:
1. Only serve the most popular videos from CDN and other videos from our high capacity storage servers.
2. For less popular content, we may not need to store many encoded video versions. Short videos can be encoded on-demond.
3. Some videos are popular only in certain regions. There is no need to distribute these videos to other regions.
4. Build your own CDN like Netflix and partern with ISPs. Building your CDN is a giant project.

### Error handling
- Upload error: Retry
- Split video error: Older version of clients? let server do
- Transcoding error: Retry
- Preprocessor error: Regenerate DAG diagram
- DAG scheduler error: Reschedule
- Resource manager queue down: Replica
- Task worker down: Retry on new node
- API server down: Stateless server so request will be directed to a different server.
- Metadata cache server down: Replica.
- Metadata DB down: Replica with Master-slave.

# Design google drive
## High-level design
### API
#### Upload a file to Google Drive
`https://api.example.com/files/upload?uploadType=resumable`    
2 types of uploads are supported:
- Simple upload. For small file.
- Resumable upload. For large file.
  - Send a initial request to retrieve the resumable URL
  - Upload the data and monitor upload state
  - If upload is distributed, resume the upload.

#### Download a file from google drive
`https://api.example.com/files/download?uploadType=resumable`    
``` JSON
Params:    
{    
  "path": "/recipes/soup/best_soup.txt"    
}    
```

#### Get file revisions
`https://api.example.com/files/list_revisions`    
``` JSON
Params:    
{    
  "path": "/recipes/soup/best_soup.txt",    
  "limit": 20    
}    
```

All APIs require user authentication and use HTTPS. Secure Socket Layer (SSL) protects data transfer between client and backend servers.

### Sync conflicts
The 1st version get processed wins, the second one receives the conflicts. The 2nd user will make the decision how to resolve the conflict.     

### Design proposal
![design](pics/gDriveDesign.png)
Term explanations:
- Block servers: Block servers upload blocks to cloud storage. Block storage, referred to as block-level storage, is a technology to store data files on cloud-based env. A file can be split into several blocks, each with a unique hash value, stored in our metadata database. Each block is treated as an independent object and stored in our storage system (e.g. AWS S3). To reconstruct a file, blocks are joined in a particular order. As for the block size, we use Dropbox as a reference: it sets the maximal size of a block to 4MB.
- Cloud storage: A file is split into smaller blocks and stored in cloud storage.
- Cold storage: Cold storage is a computer system designed for storing inactive data, meaning files are not accessed for a long time. 
- Load balancer: A load balancer evenly distributes requests among API servers. 
- API servers: These are responsible for almost everything other than the uploading flow. API servers are used for user authentication, managing user profile, updating file metadata, etc. 
- Metadata database: It stores metadata of users, files, blocks, versions, etc. Please note that files are stored in the cloud and the metadata database only contains metadata.
- Metadata cache: Some of the metadata are cached for fast retrieval.
- Notification service: It is a publisher/subscriber system that allows data to be transferred from notification service to clients as certain events happen. In our specific case, notification service notifies relevant clients when a file is added/edited/removed elsewhere so they can pull the latest changes. 
- Offline backup queue: If a client is offline and cannot pull the latest file changes, the offline backup queue stores the info so changes will be synced when the client is online. 

## Design deep dive
### Block servers
Optimizations for uploading updated files:
1. Delta sync: Only upload the modified block.
2. Compression: Use different compression algorithms to compress different file types.
**Block server**    
![Block servers](pics/blockserver.png)

**Delta sync**    
![Delta sync](pics/deltasync.png)

### High consistency requirement
String consistency should apply to metadata cache and database layers.    
The following 2 things are guaranteed for a strong consistency:
- Data in cache replcas and the master is consistent.
- Invalidate caches on database write to ensure cache and database hold the same value.

### Metadata database
![dbschema](pics/dbschema.png)    
- User: The user table contains basic information about the user.
- Device: Device table stores device info. *Push_id* is used for sending and receiving mobile push notifications. Please note a user can have multiple devices.
- Workspace: A workspace is the root directory of a user.
- File: File table stores everything related to the latest file.
- File_version: It stores version history of a file. Existing rows are read-only to keep the integrity of the file revision history.
- Block: It stores everything related to a file block. A file of any version can be reconstructed by joining all the blocks in the correct order.

### Upload flow
![upload_flow](pics/fileUploadFlow.png)    
2 requests are sent in parallel: add file metadata and upload the file to cloud storage. Both requests originate from the same client 1.
- Add file metadata.
  1. The client 1 sends a request to add the metadata of the new file. 
  2. Store the new file metadata in metadata DB and change the file upload status to "pending".
  3. Notify the notification service that a new file in being added.
  4. The notification service notifies relevant clients (client 2) that a file is being uploaded.
- Upload files to cloud storage.
  1. Client 1 uploads the content of the file to block servers.
  2. Block servers chunk the files into blocks, compress, encrypt the blocks, and upload them to cloud storage.
  3. Once the file is uploaded, cloud storage triggers upload completion callback. The request is sent to API servers.
  4. File status changed to "uploaded" in Metadata DB.
  5. Notify the notification service that a file status is changed to "uploaded".
  6. The notiification service notifies relevant clients (client 2) that a file is fully uploaded.
Editing is same as created.

### Download flow
2 ways to let a client know when a file is added or updated:
1. If client A is online while a file is changed by another client, notification service will inform client A that chagnes are made somewhere so it needs to pull the latest data.
2. If client B is offline while a file is changed by another client, data will be saved to the cache. When the offline client is online again, it pulls the latest changes.

Get the metatdata file firstly, then download the blocks to reconstruct the file. See below diagram:       
![download_flow](pics/downloadfile.png)

### Notification service
To maintain file consistency, any mutation of a file performed locally needs to be informed to other clients to reduce conflicts. Notification service is built to serve this purpose. At the high-level, notification service allows data to be transferred to clients as events happen. Here are a few options:
- Long polling. Dropbox uses long polling.
- WebSocket. WebSocket provides a persistent connection between the client and the server.
    
Even though both options work well, we opt for long polling for the following 2 reasons:
- Communication for notification service is not bi-directional.
- WS is designed for real-time bi-directional communication.

### Save storage space
- De-duplicate data blocks. 2 blocks with the same hash value will be merged as one.
- Adopt an intelligent data backup strategy. 2 optimization strategies can be applied:
  1. Set a limit: Drop the oldest version if we store too many versions.
  2. Keep valuable versions only: Limit the number of saved versions. We give more weight to recent versions.
  3. Moving infrequently used data to cold storage. Could data is the data that has not been active for months or years.

### Failure Handling
- Load balancer failure: Primary/Secondary + heartbeat
- Block server failure: Other servers pick up unfinished or pending jobs, if a block server fails.
- Cloud storage failure: S3 buckets are replicated multiple times in different regions. If files are not available in one region, they can be fetched from different regions.
- API server failure: Stateless service. Reroute request to other servers.
- Metadata cache failure: Replica.
- Metadata DB failure: Master/Slave + failover.
- Notification service failure
- Offline backup queue failure: Replica + re-subscribe


