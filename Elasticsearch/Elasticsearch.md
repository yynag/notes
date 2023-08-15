# Elasticsearch

## Elasticsearch 核心技术与实战

[初识Elasticsearch](./初识Elasticsearch.pdf)

[深入了解Elasticsearch](./深入了解Elasticsearch.pdf)

[管理 Elasticsearch集群](./管理 Elasticsearch集群.pdf)

[利用ELK做大数据分析](./利用ELK做大数据分析.pdf)

[应用实战工作坊](./应用实战工作坊.pdf)

> Elasticsearch 核心技术与实战 Github：https://github.com/geektime-geekbang/geektime-ELK 


## 简介

Elasticsearch 是一个分布式、RESTful 风格的搜索和数据分析引擎。

Elasticsearch 会集中存储您的数据，让您飞快完成搜索，微调相关性，进行强大的分析，并轻松缩放规模。

主要功能：海量数据分户式存储 和 集群管理、近实时搜索，性能卓越、海量数据近实时分析。

生态圈：作为数据平台，Kibana可视化分析、ES存储和计算、Logstah/Beats 数据抓取、X-Pack商业化套件。

X-Pack：安全、告警、监控、图查询、机器学习。

Logstah：数据处理管道， 支持从不同来源采集数据、转换数据，并将数据发送到不同的存储库。

Beats：轻量的数据采集器。

ELK客户和应用场景：网站搜索、垂直搜索、代码搜索、日志管理和分析、安全指标监控、应用性能监控、Web抓取舆情分（？？？）

日志管理：日志搜索，格式化分析，全文检索，风险告警。

ES 也可以通过同步机制与数据库集成，做读写分离。

集成 ArcSight 安全分析。

【 ES 的价值就是从海量的数据中抓取关键字，然后统计，形成结论，也可以用于搜索，这是另一个用途。 】


## 技能要求

开发：产品基本功能、底层工作原理、数据建模最佳实践。

运维：容量规划、问题诊断、滚动升级。

方案：搜索、解决搜索相关问题、大数据分析、实际场景。


## 安装

> PPT 初识Elasticsearch Page47

---

目录结构：PPT 初识Elasticsearch Page53

ES 包含 模块、插件、数据文件、集群配置文件、日志文件 这些文件夹。

ES 充分运用插件。

---

ES 作为单机的时候，不用说。

一旦还是扩展：协调节点群 Coordinate Nodes、数据节点群 Data Nodes

再扩展：热数据节点群 Hot Nodes、温数据节点群 Warm Nodes、深度学习节点群 ML Nodes

【 根据规模进行节点群的配置，跟一般分片集群还不一样。 】

ES 使用 Java，内存不要超过总内存 50%，最大不要超过 30GB。

--- 

Cerebro：集群管理的可视化工具。


## 基本概念 — 索引、文档

> PPT 初识Elasticsearch Page82 

元数据：PPT 初识Elasticsearch Page87

_score 相关性打分：我搜索的东西，一个相关性。

Index 索引：跟 RDBMS 不一样，类似于其 表。

每个节点启动后，默认就是 master eligible 节点（有资格成为master的节点），可以设置关闭，第一个节点启动会自己选举为 Master 节点，每个节点都保存了集群的状态，只有 Master 才能修改集群状态，比如 所有节点的信息、索引、settings、分片的路由。

Data Node：数据节点。

Coordinating Node：接受客户端请求，分发到合适的节点上，最终把结果汇集到一起，每个节点默认起到 Coordinating Node 的职责。—— 【 分布式的 】

Hot & Warm Node：不同硬件配置的 Data Node，降低部署成本。

Tribe Node：连接到不同集群，支持将这些集群作为一个单独集群处理。（5.3 开始 Cross Cluster Search）

## 基本概念 — 集群、节点、分片、副本

分片：

> PPT 初识Elasticsearch Page104

主分片：把数据分布到节点上，一个分片一个实例，数量创建索引的时候指定，后期不能修改，除非 Reindex。【 相当于 rehash 】

副本分片：解决高可用，对主分片的拷贝，相当于从节点，可以提高读取的吞吐。

settings 里面，number_of_shards 和 number_of_replicas 设置分片和副本，一般，一个节点一个主分片，有 number_of_shards 个节点，有 number_of_replicas 个副本。

生产环境下，提取做好容量规划，分片数太小，后期没办法水平扩展，如果重新分配，特别耗时。

分片数过大，一方面导致资源浪费，另方面影响相关性打分。

集群状况：GET _cluster/health。

> 颜色 PPT 初识Elasticsearch Page107


## 基本操作

> PPT 初识Elasticsearch Page110
> Document APIs https://www.elastic.co/guide/en/elasticsearch/reference/current/docs.html


## 倒排索引

