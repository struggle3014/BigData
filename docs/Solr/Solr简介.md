<div align="center"><img src="https://gitee.com/struggle3014/picBed/raw/master/name_code.png"></div>

# 导读

本篇我们对 Solr 搜索做简单介绍。文章主要从 Solr 简介，Solr 历史，Solr 关键特性，Solr 架构，Solr 管理界面，Solr Schema 体系，Solr Core，Solr Documents 和 Fileds，Solr 索引数据，Solr 分析，Solr 搜索过程，Solr 特性，配置 Solr 实例/Core，Solr Cloud 等方面介绍。开启 Solr 之旅吧！

***持续更新中~***



# 目录

<nav>
<a href='#导读' style='text-decoration:none;font-weight:bolder'>导读</a><br/>
<a href='#目录' style='text-decoration:none;font-weight:bolder'>目录</a><br/>
<a href='#正文' style='text-decoration:none;font-weight:bolder'>正文</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1 Solr 简介' style='text-decoration:none;${border-style}'>1 Solr 简介</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2 Solr 历史' style='text-decoration:none;${border-style}'>2 Solr 历史</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3 Solr 关键特性' style='text-decoration:none;${border-style}'>3 Solr 关键特性</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4  Solr 架构' style='text-decoration:none;${border-style}'>4  Solr 架构</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5 Solr Admin UI' style='text-decoration:none;${border-style}'>5 Solr Admin UI</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6 Solr Schema 体系' style='text-decoration:none;${border-style}'>6 Solr Schema 体系</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#7 Solr Core' style='text-decoration:none;${border-style}'>7 Solr Core</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#8 Solr Documents 和 Filelds' style='text-decoration:none;${border-style}'>8 Solr Documents 和 Filelds</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#9 Solr 索引' style='text-decoration:none;${border-style}'>9 Solr 索引</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#9.1 Lucene 索引原理' style='text-decoration:none;${border-style}'>9.1 Lucene 索引原理</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#9.2 索引数据源及创建索引' style='text-decoration:none;${border-style}'>9.2 索引数据源及创建索引</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#10 Solr 分词（Analysis）' style='text-decoration:none;${border-style}'>10 Solr 分词（Analysis）</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#10.1 基本概念' style='text-decoration:none;${border-style}'>10.1 基本概念</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#10.2 中文分词器' style='text-decoration:none;${border-style}'>10.2 中文分词器</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#11 Solr 搜索过程' style='text-decoration:none;${border-style}'>11 Solr 搜索过程</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#12 Solr 特性' style='text-decoration:none;${border-style}'>12 Solr 特性</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#13 配置 Solr 实例/Core' style='text-decoration:none;${border-style}'>13 配置 Solr 实例/Core</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#14 Solr Cloud' style='text-decoration:none;${border-style}'>14 Solr Cloud</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='# 14.1 Solr Cloud 简介' style='text-decoration:none;${border-style}'> 14.1 Solr Cloud 简介</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#14.2 Solr Cloud 特性' style='text-decoration:none;${border-style}'>14.2 Solr Cloud 特性</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#14.3 Solr Cloud 概念及术语' style='text-decoration:none;${border-style}'>14.3 Solr Cloud 概念及术语</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#14.4 Solr Cloud 架构' style='text-decoration:none;${border-style}'>14.4 Solr Cloud 架构</a><br/>
<a href='#参考文献' style='text-decoration:none;font-weight:bolder'>参考文献</a><br/>
</nav>

# 正文

## 1 Solr 简介

* Solr 是一个企业级搜索服务/web 引用。

* Solr 使用并扩展了 Lucene 搜索库。

* Solr 将 lucene Java API 公开为 REST-Full 服务。

* 可以通过 HTTP 方式以 XML，JSON，CSV 或 binary 将文档存放至 Solr（称之为 "索引"）。

* 可以通过 HTTP Get 方法来查询获取 XML，JSON，CSV 或 binary 结果。



