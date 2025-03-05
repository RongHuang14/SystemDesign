# Interviews Steps

## Step 1: Requirements Clarifications

Clarify what parts of the system we will be focusing on.

必须理清需要解决的问题的详细需求。系统设计问题由于是开放式的，没有唯一确定的答案，如果我们对问题需求理解存在偏差，那么整个面试就很难获得成功。而且面试时间通常只有 35-40 分钟，因此我们需要把精力集中在重点模块上。

**Example: designing a Twitter-like service**
- Will users of our service be able to post tweets and follow other people?
- Should we also design to create and display the user’s timeline?
- Will tweets contain photos and videos?
- Are we focusing on the backend only, or are we developing the front-end too?
- Will users be able to search tweets?
- Do we need to display hot trending topics?
- Will there be any push notification for new (or important) tweets?

**问题澄清**
- 是否需要支持用户发推特以及关注其他人？
- 是否需要展示用户的活动时间线？
- 推特内容是否需要包含照片和视频？
- 除了后端，前端是否也需要考虑？
- 是否支持用户搜索推特内容？
- 是否需要展示热点趋势话题？
- 新推特是否需要支持通知？

---

## Step 2: System Interface Definition

Define what APIs. 这一步需要定义出系统中的核心接口。定义出接口能够准确体现系统不同模块的交互，还能 review 看看我们理解是否和问题需求有偏差。

**Examples for our Twitter-like service:**
- `postTweet(user_id, tweet_data, tweet_location, user_location, timestamp, ...)`
- `generateTimeline(user_id, current_time, user_location, ...)`
- `markTweetFavorite(user_id, tweet_id, timestamp, ...)`

---

## Step 3: Back-of-the-Envelope Estimation

Estimate the scale of the system we’re going to design. This will also help later when we will be focusing on scaling, partitioning, load balancing, and caching.

估计出系统需要支持的量级，在后续进行伸缩性、分片、负载均衡和缓存设计的时候会用到：

- What scale is expected from the system (e.g., number of new tweets, number of tweet views, number of timeline generations per sec., etc.)?
- How much storage will we need? We will have different numbers if users can have photos and videos in their tweets.
- What network bandwidth usage are we expecting? This will be crucial in deciding how we will manage traffic and balance load between servers.

**系统规模估算**
- 系统需要支持的业务流量量级（发新推的量级、推特的查看量级等）
- 存储空间大小（是否需要支持照片和视频对存储空间大小评估影响很大）
- 网络带宽

---

## Step 4: Defining Data Model

Defining the data model early will clarify how data will flow among different components of the system. Later, it will guide towards data partitioning and management. The candidate should be able to identify various entities of the system, how they will interact with each other, and different aspects of data management like storage, transportation, encryption, etc.

即领域模型定义。通过定义数据模型，明确数据在系统不同模块之间的流向变得清晰，也在后续的数据分片和管理上起到指导作用。我们需要确定出系统中的不同实体、相互之间怎么交互，以及和存储、传输、加密等数据管理设施之间的关系。

**Here are some entities for our Twitter-like service:**
- **User**: UserID, Name, Email, DoB, CreationData, LastLogin, etc.
- **Tweet**: TweetID, Content, TweetLocation, NumberOfLikes, TimeStamp, etc.
- **UserFollow**: UserID1, UserID2
- **FavoriteTweets**: UserID, TweetID, TimeStamp

**在分布式系统设计中，通常会讨论不同类型的数据应该如何存储：**
- **结构化数据**（如用户信息、推文文本）→ 通常存储在 **关系型数据库（MySQL, PostgreSQL）** 或 **NoSQL 数据库（Cassandra, DynamoDB）** 中。
- **非结构化数据**（如照片、视频、音频）→ 通常存储在 **对象存储（Object Storage）或块存储（Block Storage）** 中。

---

## Step 5: High-Level Design

Draw a block diagram with 5-6 boxes representing the core components of our system.

通过图示画出系统中的核心模块。

For Twitter, at a high level, we will need multiple application servers to serve all the read/write requests with load balancers in front of them for traffic distribution. If we’re assuming that we will have a lot more read traffic (as compared to write), we can decide to have separate servers for handling these scenarios. On the backend, we need an efficient database that can store all the tweets and can support a huge number of reads. We will also need a distributed file storage system for storing photos and videos.

