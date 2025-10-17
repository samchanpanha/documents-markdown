# Three technical routes for distributed transactional databases

**Original** **cute president** [cute president](javascript:void(0);)

 *October 12, 2025, 19:19*

**preface**

**There are three main technical routes common to distributed transactional databases:**

* **Route 1 : Distributed data routing component + single-machine database , such as** **ShardingSphere-** **Proxy + MySQL**
* **Route 2:** **Shared storage architecture （storage and computing separated）, such as Alibaba CloudPolarDB、** **AWS Aurora**
* **Route 3 :** **Native distributed database , such as OceanBase**

---

**Route 1: Distributed data routing components + single database**

**The route is essentially a distributed system** **, mainly** **composed of two parts : data routing component + single-machine database :**

**1) Distributed Data Routing Component**

**Responsible for maintaining a unified set of** **data** **sharding** **routing** **rules , providing SQL parsing , SQL routing , request forwarding and result merging capabilities** **.** **It supports data partitioning , read-write separation , distributed transactions and elastic scalability , and realizes transparent SQL routing through rule configuration .**

**Data routing components are stateless and can be scaled horizontally on demand and can be divided into the following two categories:**

**○**  **Client mode** **: is usually** **integrated into the application layer as an enhanced JDBC driver, and can be expanded with the application level。Compatible with all JDBC-based ORM frameworks (MyBatis / JPA).For example** **, ShardingSphere-JDBC is such a way.**

**○**  **Agent mode:** **that is** **, as an independent deployment of the DB agent service, can be expanded as needed。Business applications unifiedly access DB through this proxy service.Like** **MyCat** **,** **ShardingSphere-** **ProxyIn this way.**

**2) A single database**

**That is , multiple single-machine databases** **form a DB cluster , and the database and table are divided according to demand** **.** **Most of the open source MySQL single-machine databases provide data storage and execution capabilities .**

**After the business request is routed by the upper layer data routing component** **into data shards, it is distributed to multiple single-machine databases。When the single-machine database has a performance bottleneck, the single-machine database can be horizontally expanded and a new single-machine database can be deployed to reduce the performance pressure.**

**If there is a requirement for domestic-made information security , the MySQL kernel can be upgraded to a domestic-made kernel to meet the information security requirements , such as** **:**

**○**  **TXSQL （** **Enterprise-level branch kernel of Tencent based on the official version of MySQL** **）**

**○**  **AliSQL（Alibaba Cloud Research and Development MySQL branch）**

![](https://mmbiz.qpic.cn/mmbiz_png/XNkyxpRH73BqV1ialEWWKaW9U0Wbu0saoam56MZnGkwxcyavkU5hq11EibskI84TRF4SHZ9QD8maCXIiaCXMRr3ug/640?wx_fmt=png&watermark=1)

**可看到，路线一无论是上层无状态的数据路由组件，还是底层有状态的单机数据库，均具备良好的水平扩展能力，因此这一** **路线可** **通过硬件堆叠，可近似线性地提升计算性能和存储容量，具有可支持超大规模集群的能力，适用于高并发、** **数据规模巨大、对延迟要求很高** **的在线交易场景。**

**Route 1 can be** **based on open source** **distributed data routing components and distributed** **mature and stable** **single-machine database implementation ,** **low cost of transformation , and flexible expansion ,** **Strong self-control, not dependent on any manufacturer。This method is widely used in many Internet companies, and is also used in some financial institutions with strong self-research capabilities.**

**Route 1: Many Internet companies should address the mainstream choice of high concurrency + large-scale data volume + transactional scenarios, which has been validated by many Internet companies' large-scale production practices for many years.**

**The route has certain requirements on the ability of the RD team , especially in disaster tolerance , elasticity , integrated operation and maintenance , data consistency , replica control , and distributed transactions** **. If there are high requirements in these areas , a large number of optimization enhancements are required .**

---

**Route 2 **:** **Distributed shared storage architecture****

**The route adopts the design of separating calculation and storage, and multiple calculation nodes share the same storage pool （such as shared disk or block storage），That is, the upper layer is** **multiple groups** **of stateless** **computing** **nodes, and the bottom layer is** **shared** **Distributed cloud storage , shared storage provides** **cross node** **read and write .** **Data consistency is mainly dependent on the distributed storage engine .**

![](https://mmbiz.qpic.cn/mmbiz_png/XNkyxpRH73BqV1ialEWWKaW9U0Wbu0saoQedeJVOUIQFfzP1vPlgdR9v5X8iaUcW0c0dfpRZUxict1LTz3fXfFdOw/640?wx_fmt=png&watermark=1)

**The route** **takes full advantage of the advanced features offered by distributed storage, with most public cloud databases** **（** **such as** **Aws Aurora** **，Alibaba Cloud PolarDB）** **Using this technical route** **。However, this route requires a heavier reliance on the cloud base.**

---

**Route 3 **: Native** **distributed database****

**The route is** **based on the distributed database theory to achieve a distributed database，The top layer is usually a stateless db proxy cluster, and the next layer** **implements components such as optimizers, executors, transaction management, storage engines, and so on, which support distributed transactions and global MVCC more thoroughly.** **The bottom layer uses self-developed or bare storage engines , and the data is automatically scattered and stored in multiple copies by DB , through** **pa** **x** **os / raft** **and other distributed** **consistency** **protocols ensure data consistency among multiple replicas .**

**The typical representatives of this route are** **Google Spanner ,** **Oce** **anBase** and so on .

**The route is more advantageous in terms of distributed transactions , replica control , data consistency , disaster tolerance , elasticity and integrated operation and maintenance** **（** **native implementation ,** **belonging to its own features ） , and does not need to be as much as route 1 .**

**This route can be further divided into different implementation paths, the main differences being whether compute and storage are separated (many are compute separated, but some, such as OB, are compute integrated), and the data strong consistency mechanism (such as OB is based on Paxos protocol, and TDSQL is based upon strong synchronization).**

For example **, OceanBase** only has a proxy layer （OBProxy） and a service layer （ **OBServer** ）。Each **Observer** **It contains an independent SQL engine , transaction engine and storage engine , that is , each** **OBServer is a storage and computing** **body .** **Multi-copy strong consistency based on Paxos protocol,** **fully** **decentralized。As shown below:**

![](https://mmbiz.qpic.cn/mmbiz_png/XNkyxpRH73BqV1ialEWWKaW9U0Wbu0saoYf1yEtwdkR90niak2uOFZdSwJHUicu5wqB6uRAr4Blyn8MGpTb3npiabg/640?wx_fmt=png&watermark=1)

AndTDSQL, GaussDB, **GoldenDB** and so on are separated from storage and computing. The computing layer and the storage layer are deployed independently. Generally, there will be a global transaction manager or a management node, as shown below:

![](https://mmbiz.qpic.cn/mmbiz_png/XNkyxpRH73BqV1ialEWWKaW9U0Wbu0saob9n6ZGVl1ktFFv3XicJQLAX8HeUYibaSMTwq3yPDGazl0M1264kxoJ1w/640?wx_fmt=png&watermark=1)

![](https://mmbiz.qpic.cn/mmbiz_png/XNkyxpRH73BqV1ialEWWKaW9U0Wbu0saoFPsZgyicmM8PNMia1IlU85IUTpXe09icyII2aCl0L87ZibN1DHvViaIeRNg/640?wx_fmt=png&watermark=1)