## 2 Solr 历史

2004 年，Solr 由 CNET Networks 的 “Yonik Seeley” 创建，起初是为公司网站增加搜索功能的内部项目。

2006 年 1 月，CNET Networks 决定开源，并将其捐赠给 Apache 软件基金会的 Lucene 顶级项目下。

2008 年 9 月，Solr 1.3 发布了，包含了很多增强的功能，如分布式搜索能力和性能增强等。

2012 年 10 月，Solr 4.0 发布，包含了 SolrCloud 特性。



## 3 Solr 关键特性

高级的全文搜索能力。

优化了高容量的网络流量问题。

标准基础的开源接口—XML，JSON 和 HTTP。

全面的 HTML 管理界面。

通过 JMX 公开用于监控服务器统计信息。

近乎实时的索引和 XML 配置适配。

线性扩缩容，自动索引备份，可扩展的插件架构。



## 4  Solr 架构

![solr-architecture](https://gitee.com/struggle3014/picBed/raw/master/solr-architecture.png)



## 5 Solr Admin UI

![solr-admin-ui](https://gitee.com/struggle3014/picBed/raw/master/solr-admin-ui.png)



## 6 Solr Schema 体系

![solr-schema-hierarchy](https://gitee.com/struggle3014/picBed/raw/master/solr-schema-hierarchy.png)



## 7 Solr Core

是运行的一个 Lucene 索引的示例，以及它所需的所有 solr 配置（SolrConfigXml，SchemaXml 等）。

一个 Solr 引用可能包含 0 个或多个 cores。

Cores 在很大程度上是独立运行的，但是必要时可以通过 CoreContainer 互相通信。

Solr 最初只支持单个索引，而 SolrCore 类是用于协调 solr 核心底层功能的单例。



## 8 Solr Documents 和 Filelds

Solr 的基础信息单元是 document，它是描述某些事情的数据集合。

Documents 由 fields 组成，它们是更具体的信息片段。

Fields 可以包括不同类型的数据。例如，名称 Field 是 text（字符数据）。

Field 类型告知 Solr 如何解释 filed，如何查询。

```xml
<doc>
    <filed name="id">9885A004</filed>
    <field name="name">Canon PowerShot SD500</field>
    <field name="manu">Canon Inc.</field>
    ...
    <field name="inStock">true</field>
</doc>
```



## 9 Solr 索引

Solr 是基于 Lucene 实现的，为了了解 Solr 的索引，我们有必要了解 Lucene 的内部索引原理。

### 9.1 Lucene 索引原理

Lucene 建立索引的过程就是创建倒排索引表的过程，为此，Lucene 设计了一个良好的索引结构，Lucene 索引结构总有几个基础概念：索引 Index，文档 Document，词 Term，域 Field，段 Segment。

* 索引（Index）
  * 在 Lucene 中一个索引是放在一个文件夹中，一个索引就是多个 Document 的集合。
  * 同一个索引目录中的所有文件构成一个 Lucene 索引。
  * 一个索引其实就是多个 Docuement 的集合。
* 词（Term）
  * 每个 Filed 的值经过分词器处理后得到的每一项成为 Term。
  * 是索引中最小单元。
  * 在两个不同的 Field 中的同一个字符串被认为是不同的 Term。
  * Field 的域经过序列化（tokenized）成 Term 集合。
* 文档（Document）
  * 文档是构建索引的基本单位，索引中的每个 Docuement 好比数据库表中每条记录 Record。
  * 新添加的文档单独保存在一个新生成的段文件中。
* 域（Field）
  * 一个 Document 是一个 Field 的集合，每个 Field 好比数据库表中的每个字段 column。
  * 不同 Filed 可以分别存储不同信息，以及拥有各自不同的存储方式。
* 段（Segment）
  * 当添加一个新文档就会生成一个新的段，并且也会触发段文件合并。
  * 一个索引可以包含多个段文件，段与段之间是相互独立的。
  * 合并段文件有助于提升创建索引的性能。
  * 段文件里记录了索引中包含多个段，每个段包含多少个文档。
  * 索引可以由读个子索引构成，这个子索引便成为段。



理解基本概念，我们进一步了解 Lucene 倒排索引表的基本结构及构建过程。倒排索引表结构的设计使得索引查询非常高效，倒排索引是大多数现代搜索引擎的基础。

**Lucene 内部倒排索引表构建过程**大致如下：

假设，有三篇文档。

> D0 = “What makes life dreary is the want of motivate.”
>
> D1 = "I figure life is a gift and I don't intend to wasting it."
>
> D2 = "Life is made up of small pleasures."



* 文档分词。

  对文档进行分词处理。英文一般按空格分词，中文需要用中文分词器分词，例如，将 D0 分词为 "What" "makes" "life" ...

  

* 词处理，获取文档 -> 词的映射。

  得到所有单词后，需要进一步处理。如剔除毫无意义的单词（“the” “are” “at” “in” “to” 等），中文中的停顿词（“的”， “是”），单词小写转换，英文词态还原（如 ”loved “ 还原为 “love”），剔除标点符号。

  我们获取下述信息：

  > Doc0 = make, life, dreary, want, motivate
  >
  > Doc1 = figure, life, gift, intend, waste
  >
  > Doc2 = life, make, small, pleasure

  

* 词统计，生成倒排列表。

  统计词频，记录词位置（关键词高亮显示）。

  我们可能得到如下倒排索引结构：

  > make -> 0, 1, [5, 9]	2, 1, [8, 11]
  >
  > life -> 0, 1, [11, 14]	1, 1, [9, 12]
  >
  > gift -> 1, 1, [19, 22]
  >
  > // ...

  0, 1, [5, 9] 依次表示文档的 id，词频，词出现的位置。



### 9.2 索引数据源及创建索引

Solr 索引可以接受不同类型的数据源，包括 XML 文件，逗号分割值文件（comma-separated value—csv），数据库抽取的表数据，和一些通用的文件格式，如 Word 或 PDF。

下面是三种最常见的加载数据到 Solr index 方式：

* 通过发送 Solr HTTP 请求上传 XML 文件。
* 使用 Index Handlers 从数据库导入数据。
* 使用 Solr Cell 框架。
* 通过 Java 客户端编写自定义 Java 应用来导入数据。



## 10 Solr 分词（Analysis）

### 10.1 基本概念

![solr-analysis](https://gitee.com/struggle3014/picBed/raw/master/solr-analysis.png)

分词就是将用户输入的一串文本分割成一个个 token，一个个 token 组成了 tokenStream，然后遍历 tokenStream 对齐进行过滤操作，比如去停用词、特殊字符、标点符号和同一小写等。分词的正确与否直接影响搜索的相关排序，从某种程度上将，分词算法的不同，都会影响页面的返回结果。分词是搜索的基础。

在分词中有三个主要概念：分词器（analyzers），标记器（tokenizers）和过滤器（filters）。

* 分词器（analyzers）在 document 索引期间和查询期间使用到。

  * 两个操作不需要使用到相同的分析过程。
  * 分析器检查字段的文本并生成一个标记流。
  * 分析器可以使单个类，也可以由一系列的分词器和过滤器组成。

* 标记器（tokenizers）将 field 数据分成词汇单元（lexical units）或词。 

* 过滤器（filters）检查流标记并保留，转换，丢弃它们，或创建新的标记。



### 10.2 中文分词器

中文分词器主要用来对中文语句中包含的词汇进行断词，分割出一个个包含独立含义的词语，当然，分出的词语最好能与原始语句语义相吻合。中文分词一般同时还附带去除标点符号，去除停用词，加载扩展字典对陌生此的分词支持，为消除分词语义的歧义，还需要添加磁性标注，语义标注。如“你真的确实太棒啦”，应该被分成：你|真的|确实|棒。此时“的确”看起来也是一个词，但是在当前域经中，不应该被分成一个词。需要涉及上下文语义环境来断词，是分词领域的难题。

#### 10.2.1 分词算法

**分词算法**主要有三大类：基于字典，基于理解，基于统计。

* 基于字典

  基于字典的分词方法就是扫描一个指定的汉语词典进行查找匹配，若匹配上，则会分词。

* 基于理解

  通过语义让计算机模拟人类理解句子的过程，达到分词的目的。目前市面上没有此类分词器。

* 基于统计

  * 基于概率

    两个字相邻出现的次数越多，那么它们组成一个词的概率越大。

  * 基于统计学

    利用统计模型学习词语的切分规律，需要大量的语料库来模拟训练。

#### 10.2.2 中文分词器

* IK 分词器
* Jieba 分词器
* Ansj 分词器



## 11 Solr 搜索过程

![solr-search-process](https://gitee.com/struggle3014/picBed/raw/master/solr-search-process.png)



* 定义字段
  在 managed-schema 中配置搜索需要的字段，包括该字段是否被搜索、是否被返回。
* 导入数据
  向 Solr 导入数据的过程也是 Solr 建立索引的过程。Solr 会根据对应的分词方式来建立索引。
* 查询请求
  请求处理器将请求映射到搜索组件，搜索组件先将关键词进行分词，再通过分词后的词组向 Lucene 进行检索，找到相关的分词进行返回。
* 结果响应
  根据用户指定的格式，将搜索结果返回给用户。



## 12 Solr 特性

* Faceting（分类）

  将搜索结果分类到各种类别中。

* Highlighting（高亮显示查询）

* Spell Checking（拼写查询）

* Query-Re-ranking（查询重排序）

* Transforming（转换）

* Suggestors（搜索检查建议）

* More Like This（查询相似文档）

* Pagination（结果分页）

* Grouping & Clustering（分组查询和聚合查询）

* Spatial Search（地理空间搜索）

* Components（组件）

* Real time（Get & Update）（读 & 更新实时操作）

* LABS



## 13 配置 Solr 实例/Core

![solr-configuration](https://gitee.com/struggle3014/picBed/raw/master/solr-configuration.png)



## 14 Solr Cloud

###  14.1 Solr Cloud 简介

* Apache Solr 包含了 Solr servers 集群建设的能力，包括错误容忍和高可用，被称为 SolrCloud。

* SolrCloud 是灵活的分布式搜索和索引，不需要一个 master 节点来分配节点，分片（shards）和副本（replicas）。

* 根据配置文件和约束（schemas），Solr 使用 ZooKeeper 来管理这些位置

* 文档可以被发送到任何服务节点，ZooKeeper 会找到它。



### 14.2 Solr Cloud 特性

* Horizontal Scaling（For Sharding & Replication）（水平扩展，对于分片和副本）
* Elasatic Scaling（弹性扩容）
* High Availability（高可用饭）
* Distributed Indexing（分布式索引）
* Distribution Searching（分布式查询）
* Central Configuration For Entire Cluster（整个集群的中央配置）
* Automatic Loading Balancing（自动负载均衡）
* Automatic Failover For Queries（查询自动故障恢复）
* Zookeeper Integration For Coordination & Configuration（集成 Zookeeper 来协调和配置）



### 14.3 Solr Cloud 概念及术语

* Collection：同一类型的索引文档的集合，但是这些索引文档并不实际存储在同一台机器上，它们通常会分割成多个分片，然后每个分片会创建多个副本，所以实际存储的是副本，同一个 Shard 下的每个副本会分散到多个节点上存储。
* Shard分片：单个 Collection 的逻辑划分，划分出来的每一份称之为 Shard ，这里的划分只是逻辑概念上的分割。
* Replica：通过逻辑划分出来的 Shard 会以多个副本的形式实际物理存储在多个节点上，每个副本其实可以看作一个物理存在的“ Core ”，只不过此时副本应用的schema.xml solrconfig.xml 是托管在 Zookeeper 上的。
* Leader：每个 Shard 会复制出多个副本，其中一个副本会被选 Leader （领导），由 Leader 来负责主导分布式环境下的索引和查询请求，与 Leader 对应的还有 Follwer。
* Core：也就是 Solr Core，一个 Solr 中包含一个或者多个 SolrCore，每个 Solr Core 可以独立提供索引和查询功能，Solr Core额提出是为了增加管理灵活性和共用资源。SolrCloud 中使用的配置是在 Zookeeper 中的，而传统的Solr Core的配置文件是在磁盘上的配置目录中。
* Node 节点：表示一个 Solr Server 实例，而一个 Solr Server 实例通常运行于 Web 容器，由于 Web 容器可以提供不同的端口号从而启动不同的 JVM 示例，意味着同一台服务器上可以部署多个 Solr Server 示例。一个 Solr Server 实例可以拥有多个 Solr Core，而每个 Core 下可以包含某个 Collection 的部分索引文档，此时这里的每个 Core 其实就是该 Collection 某个 Shard 下的 Replica（副本）。

- Cluster 集群：一个 Solr 集群，所有的 Solr Server 实例一起托管着所有 Solr Core，在集群环境下，每个 Solr Server 实例下管理的每个 Core 其实就是 Collection 下某个 Shard 的某一个 Replica（副本）。

一个 Solr Core 是 Solr Server 中被唯一命名、可配置的、受管理的索引集合的总称。一个 Solr Server 实例可以拥有 SolrCore。Solr 中的 Core 通常是通过定义不同的 Schema 来分割索引文档的。SolrCloud 中的 Collection 虽然也是索引集合的总称，但它是逻辑概念 Collection 会将索引文档分割成多个 Shard ，这些 Shard 会分布在多个 Solr Server 节点之上，每个 Shard Replica 是真正物理存在的 Core ，而 Collection 并没有物理存在于任意一个节点上。



### 14.4 Solr Cloud 架构

![solr-cloud-architecture](https://gitee.com/struggle3014/picBed/raw/master/solr-cloud-architecture.png)



![solrcloud-architecture](https://gitee.com/struggle3014/picBed/raw/master/solrcloud-architecture.png)

一个 Collection 被分成 Shard 1 和 Shard 2 两个 Shard ，每个 Shard 又由两个 Replica 组成，Shard1 的两个 Replica 分别存储于 Solr Server1 和 Solr Sever2 两个节点上， Shard2 同理。每个 Shard 的 Replica 会经过 Zookeeper 选举出一个 Leader, Solr Server1 节点成为 Shard 1 的 Leader ，而 Solr Server 2 节点成为 Shard 2 的 Leader。当 Shard 1 的 Leader 即 Solr Server 1 节点挂掉了，那么 Shard 1 的另一个 Replica 所在节点 Solr Server 2 会被 Zookeeper 选举为 Shard 1 新的 Leader。而 Shard 1 和 Shard 2 共同组成一个 Collection。当一个 Client 请求集群中任意一个 Solr Server 节点并将索引文档发送该节点时，该节点会首先判断当前索引文档属于哪个 Shard ，确定该 Shard 的 Leader 所在 Solr Server 节点即最终目标节点，然后接收索引文档的源节点将索引文档再转发给最终目标节点，最后在该节点创建索引。



# 参考文献

[1] [Solr 官方指南](https://lucene.apache.org/solr/guide/8_5/solr-tutorial.html)

[2] [Solr 权威指南](https://99baiduyun.com/baidu/Solr权威指南)

[3] [YouTube—Solr 系列视频](https://www.youtube.com/results?search_query=solr)

[4] [Cloudera Search](https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/search.html)