类 Twitter 系统的概要设计图如下：
<img width="714" alt="image" src="https://github.com/user-attachments/assets/80fb53fb-e081-4c20-87af-7d3aac33fbc3" />
<img width="902" alt="image" src="https://github.com/user-attachments/assets/90312430-8ad8-44dd-951e-01d89fe104af" />

- 如果流量较大，可能需要考虑采用多台服务器，还需要负载均衡设备。
- 可能需要读写分离，即：
  - **写操作** 由主数据库（Primary DB）处理
  - **读操作** 由多个从数据库（Read Replica）处理
- 图片和视频可能需要使用 **分布式存储（如 S3、HDFS）**
- 使用 **CDN（内容分发网络）** 来提高媒体加载速度

---

## Step 6: Detailed Design  
Dig deeper into two or three components; interviewer’s feedback should always guide us on what parts of the system need further discussion. We should be able to present different approaches, their pros and cons, and explain why we prefer one approach over the other. The important thing is to consider trade-offs between different options while keeping system constraints in mind.  
对核心模块进行深挖，进行深入讨论。可能会有多种方案，我们需要对多种方案的优缺点进行权衡，并考虑系统的约束条件。

- **Data Partitioning and Distribution**  
  **Q: Since we will be storing a massive amount of data, how should we partition our data to distribute it to multiple databases? Should we try to store all the data of a user on the same database? What issues could it cause?**  
  **数据分区与分布**  
  **问题：由于我们需要存储大量数据，应该如何对数据进行分区并分布到多个数据库？是否应该将用户的所有数据存储在同一个数据库？这样做可能会引发哪些问题？**  

  - **Approach 1: Partition by UserID**  
    - ✅ Easy to fetch all user-related data  
    - ❌ Hot users (high activity accounts) can overload a single database  
  - **Approach 2: Partition by TweetID (Sharding by time or hash)**  
    - ✅ Better load balancing  
    - ❌ Requires joining multiple databases to retrieve all user data  

  **方案 1：按 UserID 分区**  
  - ✅ 方便查询用户的所有数据  
  - ❌ 热点用户（高活跃账户）可能导致数据库负载过载  

  **方案 2：按 TweetID 进行分片（按时间或哈希分片）**  
  - ✅ 负载均衡更好  
  - ❌ 需要跨多个数据库查询用户所有数据  

- **Handling Hot Users**  
  **Q: How will we handle hot users who tweet a lot or follow lots of people?**  
  **如何处理高活跃用户（热点用户）？**  

  - **Solution 1: Cache hot users’ data using Redis**  
    - ✅ Fast read performance  
    - ❌ Cache invalidation complexity  
  - **Solution 2: Distribute hot users’ data across multiple shards**  
    - ✅ Avoids database overload  
    - ❌ Increases query complexity  

  **方案 1：使用 Redis 缓存热点用户数据**  
  - ✅ 提高查询速度  
  - ❌ 需要处理缓存失效问题  

  **方案 2：将热点用户数据分布到多个分片**  
  - ✅ 避免数据库过载  
  - ❌ 增加查询复杂度  

- **Optimizing Timeline Storage**  
  **Q: Since users’ timeline will contain the most recent (and relevant) tweets, should we try to store our data in such a way that is optimized for scanning the latest tweets?**  
  **优化用户时间线存储**  
  **问题：由于用户时间线主要包含最新的推文，我们是否应该对存储结构进行优化，以便高效查询最新推文？**  

  - **Fan-out on Write (Precompute timelines)**  
    - ✅ Faster reads for users  
    - ❌ High write amplification  
  - **Fan-out on Read (Compute on demand)**  
    - ✅ Lower write cost  
    - ❌ Slower reads  

  **写时扩散（预计算时间线）**  
  - ✅ 读取速度更快  
  - ❌ 写入成本高（写放大问题）  

  **读时扩散（按需计算时间线）**  
  - ✅ 写入开销小  
  - ❌ 读取时延较高  

