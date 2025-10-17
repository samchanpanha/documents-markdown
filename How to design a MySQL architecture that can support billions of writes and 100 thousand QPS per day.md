# Interviewer: How to design a MySQL architecture that can support billions of writes and 100 thousand QPS per day?

**Original** **Fox** [Fox爱分享](javascript:void(0);)

 *September 27, 2025, 06:01*

**If you're a back-end engineer, you know that there's a huge gap between being able to write proficient SQL and being able to handle a highly concurrent, massive data system. This gap is "architecture design ability."**

**In today's interviews, especially for senior positions, the interviewer is no longer satisfied with investigating whether you know "why index uses B + tree." They want to know more about whether you can build a strong levee when the flow of floodwaters comes.**

**Below, we'll break down a high-pressure interview scene from a real business. It is not an isolated knowledge-based question and answer, but rather an ultimate test of the ability to design a system.**

### Scenes Coming: An Unavoidable Challenge

**Imagine you walk into an interview room, and after a brief exchange with the interviewer, he throws this question:**

**“Let's talk about a challenging scene. Suppose you are responsible for the database architecture design of a core system, which has several distinct characteristics:**

* **Business ** **: a national-level app**  **billing log system ** **, or similar to WeChat Moments**  **dynamic （ feed ） flow ** **.**
* **Write ** **: The pressure is enormous , and the number of new data added to the core table will reach** **100 million **lines per day .
* **Read ** **: equally amazing , peak QPS is expected to touch** **100,000 **times / second , read requests account for the absolute majority .
* **Data ** **: Typical time-series data, continuously growing, with few updates and deletions。The query dimensions are typically user ID and time range.**

**It's clear that a single MySQL instance is not going to cut it here。Now,** **please give us a complete optimization plan from the database architect's point of view, and tell us why you designed it this way。** **”**

