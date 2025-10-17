# Enterprise Big Data Security Planning: Risk prevention and system building implementation plan

**Original** **Old Line Senior** [Pre-Sales Solution Research Institute](javascript:void(0);)

 *October 11, 2025, 08:40*

**Click the blue text to follow me** **↑**

**Look at the world from a multidimensional perspective!**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/L5lr1zMLMLphfbEsEVUSfOiaLwYxuiafE63KcquSE0snf8XBnMD8MuwtUucxqpVzhicFww3m7pdpdoHmB9xJv2aZg/640?wx_fmt=png&from=appmsg&randomid=d80g46hq#imgIndex=0)

**I. Project background introduction**

At a time when digital transformation is accelerating, the scale of enterprise data is exploding, and big data platform has become the core infrastructure to support business decision-making and optimize operational efficiency。However, while data value increases, security risks also increase - in many provinces **Distributed storage architecture** （such as 31 provinces HDFS resource pool） brings data control complexity, multi-role access of internal and external personnel （business operation, partners, third-party operation and maintenance） increases the difficulty of authority management，The default configuration vulnerabilities of open source components such as Hadoop, as well as the explicit exposure and difficulty in tracing leaks throughout the data life cycle (collecting, storing, processing, applying, sharing), all place higher requirements on data security protection.To this end, we have developed this set of big data security planning general plan covering risk analysis, protection system and landing path, aiming to escort big data security for enterprises.

**II. Deep analysis of big data security risks**

**2.1 Data life cycle security risks: the full chain of risks from source to consumption**

**Every step of data - from generation to consumption - has security vulnerabilities that cannot be ignored. The first is the issue of data distribution and storage. The current system lacks a clear definition of which data are core sensitive data (such as user identity information and business operations data), which leads to unclear protection priorities. What's more, 70 percent of structured relational data and 30 percent of unstructured text files are stored in plaintext, exposing "core assets" to unprotected conditions.**

**In the use and flow of data, the risk is further amplified: the data in the application system is explicitly displayed regardless of user rights, and both business and operation personnel have direct access to the complete data. After the data is exported to the personal terminal, it can be sent freely through the network transmission, U disk copy and other peripherals, lacking effective interception means; Data is also clearly displayed during background maintenance, and the document server that shares data can be connected at will, and whether the end-to-end access subject is compliant or qualified to access is not managed.**

**What's more, once a data breach occurs, the existing system cannot trace the source and accountability - the route of the breach (whether it is an insider's mistake or an external attack) is difficult to locate in time, and the path of the leaked data cannot be traced. At the same time, the export of production environment data to the development test environment without security treatment such as desensitization amounts to an additional "export" for data breaches, while the legality of data output lacks auditing standards, further amplifying security risks.**

**2.2 Third party personnel security risks: control blind areas under multi-role access**

**With the deepening of business cooperation, third-party personnel (partners, third party development and operation teams) have become important users of big data platforms, but there are clear blind areas in related security control. First, there is a blurred tenant boundary. The existing platform does not clearly delineate the resource boundaries of internal tenants and third-party tenants, and partners may use illegal means to steal, use data that is not within their own authority, or even tamper with core business data.**

**Second, there is a lack of access and operational control: the process for third-party personnel to enter the production environment is unclear, and there is always a risk of circumventing regulation (such as borrowing internal account numbers and cracking weak passwords) to enter the system. In the production process, key activities such as operational commands, script uploading, and data exporting by third party personnel are not strictly monitored, especially operations involving core data (such as user consumption records, business operating statements), and there is a lack of targeted management measures. At the same time, the terminal access conditions for third-party personnel are lax, and the terminal is not detected for viruses or Trojans. Once the terminal carries malicious programs, it is very easy to penetrate the internal network and cause infection of the entire big data platform.**

**2.3 Hadoop and Component Security Risks: Fundamental Vulnerabilities in Open Source Architecture**