- **Caching Strategy**  
  **Q: How much and at which layer should we introduce cache to speed things up?**  
  **缓存策略：在哪些层引入缓存以提高速度？**  

  - **Layer 1: Redis caching for hot data (recent tweets, hot users)**  
    - ✅ Reduces database load  
    - ❌ Cache invalidation needed  
  - **Layer 2: CDN for images and videos**  
    - ✅ Low latency for media content  
    - ❌ Storage cost  

  **缓存层 1：使用 Redis 缓存热点数据（如热门推文、热点用户）**  
  - ✅ 降低数据库压力  
  - ❌ 需要缓存失效管理  

  **缓存层 2：使用 CDN 缓存图片和视频**  
  - ✅ 提高媒体加载速度  
  - ❌ 存储成本较高  

- **Load Balancing Strategy**  
  **Q: What components need better load balancing?**  
  **哪些组件需要更好的负载均衡？**  

  - **API Layer: Nginx + Round Robin**  
    - ✅ Distributes requests across servers  
  - **Database Layer: Read Replicas + Sharding**  
    - ✅ Handles high query loads  
  - **Cache Layer: Redis Cluster**  
    - ✅ Ensures high availability  

  **API 层：Nginx + 轮询策略（Round Robin）**  
  - ✅ 均衡请求流量  

  **数据库层：读写分离（Read Replicas）+ 数据分片（Sharding）**  
  - ✅ 处理高并发查询  

  **缓存层：Redis 集群**  
  - ✅ 提高系统高可用性  

- **Sharding (Data Partitioning)**
  **Q: What is sharding, and how should we use it for large-scale distributed systems?**  
  **分片（Sharding）是什么？在大规模分布式系统中如何使用它？**  

  - **Definition**  
    - Sharding is a method of distributing data across multiple databases to improve performance and scalability.  
    - **分片是一种将数据分布到多个数据库的方法，以提高性能和可扩展性。**

  - **Common Sharding Strategies**  
    - **UserID-based Sharding (Range-based Sharding)**  
      - ✅ Easy to fetch all user-related data  
      - ❌ Hot users may overload a single database  
    - **TweetID-based Sharding (Hash-based or Time-based Sharding)**  
      - ✅ Better load balancing  
      - ❌ Querying all user data requires multiple database joins  

  **常见的分片策略**  
  - **基于 UserID 分片（按范围 Range-based Sharding）**  
    - ✅ 方便查询用户的所有数据  
    - ❌ 热点用户可能导致数据库负载不均衡  
  - **基于 TweetID 分片（按哈希或时间 Sharding）**  
    - ✅ 负载均衡更好  
    - ❌ 需要跨多个数据库查询用户的所有数据  

  - **Real-world Application (Example: Twitter-like system)**  
    - **User Data Sharding**: Partition by UserID to keep all user-related data together  
    - **Tweet Data Sharding**: Partition by TweetID to distribute load evenly across servers  
    - **Kafka Sharding**: Distribute message queues into different partitions for scalability  

  **实际应用（类 Twitter 系统）**
  - **用户数据分片**：按 UserID 进行分片，保证用户数据存储的一致性  
  - **推文数据分片**：按 TweetID 进行分片，确保数据库负载均衡  
  - **Kafka 消息队列分片**：将消息队列拆分到多个分区，以提高系统可扩展性  

- **CDN (Content Delivery Network)**
  **Q: What is a CDN, and why is it important for large-scale systems?**  
  **CDN（内容分发网络）是什么？为什么在大规模系统中很重要？**  

  - **Definition**  
    - A CDN is a distributed network of servers that helps deliver static content (e.g., images, videos, CSS, JavaScript) efficiently to users.  
    - **CDN 是一组分布式服务器网络，用于高效传输静态内容（如图片、视频、CSS、JavaScript）。**  

  - **Benefits of Using a CDN**  
    - **Faster content delivery**: Serves users from the nearest CDN node  
    - **Reduces server load**: Offloads traffic from the origin server  
    - **Better scalability**: Handles large amounts of traffic efficiently  

  **CDN 的优势**
  - **加速内容交付**：通过最近的 CDN 节点向用户提供内容  
  - **减少服务器负载**：减少对原始服务器的直接请求  
  - **提高系统扩展性**：有效处理大规模流量  

  - **CDN Usage in a Twitter-like System**  
    - **User Profile Pictures**: Cached in CDN for fast retrieval  
    - **Tweet Images/Videos**: Stored and delivered via CDN to reduce bandwidth usage  
    - **Static Web Resources (CSS/JS)**: Served through CDN to enhance page load speed  

  **CDN 在类 Twitter 系统中的应用**
  - **用户头像**：缓存到 CDN 以提高访问速度  
  - **推文中的图片/视频**：通过 CDN 存储和分发，减少服务器带宽压力  
  - **网页静态资源（CSS/JS）**：通过 CDN 传输，提高页面加载速度  

