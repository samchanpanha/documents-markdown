# Talk about the five Redis deployment patterns

**Original** **Yong Ge** [Yong Ge Java Practice](javascript:void(0);)

 *September 22, 2025, 07:30*

**In this article, I share five Redis deployment patterns that I have experienced in my career, hoping to inspire everyone.**

# 1 Single Instance

**This is the simplest and most basic way to deploy Redis : the entire Redis service runs on** **a single server **and  **single process ** **.**

**The first time I used Redis in the production environment, it was in the eLong red envelope system, using Redis to implement distributed locks.**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/V71JNV78n2ibiaQI3WG5GFrpRC9ibIjbsSqdiaX1hmeAsscO0ABadzarykpAXiaibVvTPbKV7a6CgyYibvjFjo7bYwDEA/640?wx_fmt=png&from=appmsg)

**Because the launch time requirement was relatively anxious, operations said that one instance could be used without application, so the single instance model was adopted. I also specifically told O & M that if Redis hangs, it will restart through Linux timing tasks.**

**The advantages of a single instance model are obvious :**  **simple （ deployment , configuration , maintenance ） ** **, but the disadvantages are equally prominent : if the server goes down , the service is completely unavailable , and the memory size is limited by the server .**

# 2 Master + Sentinel

**After the initial version of the Yilong red envelope system was launched, the team architect introduced me to the high-availability solution of Redis -** **the master-slave replication + sentinel cluster **mode。This deployment mode realizes data backup through master-slave data synchronization, and with the automatic fault detection and master-slave switching capability of the Sentinel cluster, it can effectively ensure the high availability of the service.

**In the architecture shown in the diagram:**

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/V71JNV78n2ibiaQI3WG5GFrpRC9ibIjbsSqUvMP5Mt76UflpjbHdQavXmfciabYVbGnkiae7oocWrcCNeSHvAibdYwvg/640?wx_fmt=jpeg&from=appmsg)

1. **The main node is responsible for handling all write requests**
2. **Main node data is synchronized from nodes in real time, and read requests can be shared**
3. **Sentinels continuously monitor the state of health of peers**
4. **When the main node fails, the sentinel automatically elects a new main node**

**Through this transformation, the cache architecture of the redback system has been improved: not only avoided the single point of failure risk, but also read and write separation, and the overall system stability and usability have been significantly enhanced. Even in the case of sudden failure, it can ensure the continuous and stable operation of red envelope business.**

# 3-Part Cluster + Consistency Hash

**The "master-slave + sentinel" model is very robust, but if the cache data volume is very large, this model has a bottleneck, so multiple sets of Redis instances are required to meet the business requirements.**

**The computing process of eLong's streaming computing service relies heavily on this multi-Redis instance model, as shown below:**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/V71JNV78n2ibiaQI3WG5GFrpRC9ibIjbsSqRrS3sdv5biazuiaezQfXvb4mRnFVKe34lrwE3CfvpicPooPj69xFFwg3g/640?wx_fmt=png&from=appmsg)

**We can use** **the Consistent Hash Algorithm **to achieve data sharding :

![](https://mmbiz.qpic.cn/sz_mmbiz_png/V71JNV78n2ibiaQI3WG5GFrpRC9ibIjbsSqmgRia56Fiaosiam2tZlSyGyibC0pEuqkyZrXQDOUoIZicVaiat7hs1bOaMfA/640?wx_fmt=png&from=appmsg)

1. **Hash ring construction ** **: organizes the entire hash space （ 0 ~ 2 ^ 32 - 1 ） into a ring structure .**
2. **Node mapping ** **: Compute the hash values of multiple virtual nodes （ typically 200-300 ） for each Redis node and evenly distribute them across the ring .**
3. **Data routing ** **: Compute the hash value of each key and find the nearest node clockwise on the ring .**

**The Redis cluster of streaming computing only adopts the single master cluster mode, which has certain high availability risks, such as a shard hanging, and the whole system will have problems.**

**The solution is actually simple:**

* **Each shard is in a master mode.**
* **Sentinel cluster monitoring (automated switching of master from master)**

**The architecture diagram becomes the cache deployment architecture of the following diagram (the cache deployment architect for the Shinju Express orders):**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/V71JNV78n2ibiaQI3WG5GFrpRC9ibIjbsSqdIVs1vVGTo0VCbiakIK8T04rgCx5LDh83Ynm1cdtP7sKS6ONdhMJZWQ/640?wx_fmt=png&from=appmsg)