> PPT 初识Elasticsearch Page124

倒排索引 就是 词典的索引页，Key 就是关键词，Value 就是 个数、位置。


### Analyzer 分词

> PPT 初识Elasticsearch Page135

分词不用说，有很多插件可以选择，模块化设计，一个分词器由多个分析器组合而来。

可以用 _analyzer API 进行测试效果。

中文比较难点，但有很多选择，基于机器学习方案的选择。


### Search API

> PPT 初识Elasticsearch Page153

URI Search：在 URL 中使用查询参数

Request Body：在 Body 里面提供 JSON，DSL（Query Domian Specific Language 特定领域查询语言）。

也可以看看搜索的 Response 结构体：PPT 初识Elasticsearch Page159

---

搜索相关性：

Page Rank 算法：文档被多少人引用，相关性更高，可信度越好。

电商：购物体验（相关）、门店销售业绩、库存的问题

衡量相关性：

1. 查准率：尽可能返回较少的无关文档。

2. 查全率：尽可能返回较多的相关文档。

3. 能否按照相关读排序。


### URI Search

自己看

> PPT 初识Elasticsearch Page165


### Request Body & Query DSL

自己看

> PPT 初识Elasticsearch Page174


### Dynamic Mapping

> PPT 初识Elasticsearch Page188

Mapping 类似于数据库 schema 定义，定义字段名称、数据类型、倒排索引的相关配置，相当于 表设计。

Dynamic Mapping：不需要刻意去 DDL，自动生成，但问题是，有时候推断不对，比如 地理位置信息。

动态Mapping 如果出现文档字段不一致的情况：

1. Dynamic 设置为 true，字段自动更新，自动 DDL。

2. Dynamic 设置为 true，字段不会更新，不能被索引，但信息会出现在 _source 里。

3. Dynamic 设置为 strict，写入失败。

4. 对已有的字段，不支持修改字段定义，如果想修改，必须重建索引。

【 我个人建议 strict 比较好，可以提高性能。 】

--- 

可以显示定义 Mapping，参阅 PPT 初识Elasticsearch Page198

1. 可以控制字段是否参与倒排索引。

2. 可以控制索引记录的内容，比如有些索引记录位置就可以了，有些索引需要记录 文档名称，具体偏移 这些信息。

3. copy_to：理解为构造一个虚拟字段。

4. 多字段类型，必须同一个字段，不同语言设置分词，拼音。

5. Exact Values：专有词，不需要被分词。

 多看看 PPT。


### Index Template 和 Dynamic Template

> PPT 初识Elasticsearch Page218

Index Template 类似于 分表，比如日志，日志-2011，日志2012，拥有共同的 Index表模板。

Dynamic Template 就是在 Index Template 基础上，自动设置数据类型。


### Aggregation 聚合分析

> PPT 初识Elasticsearch Page227

Bucket Aggregation：满足特定条件文档的集合，类似于 SQL 的 GROUP BY。

Metric Aggregation：数学运算，对文档字段统计分析，类似于 SQL 的统计函数，比如 COUNT(*)

Pipeline Aggregation：其他聚合结果的二次聚合。

Matrix Aggregation：对多个字段操作，提供一个结果矩阵。

## 高级话题

### 词项 和 全文搜索

> 深入了解Elasticsearch p4

Term 查询：对输⼊入不不做分词，将输⼊入作为⼀一个整体，在倒排索引中查找准确的词项，并 且使⽤用相关度算分公式为每个包含该词项的⽂文档进⾏行行相关度算分，例例如“Apple Store”。

可以通过 Constant Score 将查询转换成⼀一个 Filtering，避免算分，并利利⽤用缓存，提⾼高性能。示例：深入了解Elasticsearch p8。

全⽂查询：查询时候，先会对输⼊入的查询进⾏行行分词，然后每个词项逐个进⾏行行底层的查询，最终将结果进⾏行行合 并。并为每个⽂文档⽣生成⼀一个算分。- 例例如查 “Matrix reloaded”，会查到包括 Matrix 或者 reload 的所有结果。

### 结构化搜索

> 深入了解Elasticsearch p17

结构化搜索(Structured search) ：指对结构化数据的搜索，⽇日期，布尔类型、数字 和 文本。

如果不不需要算分，可以通过 Constant Score，将查询转为 Filtering。

#### 搜索的相关性算分

> 深入了解Elasticsearch p27

搜索的相关性算分，描述了了⼀一个⽂文档和查询语句句匹配的程度。ES 会对每个匹配查询条件的结
果进⾏行行算分 _score。

打分的本质是排序，需要把最符合⽤用户需求的⽂文档排在前⾯面。ES 5 之前，默认的相关性算分 采⽤用 TF-IDF，现在采⽤用 BM 25。—— 【 写下用于搜索。 】

---

