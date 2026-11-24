# Codebase Studies

> **Table of Contents**
>
> - [1. Backend Engineering Repositories](#1-backend-engineering-repositories)
>   - [1.1 Business Application Systems](#11-business-application-systems)
>   - [1.2 Flash-Sale and High-Concurrency References](#12-flash-sale-and-high-concurrency-references)
>   - [1.3 Architecture References](#13-architecture-references)
>   - [1.4 Burst-Traffic and Transactional-Concurrency Systems](#14-burst-traffic-and-transactional-concurrency-systems)
>   - [1.5 Connection and Event-Stream Systems](#15-connection-and-event-stream-systems)
>   - [1.6 Distributed Execution and Scheduling Systems](#16-distributed-execution-and-scheduling-systems)
>   - [1.7 Framework Learning Repositories](#17-framework-learning-repositories)
>   - [1.8 Exchange and Order-Matching Systems](#18-exchange-and-order-matching-systems)
>   - [1.9 Redis Replication, High-Availability, and Cluster Application Systems](#19-redis-replication-high-availability-and-cluster-application-systems)
>   - [1.10 Database Replication, Read/Write Separation, and Sharding Application Systems](#110-database-replication-readwrite-separation-and-sharding-application-systems)
>   - [1.11 NageOffer Projects](#111-nageoffer-projects)
>   - [1.12 Insurance and Banking Systems](#112-insurance-and-banking-systems)
> - [2. Spring Repositories](#2-spring-repositories)

> Popularity and Last Commit retain the recorded research snapshots.

## 1. Backend Engineering Repositories

> Studies of how backend systems are designed and implemented.
>
> - Depth (first line): Low / High; `—` = unassigned.
> - Status (second line): Pending / Active / Complete.
> - Source language: `ZH` = primarily Chinese documentation; `EN` = primarily English documentation; `ZH/EN` = substantial documentation in both languages.

### 1.1 Business Application Systems

| Repository | Study | Description | Popularity | Last Commit | Repository Location |
| --- | --- | --- | --- | --- | --- |
| [mall](backend-engineering/mall/README.md) | Low<br>Complete | A mature Spring Boot commerce backend with separate administration, storefront, and search APIs and extensive Chinese documentation. | 84,850 stars · 29,834 forks | 2026-09-15 | [macrozheng/mall](https://github.com/macrozheng/mall) · `ZH` |
| [mall-learning](backend-engineering/mall-learning/README.md) | Low<br>Complete | Companion tutorials and Spring Boot examples for mall. | 13,422 stars · 8,229 forks | 2024-09-15 | [macrozheng/mall-learning](https://github.com/macrozheng/mall-learning) · `ZH` |
| Mall4Cloud | —<br>Pending | A Spring Cloud Alibaba commerce system with distributed transactions and asynchronous data integration. | 6,951 stars · 2,654 forks | 2026-09-29 | [gz-yami_admin/mall4cloud](https://gitee.com/gz-yami_admin/mall4cloud) · `Gitee` · `ZH` |
| [novel-plus 小说精品屋](backend-engineering/novel-plus/README.md) | High<br>Complete | An online literature CMS with reader and author interfaces, administration, and content crawling. | 4.7k stars · 1.5k forks | 2026-09-04 | [201206030/novel-plus](https://github.com/201206030/novel-plus) · `ZH` |
| [waynboot-mall](backend-engineering/waynboot-mall/README.md) | High<br>Active | A Spring Boot commerce backend with three applications. | 1,264 stars · 316 forks | 2026-07-09 | [wayn111/waynboot-mall](https://github.com/wayn111/waynboot-mall) · `ZH` |
| [GuliMall 谷粒商城](backend-engineering/gulimall/README.md) | High<br>Active | A Spring Boot e-commerce system with 12 main services. | 112 stars · 40 forks | 2022-06-25 | [shuhongfan/GuliMall](https://github.com/shuhongfan/GuliMall) · `ZH` |

### 1.2 Flash-Sale and High-Concurrency References

| Repository | Study | Description | Popularity | Last Commit | Repository Location |
| --- | --- | --- | --- | --- | --- |
| JD HotKey | —<br>Pending | JD's production-origin system for detecting hot keys and distributing them to local caches. | 4,557 stars · 1,447 forks | 2023-08-26 | [jd-platform-opensource/hotkey](https://gitee.com/jd-platform-opensource/hotkey) · `Gitee` · `ZH` |
| [seconds-kill](backend-engineering/seconds-kill/README.md) | Low<br>Complete | A Spring Boot flash-sale experiment with synchronous and Kafka-based order creation. | 490 stars · 142 forks | 2019-06-18 | [daydreamdev/seconds-kill](https://github.com/daydreamdev/seconds-kill) · `ZH` |
| [shop-flashsale_cd05 限时抢购](backend-engineering/shop-flashsale-cd05/README.md) | Low<br>Active | A Spring Boot flash-sale system with nine backend applications. | 76 stars · 61 forks | 2023-06-15 | [leoen/shop-flashsale_cd05](https://gitee.com/leoen/shop-flashsale_cd05) · `Gitee` · `ZH` |

### 1.3 Architecture References

| Repository | Study | Description | Popularity | Last Commit | Repository Location |
| --- | --- | --- | --- | --- | --- |
| DDD by Examples: Library | —<br>Pending | A public-library DDD reference built as a modular monolith with context-specific architectures. | 5.9k stars · 831 forks | 2026-07-01 | [ddd-by-examples/library](https://github.com/ddd-by-examples/library) · `EN` |

### 1.4 Burst-Traffic and Transactional-Concurrency Systems

| Repository | Study | Description | Popularity | Last Commit | Repository Location |
| --- | --- | --- | --- | --- | --- |
| [miaosha v2](backend-engineering/miaosha/README.md) | Low<br>Active | A legacy Spring Boot flash-sale learning system with a web application and a separate Dubbo goods service. | 26,590 stars · 6,556 forks | 2025-04-18 | [qiurunze123/miaosha](https://github.com/qiurunze123/miaosha) · `ZH` |
| [Jeepay](backend-engineering/jeepay/README.md) | High<br>Complete | A Spring Boot payment platform with three applications, emphasizing order consistency and asynchronous notifications. | 6,402 stars · 2,021 forks | 2026-09-22 | [jeequan/jeepay](https://github.com/jeequan/jeepay) · `ZH` |
| [yu-like](backend-engineering/yu-like/README.md) | Low<br>Active | A like system with Redis-first writes and Pulsar-based persistence. | 247 stars · 73 forks | 2025-04-25 | [liyupi/yu-like](https://github.com/liyupi/yu-like) · `ZH` |

- Related: [shortlink](#111-nageoffer-projects).

### 1.5 Connection and Event-Stream Systems

| Repository | Study | Description | Popularity | Last Commit | Repository Location |
| --- | --- | --- | --- | --- | --- |
| CIM | —<br>Pending | A Spring Boot 3 instant-messaging system with Netty connections and stateless clustered routing. | 9,491 stars · 2,865 forks | 2026-04-05 | [crossoverJie/cim](https://github.com/crossoverJie/cim) · `ZH` |
| JetLinks Community | —<br>Pending | A reactive Spring Boot 3 IoT platform managing heterogeneous devices and continuous event streams. | 6,648 stars · 1,944 forks | 2026-08-28 | [jetlinks/jetlinks-community](https://github.com/jetlinks/jetlinks-community) · `ZH` |
| MallChat | —<br>Pending | A Spring Boot chat and commerce system with Netty/WebSocket messaging and multilevel caching. | 4,656 stars · 1,019 forks | 2024-01-31 | [zongzibinbin/MallChat](https://github.com/zongzibinbin/MallChat) · `ZH` |
| HuLa Server | —<br>Pending | A reactive instant-messaging system with transactional and ordered messaging, requiring Java 25 and Spring Boot 4. | 914 stars · 314 forks | 2026-07-04 | [HuLaSpark/HuLa-Server](https://github.com/HuLaSpark/HuLa-Server) · `ZH/EN` |

### 1.6 Distributed Execution and Scheduling Systems

| Repository | Study | Description | Popularity | Last Commit | Repository Location |
| --- | --- | --- | --- | --- | --- |
| XXL-JOB | —<br>Pending | A distributed Spring Boot task scheduler with dynamic sharding, failover, and overload control. | 30,603 stars · 11,489 forks | 2026-07-22 | [xuxueli/xxl-job](https://github.com/xuxueli/xxl-job) · `ZH/EN` |
| PowerJob | —<br>Pending | A distributed scheduling and computing platform with MapReduce workers, retries, and fault recovery. | 7,794 stars · 1,381 forks | 2026-10-01 | [PowerJob/PowerJob](https://github.com/PowerJob/PowerJob) · `ZH/EN` |

### 1.7 Framework Learning Repositories

1. [Spring Repositories](#2-spring-repositories): Repositories for learning Spring fundamentals and Spring Boot.

| Repository | Study | Description | Popularity | Last Commit | Repository Location |
| --- | --- | --- | --- | --- | --- |
| houbb/validator | —<br>Pending | A Java validation library for studying reusable constraints, conditional checks, and validation result handling. | — | — | [houbb/validator](https://github.com/houbb/validator) · `ZH` |

### 1.8 Exchange and Order-Matching Systems

| Repository | Study | Description | Popularity | Last Commit | Repository Location |
| --- | --- | --- | --- | --- | --- |
| Bisq | —<br>Pending | A Java peer-to-peer Bitcoin exchange over Tor. | 5,137 stars · 1,310 forks | 2026-10-06 | [bisq-network/bisq](https://github.com/bisq-network/bisq) · `EN` |
| exchange-core | —<br>Pending | A Java exchange core with Disruptor-based matching, risk checks, and journal recovery. | 2,645 stars · 974 forks | 2022-05-14 | [exchange-core/exchange-core](https://github.com/exchange-core/exchange-core) · `EN` |
| CoinExchange | —<br>Pending | A Spring Cloud cryptocurrency exchange with in-memory matching and Kafka-based order messaging. | 1,731 stars · 1,306 forks | 2021-11-05 | [jammy928/CoinExchange_CryptoExchange_Java](https://github.com/jammy928/CoinExchange_CryptoExchange_Java) · `ZH/EN` |
| Order Matching Engine | —<br>Pending | A financial order matching engine with parallel lanes, write-ahead journaling, and crash recovery. | 0 stars · 0 forks | 2026-01-31 | [Anixton/Order-Matching-Engine](https://github.com/Anixton/Order-Matching-Engine) · `EN` |

- [WeChat guide](https://mp.weixin.qq.com/s/kmuG5azJnqjKRYlkiVHWqQ) · `ZH`

### 1.9 Redis Replication, High-Availability, and Cluster Application Systems

| Repository | Study | Description | Redis Topology | Popularity | Last Commit | Repository Location |
| --- | --- | --- | --- | --- | --- | --- |
| JeecgBoot | —<br>Pending | An AI and low-code business platform with Redis caching and annotation-based locks; Redis provisioning is external. | Primary–replica · Sentinel · Cluster | 48,105 stars · 16,205 forks | 2026-09-22 | [jeecgboot/JeecgBoot](https://github.com/jeecgboot/JeecgBoot) · `ZH/EN` |
| ThingsBoard | —<br>Pending | A distributed IoT platform using Redis as a shared cache across application and transport services. | Sentinel · Cluster | 22,512 stars · 6,501 forks | 2026-10-02 | [thingsboard/thingsboard](https://github.com/thingsboard/thingsboard) · `EN` |
| Apereo CAS | —<br>Pending | A Spring Boot single sign-on platform with Redis ticket storage, local-cache coherence, and distributed locking. | Sentinel · Cluster | 11,382 stars · 3,954 forks | 2026-10-08 | [apereo/cas](https://github.com/apereo/cas) · `EN` |
| Plumelog | —<br>Pending | A Java distributed-log platform with separate Redis stores for queuing and management. | Sentinel · Cluster for management Redis | 4,809 stars · 1,374 forks | 2025-10-09 | [plumeorg/plumelog](https://gitee.com/plumeorg/plumelog) · `Gitee` · `ZH` |
| RuoYi-Vue-Plus | —<br>Pending | A multi-tenant Spring Boot application with Redis Cluster replica reads and a standalone development default. | Cluster | 2,754 stars · 789 forks | 2026-09-28 | [dromara/RuoYi-Vue-Plus](https://github.com/dromara/RuoYi-Vue-Plus) · `ZH` |
| SiGa | —<br>Pending | A digital-signature gateway with Redis sessions, renewable locks, and reprocessing queues for clustered or managed Redis. | Cluster | 24 stars · 16 forks | 2026-10-01 | [open-eid/SiGa](https://github.com/open-eid/SiGa) · `EN` |

### 1.10 Database Replication, Read/Write Separation, and Sharding Application Systems

| Repository | Study | Description | Database Topology | Popularity | Last Commit | Repository Location |
| --- | --- | --- | --- | --- | --- | --- |
| SuperMarket | —<br>Pending | A Spring Cloud commerce system with MyCat routing across four MySQL 5.7 instances. | 2 shards × 2 MySQL nodes · bidirectional replication · read/write separation · database and table sharding | 2,109 stars · 692 forks | 2024-03-06 | [ZongXR/SuperMarket](https://github.com/ZongXR/SuperMarket) · `ZH` |
| Damai | —<br>Pending | A Spring Cloud ticketing system using ShardingSphere for order sharding. | 2 databases × 4 tables per database · no replicas or read/write splitting | 1,177 stars · 108 forks | 2026-09-24 | [java-up-up/damai](https://github.com/java-up-up/damai) · `ZH` |

- Related: [12306](#111-nageoffer-projects) — order sharding across two databases and 32 physical tables per logical order table; no replicas or read/write splitting.

### 1.11 NageOffer Projects

> The [NageOffer 拿个 offer](https://github.com/nageoffer) project family; Hippo4j is hosted under `opengoofy`.

| Repository | Type | Study | Description | Popularity | Last Commit | Repository Location |
| --- | --- | --- | --- | --- | --- | --- |
| [12306](backend-engineering/12306/README.md) | Application | High<br>Active | A railway ticketing system with aggregate and microservice deployment modes. | 3,752 stars · 371 forks | 2026-03-12 | [nageoffer/12306](https://github.com/nageoffer/12306) · `ZH` |
| [shortlink](backend-engineering/shortlink/README.md) | Application | High<br>Active | A short-link platform with aggregate and microservice deployment modes. | 773 stars · 125 forks | 2026-03-12 | [nageoffer/shortlink](https://github.com/nageoffer/shortlink) · `ZH` |
| CongoMall 刚果商城 | Application | —<br>Pending | A DDD-based commerce microservice system. | 1,117 stars · 149 forks | 2026-03-12 | [nageoffer/congomall](https://github.com/nageoffer/congomall) · `ZH` |
| Hippo4j | Component | —<br>Pending | Dynamic thread-pool management with monitoring and alerts. | 5,999 stars · 1,237 forks | 2026-03-12 | [opengoofy/hippo4j](https://github.com/opengoofy/hippo4j) · `ZH/EN` |
| Seraph | Component | —<br>Pending | An idempotency component for HTTP requests and message processing. | 10 stars · 1 fork | 2023-07-21 | [nageoffer/seraph](https://github.com/nageoffer/seraph) · `ZH` |

### 1.12 Insurance and Banking Systems

> References for studying insurance business workflows and banking operations.

| Repository | Study | Description | Popularity | Last Commit | Repository Location |
| --- | --- | --- | --- | --- | --- |
| openIMIS | —<br>Pending | A Python/Django health-insurance administration system with documented national and community scheme deployments; business modules are maintained in separate repositories. | 13 stars · 83 forks | 2026-09-07 | [openimis/openimis-be_py](https://github.com/openimis/openimis-be_py) · `EN` |
| 中睿保险中介代理系统 2.0 | —<br>Pending | A historical Java/Struts/JSP insurance-agency system with policy records, commissions, reconciliation, and settlement; the author describes building it with colleagues in 2009 and later upgrading it. | 4 stars · 4 forks | 2018-02-22 | [13014657675/agent2.0](https://github.com/13014657675/agent2.0) · `ZH` |
| Titanium 保险核心系统 | —<br>Pending | A Java/Spring insurance domain reference with policy lifecycle and policy-loan logic; several referenced services and build dependencies are not publicly accessible, and production use is unverified. | 18 stars · 0 forks | 2026-10-16 | [chisss/titanium-policy](https://github.com/chisss/titanium-policy) · `ZH` |
| Faktor-IPS | —<br>Pending | A Java insurance product-modeling platform with a documented Achmea implementation; the public framework supports product generations and pricing models rather than exposing an insurer's complete application. | 17 stars · 10 forks | 2026-10-15 | [faktorips/faktorips.base](https://github.com/faktorips/faktorips.base) · `EN` |
| OpenUnderwriter | —<br>Pending | A legacy Java insurance distribution platform for quotation, rating, and policy issuance; the latest listed release is dated 2019-08-22, while current source history remains unverified. | — | — | [OpenUnderwriter](https://sourceforge.net/projects/oquote/) · `SourceForge` / `Bitbucket` · `EN` |
| Apache Fineract | —<br>Pending | A Java/Spring core-banking backend used by financial institutions, with loan and savings account lifecycles connected to accounting. | 2,555 stars · 2,868 forks | 2026-10-17 | [apache/fineract](https://github.com/apache/fineract) · `EN` |

## 2. Spring Repositories

| Repository | Status | Description | Popularity | Last Commit | Repository Location |
| --- | --- | --- | --- | --- | --- |
| SpringAll | Pending | An older Spring Boot curriculum spanning application development, startup mechanisms, and container internals. | 28,948 stars · 8,077 forks | 2021-01-31 | [wuyouzhuguli/SpringAll](https://github.com/wuyouzhuguli/SpringAll) · `ZH` |
| source-code-hunter | Pending | Spring source analysis tracing the Bean lifecycle from discovery to post-processing. | 23,135 stars · 4,224 forks | 2026-09-24 | [doocs/source-code-hunter](https://github.com/doocs/source-code-hunter)<br>[Doocs/source-code-hunter](https://gitee.com/Doocs/source-code-hunter) · `Gitee` · `ZH` |
| spring-analysis | Pending | Spring source-reading notes organized by framework component, from the container to MVC. | 13,753 stars · 4,112 forks | 2023-03-24 | [seaswalker/spring-analysis](https://github.com/seaswalker/spring-analysis) · `ZH` |
| mini-spring | Pending | A handwritten Spring implementation for understanding container mechanics, AOP, and circular dependencies. | 6,368 stars · 1,444 forks | 2026-08-13 | [DerekYRC/mini-spring](https://github.com/DerekYRC/mini-spring) · `ZH` |
| small-spring | Pending | A handwritten Spring course building the container incrementally, with separate chapter repositories on Gitee. | 4,899 stars · 1,422 forks | 2023-07-17 | [fuzhengwei/small-spring](https://github.com/fuzhengwei/small-spring)<br>[small-spring (course organization)](https://gitee.com/small-spring) · `Gitee` · `ZH` |
| spring-boot-demo | Pending | A modular Spring Boot laboratory for practical application integrations. | 34,086 stars · 10,868 forks | 2022-09-02 | [xkcoding/spring-boot-demo](https://github.com/xkcoding/spring-boot-demo)<br>[xkcoding/spring-boot-demo](https://gitee.com/xkcoding/spring-boot-demo) · `Gitee` · `ZH` |
| spring-boot-examples | Pending | Minimal runnable Spring Boot 3.0 examples that isolate one capability at a time. | 30,490 stars · 12,134 forks | 2022-12-29 | [ityouknow/spring-boot-examples](https://github.com/ityouknow/spring-boot-examples)<br>[ityouknow/spring-boot-examples](https://gitee.com/ityouknow/spring-boot-examples) · `Gitee` · `ZH` |
| SpringBoot-Labs | Pending | A Spring Boot 2.x laboratory covering application development and distributed Spring systems. | 20,133 stars · 6,132 forks | 2022-05-03 | [yudaocode/SpringBoot-Labs](https://github.com/yudaocode/SpringBoot-Labs)<br>[yudaocode/SpringBoot-Labs](https://gitee.com/yudaocode/SpringBoot-Labs) · `Gitee` · `ZH` |
| SpringBoot-Learning | Pending | A Spring Boot 1.x/2.x tutorial with versioned examples and explanations. | 15,696 stars · 4,780 forks | 2022-02-14 | [dyc87112/SpringBoot-Learning](https://github.com/dyc87112/SpringBoot-Learning)<br>[didispace/SpringBoot-Learning](https://gitee.com/didispace/SpringBoot-Learning) · `Gitee` · `ZH` |
| spring-boot-best-practice | Pending | Java 21 examples for Spring Boot 4 and Spring Framework 7, including startup behavior and custom starters. | 5,113 stars · 1,355 forks | 2026-03-09 | [javastacks/spring-boot-best-practice](https://github.com/javastacks/spring-boot-best-practice)<br>[mirrors_javastacks/spring-boot-best-practice (mirror)](https://gitee.com/mirrors_javastacks/spring-boot-best-practice) · `Gitee` · `ZH` |

[Trading study notes](trading/README.md) · [Studied revisions](SOURCES.md)