**As the core architecture of big data platform, Hadoop has many shortcomings in its default security configuration, which has become the weak link of security protection. First, the authentication mechanism is weak: Hadoop's permission model is similar to Linux's. The default permission for the new file / directory is 755 (rwxr-xr-x), which means that any user has access right. Moreover, the account of the big data system is directly bound to the host account, and the account is not equal to a natural person, there are situations where multiple people share the same account, and once security problems occur, it is impossible to locate the specific responsible person.**

**Permissions and audit issues: Although the Hadoop component supports Kerberos third-party authentication, the existing platform does not open this function, resulting in insufficient authentication strength; At the same time, component logs are recorded in a professional development language (such as HDFS access logs), which makes it difficult for the average manager to quickly understand the log content, enable effective auditing, and make it difficult to trace the source of the problem through logs even when an exception occurs.**

**Some Hadoop components also have specific security vulnerabilities, such as web services that can be accessed without authentication (HDFS, HBase web services can be opened directly), XSS injection protection is inadequate (webhdfs page path input boxes can be easily injected into scripts), Poor audit logging (Hive's metastore has no audit log), and sensitive information (such as TGT authentication credentials) is printed in clear text in the debug log, all of which provide opportunities for external attacks and internal abuse.**

**2.4 Security management supports risks: inadequate institutional control capacity**

**In addition to the risks at the technical level, the inadequacy of the safety management support system also exacerbates the overall safety risks. First, there is a lack of risk perception: existing systems cannot sense risk trends in data centers, and they lack a mechanism for public awareness and security intelligence updates to respond to new security threats (such as new ransomware and APT attacks).**

**Poor implementation of policies and norms: There is a lack of capacity to manage and analyze security policies issued by regulators (such as data security laws and personal information protection laws) and to translate policy requirements into concrete implementation measures. At the same time, the company's internal security specifications have not established an effective implementation monitoring mechanism, and the specifications are disconnected from actual operations, resulting in a widespread situation of "norms not enforced."**

**Poor management of basic information: the distribution, location, and attribution status of basic equipment (servers, network equipment) is unknown, and there is a lack of uniform management of equipment vulnerabilities, patch installations, health, and compliance; User identities, application systems, organizational organizations, role roles and other information are not uniform, and there are problems such as duplicate research, difficulty in connecting systems, duplicate application or fraudulent use of accounts, further weakening the effectiveness of security management.**

**III. Building a big data security protection system**

**3.1 Core protection framework: five areas to work together**

**Based on the above risk analysis, we have established a The five fields of protection underpinned by "data application security protection," "big data security control" and "big data infrastructure security" form a three-tiered protection architecture of "foundation - core - support," ensuring that there is no end to big data security protection.**

**Among them, data security management support is the "foundation" of the entire system, providing a management basis for other areas by clarifying data classification and grading standards, standardizing business processes, dividing security responsibilities, and strengthening partner management. Data life cycle security management focuses on the whole process of data from collection to consumption, and realizes data full cycle security through data discovery, rights control, encryption desensitization and other means. Data application security protection for the application layer (such as open application platform), service layer (DAAS, PAAS) security requirements, to prevent illegal access and data leakage; Big data security control achieves full-dimensional control of users of big data platforms through unified accounts, unified authentication, operational auditing and other functions. Big data infrastructure security ensures the security of servers, network equipment, virtualization environments and other infrastructures, providing a stable and secure operational environment for the upper layer business.**

**3.2 Data security management support: establishing a standardized management system**

**At the core of data security management is "standardization" - solving the problem of "what to do, who to do it and how to do it" by clarifying rules and dividing responsibilities. The first is the classification and classification of sensitive data, based on the Guidelines for the Protection of Personal Information of Telecommunication and Internet Service Users. Combining the actual business of the enterprise, the data is divided into four categories (A (user identity-related), B (user service content), C (user service derived), and D (enterprise management) and further subdivided into four (extremely sensitive, sensitive, more sensitive, and low sensitive).**

**For example, highly sensitive data, including physical identification, user passwords and associated information, and internal core management data, require the highest level of protection using multiple approval + encrypted storage + full audit. Sensitive levels of data (such as user basic data, location data) need to be implemented with fine-grained permission control and transmission encryption; More sensitive and less sensitive data are subject to relatively simplified security measures based on actual needs. Through classification, we can clarify the protection priorities of different data and avoid "over-protection" and "inadequate protection."**

**Business process specifications and partner management: For external big data collaborations, we have established from“Cooperation Requests - Business Negotiations - Security Audits - Credentials Audit - Resource Opening”There is a full-process management mechanism, each step of the process is clearly audited (e.g. security audits need to verify the security credentials of partners and the extent of data use), and the process is solidified through an online system to achieve traceability of the cooperation entire process. At the same time, establish a partner data application process, provide encryption and key management services for the requested data, and ensure that the data is not misused during the cooperation process.**

**We have also built a security responsibility matrix that identifies the security responsibilities of different roles such as business managers, platform operators, tenants, and data consumers (e.g. business managers are responsible for requirements auditing and security strategy development). Platform operators are responsible for log collection and security auditing, avoiding burden-sharing, and ensuring that security measures are implemented.**

**3.3 Data Lifecycle Security Management: All Process Closing Loop Control**

**Data life cycle security management focuses on "the security of every step of data from generation to demise," and prevents the risk of data breach through closed-loop control with "prior prevention, in-fact control and post-fact traceability." The first is data acquisition and storage security: in the data acquisition link (such as from the data source system through the mirror), Identify the sensitive information of the collected source data. If it contains sensitive data (such as mobile phone number and ID card number), it will be encrypted or desensitized in real-time to avoid the sensitive data leaking during the collection. In the storage process, Hive / HBase fine-grained encryption (supporting AES, national secret SM4 and other algorithms) is used to achieve table-level and column-level encryption, and the decryption process is transparent to the business and does not affect the normal operation of the business.**

**Data use and sharing security: In the data use process, establish a "data access request - permission approval - access control" process, and users need to apply and obtain approval before they can access the corresponding data; At the same time, implementing "fine-grained permission control" can be controlled to tables, files, and even field and row levels, so that different users can only access data within their own responsibilities according to their rights. In the data sharing process, the shared data is desensitized (e.g. the user's mobile phone number is shown as "138****5678"), and the source tracing can be achieved through document watermarking (e. g. web page watermarking, document watermarks). Once the data is compromised, the source of the compromised data can be located through the watermarking.**

**Finally, data output and destruction security: data outputs (e.g. outbound APIs, exported to terminals) need to be subjected to multiple validations - label dimension enumeration validation (e. g. credit ratings can only be A / B / C / D, If the output data is exceeded, the output data will be checked to ensure the compliance of the output data. The data destruction process uses a "repeated rewriting + physical destruction" method to ensure that the data is completely deleted and not recovered.**

**3.4 Big data security control: full-dimensional user and operation control**

The core of big data security control is "people". Through the full-dimensional management of account, authentication, authority and operation of platform users （internal personnel, third-party personnel, application system）, we can prevent "inside man" and "excessive operation"。First of all, unified account numbers and unified authentication: we are based on Kerberos authentication mechanism, Implement the decoupling of big data system accounts and server accounts, each natural person corresponds to a unique account (not a shared account), and account passwords must meet strong password requirements (such as 8 bits or more, containing upper and lower case letters + numbers + special symbols);At the same time, docking the 4A platform to achieve **single sign-on** , users only need to authenticate once to access multiple authorized systems, which not only improves the user experience, but also avoids the chaos of multi-account management.

Access control and operation audit: access control is divided into "pre-event" and "event" two stages - pre-event according to the access area （such as only allow office network access）, access time （such as working days 9: 00-18: 00） set access policy；During the process, "order set control" （only authorized orders are allowed to be executed） and " **treasury mode** " （sensitive operations need to be approved by multiple people） are used to prevent unauthorized operations。Operational audits implement "5W1H" (who, when, where, what was done,Why to do, how to do） of all dimensions, whether the user accesses the big data platform through a command or the application accesses it through an API, all operation logs are collected, stored in real time, and support online audit and manual review. Once an abnormal operation occurs, it can be traced quickly.

**For third-party personnel, we have also established a terminal access and operation monitoring mechanism: third-Party personnel's terminals need to install security software and conduct virus investigation before they can access the internal network. Its operational processes (such as script upload, data export) are monitored in real time, and core data-related operations (such as access to user password data) require additional approval to ensure compliance of third-party personnel.**

**3.5 Big Data Infrastructure Security: Strengthen the Infrastructure Security Line**

**Big data infrastructure security is the "base" of the entire security system, which needs to ensure the security and stability of servers, network equipment, virtualization environment and other infrastructure. The first is virtualized environment security: addressing the security risks posed by virtualization technology (e.g. virtual machine escape, We adopt the technology of "agentless virtualization protection" to realize virus protection, intrusion detection and integrity monitoring of virtual host without agent. At the same time, the virtual network is isolated and divided into different security domains (such as Internet subdomain, external interface subdomain, core domain), and each security domain is controlled by firewall, IDS / IPS to prevent cross-domain attacks.**

Vulnerability and patch management: establish a closed-loop management process of "vulnerability scanning - patch testing - patch deployment - effect verification", and regularly （such as monthly） conduct vulnerability scanning of servers, network equipment and big data components （using **Nessus** , AppscanAnd other tools）, after finding the vulnerability, the priority is to use **virtual patch** （temporary protection measures before deploying the official patch） for protection, and then deploy the patch in time after the patch test is passed；For operating systems and applications that are no longer supported, extend their security protection period through virtual patches to avoid security vulnerabilities caused by the inability to upgrade patches.

**We have also strengthened APT attack protection, deploying APT Web Security Gateway, APT Mail Security Gateway, Deep threat discovery platforms and other devices can effectively identify and block APT attacks by detecting C & C communications, analyzing malware behavior, and monitoring attacker activities (such attacks are highly hidden and long-lasting, which are difficult to prevent with traditional security devices); At the same time, establish data disaster-tolerant backup mechanisms to ensure that data is not lost and business recoverable in extreme cases (e.g., natural disasters, major attacks) through cross-data center backups (such as HBase clusters, HLOG quasi-real-time replication, HDFS asynchronous replication).**

**IV. Phased construction planning: steadily advancing the implementation of the security system**

**Given the complexity of the security system construction, we adopted a "phased and progressive" implementation strategy, dividing the entire construction process into three phases, each focusing on core objectives to ensure the effectiveness of the construction.**

**4.1 Phase I: Building foundations and addressing core risks**

**The core objectives of the first phase (1-3 months) are:“Fix the most urgent, The core security risks”We will focus on three tasks: First, we will complete the classification and classification of sensitive data, identify sensitive data in existing systems through data scanning tools, clarify the classification results, and formulate protection strategies for different levels of data; The second is to achieve the core control of the big data platform, deploy unified account management, unified authentication (Kerberos), and operate audit functions to solve the problems of account sharing, weak authentication, and lack of audit; Third, establish basic security protection, carry out security baseline configuration for core servers and network equipment (such as closing unnecessary ports and setting strong passwords), deploy firewalls, IDS / IPS and other basic security devices to prevent common attacks.**

**4.2 Stage 2: Deepen protection and improve the system**

**The second phase (4-6 months) aims to "deepen the protection capability and improve the security system." The key tasks include: 1. Data lifecycle security control, data acquisition encryption, storage encryption, transmission encryption, application desensitization, and the establishment of data access application and approval process; Second, application and service layer security protection, API security control for the DAAS and PAAS service layers (such as API access authentication, parameter verification, transmission encryption), security testing for the application layer (such as code scanning, penetration testing), and application vulnerability repair. Third, third party personnel full-process management, online partner management system, solidifying the cooperation process and data application process, and realizing terminal access and operation monitoring for third party staff.**

**4.3 Phase 3: Optimizing operations and continuing improvement**

**The objectives of the third phase (7-12 months) are: "Optimizing security operations, Enhance active defense capabilities" The key tasks include: (1) optimizing the security operation system, establishing security incident response process (such as data breach emergency process), realizing security log automatic analysis and alarm, and improving the efficiency of security incident handling; Second, active risk perception, access to security intelligence platforms, realize public opinion perception and new threat identification, and turn "passive defense" into "active defense"; Third, continuous optimization and training, regular security assessment (such as penetration test), find and repair new security vulnerabilities; At the same time, conduct security training (developing training content for different roles) to enhance the safety awareness of the entire staff.**

**PPT文末**

**I. Big data security solutions are explained as follows:**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNynAwM5v6QGzQicBzZfqvqRgbclxic7gFyzuwuRAT6YcpyJNibMCeDO8Cgw/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyjnE9JbMp7OZmRz7iaicagrNP3fG8bt12MibFxLNszluY1d4CelbIgjohw/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyIz1yqSafGokvbw6HarPpPxib8bSUGbrLicvbe6wLY9N3S0gvam37flZQ/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNy8ShAsicpM3iaagEicMBScNfhLrnyqicx9qTN0MlhIo2C8iaRD04XzicqCichQ/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyDAVWM5MyFewQwZWYvE6gRSIEI8t8Wj7icLZ5AB86quvLicIjA2ZZKnXQ/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNycBqPkmE02NQHL92emb8JpAtsE67iaqNObxjyEBjD0TlKOibLwtbXqR6Q/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyocam26WQ8ayiczT2tpKYk8oWEKV1c9GCda3sOIUDb2LRaJaLb5ULhgg/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyicjh1a2nb5SwTq7COEdfzQgE5PCN7qEHPY575InXib4p7xV9c7ZPXoGA/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyJRgfeSsyMf8k0CrRh3JmZmSeAIpiaNnjmyLTQVHib26Z8BxJRIHRicxHA/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyWKR5OoqdNqPlOZccUCicMKnuDhhOwyLvZTlOAKYwOic22XkSBZpibXicnA/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyVCf7BPxrtRlmMG0BV6DiaByLoyG91rYGFLN6gmD4dro2hMPm1Pb4MWg/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyF6kRZ4kUicvKRY4nsscEN0P2kWbkK39xnhZxccAZEUZG4UjyRuY1jrA/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyFznQvjZVehxWYZTMQ8UWC1iaPjhvcNZImHuwdDH3OwGoNXHvmGD4p0w/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNy1JibEJ9dgeZMkL3mGs7dibgXEUTKFrT98JuyfBhwfJJ5VrXlT2UNpibKg/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyOu1Tg3W6ARjMicLq0dPSVPic3M65sjkkfpTJqPXhGEzMrEKtAOvgA1mg/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNy9ptPqXdvVsW9tqn6S1vudUhFL9Kz1iaiadwloW4unNa4fcDcOnvYibCGg/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyDNEnJX7CODKNVLHaMK8GTYAukiacsH3Gxuh7ymRh40diaYRnqQWz1ncg/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyxA3lePiaVicich3qSBRWmLVnrSnBojGHpuIArZrLxwM65ibKovjnsst0aQ/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNysRMQaXLqB8Qqm9Z2VjsXvOB86oKRl7gib9emXca26eqI75FgdaDXhDQ/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyLwvlpjdHPic2aF3Ecjll9x2r07IU4rHZJoziawMVGnjhz2HBUPBhcU7Q/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyHnc5ch78W3pR0vSvVWyJgPSPKWCS9LEzPeCSOrWD7khWqmVHduAMJg/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNy5fTUUZ2TCpOib980EKvoamjraD5chGlcv1icapbIaZT8Q8mZ1yL3kyBw/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNylPryo8I1UC76RJ67PyRVq3icULMzqua0TJGlygMVHsaYC4LcHibzWr5g/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyao3KcVw1dMlc68I9SgcHjiae1jv7iaS3BUdH62V4PgGt7WKHJcG0suzw/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNydaRcI8ndBFAFbDiaC8EKuTicq3uLPzfP0g92snLz5mVadGPV0jr9Zb1Q/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyuLjP5hdpACxj6K2QEfn97St9tDDvF7icUgnvbC50BUyPsPB7yeuYfEw/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyqMRKvgSmEd7VM9iaRvAuyk5PqMtZg0ibSFHx7MXriahLLFpyHNKUNz2TQ/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyRibicJnFB8MEjrGib852Cey6mLXJxc7sPNXGIzREnJwromMMF8oh6iaaIA/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNywa3kGiahmNbcEmvpJlk1w5DoePrgHMxZvf7Sr4eVBUWNA5HcERo0gIQ/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNy8vGG7aQ01kdp8kMibRaJ2SJI0XN6h9uGSuuOo1tNV2cWJXcZNlVvZzw/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNy6MibG5HLsTXpDsNKibKibQrJBxicppuXhzsqrmYZ1hOlPiboOE59ZqRoiadQ/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyV4zmFAu0L0AmDicXYXB0YwDxvzAaicib5ryhbaHTdXu4b4Z0ImsHicQ3Ig/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyXhreVyFkutTEATDmF5S5TEf3QRjxFiaLDMXjg9RbOOlxdj30I0yVWiaA/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNy9DxcyBZniaRnPS92hdv4nTYk3mDN3PCWucVWv7aD5iceLODd08icAb2Kg/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyMUPrUfItt7Mmvuiaa9yWOnGddKnAAetnib5pViaLfaqlpKwWhxOyvX14w/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyqUZwKMvBcl0lXmIqyjp7nXfX0qxBqM3qbZY4iby5VYAI3KzmourHSYw/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyyEbRr4giakdxZckaiaMIvUJGP7WANTOZn58hWibcm7aPLeL4Enic8hqyGA/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNy78uhLLd8wmeT4IezyicOhBPmdtWtlSDRkmS4nkv4Z3xOpLhF5rwMiaMA/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyFPsJibeIPPCbH4pRra3bwE7LIfyUUJnJKjYerYE3rQL47uLlE8Khfuw/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNypMo6IIunrjibicVJrH1Qkv5aia8GCIaoMib7ERptFepR1dpibfftEE1hib1w/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyozicFkNpLav8vujaJYGwUH1cyibLb97rHjvichk99VQzVmaemyuDtyNog/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyxbibWuXuxDkEM7SS5QDz7yEPU289k0qbbnPsz6gicWpRJ4GTWm0UFRzg/640?wx_fmt=png&from=appmsg&watermark=1)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNypP9pV37k5E0BRm2cyS8xUxFvzbcW4CH123HILfrbxEBiaHrKNRPkV7A/640?wx_fmt=png&from=appmsg&watermark=1)

**Key words :** **risk analysis , classification and grading , full-cycle protection , unified management and control , phased implementation**

**📌Please note: Due to the WeChat push rules, you may miss the updates of the Pre-Sales Solution Institute. Remember to click on the home page and set it as a star⭐️Live dry goods can't be missed. They're delivered the first time!**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NtyaPSlTkqDAyH1Z7UfXhNyvjyHKQtiah0CdVBaTzmMykicYHxx09kx9GNo0gwWYPWkyiarcNglLaZHA/640?wx_fmt=png&from=appmsg&randomid=660tld5v&watermark=1#imgIndex=81)

**Knowledge Planet**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/I0HOTFiad9NsVfhIQ14oQKHLnoXhT6vd619GHTEMcLzDX9bSsG5WHp0TLQy9m2ic6cQ2GWr7DamZ4SOibIgHO6xjg/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp&randomid=jrxtwro2#imgIndex=82)

**PPT document** **, learning and growth**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/L5lr1zMLMLphfbEsEVUSfOiaLwYxuiafE6NtHFPsJxZgmMcheWZOEgiay3jibxkkQM4CX41ibWCibu0niaU2DG5BOdEyQ/640?wx_fmt=png&from=appmsg&randomid=vim0kgfw#imgIndex=83)

**Digital Transformation · Directory**

**The previous article**105 full scene coverage of the digital PPT tools

Reads 587

[]()