## Step 7: Identifying and Resolving Bottlenecks  
Identify system bottlenecks and suggest optimizations.  
讨论系统中可能的瓶颈，并讨论解决方案。

- **Single Point of Failure (SPOF)**
  **Q: Is there any single point of failure in our system? What are we doing to mitigate it?**  
  **系统是否存在单点故障？如何解决这个问题？**  

  - **Issue**: If a single server, database, or service instance fails, the entire system may become unavailable.  
  - **Solution**:  
    - **Primary-Replica Database Architecture**: Use read replicas to handle traffic and ensure failover.  
    - **Multi-Region Deployment**: Replicate data across multiple data centers.  
    - **Load Balancing**: Use Nginx, AWS ALB, or HAProxy to distribute traffic dynamically.  

  **问题**：如果某个服务器、数据库或服务实例发生故障，整个系统可能不可用。  
  **解决方案**：  
  - **数据库主备架构**：使用主从复制（Primary-Replica）数据库，确保主库故障时可以自动切换到从库。  
  - **多区域部署**：跨数据中心复制数据，防止单点崩溃影响整个系统。  
  - **负载均衡**：使用 **Nginx、AWS ALB、HAProxy** 进行流量分发，确保高可用性。  

- **Data Replication**
  **Q: Do we have enough replicas of the data so that if we lose a few servers, we can still serve our users?**  
  **我们的数据是否有足够的副本？如果部分服务器宕机，系统是否仍然可用？**  

  - **Issue**: If data is stored in a single place, losing the server means data loss and service disruption.  
  - **Solution**:  
    - **Database Replication (Primary-Replica, Multi-Master, Sharded DBs)**  
    - **Distributed Storage (Cassandra, MongoDB, DynamoDB with multi-region replication)**  
    - **Cloud Storage (AWS S3, Google Cloud Storage) with built-in redundancy**  

  **问题**：如果数据存储在单个服务器上，服务器宕机可能导致数据丢失或服务中断。  
  **解决方案**：  
  - **数据库副本（主从复制、多主架构、分片数据库）**：保证数据高可用性。  
  - **分布式存储（Cassandra, MongoDB, DynamoDB 多区域复制）**：分布数据以增强容错能力。  
  - **云存储（AWS S3, GCS）**：自动复制数据，防止数据丢失。  

- **Service Redundancy**
  **Q: Do we have enough copies of different services running such that a few failures will not cause total system shutdown?**  
  **我们的微服务架构是否有足够的冗余？如果部分服务宕机，系统是否仍然可用？**  

  - **Issue**: If a critical microservice crashes, it could impact the entire system.  
  - **Solution**:  
    - **Kubernetes (K8s) for Auto-Scaling & Redundancy**  
    - **API Load Balancing with Nginx or AWS ALB**  
    - **Using Multiple Service Instances with Auto Recovery**  

  **问题**：如果某个关键微服务崩溃，可能导致整个系统部分功能瘫痪。  
  **解决方案**：  
  - **使用 Kubernetes 进行自动扩展和冗余部署**，确保服务实例动态扩容和故障恢复。  
  - **API 层负载均衡（Nginx, AWS ALB）**，避免单个 API 服务器负载过高。  
  - **多个服务实例（Auto Recovery）**，确保某些实例宕机时，系统仍然可以正常运行。  