TF-IDF

TF：Term Frequency，词频，检索词在⼀一篇⽂文档中出现的频率。

DF：Document Frequency，检索词在所有⽂文档中出现的频率。

IDF：Inverse Document Frequency，逆⽂文档频率，log(全部⽂文档数/检索词出现过的⽂文档总数)。

TF-IDF 本质上就是将 TF 求和变成了了加权求和：TF(区块链)*IDF(区块链) + TF(的)*IDF(的)+ TF(应用)*IDF(应用)。

【 TF是正向的，越高，说明文档的相关性越强，IDF是逆向的，越高意味着这个词在其他文档中出现的次数小。 】 

---

BM 25

和经典的TF-IDF相⽐比，当 TF ⽆无限增加时， BM 25算分会趋于⼀一个数值。


### Query & Filtering 和 多字符串串多字段查询

> PPT 深入了解Elasticsearch p38

高级搜索的功能：支持多项⽂本输入，针对多个字 段进行搜索。搜索引擎一般也提供基于时间，价格等条件的过滤。

在 Elasticsearch 中，有Query 和 Filter 两种不不同的 Contex：

1. Query Context:相关性算分

2. Filter Context:不不需要算分( Yes or No)， 可以利用 Cache， 获得更好的性能。

条件组合：对于一个搜索来说，需要多个条件，这个时候用 bool Query 查询。

> 参见：PPT 深入了解Elasticsearch p42


### 单字符串多字段查询：Disjunction Max Query

> PPT 深入了解Elasticsearch p54

单字符串多字段查询：Google 只提供⼀一个输⼊入框，查询相关的多个字段。

Disjunction Max Query：将评分最高的字段评分作为结果返回，满⾜两个字段是竞争关系的场景。

对最佳字段查询进行调优：通过控制 Tie Breaker 参数，引⼊其他字段对算分的⼀一些影响。

我没看懂，但是这一章要解决的问题是：单字符串多字段查询时，如何在多个字段上进行算分？


### 单字符串多字段查询：Multi Match

我的理解，在多个字段中去匹配。

> PPT 深入了解Elasticsearch p63

### 多语言及中⽂分词与检索

> PPT 深入了解Elasticsearch p71

### Search Template 和 Index Alias

> PPT 深入了解Elasticsearch p95

Search Template：解耦程序 & 搜索 DSL

Index Alias：实现零停机运维


### Suggester

> PPT 深入了解Elasticsearch p110

---

Elasticsearch Suggester API，搜索引擎中类似的功能，在 Elasticsearch 中是通过 Suggester API 实现的。

原理: 将输入的⽂本分解为 Token，然后在索引的字典里查找相似的 Term 并返回。

根据不同的使用场景，Elasticsearch 设计了了 4 种类别的 Suggesters：

- Term & Phrase Suggester
- Complete & Context Suggester

Term Suggester：

Phrase Suggester：Phrase Suggester 在 Term Suggester 上增加了了一些额外的逻辑，自己看PPT。

Completion Suggester：提供了了“自动完成” (Auto Complete) 的功能。⽤户每输⼊入一个 字符，就需要即时发送⼀个查询请求到后段查找匹配项。

Context Suggester：Completion Suggester 的扩展，可以在搜索中加入更多的上下文信息，例如，输入 “star”，咖啡相关:建议 “Starbucks”，电影相关:”star wars”。


### 跨集群搜索（Cross Cluster Search）

> PPT 深入了解Elasticsearch p132


### 集群分布式模型及选主与脑裂问题

> PPT 深入了解Elasticsearch p138

Coordinating Node

Data Node

Master Node

Master Eligible Nodes & 选主流程？

集群状态？

Master Eligible Nodes & 选主的过程？

如何避免脑裂问题？


### 分⽚与集群的故障转移

> PPT 深入了解Elasticsearch p154

Primary Shard - 提升系统存储容量量

Replica Shard - 提⾼高数据可⽤用性

集群健康状态


### 文档分布式存储

> PPT 深入了解Elasticsearch p165

文档到分片的路由算法。

可以通过设置 Index Settings，控制数据的分⽚。


### 分片及其生命周期

> PPT 深入了解Elasticsearch p173

倒排索引不不可变性，一旦⽣成，不可更改，好处就是没有并发问题，好处就是需要重建整个索引。

在 Lucene 中，单个倒排索引⽂件被称为 Segment。Segment 是⾃包含的，不可变更更的。 多个 Segments 汇总在⼀起，称为 Lucene 的 Index，其对应的就是 ES 中的 Shard。

看PPT。


### 剖析分布式查询及相关性算分

> PPT 深入了解Elasticsearch p180

分布式查询机制：