![](https://mmbiz.qpic.cn/mmbiz_png/aSJ8tDK6zEvMSNh5QvU6JDZLP9SicHDzSU5n5xEHHXY2dOGCMAS2eR1SIX65CIHMbiawnb0slNhhTZfmRiaXmNvXg/640?wx_fmt=png&from=appmsg&watermark=1)

**The atmosphere became serious immediately when the question arose. This is no longer a "paper talk" about military force, but an architectural offensive and defensive battle with real guns.**

### Levels Dismantled: Deep Questioning of Five Core Issues

**An experienced interviewer will learn about your technical depth through a series of follow-up questions, step by step.**

#### The First Level: The Foundation Debate - Partitioning or Sharding?

**“ **Every day, 100 million new data, single table can not withstand is certain。To solve a storage bottleneck, do you plan to use Partitioning or Sharding? Tell me about your choice and your reasons.** **”****

**This question may seem like a binary choice, but it's actually a proposition.**

**An excellent answer should not hesitate to choose**  **the level table （Sharding） ** **。The reason is simple:** **the partitioned table** **essentially just stores the data blocks of a large table in different physical files according to a rule （e.g. month）, but all its data and indexes are still on** **the same server **Up。Facing the increment of 100 million per day and 36.5 billion per year, the disk I / O and capacity of any single machine will be quickly penetrated. **Partition table can not solve the problem of physical limit , and horizontal partition table is the " dragon technique " to disperse data and pressure to different servers .**

![](https://mmbiz.qpic.cn/mmbiz_png/aSJ8tDK6zEvMSNh5QvU6JDZLP9SicHDzS82J9NiaWBWibh12aQjrHIHaLlKheIbCe6yHXpiaGkmmKsicHljia3DKobIw/640?wx_fmt=png&from=appmsg&watermark=1)

**If you could go further and point out that partitioned tables become new bottlenecks in their metadata management itself when the volume of data is very large, that would undoubtedly be a plus.**

#### Level 2: Blueprint mapping - sharding keys, quantity, and architectural selection

**“ All right, now that we've set the level, let's break it down。So how exactly is this going to work?Sharding Key （ **Sharding Key** ）How do I choose? How many films are likely to be divided? Use client sharding or middleware sharding? **”****

**This question pushes the exploration point from "what to do" to "how to do it," hitting the details of the ground.**

**An answer that stands up to scrutiny needs to include the following:**

* **Shard Key ** **: Closely follow the clue that "queries are mostly based on user IDs" and choose** `<span class="js_darkmode__50 js_darkmode__text__131"><span leaf="" class="js_darkmode__text__132">user_id</span></span>`Being a shard key is logical. This ensures that data for the same user ends up in the same bucket, avoiding costly cross-bucket queries. Further, you also need to consider the potential "data skew" problem (such as large V users far more data than the average person), and give a preliminary idea of how to deal with it.
* **Number of shards ** **: This is not a feel-good number, but a process that needs to be quantified。As a rule of thumb, MySQL is generally considered to perform better with 50 million to 100 million rows per table.So a year would require** `<span class="js_darkmode__57 js_darkmode__text__142"><span leaf="" class="js_darkmode__text__143">(1亿/天 * 365天) / 5000万 ≈ 730</span></span>`A fragment. Given the ease of business growth and expansion, designing a solution that supports either 1024 or 2048 shards would be a more forward-looking option.
* **Architecture selection ** **: You need to clearly compare the pros and cons of the two mainstream options。Is the choice of integrated in the business code**  **client segmentation ** **（such as Sharding-JDBC）, it has little performance loss but business coupling；Or choose an independent**  **middleware sharding ** **（such as MyCAT）, which is transparent to the application but increases the complexity of the architecture and operation and maintenance costs。There is no absolute right or wrong choice here. The key is whether you can make a reasonable trade-off based on the technology stack of your team and the stage of your business development.**

#### Level 3: Peak Traffic - How to Handle 100 thousand QPS Gracefully?

**"The writes and storage were addressed, but there were 100 thousand read QPS. The database would definitely not be able to handle it. How are you going to design it to support this high readability?"**

**This is a central question about the system's overall throughput capacity.**

**The answer points to two classic weapons:** **read-write separation **and  **multi-level caching ** **。Your answer should not just be a pile of nouns, but a clear flow chart:**

1. **Read-write separation ** **: Within each shard library, a master-slave cluster is built。Write requests are only made to the main library, while a large number of read requests are shared by multiple libraries. Isn't there enough from the library? Yeah!This is the most direct way to horizontally scale read capacity.**

![](https://mmbiz.qpic.cn/mmbiz_png/aSJ8tDK6zEvMSNh5QvU6JDZLP9SicHDzSeS5uX946qOHrsAHBmPEJxaFCrIDO8PbRNiaWjcavicmSRicvHiabJK0JRw/640?wx_fmt=png&from=appmsg&watermark=1)

2. **Multi-level cache ** **: This is the first and most powerful defense against 100,000 QPS。A request journey should look like this:**

* **First access to the application server's**  **local cache ** **（L1 Cache，Caffeine, for example, is the fastest.**
* **If not, then accesses**  **the distributed cache cluster ** **（L2 Cache，Such as Redis).The vast majority of read requests should end up here.**
* **Only when both levels of the cache are "lost" will the request be allowed to "penetrate" to the final database layer.**

![](https://mmbiz.qpic.cn/mmbiz_png/aSJ8tDK6zEvMSNh5QvU6JDZLP9SicHDzSJYYiciaR3qqygib49laU0b6Bxs18WB4Y46fEowAIcaOcYjR7PN7AcczEQ/640?wx_fmt=png&from=appmsg&watermark=1)

#### Chapter 4: The Soul of Architecture - Caching Consistency and Hotspots Conundrum

**"When you talk about caching, you can't avoid consistency. How do you make sure the data is consistent between the cache and the database when the data changes? In addition, if a large V user is frequently visited, forming a hot spot, how will your cache architecture respond?"**

**If the previous question tested breadth, then this question tested the depth.**

* **Consistency ** **: You need to be able to articulate** **the Cache-Aside Pattern （Bypass cache） **This classic mode: read, first read the cache, then read the library, and then write back to the cache；When writing, first update the database, and then **remove **the cache。Why delete rather than update?Because the delete operation is more lightweight and can avoid data inconsistency in complex scenarios.
* **Hot problem ** **: This is almost the "Achilles heel" of all high-concurrency systems。You need to be like an experienced soldier who can accurately identify and dismantle the three "enemies":**
* **Cache penetration ** **（ looking up data that does not exist ） : use a bloom filter at the entry point .**
* **Cache flushes ** **（ hot keys suddenly fail ） : use a distributed lock to ensure that only one thread requests the database at a time when the cache is rebuilt .**
* **Cache avalanche ** **（ lots of keys expire at the same time ） : avoid by adding random value to cache expiry time .**

#### The ultimate test: vision and landing - historical data and online change

**“In a few years, the data will be petabytes. Those logs from 3 years ago, access frequency is extremely low, but they have been placed in MySQL at a high cost. What will you do with this cold data? Also, in case the business needs to add a field, how do you do DDL operations on so many shards?”**

**The question examines the architect's vision, cost awareness and ability to "change engines on a high-speed plane."**

* **Cold and hot separation ** **: You need to demonstrate a data lifecycle management mindset。Archive "cold data" （data that will not be accessed for a certain period of time, such as a year） from expensive MySQL clusters to cheaper storage systems such as** **HBase, ClickHouse **, or object storage （OSS/S3） from cloud providers。This is not only to reduce the burden on the core library, but also to save money for the company.

![](https://mmbiz.qpic.cn/mmbiz_png/aSJ8tDK6zEvMSNh5QvU6JDZLP9SicHDzS4yYm2cgsbd1htlekb2ibVAR1f3kDQUXCuUDcqgQ31Bv7WHE2GPxZicXQ/640?wx_fmt=png&from=appmsg&watermark=1)

* **OnlineDDL ** **：Execute directly on the sharded large table** `<span class="js_darkmode__125 js_darkmode__text__306"><span leaf="" class="js_darkmode__text__307">ALTER TABLE</span></span>`It was a catastrophic operation. You have to know that this needs help. `<span class="js_darkmode__130 js_darkmode__text__311"><span leaf="" class="js_darkmode__text__312">gh-ost</span></span>`or `<span class="js_darkmode__135 js_darkmode__text__316"><span leaf="" class="js_darkmode__text__317">pt-online-schema-change</span></span>`Such a professional tool can complete smooth table structure changes by creating a "shadow table" without locking the main table and without affecting online business.

![](https://mmbiz.qpic.cn/mmbiz_png/aSJ8tDK6zEvMSNh5QvU6JDZLP9SicHDzSHkBGUXxM65YfkgS1w0yiblPMsfJb2uPgiaIr67MdcC3qHIIgCQvxRYQA/640?wx_fmt=png&from=appmsg&watermark=1)

### Conclusion: From "respondent" to "architect"

**If you can get through all of these questions clearly and methodically, then what you're demonstrating is not a "code implementer" skill, but a global perspective, weighing ability, and hands-on experience necessary for a good architect.**

**On the road to technology, real growth is often reflected in these point-to-point shifts in thinking. And a remarkable system is precisely about every "if..." It was carefully honed in deep reflection.**

**If you think it is useful ,**  **give a like and save it ** **, maybe you will use it in the next interview**