# 4 sharded clusters + pre-allocation

**When we look at the "sharded cluster + consistent hash" model, although it looks perfect, there is a hidden disadvantage:**

**When a new shard is added, how can the data be smoothly migrated to the new shard nodes?**

**The most efficient way to solve this problem is to :**  **pre-allocate slots ** **.**

**I have introduced the car sub-database sub-table algorithm, assuming that the order table now needs to be divided into 4 sub-database shard0, shard1, shard2, shard3.**

**First , divide 【 0-1023 】 into 4 segments : 【 0-255 】 , 【 256-511 】 , 【 512-767 】 , 【 768-1023 】 , then hash the string （ or substring , user-defined ） , and the hash result is mod 1024 , and the final result is** **slot **The segment to which it falls is the segment to which it is routed.

![](https://mmbiz.qpic.cn/sz_mmbiz_png/V71JNV78n2ibiaQI3WG5GFrpRC9ibIjbsSquf4yuiaysIicVHwtyhEGtFyEibuhabGEmDnwzQpB7LTDGoWibDqN7IkCdQ/640?wx_fmt=png&from=appmsg)

**We can apply the pre-allocation theory of sub-library and sub-table to a Redis shard cluster, as shown below:**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/V71JNV78n2ibiaQI3WG5GFrpRC9ibIjbsSqibE7cx60BmWFRib0cnVj75qF0Hjv0Jxo9vcUPPQzib4XanFk4DuhBVhmA/640?wx_fmt=png&from=appmsg)

**The well-known open source project Codis also uses the technique of pre-allocating ,** **" Sharded cluster + pre-allocation " can not only retain the scalable advantages of sharded cluster , but also achieve more smooth data migration through the technique of pre-allocating slots , but data migration is still a test of the architect 's foundation .**

**Is there a solution that supports all the features?**

**Yes , it 's here , and it 's called :**  **the official **Redis Cluster** ** **.**

# 5 Official Redis Cluster

**The author has used the Redis Cluster model in Peanut Good Car and Iflytek.**

![1](https://mmbiz.qpic.cn/sz_mmbiz_jpg/V71JNV78n2ibiaQI3WG5GFrpRC9ibIjbsSq7maM3opfnn9xWEDDiaAmhxgrjPucTSSpaRYSlicVcIWhcngRf561icYBg/640?wx_fmt=jpeg&from=appmsg)

**The RedisCluster cluster has the following features:**

* **The cluster is completely decentralized and uses multiple masters and multiple delegations;**
* **Each partition is composed of a Redis host and multiple slaves, and the shards are parallel to each other.**
* **RedisCluster does not need to deploy sentinel cluster. Redis peers in the cluster detect each other's state of health through Gossip protocol, and initiate automatic switch in case of failure.**
* **The Redis Cluster divides data into 16,384 slots, with each node responsible for managing a portion of the slots.**
* **When a guest sends a request to the Redis Cluster, the Cluster routes the request to the corresponding node based on the hash value of the key. Specifically, Redis Cluster uses the CRC16 algorithm to calculate the hash of the key, and then modulates 16384 to get the slot number.**
* **RedisCluster provides a "companion" SDK that can be integrated with RedisCluster as long as the client upgrades the SDK. The SDK will help you find the Redis node corresponding to the key to read and write, and will automatically adapt to the addition and deletion of Redis nodes. The business side is not aware.**

**In terms of functionality, Redis Cluster has approached perfection, providing high availability while achieving data fragmentation and negative load balancing, suitable for large-scale data storage and high-performance scenarios. However, configuration and operation are relatively complex, and some complex multi-key operations may be limited.**

# 6 Are there really silver bombs?

**In the evolution of Redis's deployment model, from single-instance to Redis Cluster, we see the advantages and disadvantages of different architectures.**

**But**  **no one solution is a perfect silver bullet ** **, and each model has its own use cases and limitations .**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/V71JNV78n2ibiaQI3WG5GFrpRC9ibIjbsSqaicBic5JCvsChtxesOWxGz6L76DFTU9DhrOwXiayUlEZp2bNNFYLtdkOA/640?wx_fmt=png&from=appmsg)

**Therefore, we need to understand the business requirements and weigh performance, scaling and operating costs to make the best choice.**