1. ⽤户发出搜索请求到 ES 节点。节点收到请求 后， 会以 Coordinating 节点的身份，在 6 个 主副分⽚中随机选择 3 个分⽚，发送查询请求。

2. 被选中的分片执⾏行行查询，进行排序。然后，每 个分片都会返回 From + Size 个排序后的文档 Id 和排序值 给 Coordinating 节点。

3. Coordinating Node 会将 Query 阶段，从 从每个分片获取的排序后的文档 Id 列表， 重新进行排序。选取 From 到 From + Size 个⽂文档的 Id。

4. 以 multi get 请求的⽅式，到相应的分片获 取详细的⽂档数据。

Query Then Fetch 的问题：性能 和 相关性算分。

---

每个分⽚片都基于⾃己的分⽚上的数据进⾏相关度计算。这会导致打分偏离的情况，特别是数据量很少时。相关性算分在分⽚之间是相互独⽴。当文档总数很少的情况下，如果主分片⼤于 1，主分⽚片数越多 ，相关性算分会越不不准

解决：

1. 数据量量不大的时候，可以将主分⽚片数设置为 1。
2. 搜索的URL 中指定参数 “_search?search_type=dfs_query_then_fetch”，到每个分片把各分片的词频和文档频率进⾏行搜集，然后完整的进行⼀次相关性算分， 耗费更加多的 CPU 和内存，执⾏性能低下，一般不建议使⽤用。


### 排序及 Doc Values & Field Data

> PPT 深入了解Elasticsearch p188

排序过程：排序是针对字段原始内容进行的。 倒排索引⽆法发挥作⽤。

Doc Values：列式存储，ES2.x之后默认。

Field Data：动态创建。


### 分⻚页与遍历 – From，Size，Search After & Scroll API

> PPT 深入了解Elasticsearch p201

分页方案：From:开始位置；Size:期望获取⽂文档的总数。

会在每个分⽚上先都获取 1000 个文档。然后， 通过 Coordinating Node 聚合所有结果。最后 再通过排序选取前 1000 个⽂文档。

⻚页数越深，占⽤内存越多。为了避免深度分页带 来的内存开销。ES 有⼀个设定，默认限定到 10000 个⽂文档

Search After 避免深度分页的问题，指定在XXX之后，原因是通过唯一排序值定位。

Scroll API 通过快照查询。


### 处理理并发读写操作

> PPT 深入了解Elasticsearch p212

ES 的乐观并发控制：ES 中的⽂档是不不可变更更的。如果你更更新⼀一个⽂文档，会将 就⽂文档标记为删除，同时增加⼀一个全新的⽂文档。同时⽂文档 的 version 字段加 1。


### Bucket & Metric 聚合分析及嵌套聚合

> PPT 深入了解Elasticsearch p216


### Pipeline 聚合分析

> PPT 深入了解Elasticsearch p229

管道的概念: ⽀持对聚合分析的结果，再次进行聚合分析

### 聚合的作⽤用范围及排序

> PPT 深入了解Elasticsearch p237

---

作用范围：

Filter：过滤

Post_Filter：对分析后的文档再次过滤，

Global：无视 query，对全部文档进行统计。

---

排序

默认情况，按照 count 降序排序


### 聚合的精准度问题

> PPT 深入了解Elasticsearch p244


### 对象及 Nested 对象

> PPT 深入了解Elasticsearch p252


### 文档的⽗子关系

> PPT 深入了解Elasticsearch p266


### Update By Query & Reindex API

> PPT 深入了解Elasticsearch p279


### Ingest Pipeline 与 Painless Script

> PPT 深入了解Elasticsearch p289

Ingest Node vs Logstash  参阅 p301

### Elasticsearch 数据建模实例例

> PPT 深入了解Elasticsearch p311


### Elasticsearch 数据建模最佳实践

> PPT 深入了解Elasticsearch p327


## 管理 Elasticsearch 集群

> PPT 管理 Elasticsearch 集群

---

集群身份认证与用户鉴权 3

集群内部间的安全通信 20

集群与外部间的安全通信 25

常见的集群部署方式 31

Hot & Warm 架构与 Shard Filtering 43

分片设定及管理 58

如何对集群进行容量规划 66

在私有云上管理 Elasticsearch 的一些方法 80

在公有云上管理与部署 Elasticsearch 89

生产环境常用配置和上线清单 96

集群写性能优化 110

集群读性能优化 122

诊断集群的潜在问题 132

解决集群 Yellow 与 Red 的问题 140

集群压力测试 148

段合并优化及注意事项 163

缓存及使用 Circuit Breaker 限制内存使用 167

监控 Elasticsearch 集群 180

一些运维相关的建议 185

使用 Shrink 与 Rollover API 管理索引 203

索引全生命周期管理及工具介绍 214


## ELK大数据分析

> PPT 利用 ELK 做大数据分析 p1