- **Monitoring & Alerting**
  **Q: How are we monitoring the performance of our service? Do we get alerts whenever critical components fail or their performance degrades?**  
  **如何监控系统的性能？关键组件发生故障或性能下降时，我们是否会收到警报？**  

  - **Issue**: If a system component fails or slows down, there should be real-time monitoring.  
  - **Solution**:  
    - **Log Monitoring (ELK Stack, Splunk, AWS CloudWatch Logs)**  
    - **Metrics Monitoring (Prometheus + Grafana, Datadog, New Relic)**  
    - **Automated Alerts (PagerDuty, AWS SNS, OpsGenie)**  

  **问题**：如果某个系统组件故障或变慢，我们需要实时监控和告警。  
  **解决方案**：  
  - **日志监控（ELK Stack, Splunk, AWS CloudWatch）**，实时分析日志数据。  
  - **指标监控（Prometheus + Grafana, Datadog）**，监控 CPU、内存、请求速率、数据库性能。  
  - **自动告警（PagerDuty, AWS SNS, OpsGenie）**，异常情况时触发警报，通知运维人员。  

## **Summary Table**
| **Bottleneck** | **Potential Risk** | **Solution** |
|---------------|------------------|------------|
| **Single Point of Failure (SPOF)** | A single server crash can bring down the system | Multi-region deployment, Load Balancing, Database Replication |
| **Data Replication** | Losing a few servers may result in data loss | Primary-Replica DBs, Distributed Storage, Cloud Storage |
| **Service Redundancy** | Crashing of critical microservices | Kubernetes Auto Scaling, API Load Balancing |
| **Lack of Monitoring** | No alerts when performance degrades | Log Monitoring, Metrics Tracking, Automated Alerts |
---

# Examples
## Examples 1：High-Concurrency Shopping Cart System 高并发购物车系统设计

## Step 1: Requirements Clarifications 需求澄清
Clarify what parts of the system we will be focusing on.  
必须理清需要解决的问题的详细需求。

### **Scenario 场景**
Design an online shopping cart system that supports millions of users operating simultaneously. The system must ensure:  
设计一个支持数百万用户同时操作的在线购物车系统，系统必须确保：
- **Real-time synchronization 实时同步**: Shopping cart data is updated across multiple devices in real-time.  
  购物车数据需在不同设备间实时同步。
- **Data consistency 数据一致性**: Handling concurrent updates to prevent conflicts when multiple users modify the same product quantity.  
  处理并发更新，防止多个用户修改同一商品数量时发生冲突。
- **High availability 高可用性**: The system must handle extreme traffic conditions, such as flash sales.  
  需要应对极端高并发情况，如秒杀活动。

### **Key Questions 关键问题**
- Should the system support real-time cart synchronization across multiple devices?  
  系统是否需要支持跨设备购物车实时同步？
- How do we handle concurrent updates to the same cart item?  
  如何处理多个用户同时修改同一购物车商品的问题？
- Should we optimize for high-traffic events like flash sales?  
  是否需要针对秒杀等高并发场景进行优化？
- Do we need to consider both frontend and backend scalability?  
  是否需要考虑前后端的可扩展性？

---

## Step 2: System Interface Definition 系统接口定义
Define core system APIs to ensure proper interaction between components.  
定义核心 API，确保系统不同模块之间的交互。

### **APIs for Shopping Cart System 购物车系统 API**
- `addToCart(user_id, item_id, quantity, timestamp, device_id, …)`  
- `removeFromCart(user_id, item_id, …)`  
- `updateCart(user_id, item_id, new_quantity, version_number, …)`  
- `checkout(user_id, cart_id, payment_info, …)`  
- `syncCart(user_id, device_id, last_sync_time, …)`

---

## Step 3: Back-of-the-Envelope Estimation 粗略估算
Estimate system scale for scalability, partitioning, and caching.  
估算系统的流量和存储需求，以指导后续的伸缩性、分片和缓存设计。

### **Estimation Factors 估算因素**
- **Active users per second 并发用户数**: 1M concurrent users (100 万用户并发)
- **Shopping cart updates per second 购物车更新频率**: 500K updates/sec (50 万次/秒)
- **Read-heavy operations 读多写少**: Cart retrieval should be optimized (优化购物车读取)
- **Storage requirements 存储需求**:  
  - If each cart stores 1KB, with 100M active users → **100GB of storage per session**  
    如果每个购物车占 1KB，活跃用户 1 亿，则单次存储需求为 **100GB**

---

## Step 4: Defining Data Model 数据模型定义
Define entities and relationships to guide database design.  
定义数据模型，明确系统中的关键实体及交互方式。

