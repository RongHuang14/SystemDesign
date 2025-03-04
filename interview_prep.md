# Interviews steps
## Step 1: Requirements clarifications
clarify what parts of the system we will be focusing on.
必须理清需要解决的问题的详细需求.系统设计问题由于是开放式的，没有唯一确定的答案，如果我们对问题需求理解存在偏差，那么整个面试就很难获得成功。而且面试时间通常只有35-40分钟，因此我们需要把精力集中在重点模块上。

Example: designing a Twitter-like service
• Will users of our service be able to post tweets and follow other people?
• Should we also design to create and display the user’s timeline?
• Will tweets contain photos and videos?
• Are we focusing on the backend only or are we developing the front-end too?
• Will users be able to search tweets?
• Do we need to display hot trending topics?
• Will there be any push notification for new (or important) tweets?

• 是否需要支持用户发推特以及关注其他人
• 是否需要展示用户的活动时间线
• 推特内容是否需要包含照片和视频
• 除了后端，前端是否也需要考虑
• 是否支持用户搜索推特内容
• 是否需要展示热点趋势话题
• 新推特是否需要支持通知

## Step 2: System interface definition
Define what APIs.这一步需要定义出系统中的核心接口。定义出接口能够准确体现系统不同模块的交互，还能review看看我们理解是否和问题需求有偏差。
Examples for our Twitter-like service will be:
• postTweet(user_id, tweet_data, tweet_location, user_location, timestamp, ...)
• generateTimeline(user_id, current_time, user_location, ...)
• markTweetFavorite(user_id, tweet_id, timestamp, ...)


## Step 3: Back-of-the-envelope estimation
estimate the scale of the system we’re going to design.This will also help later when we will be focusing on scaling, partitioning, load balancing and caching.
估计出系统需要支持的量级，在后续进行伸缩性、分片、负载均衡和缓存设计的时候会用到：
• What scale is expected from the system (e.g., number of new tweets, number of tweet views, number of timeline generations per sec., etc.)?
• How much storage will we need? We will have different numbers if users can have photos and videos in their tweets.
• What network bandwidth usage are we expecting? This will be crucial in deciding how we will manage traffic and balance load between servers.

• 系统需要支持的业务流量量级（发新推的量级、推特的查看量级等）
• 存储空间大小（是否需要支持照片和视频对存储空间大小评估影响很大）
• 网络带宽

## Step 4: Defining data model
Defining the data model early will clarify how data will flow among different components of the system. Later, it will guide towards data partitioning and management. The candidate should be able to identify various entities of the system, how they will interact with each other, and different aspect of data management like storage, transportation, encryption, etc. 
即领域模型定义。通过定义数据模型，明确数据在系统不同模块之间的流向变得清晰，也在后续的数据分片和管理上起到指导作用。我们需要确定出系统中的不同实体、相互之间怎么交互、以及和存储、传输、加密等数据管理设施之间的关系。

Here are some entities for our Twitter- like service:
• User: UserID, Name, Email, DoB, CreationData, LastLogin, etc.
• Tweet: TweetID, Content, TweetLocation, NumberOfLikes, TimeStamp, etc. 
• UserFollowo: UserdID1, UserID2
• FavoriteTweets: UserID, TweetID, TimeStamp

在分布式系统设计中，通常会讨论不同类型的数据应该如何存储：
	•	结构化数据（如用户信息、推文文本） → 通常存储在**关系型数据库（MySQL, PostgreSQL）或NoSQL数据库（Cassandra, DynamoDB）**中。
	•	非结构化数据（如照片、视频、音频） → 通常存储在**对象存储（Object Storage）或块存储（Block Storage）**中。

## Step 5: High-level design
Draw a block diagram with 5-6 boxes representing the core components of our system.
通过图示画出系统中的核心模块。
For Twitter, at a high-level, we will need multiple application servers to serve all the read/write requests with load balancers in front of them for traffic distributions. If we’re assuming that we will have a lot more read traffic (as compared to write), we can decide to have separate servers for handling these scenarios. On the backend, we need an efficient database that can store all the tweets and can
support a huge number of reads. We will also need a distributed file storage system for storing photos and videos.
类Twitter系统的概要设计图如下。如果流量较大，可能需要考虑采用多台服务器，还需要负载均衡设备。另外还可以考虑读写分离。



## Step 6: Detailed design
## Step 7: Identifying and resolving bottlenecks
