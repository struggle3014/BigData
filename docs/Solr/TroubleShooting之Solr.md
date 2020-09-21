<div align="center"><img src="https://gitee.com/struggle3014/picBed/raw/master/name_code.png"></div>

# 导读

在使用 Solr 的过程中，自己挖了很多的坑，同时的确也存在很多的坑，此处做总结，方便后续规避掉此类问题。

***持续更新中~***



# 目录

<nav>
<a href='#导读' style='text-decoration:none;font-weight:bolder'>导读</a><br/>
<a href='#目录' style='text-decoration:none;font-weight:bolder'>目录</a><br/>
<a href='#正文' style='text-decoration:none;font-weight:bolder'>正文</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1 写数据问题' style='text-decoration:none;${border-style}'>1 写数据问题</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2 查数据问题' style='text-decoration:none;${border-style}'>2 查数据问题</a><br/>
<a href='#总结' style='text-decoration:none;font-weight:bolder'>总结</a><br/>
<a href='#参考文献' style='text-decoration:none;font-weight:bolder'>参考文献</a><br/>
</nav>

# 正文

## 1 写数据问题

在使用 Solr 进行企业征信开发的过程中，我们经历过一些事情。起初数据量较小，基本上没有问题，随着后业务的扩展和数据量的急剧增大我们遇到了诸多问题。如：

* 业务变动，导致需要对 Solr 的 Collection 进行升级。
* 大数据量（约 3000 多万）导入 Solr，导入响应超时，查询缓慢。

为了解决上述遇到的问题，我们对代码层面及集群层面分别进行优化，经历了多次版本迭代：

1. **第一版**

   使用 HttpSolrClient 导入。在数据量较小的情况下可以胜任，但是针对数据量庞大的情况下就显得捉襟见肘了。

   ```scala
   val solrClient = new HttpSolrClient.Builder("http://my-solr-server:8983/solr/testcollection").build()
   val documents = new ArrayBuffer[SolrInputDocument]()
   
   val document = new SolrInputDocument()
   document.addField("key1", "value1")
   document.addField("key2", "value2")
   
   documents += document
   
   solrClient.add(documents.asJava)
   solrClient.commit()
   ```

   后续，我们分析了代码。创建 HttpSolrClient 时，指定数据发送到特定节点。显然，在高并发的情况下，这种方式很不合理。

   

2. **第二版**

   针对于第一版的情况，我们首先想到去解决单节点处理问题。查询官方指南，使用 CloudSolrClient。CloudSolrClient 会连接 ZK，由 ZK 返回相对节点处理请求，达到负载均衡的目的。

   ```scala
   val zkHosts = Array[String]("cdh-85:2181", "cdh-86:2181").toList
   val cloudSolrClient = new CloudSolrClient.Builder(zkHosts.asJava).build()
   
   val documents = new ArrayBuffer[SolrInputDocument]()
   
   val document = new SolrInputDocument()
   document.addField("key1", "value1")
   document.addField("key2", "value2")
   
   documents += document
   
   cloudSolrClient.add(documents.asJava)
   cloudSolrClient.commit()
   ```

   改造完后，使用该方式，发现仍然存在大量超时的现象。为此，我们经过排查，发现由于大量的请求，会导致磁盘 IO 频繁刷新，尝试使用 CloudSolrClient 软提交的方式。

   下方是官方的说明。太频繁的提交会影响 Solr 性能。

   > Be very careful when triggering commits from the client side. Commits are heavy operations and WILL impact Solr performance when executed too often or too close together. Instead, consider using ‘commitWithin’ when adding documents or rely on your core’s/collection’s ‘autoCommit’ settings.

   在生产环境中使用，发现该方式并不能解决遇到的问题。

   

3. **第三版**

   后续，我们寻找了 Spark 与 Solr 的整合方案，spark-solr。经过实践，发现性能还不错，之前的千万级数据导入只需要几分钟。很好地解决了我们地问题。

   spark-solr 使用方式很简单，大致步骤如下：

   * 导入依赖

     ```xml
     <dependency>
       <groupId>com.lucidworks.spark</groupId>
       <artifactId>spark-solr</artifactId>
       <version>3.6.0</version>
     </dependency>
     ```

     

   * 编码

     ```scala
     val configMap = Map[String, String](("zkhost", zkhost + "/solr"), ("collection", COLLECTION_NAME), ("batch_size", "2000000"), ("commit_within", "5000"))
     dataFrame.repartition(3).write.format("solr").options(configMap).save()
     ```

     

## 2 查数据问题

Solr 数据量在达到千万级别时，发现查询性能很低。而且，在某一段时间凌晨5点，Solr 各项性能指标告警。但这段时间内，并不会有大量的 Solr 操作，当时很困惑。我们的配置和代码都不存在较大问题，通过进一步排查，发现 Solr 集群的磁盘 IO 操作导致的问题。由于我们的 Solr 是搭建在大数据集群上的，凌晨存在大量的离线任务，会导致节点磁盘 IO 过高。为此，我们后续将 Solr 部署在单独的集群上，后续未发现延迟问题。



# 总结

在学习新技术时，需要阅读官方指南和相关的权威书籍。可以把握技术点的整体框架，同时避免走很多弯路，做很多无用功。



# 参考文献

[1] [Solr 官方指南](https://lucene.apache.org/solr/guide/8_5/solr-tutorial.html)

[2] [Solr 权威指南](https://99baiduyun.com/baidu/Solr权威指南)