### **Entities 实体**
- **User 用户**: `UserID, Name, Email, LastLogin, …`
- **Cart 购物车**: `CartID, UserID, Timestamp, Version, Items[]`
- **CartItem 购物车商品**: `ItemID, CartID, Quantity, LastUpdated, Version`
- **Order 订单**: `OrderID, CartID, PaymentStatus, CreatedAt`

### **Storage Considerations 存储考虑**
- **Structured data 结构化数据 (cart, users, orders)** → **Stored in SQL/NoSQL databases (存入 SQL/NoSQL 数据库)**
- **Session data 会话数据 (real-time updates)** → **Stored in Redis/Memcached (存入 Redis/Memcached)**
- **Persistent logs 永久日志 (cart history, checkout logs)** → **Stored in distributed storage (S3, HDFS) (存入分布式存储，如 S3, HDFS)**

---

## Step 5: High-Level Design 高层架构设计
Draw a block diagram with core system components.  
画出系统架构的高层设计，定义核心模块。

### **System Architecture 系统架构**
1. **Frontend (Web/Mobile App) 前端 (网页/移动应用)**
2. **API Gateway (Rate limiting, Authentication) API 网关 (限流，认证)**
3. **Shopping Cart Service 购物车服务**
   - **WebSocket for real-time updates** (基于 WebSocket 进行实时同步)
   - **DynamoDB for cart storage** (使用 DynamoDB 存储购物车数据)
   - **Redis/Memcached for caching** (使用 Redis/Memcached 进行缓存)
4. **Checkout & Order Processing 结算和订单处理**
   - **Queue-based order processing (SQS/Kafka)** (基于消息队列 (SQS/Kafka) 处理订单)
   - **Payment Service** (支付服务)
5. **Analytics & Monitoring 分析和监控**
   - **Logging & Metrics (ELK, Prometheus)** (使用 ELK, Prometheus 进行日志和指标监控)

---

## Step 6: Detailed Design 详细设计
### **Concurrency Control: Resolving Conflicts 并发控制：解决冲突**
**Problem 问题:** How do we handle two users modifying the same cart item at the same time?  
**如何处理两个用户同时修改购物车商品的问题？**  
**Solution 解决方案:**  
- **Vector Clocks / Versioning 向量时钟/版本控制**: Assign a **version number** to each cart update. If a client tries to update an outdated version, reject the update.  
  为每次购物车更新分配**版本号**，如果客户端更新的版本过时，则拒绝该更新。
- **DynamoDB Conditional Writes 条件写入**: Only allow updates if the **server version matches the client version**.  
  仅允许当**服务器版本匹配客户端版本**时执行更新。

### **Real-time Synchronization 实时同步**
- **WebSocket (AWS API Gateway WebSocket)**  
- **DynamoDB Streams + Lambda for data synchronization**  
  (使用 DynamoDB Streams + Lambda 进行数据同步)

### **Handling Flash Sales (High Traffic Optimization) 处理秒杀流量优化**
- **API Rate Limiting API 限流**: Limit excessive cart updates per user.  
  限制每个用户的购物车更新频率。
- **Preloading Hot Items in Redis 热门商品预加载**: Store frequently bought items in **cache** to avoid database overload.  
  将热门商品预加载到缓存中，减少数据库压力。
- **Asynchronous Order Processing 异步订单处理**: Use **SQS/Kafka** to queue orders and process them asynchronously.  
  使用 SQS/Kafka 队列异步处理订单。

---

## Step 7: Identifying and Resolving Bottlenecks 识别和解决瓶颈
### **Single Point of Failure 单点故障**
**Issue 问题:** If a service component crashes, the system may be unavailable.  
**如果某个服务组件崩溃，系统可能无法使用。**  
**Solution 解决方案:**  
- **Multi-region Deployment** (多区域部署)
- **Load Balancing with Nginx, AWS ALB** (使用 Nginx, AWS ALB 进行负载均衡)
- **Primary-Replica DB for automatic failover** (主从数据库自动故障转移)

### **Monitoring & Alerting 监控与告警**
- **ELK Stack for log monitoring** (使用 ELK 监控日志)
- **Prometheus + Grafana for real-time metrics** (使用 Prometheus + Grafana 进行实时监控)
- **PagerDuty for automated alerts** (使用 PagerDuty 进行自动告警)
