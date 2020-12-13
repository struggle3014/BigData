<div align="center"><img src="https://gitee.com/struggle3014/picBed/raw/master/name_code.png"></div>

# 导读

在大数据计算领域，Spark 变得越来越流行。使用 Spark 的初衷就是能够解决大数据计算作业快速、高效执行的问题。但是需要使用 Spark 开发高性能的作业，并不那么容易。

本文参考了美团技术团队 Spark 性能优化指南及官网优化指南，对 Spark 优化部分的内容做下总结。主要从以下几个方面进行阐述，多个部分之间可能有重合的部分。包括：

* 开发调优
* 资源调优
* 并行度
* 数据本地化调优
* 数据倾斜调优
* Shuffle 优化

***持续更新中~***



# 目录

<nav>
<a href='#导读' style='text-decoration:none;font-weight:bolder'>导读</a><br/>
<a href='#目录' style='text-decoration:none;font-weight:bolder'>目录</a><br/>
<a href='#正文' style='text-decoration:none;font-weight:bolder'>正文</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1 Spark 开发优化' style='text-decoration:none;${border-style}'>1 Spark 开发优化</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1.1 调优概述' style='text-decoration:none;${border-style}'>1.1 调优概述</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='# 1.2 避免创建重复 RDD' style='text-decoration:none;${border-style}'> 1.2 避免创建重复 RDD</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1.3 尽可能复用同一 RDD' style='text-decoration:none;${border-style}'>1.3 尽可能复用同一 RDD</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1.4 对多次使用的 RDD 进行持久化' style='text-decoration:none;${border-style}'>1.4 对多次使用的 RDD 进行持久化</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1.5 尽量避免使用 Shuffle 类算子' style='text-decoration:none;${border-style}'>1.5 尽量避免使用 Shuffle 类算子</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1.6 使用 map-side 预聚合的 shuffle 操作' style='text-decoration:none;${border-style}'>1.6 使用 map-side 预聚合的 shuffle 操作</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1.7 使用高性能算子' style='text-decoration:none;${border-style}'>1.7 使用高性能算子</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1.7.1 reduceByKey/aggregateByKey 替代 groupByKey' style='text-decoration:none;${border-style}'>1.7.1 reduceByKey/aggregateByKey 替代 groupByKey</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1.7.2 mapPartitions 替代 map' style='text-decoration:none;${border-style}'>1.7.2 mapPartitions 替代 map</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1.7.3 foreachPartitions 替代 foreach' style='text-decoration:none;${border-style}'>1.7.3 foreachPartitions 替代 foreach</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1.7.4 使用 filter 之后进行 coalesce 操作' style='text-decoration:none;${border-style}'>1.7.4 使用 filter 之后进行 coalesce 操作</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1.7.4 repartitionAndSortWithinPartitions 替代 repartition 和 sort 类操作' style='text-decoration:none;${border-style}'>1.7.4 repartitionAndSortWithinPartitions 替代 repartition 和 sort 类操作</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1.8 广播大变量' style='text-decoration:none;${border-style}'>1.8 广播大变量</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1.9 使用 Kryo 优化序列化性能' style='text-decoration:none;${border-style}'>1.9 使用 Kryo 优化序列化性能</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1.10 优化数据结构' style='text-decoration:none;${border-style}'>1.10 优化数据结构</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2 资源调优' style='text-decoration:none;${border-style}'>2 资源调优</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.1 调优概述' style='text-decoration:none;${border-style}'>2.1 调优概述</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.2 Spark 作业基本运行原理' style='text-decoration:none;${border-style}'>2.2 Spark 作业基本运行原理</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.2.1 作业运行原理' style='text-decoration:none;${border-style}'>2.2.1 作业运行原理</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.2.2 说明' style='text-decoration:none;${border-style}'>2.2.2 说明</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.3 资源参数调优' style='text-decoration:none;${border-style}'>2.3 资源参数调优</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.3.1 num-executors' style='text-decoration:none;${border-style}'>2.3.1 num-executors</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.3.2 executor-memory' style='text-decoration:none;${border-style}'>2.3.2 executor-memory</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.3.3 executor-cores' style='text-decoration:none;${border-style}'>2.3.3 executor-cores</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.3.4 driver-memory' style='text-decoration:none;${border-style}'>2.3.4 driver-memory</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.3.5 spark.default.parallelism' style='text-decoration:none;${border-style}'>2.3.5 spark.default.parallelism</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.3.6 spark.storage.memoryFraction' style='text-decoration:none;${border-style}'>2.3.6 spark.storage.memoryFraction</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.3.7 spark.shuffle.memoryFraction' style='text-decoration:none;${border-style}'>2.3.7 spark.shuffle.memoryFraction</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.4 资源参数调优参考示例' style='text-decoration:none;${border-style}'>2.4 资源参数调优参考示例</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3 并行度调优' style='text-decoration:none;${border-style}'>3 并行度调优</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3.1 并行度调优概述' style='text-decoration:none;${border-style}'>3.1 并行度调优概述</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3.2 并行度调优原理' style='text-decoration:none;${border-style}'>3.2 并行度调优原理</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3.3 并行度调优常用方案' style='text-decoration:none;${border-style}'>3.3 并行度调优常用方案</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4 数据本地化调优' style='text-decoration:none;${border-style}'>4 数据本地化调优</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4.1 本地化级别' style='text-decoration:none;${border-style}'>4.1 本地化级别</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4.2 本地化过程' style='text-decoration:none;${border-style}'>4.2 本地化过程</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5 数据倾斜调优' style='text-decoration:none;${border-style}'>5 数据倾斜调优</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5.1 调优概述' style='text-decoration:none;${border-style}'>5.1 调优概述</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5.2 数据倾斜发生时的现象' style='text-decoration:none;${border-style}'>5.2 数据倾斜发生时的现象</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5.3 数据倾斜发生的原理' style='text-decoration:none;${border-style}'>5.3 数据倾斜发生的原理</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5.4 如何定位导致数据倾斜的代码' style='text-decoration:none;${border-style}'>5.4 如何定位导致数据倾斜的代码</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5.4.1 某个 task 执行特别慢时' style='text-decoration:none;${border-style}'>5.4.1 某个 task 执行特别慢时</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5.4.2 某个 task 内存溢出' style='text-decoration:none;${border-style}'>5.4.2 某个 task 内存溢出</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5.4.3 查看导致数据倾斜的 key 的数据分布情况' style='text-decoration:none;${border-style}'>5.4.3 查看导致数据倾斜的 key 的数据分布情况</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5.5 数据倾斜的解决方案' style='text-decoration:none;${border-style}'>5.5 数据倾斜的解决方案</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5.5.1 使用 Hive ETL 预处理数据' style='text-decoration:none;${border-style}'>5.5.1 使用 Hive ETL 预处理数据</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5.5.2 过滤少数导致倾斜的 key' style='text-decoration:none;${border-style}'>5.5.2 过滤少数导致倾斜的 key</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5.5.3 提高 shuffle 操作的并行度' style='text-decoration:none;${border-style}'>5.5.3 提高 shuffle 操作的并行度</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5.5.4 两阶段聚合（局部聚合+全局聚合）' style='text-decoration:none;${border-style}'>5.5.4 两阶段聚合（局部聚合+全局聚合）</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5.5.5 将 reduce join 转为 map join' style='text-decoration:none;${border-style}'>5.5.5 将 reduce join 转为 map join</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5.5.6 采样倾斜 key 并分拆 join 操作' style='text-decoration:none;${border-style}'>5.5.6 采样倾斜 key 并分拆 join 操作</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5.5.7 使用随机前缀和扩容 RDD 进行 join' style='text-decoration:none;${border-style}'>5.5.7 使用随机前缀和扩容 RDD 进行 join</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5.5.8 多种方案组合使用' style='text-decoration:none;${border-style}'>5.5.8 多种方案组合使用</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6 Shuffle 调优' style='text-decoration:none;${border-style}'>6 Shuffle 调优</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6.1 调优概述' style='text-decoration:none;${border-style}'>6.1 调优概述</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6.2 ShuffleManager 发展概述' style='text-decoration:none;${border-style}'>6.2 ShuffleManager 发展概述</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6.3 HashShuffleManager 运行原理' style='text-decoration:none;${border-style}'>6.3 HashShuffleManager 运行原理</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6.3.1 未经优化的 HashShuffleManager' style='text-decoration:none;${border-style}'>6.3.1 未经优化的 HashShuffleManager</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6.3.2 优化后的 HashShufffleManager' style='text-decoration:none;${border-style}'>6.3.2 优化后的 HashShufffleManager</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6.4 SortShuffleManager 运行原理' style='text-decoration:none;${border-style}'>6.4 SortShuffleManager 运行原理</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6.4.1 普通运行机制' style='text-decoration:none;${border-style}'>6.4.1 普通运行机制</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6.4.2 bypass 运行机制' style='text-decoration:none;${border-style}'>6.4.2 bypass 运行机制</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6.5 shuffle 相关参数调优' style='text-decoration:none;${border-style}'>6.5 shuffle 相关参数调优</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6.5.1 spark.shuffle.file.buffer' style='text-decoration:none;${border-style}'>6.5.1 spark.shuffle.file.buffer</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6.5.2 spark.reducer.maxSizeInFlight' style='text-decoration:none;${border-style}'>6.5.2 spark.reducer.maxSizeInFlight</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6.5.3 spark.shuffle.io.maxRetries' style='text-decoration:none;${border-style}'>6.5.3 spark.shuffle.io.maxRetries</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6.5.4 spark.shuffle.io.retryWait' style='text-decoration:none;${border-style}'>6.5.4 spark.shuffle.io.retryWait</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6.5.5 spark.shuffle.memoryFraction' style='text-decoration:none;${border-style}'>6.5.5 spark.shuffle.memoryFraction</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6.5.6 spark.shuffle.manager' style='text-decoration:none;${border-style}'>6.5.6 spark.shuffle.manager</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6.5.7 spark.shuffle.sort.bypassMergeThreshold' style='text-decoration:none;${border-style}'>6.5.7 spark.shuffle.sort.bypassMergeThreshold</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6.5.8 spark.shuffle.consolidateFiles' style='text-decoration:none;${border-style}'>6.5.8 spark.shuffle.consolidateFiles</a><br/>
<a href='#总结' style='text-decoration:none;font-weight:bolder'>总结</a><br/>
<a href='#参考文献' style='text-decoration:none;font-weight:bolder'>参考文献</a><br/>
</nav>


# 正文

## 1 Spark 开发优化

### 1.1 调优概述

Spark 性能优化的第一步，是需要在开发 Spark 作业的过程中注意和应用一些性能优化的基本原则。开发调优，是要让大家了解 Spark 基本开发原则，包括：RDD lineage 设计、算子的合理使用、特殊操作的优化等。在开发过程中，需要注意以上原则，并根据具体的业务及实际使用场景，将其灵活地运用到 Spark。

###  1.2 避免创建重复 RDD

对于同一份数据，只应该创建一个 RDD，不能创建多个 RDD 来代表同一份数据。

对于 Spark 初学者刚开始开发 Spark 作业或有经验的工程师开发 RDD lineage 极其冗长时，忘记之前对于某一份数据已经创建过一个 RDD。如果创建了重复地 RDD，那么 Spark 作业会及逆行多次重复计算来创建代表相同数据的 RDD，进而增加了作业的性能开销。



### 1.3 尽可能复用同一 RDD

在对不同的数据执行算子操作时还要尽可能地复用一个 RDD 。比如说，有一个 RDD 的数据格式是 key-value 类型的，另一个是单 value 类型的，这两个 RDD 的 value 数据是完全一样的。那么此时我们可以只使用 key-value 类型的那个 RDD，因为其中已经包含了另一个的数据。对于类似这种多个RDD的数据有**重叠**或者**包含**的情况，我们应该尽量复用一个 RDD，这样可以尽可能地减少 RDD 的数量，从而尽可能减少算子执行的次数。

```scala
// 错误的做法。

// 有一个<Long, String>格式的RDD，即rdd1。
// 接着由于业务需要，对rdd1执行了一个map操作，创建了一个rdd2，而rdd2中的数据仅仅是rdd1中的value值而已，也就是说，rdd2是rdd1的子集。
JavaPairRDD<Long, String> rdd1 = ...
JavaRDD<String> rdd2 = rdd1.map(...)

// 分别对rdd1和rdd2执行了不同的算子操作。
rdd1.reduceByKey(...)
rdd2.map(...)


// 正确的做法。

// 上面这个case中，其实rdd1和rdd2的区别无非就是数据格式不同而已，rdd2的数据完全就是rdd1的子集而已，却创建了两个rdd，并对两个rdd都执行了一次算子操作。
// 此时会因为对rdd1执行map算子来创建rdd2，而多执行一次算子操作，进而增加性能开销。

// 其实在这种情况下完全可以复用同一个RDD。
// 我们可以使用rdd1，既做reduceByKey操作，也做map操作。
// 在进行第二个map操作时，只使用每个数据的tuple._2，也就是rdd1中的value值，即可。
JavaPairRDD<Long, String> rdd1 = ...
rdd1.reduceByKey(...)
rdd1.map(tuple._2...)

// 第二种方式相较于第一种方式而言，很明显减少了一次rdd2的计算开销。
// 但是到这里为止，优化还没有结束，对rdd1我们还是执行了两次算子操作，rdd1实际上还是会被计算两次。
// 因此还需要配合“原则三：对多次使用的RDD进行持久化”进行使用，才能保证一个RDD被多次使用时只被计算一次。
```



### 1.4 对多次使用的 RDD 进行持久化

在完成对 RDD 复用的基础上，可以进行进一步优化，保证对一个 RDD 执行多次算子操作，该 RDD 只被计算一次。

Spark 中对于一个 RDD 执行算子操作的默认原理是这样的：每次对一个 RDD 执行一个算子操作，都会重新从源头计算一遍，计算出 RDD 来，然后再对该 RDD 执行算子操作。该种方式的性能很差。

因此对于这种情况，建议是：对多次使用的 RDD 进行持久化。此时 Spark 会根据你的持久化策略，将RDD 中的数据保存在内存或磁盘中。以后每次对这个 RDD 进行算子操作时，都直接从内存或磁盘上获取，再执行算子，而不需要从源头重新计算一边该 RDD，再执行算子操作。

```scala
// 如果要对一个RDD进行持久化，只要对这个RDD调用cache()和persist()即可。

// 正确的做法。
// cache()方法表示：使用非序列化的方式将RDD中的数据全部尝试持久化到内存中。
// 此时再对rdd1执行两次算子操作时，只有在第一次执行map算子时，才会将这个rdd1从源头处计算一次。
// 第二次执行reduce算子时，就会直接从内存中提取数据进行计算，不会重复计算一个rdd。
val rdd1 = sc.textFile("hdfs://192.168.0.1:9000/hello.txt").cache()
rdd1.map(...)
rdd1.reduce(...)

// persist()方法表示：手动选择持久化级别，并使用指定的方式进行持久化。
// 比如说，StorageLevel.MEMORY_AND_DISK_SER表示，内存充足时优先持久化到内存中，内存不充足时持久化到磁盘文件中。
// 而且其中的_SER后缀表示，使用序列化的方式来保存RDD数据，此时RDD中的每个partition都会序列化成一个大的字节数组，然后再持久化到内存或磁盘中。
// 序列化的方式可以减少持久化的数据对内存/磁盘的占用量，进而避免内存被持久化数据占用过多，从而发生频繁GC。
val rdd1 = sc.textFile("hdfs://192.168.0.1:9000/hello.txt").persist(StorageLevel.MEMORY_AND_DISK_SER)
rdd1.map(...)
rdd1.reduce(...)
```

关于 RDD 持久化级别和如何选择合适的持久化策略参考 Spark Core 文档中 RDD 部分。



### 1.5 尽量避免使用 Shuffle 类算子

可能的话，尽量避免使用 shuffle 类算子。因为在 Spark 作业中，shuffle 操作涉及磁盘 I/O，序列化和反序列化，网络 I/O，该过程最消耗性能。

shuffle 过程会将集群中多个节点相同的 key 拉去到同一节点上，进行聚合操作或 Join 操作。常见的**聚合操作**和 **Join** 操作可以参考 [Spark Core 文档的 RDD 部分](./SparkCore.md)。

shuffle 过程：

* 首先各个节点会将相同的 key 先写入本地磁盘文件中；

* 然后其他节点通过网络传输拉取各个节点上磁盘文件中相同的 key，在此过程中，相同的 key 被拉取到同一节点进行操作，可能某一节点处理的 key 过多，导致内存容纳不下，从而溢写到磁盘上。

从 shuffle 的过程可看出，当中存在大量的磁盘文件读写 I/O，网络 I/O，数据序列化及反序列化过程。这也即使 shuffle 性能差的原因。

如果我们在开发过程中，避免 shuffle 类操作（reduceByKey，join，distinct，repartition），使用 map 类的非 shuffle 操作替代，可以大大减少性能开销。

**broadcast与map进行join代码示例：**

```scala
// 传统的 join 操作会导致 shuffle 操作。
// 因为两个 RDD 中，相同的 key 都需要通过网络拉取到一个节点上，由一个 task 进行 join 操作。
val rdd3 = rdd1.join(rdd2)

// broadcast+map 的 join 操作，不会导致 shuffle 操作。
// 使用 broadcast 将一个数据量较小的 RDD 作为广播变量。
val rdd2Data = rdd2.collect()
val rdd2DataBroadcast = sc.broadcast(rdd2Data)

// 在 rdd1.map 算子中，可以从 rdd2DataBroadcast 中，获取 rdd2 的所有数据。
// 然后进行遍历，如果发现 rdd2 中某条数据的 key 与 rdd1 的当前数据的 key 是相同的，那么就判定可以进行 join。
// 此时就可以根据自己需要的方式，将 rdd1 当前数据与 rdd2 中可以连接的数据，拼接在一起（String 或 Tuple）。
val rdd3 = rdd1.map(rdd2DataBroadcast...)

// 注意，以上操作，建议仅仅在 rdd2 的数据量比较少（比如几百M，或者一两G）的情况下使用。
// 因为每个 Executor 的内存中，都会驻留一份 rdd2 的全量数据。
```



### 1.6 使用 map-side 预聚合的 shuffle 操作

如果因为业务需要，**一定要使用shuffle操作，无法用 map 类的算子来替代**，那么尽量使用可以 **map-side 预聚合的算子**。

**map-side 预聚合**是在每个节点本地对相同的 key 进行一次聚合操作，类似于 MapReduce 中的本地 combiner。map-side 预聚合之后，每个节点本地就只会有一条相同的 key，因为多条相同的 key 被聚合起来了。其他节点在拉去所有节点上的相同 key 时，会大大减少需要拉取的数据数量，从而也就减少了磁盘 I/O 及网络 I/O 的开销。

**建议：**使用 reduceByKey 或 aggregateByKey 算子来替代 groupByKey 算子。因为 reduceByKey 和 aggregateByKey 算子都会在使用用户自定义的函数对每个节点本地的相同 key 进行预聚合。而 groupByKey 算子是不会进行预聚合的，全量数据会在集群的各个节点之间分发，传输，性能相对较差。

下图分别是基于 groupByKey 和 reduceByKey  单词统计的原理图。

![rdd-groupByKey](https://gitee.com/struggle3014/picBed/raw/master/rdd-groupByKey.png)

![rdd-reduceByKey](C:\Users\yue_zhou\Desktop\images\rdd-reduceByKey.png)



### 1.7 使用高性能算子

除了shuffle相关的算子有优化原则之外，其他的算子也都有着相应的优化原则。

#### 1.7.1 reduceByKey/aggregateByKey 替代 groupByKey



#### 1.7.2 mapPartitions 替代 map

mapPartitions 类的算子，一次函数调用会处理一个 partition 所有的数据，而不是一次函数调用处理一条，性能相对来说会高一些。但是有的时候，使用 mapPartitions 会出现 OOM（内存溢出）的问题。因为单次函数调用就要处理掉一个 partition 所有的数据，如果内存不够，垃圾回收时是无法回收掉太多对象的，很可能出现 OOM 异常。所以使用这类操作时要慎重！



#### 1.7.3 foreachPartitions 替代 foreach

原理和 mapPartitions 替代 map 类似。foreachPartitions 一次函数调用处理一个 partition 中所有数据。

在实践中，如果有很重的操作，如数据库连接，可以使用 foreachPartitions 算子一次性处理一个 partition 中的数据。



#### 1.7.4 使用 filter 之后进行 coalesce 操作

通常对一个 RDD 执行 filter 算子过滤掉 RDD 中较多数据后（比如 30% 以上的数据），建议使用 coalesce 算子，手动减少 RDD 的 partition 数量，将 RDD 中的数据压缩到更少的 partition 中去。因为 filter 之后，RDD 的每个 partition 中都会有很多数据被过滤掉，此时如果照常进行后续的计算，其实每个 task 处理的 partition 中的数据量并不是很多，造成资源浪费，而且此时处理的 task 越多，可能速度反而越慢。因此**用 coalesce 减少 partition 数量**，将RDD中的数据压缩到更少的 partition 之后，只要使用更少的 task 即可处理完所有的 partition。在某些场景下，对于性能的提升会有一定的帮助。



#### 1.7.4 repartitionAndSortWithinPartitions 替代 repartition 和 sort 类操作

如果需要在 repartition 重分区之后，还要进行排序，建议直接使用 repartitionAndSortWithinPartitions 算子。因为该算子可以一边进行重分区的 shuffle 操作，一边进行排序。shuffle 与 sort 两个操作同时进行，比先  shuffle 再 sort 来说，性能可能是要高的。



### 1.8 广播大变量

有时在开发过程中，会遇到需要在算子函数中使用外部变量的场景（尤其是大变量，比如100M以上的大集合），那么此时就应该使用 Spark 的广播（broadcast）功能来提升性能。

在算子函数中使用到外部变量时，默认情况下，Spark 会将该变量复制多个副本，通过网络传输到 task 中，此时每个 task 都有一个变量副本。如果变量本身比较大的话（比如100M，甚至1G），那么大量的变量副本在网络中传输的性能开销，以及在各个节点的 Executor 中占用过多内存导致的频繁 GC ，都会极大地影响性能。

因此对于上述情况，如果使用的外部变量比较大，建议使用 Spark 的广播功能，对该变量进行广播。广播后的变量，会保证每个 Executor 的内存中，只驻留一份变量副本，而 Executor 中的 task 执行时共享该 Executor 中的那份变量副本。这样的话，可以大大减少变量副本的数量，从而减少网络传输的性能开销，并减少对 Executor 内存的占用开销，降低 GC 的频率。

**广播大变量示例代码：**

```scala
// 以下代码在算子函数中，使用了外部变量
// 此时如果没有任何操作，那么每个 task 都会有一份 list1 的副本
val list1 = ...
rdd1.map(list1...)

// 接着，我们将 list1 封装成 broadcast 类型的广播变量。
// 在算子函数中，使用广播变量，首先会判断当前 task 所在 Executor 内存中，是否有变量副本。若有，则直接使用，否则从 Driver 或其他 Executor 节点上远程拉取一份到本地 Executor 内存中。
// 每个 Executor 内存中，只有一份广播变量副本。
val list1 = ...
val list1Broadcast = sc.broadcast(list1)
rdd1.map(list1Broadcat...)
```



### 1.9 使用 Kryo 优化序列化性能

在 Spark 中，主要有三个地方涉及序列化：

* 在算子函数中使用到外部变量时，该变量会被序列化进行网络传输。

* 将自定义的类型作为 RDD 的泛型类型时，所有自定义类型对象，都会进行序列化。

* 使用可序列化的持久化策略时候（如 MEMORY_ONLY_SER），Spark 会将 RDD 中的每个 partition 序列化成一个大的字节数组。

对于上述出现序列化的地方，我们可以使用 Kryo 序列化库，来优化序列化和反序列化。Spark 默认使用的是 Java 的序列化机制（ObjectOutputStream/ObjectInputStream API）。Kryo序列化类库的性能比 Java 序列化类库的性能要高很多。

> The only reason Kryo is not the default is because of the custom registration requirement, but we recommend trying it in any network-intensive application. Since Spark 2.0.0, we internally use Kryo serializer when shuffling RDDs with simple types, arrays of simple types, or string type.

**Kryo 使用示例：**

```scala
// 创建 SparkConf 对象
val conf = new SparkConf().setMaster(...).setAppName(...)
// 设置序列化器为 KryoSerializer
conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
// 注册要序列化的自定义类型
conf.registerKryoClasses(Array(classOf[MyClass1], classOf[MyClass2]))
```



### 1.10 优化数据结构

Java中，有三种类型比较耗费内存：

* 对象，每个Java对象都有对象头、引用等额外的信息，因此比较占用内存空间。
* 字符串，每个字符串内部都有一个字符数组以及长度等额外信息。
* 集合类型，比如HashMap、LinkedList等，因为集合类型内部通常会使用一些内部类来封装集合元素，比如Map.Entry。

Spark 官网调优建议：在算子函数的代码中，不要使用上述三种数据结构。尽量使用字符串代替对象，使用原始类型（如 Int，Long）代替字符串，使用数组代替集合。这样可以减少内存的使用，降低 GC 频率，提升性能。

在实践中，优化数据结构并不容易。因为需要在代码可维护性和效率做权衡。



## 2 资源调优

### 2.1 调优概述

在开发完 Spark 作业后，需要为作业配置合理的资源。Spark 资源参数，可以通过 spark-submit 命令中作为参数进行提交。在进行资源调优之前，我们需要了解 Spark 作业运行的原理以及在作业过程中，哪些参数是可以设置的。



### 2.2 Spark 作业基本运行原理

![spark-job-principle](https://gitee.com/struggle3014/picBed/raw/master/spark-job-principle.png)

#### 2.2.1 作业运行原理

1. **启动 driver 进程。**使用 spark-submit 提交 Spark 作业后，该作业会启动对应的 driver 进程（根据部署模式 deploy-mode 的不同， driver 可能在本地，或在集群某个节点）。driver 会根据设定的参数占用一定数量的内存和 CPU core。
2.  **driver 进程向资源管理器申请资源。**driver 进程会向集群资源管理器申请 Spark 作业所需要的的资源（即 executor 进程）。
3. **资源管理器为 executor 分配资源。**YARN 资源管理器会根据 Spark 作业设置的资源参数，在各工作节点上，启动一定数量的 executor 进程，每个 executor 进程占有一定数量的内存和 CPU core。
4. **driver 为 executor 分配 task。**申请到作业执行所需资源后，driver 进程会调度和执行我们编写的作业代码。driver 进程会将编写的 Spark 作业拆分成多个 stage，每个 stage 执行一部分代码片段，并为每个 stage 创建一批 task，然后将这些 task 分配到各 executor 进程中执行。task 是最小的计算单元，负责执行一模一样的计算逻辑（用户编写的代码片段），只是每个 task 处理的数据不同。一个 stage 的所有 task 都执行完毕之后，会在各个本地的磁盘文件中写入中间结果，之后 driver 就会调度运行下一个 stage。而下一个 stage 的输入数据就是上一个 stage 的输出的中间结果。如此循环，直到完成用户编写的代码逻辑全部执行完，并计算完所有数据。得到结果。

#### 2.2.2 说明

* **shuffle 过程**。Spark 根据 shuffle 类算子进行 stage 划分。如果我们的代码中执行了某个 shuffle 类算子（详见 [Spark Core 文档 shuffle 部分]()），那么在该算子处，划分出一个 stage 界限。可以大致理解为：shuffle 算子执行之前的代码被划分到一个 stage，shuffle 算子执行及之后的代码被划分到下一个 stage。由于一个 stage 刚开始执行的时候，它的每个 task 都可能从上一个 stage 的 task 所在的节点，通过网络传输拉取需处理的所有 key，然后对拉取到的相同的 key 使用编写的算子函数执行聚合操作或连接操作。

* **持久化**。当代码中执行 cache/persist 等持久化操作时，会根据持久化级别，将每个 task 计算出的数据保存到 executor 进程的内存或所在节点的磁盘文件上。

* **executor 内存详解。**<br>executor 内存主要分为三块：

  * 第一块是 task 执行用户编写的代码时使用，默认占 executor 总内存的 20%。
  * 第二块是 task 通过 shuffle 过程拉取上一个 stage 的 task 的输出，进行聚合或连接操作时使用，默认占 executor 总内存的 20%。
  * 第三块是给 RDD 持久化使用，默认占 executor 总内存的 60%。
* **task 执行速度与 CPU core 的关系**。task 的执行速度与 executor 进程的 CPU core 数量直接相关。一个 CPU core 同一时间只能运行一个线程。而每个 executor 进程上分配到多个 task，那么以每个 task 一个线程的方式。如果 CPU core 数量比较充足而且分配到的 task 数量比较合理，通常，可以比较快速和高效地执行完 task 线程。



### 2.3 资源参数调优

在了解完 Spark 作业运行的基本原理，我们对资源相关的参数就比较容易理解。所谓 Spark 资源参数调优，就是主要对 Spark 运行过程中各个使用资源的地方，通过调节各种参数，来优化资源的使用效率，从而提升 Spark 作业的执行性能。以下是 Spark 中主要的资源参数，每个参数对应着作业运行原理中的某个部分，同时也给出了调优的参考值。

#### 2.3.1 num-executors

- *<font color="red">参数说明：</font>该参数用于设置  Spark 作业总共要用多少个 executor 进程来执行。driver 在向 YARN 集群管理器申请资源时，YARN集群管理器会尽可能按照你的设置来在集群的各个工作节点上，启动相应数量的  executor进程。这个参数非常之重要，如果不设置的话，默认只会给你启动少量的 executor 进程，此时你的 Spark 作业的运行速度是非常慢的。*
- *<font color="red">参数调优建议：</font>每个 Spark 作业的运行一般设置 50~100 个左右的 executor 进程比较合适，设置太少或太多的  executor 进程都不好。设置的太少，无法充分利用集群资源；设置的太多的话，大部分队列可能无法给予充分的资源。*

#### 2.3.2 executor-memory

- *<font color="red">参数说明：</font>该参数用于设置每个Executor进程的内存。Executor内存的大小，很多时候直接决定了Spark作业的性能，而且跟常见的JVM OOM异常，也有直接的关联。*
- *<font color="red">参数调优建议：</font>每个Executor进程的内存设置 4G~8G 较为合适。但是这只是一个参考值，具体的设置还是得根据不同部门的资源队列来定。可以看看自己团队的资源队列的最大内存限制是多少，num-executors乘以executor-memory，是不能超过队列的最大内存量的。此外，如果你是跟团队里其他人共享这个资源队列，那么申请的内存量最好不要超过资源队列最大总内存的1/3~1/2，避免你自己的 Spark 作业占用了队列所有的资源，导致别的同学的作业无法运行。*

#### 2.3.3 executor-cores

- *<font color="red">参数说明：</font>该参数用于设置每个 executor进程的 CPU core数量。这个参数决定了每个Executor进程并行执行 task 线程的能力。因为每个 CPU core 同一时间只能执行一个 task 线程，因此每个 executor进程的 CPU core 数量越多，越能够快速地执行完分配给自己的所有 task 线程。*
- *<font color="red">参数调优建议：</font>  executor 的  CPU core 数量设置为2~4个较为合适。同样得根据不同部门的资源队列来定，可以看看自己的资源队列的最大 CPU core 限制是多少，再依据设置的 executor数量，来决定每个  executor 进程可以分配到几个 CPU core。同样建议，如果是跟他人共享这个队列，那么 num-executors \* executor-cores 不要超过队列总 CPU core 的1/3~1/2左右比较合适，也是避免影响其他同学的作业运行。*

#### 2.3.4 driver-memory

- *<font color="red">参数说明：</font>该参数用于设置 driver进程的内存。*
- *<font color="red">参数调优建议：</font> driver的内存通常来说不设置，或者设置 1G 左右应该就够了。唯一需要注意的一点是，如果需要使用 collect 算子将 RDD 的数据全部拉取到  driver上进行处理，那么必须确保 driver 的内存足够大，否则会出现 OOM 内存溢出的问题。*

#### 2.3.5 spark.default.parallelism

- *<font color="red">参数说明：</font>该参数用于设置每个 stage 的默认 task 数量。这个参数极为重要，如果不设置可能会直接影响你的 Spark 作业性能。*
- *<font color="red">参数调优建议：</font> Spark 作业的默认 task 数量为 500~1000 个较为合适。很多同学常犯的一个错误就是不去设置这个参数，那么此时就会导致 Spark 自己根据底层 HDFS 的  block 数量来设置task的数量，默认是一个 HDFS block 对应一个 task。通常来说，Spark 默认设置的数量是偏少的（比如就几十个 task），如果 task 数量偏少的话，就会导致你前面设置好的 executor 的参数都前功尽弃。试想一下，无论你的 executor 进程有多少个，内存和 CPU 有多大，但是 task 只有 1 个或者  10个，那么 90% 的 executor 进程可能根本就没有 task 执行，也就是白白浪费了资源！因此 Spark 官网建议的设置原则是，设置该参数为 num-executors \* executor-cores的2~3倍较为合适，比如 executor 的总 CPU core 数量为 300 个，那么设置 1000 个 task 是可以的，此时可以充分地利用 Spark 集群的资源。*

#### 2.3.6 spark.storage.memoryFraction

- *<font color="red">参数说明：</font>该参数用于设置 RDD 持久化数据在 executor 内存中能占的比例，默认是 0.6。也就是说，默认 executor 60%的内存，可以用来保存持久化的 RDD 数据。根据你选择的不同的持久化策略，如果内存不够时，可能数据就不会持久化，或者数据会写入磁盘。*
- *<font color="red">参数调优建议：</font>如果 Spark 作业中，有较多的 RDD 持久化操作，该参数的值可以适当提高一些，保证持久化的数据能够容纳在内存中。避免内存不够缓存所有的数据，导致数据只能写入磁盘中，降低了性能。但是如果Spark作业中的 shuffle 类操作比较多，而持久化操作比较少，那么这个参数的值适当降低一些比较合适。此外，如果发现作业由于频繁的 gc 导致运行缓慢（通过 spark web ui 可以观察到作业的 gc 耗时），意味着 task 执行用户代码的内存不够用，那么同样建议调低这个参数的值。*

#### 2.3.7 spark.shuffle.memoryFraction

- *<font color="red">参数说明：</font>该参数用于设置 shuffle 过程中一个task拉取到上个 stage 的 task 的输出后，进行聚合操作时能够使用的 executor 内存的比例，默认是 0.2。也就是说， executor 默认只有 20% 的内存用来进行该操作。shuffle操作在进行聚合时，如果发现使用的内存超出了这个 20% 的限制，那么多余的数据就会溢写到磁盘文件中去，此时就会极大地降低性能。*
- *<font color="red">参数调优建议：</font>如果 Spark 作业中的 RDD 持久化操作较少，shuffle 操作较多时，建议降低持久化操作的内存占比，提高 shuffle 操作的内存占比比例，避免 shuffle 过程中数据过多时内存不够用，必须溢写到磁盘上，降低了性能。此外，如果发现作业由于频繁的 gc 导致运行缓慢，意味着 task 执行用户代码的内存不够用，那么同样建议调低这个参数的值。*



### 2.4 资源参数调优参考示例

资源参数的调优，没有固定的值，需要根据实际情况（包括 Spark 作业 shuffle 操作数量，RDD 持久化数量以及 spark web ui 中显示的作业 gc 情况），根据文章中给出的原理及调优建议，合理地设置上述参数。

下面是 spark-submit 的示例：

```shel
./bin/spark-submit \
  --master yarn-cluster \
  --num-executors 100 \
  --executor-memory 6G \
  --executor-cores 4 \
  --driver-memory 1G \
  --conf spark.default.parallelism=1000 \
  --conf spark.storage.memoryFraction=0.5 \
  --conf spark.shuffle.memoryFraction=0.3 \
```



## 3 并行度调优

Spark作业资源配置合理的情况下，运行也很长，可能是由于不合理的并行度，导致不能充分利用资源，从而导致作业执行效率低。通常需要对并行度进行调优。

### 3.1 并行度调优概述

> 除非为每个算子操作设置足够高的并行度（parallelism）级别，否则集群将无法得到充分利用。Spark 会根据文件的大小（sizes）自动设置在每个文件上运行的 “map” 任务数量（尽管可以通过对 SparkContext.textFile 等进行可选的参数进行控制），对于分布式的 "reduce" 任务，例如 groupByKey 和 reduceByKey 等，它使用最大父 RDD 的分区数量作为其并行度。可以通过**传递并行度**作为第二个参数或**配置 spark.default.parallelism** 来**更改默认值**。一般，建议集群中每个 CPU 核分配 2-3 个任务。



### 3.2 并行度调优原理

官网中为什么建议将集群中的每个 CPU Core 分配 2-3 个 tasks？

**充分利用集群资源，提高任务执行效率**。因为集群中核的性能有高有低，如果每个 Core 分配多个 tasks（每个 task 是以线程的形式跑在 Executor 进程中），那么在相同的时间内高性能的 Core 可能已处理完分配给它的 tasks，而低性能的 Core 还未处理完分配给它的 tasks，此时可以由操作系统将未处理完的 tasks 调度给高性能 Core，充分发挥高性能核的特性，从而减少 stage 整体运行时间，进一步减少作业运行时间。



### 3.3 并行度调优常用方案

（1）**改变 HDFS 上文件的 Block 块数（不常用）**

HDFS 默认情况下 split 和 block 块一一对应，而 split 与 RDD 中的 partition 对应，因此增加 block 块，可以提高并行度。

（2）**创建 RDD 时指定分区数（map 任务阶段）**

sc.textFile(path: String, minPartitions: Int = defaultMinPartitions)

sc.parallelism(seq: Seq[T], numSlices: Int = defaultParallelism)

（3）**使用 Shuffle 算子指定分区数（reduce 任务阶段）**

reduceByKey(func, numPartitions: Int)

join(other: RDD[(K, W)], numPartitions: Int)

（4）**使用 coalesce 或 repartition 算子进行重分区**

repartition(numPartitions) <=> coalesce(numPartitions, shuffle=true)

（5）**参数设定**

spark.default.parallelism：Spark默认并行度

spark.sql.shuffle.partitions：Spark SQL shuffle 过程中默认的 partition 数



## 4 数据本地化调优

数据本地化对 Spark 作业的性能产生重大影响。如果数据和对其进行操作的代码在一起，那么计算速度很快。如果数据和代码分开，其中一个必须向另一个移动。通常，将序列化的代码从一处运送到另一处要比将数据块运送地更快，因为代码的大小比数据的大小要小得多。Spark 围绕数据本地化原则来构建调度。

### 4.1 本地化级别

**数据本地化是指数据与处理数据的代码（计算逻辑）之间的距离**。基于数据的当前位置，本地化级别从最近到最远：

* **PROCESS_LOCAL** 数据和计算逻辑位于同一个 JVM 中。

  ![PROCESS_LOCAL](https://gitee.com/struggle3014/picBed/raw/master/PROCESS_LOCAL.png)

* **NODE_LOCAL** 数据和计算逻辑在同一个节点上。数据可能位于同一个节点的 HDFS 中，或在同一个节点的其他 Executor 进程中。比 PROCESS_LOCAL 稍微慢些，因为数据需要在进程间传输。

  ![NODE_LOCAL](https://gitee.com/struggle3014/picBed/raw/master/NODE_LOCAL.png)

* **NO_PREF** 从任何地方访问数据都同等快，没有本地化偏好。

  ![NO_PREF](https://gitee.com/struggle3014/picBed/raw/master/NO_PREF.png)

* **RACK_LOCAL** 数据和计算逻辑在同一个服务器机架上。数据和计算逻辑位于同一机架上的不同服务器，需要通过交换机在网络上传输。

  ![RACK_LOCAL](https://gitee.com/struggle3014/picBed/raw/master/RACK_LOCAL.png)

* **ANY** 数据在网络上的其他位置，和计算逻辑不在同一机架上。



### 4.2 本地化过程

Spark 倾向于在最佳本地化级别调度所有任务，但这并不总是可能的。在任何空闲的 Executor 上都没有未处理的数据情况下，Spark 将切换到较低级别的本地化。有两种选择：

* 等待，直到繁忙的 CPU 空闲下来，在数据的所在服务器启动一个 task。
* 立即在需将数据移动到那里的更远的地方启动一个新任务。

Spark 通常是等待一段时间，希望繁忙的 CPU 空闲下来。一旦超时过期，它将数据从远处移动到空闲 CPU。每个本地化级别之间的回退等待超时时间可以单独配置或在单个参数中配置。如果任务耗时很长并且本地化级别差，应该增加参数所设置的等待时间，但通常默认配置能够很好地运行。

| 配置                        | 默认值              | 含义                                                         |
| --------------------------- | ------------------- | ------------------------------------------------------------ |
| spark.locality.wait         | 3s                  | **放弃启动数据本地化任务之前的等待时长**，之后启动低一级别的数据本地化节点启动该任务。相同的等待将用于**单步执行多个本地级别**（PROCESS_LOCAL, NODE_LOCAL, RACK_LOCAL 然后 ANY），也可单独设置。如果任务很长，并且本地化级别很差，则应该增加该值。 |
| spark.locality.wait.node    | spark.locality.wait | 自定义节点本地化（NODE_LOCAL）等待时长。                     |
| spark.locality.wait.process | spark.locality.wait | 自定义进程本地化（PROCESS_LOCAL）等待时长。                  |
| spark.locality.wait.rack    | spark.locality.wait | 自定义机架本地化（RACK_LOCAL）等待时长。                     |

**Spark 调度时，TaskScheduler 在分发前，会依据数据的位置进行分发，尽可能使用最佳本地化级别调度任务**。如果在某个本地化级别调度任务失败超过5次，那么选择从该本地化级别的低一级别的本地化调度任务，重复上述过程，直到成功调度为止。![Spark 数据本地化调度流程](https://gitee.com/struggle3014/picBed/raw/master/Spark 数据本地化调度流程.png)



## 5 数据倾斜调优

### 5.1 调优概述

我们在大数据计算中会遇到一个棘手的问题—数据倾斜，此时 Spark 作业的性能会比期望的差很多。



### 5.2 数据倾斜发生时的现象

数据倾斜发生的一般现象包括：

* 绝大多数 task 都执行得非常快，极个别 task 执行极慢。如，有 1000 个 task，990 个 task 在几分钟内执行完了，剩余的需要执行几个小时。**该情况比较常见。**
* 原本能正常执行的 Spark 作业，某天突然报 OOM 异常，观察异常栈，是由于业务代码造成的。**该情况比较少见。**



### 5.3 数据倾斜发生的原理

在 shuffle 时过程中，必须将各个节点上相同的 key 拉取到某个节点上的一个 task 来进行操作，比如按 key 进行聚合或 join 操作。此时，如果某个 key 对应的数据量非常大，就会发生倾斜。

例如：大部分 key 对应个位数记录数，但是个别 key 对应百万条记录。那么此时会发生这种情况，大部分 task 分配到 10 条数据，几秒钟就执行完；但是个别 task 分配到百万条记录，需要运行几个小时。因此，整个 Spark 作业运行进度是由运行时间最长的 task 所决定的。

在发生数据倾斜时，Spark 的作业运行得会很慢，甚至可能由于某个 task 处理的数据量过于庞大，导致内存溢出。

示例： hello 这个 key，在三个节点上对应 7 条记录，这些数据会被拉取到同一个 task 中处理；而 word 和 you 这两个 key 分别对应 1 条记录，所以另外两个 task 只要分别处理 1 条数据即可。此时，第一个 task 运行时间大致是另外两个 task 的 7 倍，而整个 stage 的运行速度由运行最慢的那个 task 所决定。

![spark-data-skew](https://gitee.com/struggle3014/picBed/raw/master/spark-data-skew.png)



### 5.4 如何定位导致数据倾斜的代码

数据倾斜只会发生在 shuffle 过程中。常用触发 shuffle 操作的算子：distinct， groupByKey，reduceByKey，aggregateByKey，join，cogroup，repartition 等。详见 [Spark Core 文档]()。



#### 5.4.1 某个 task 执行特别慢时

1. **确定数据倾斜发生在第几个 stage 中。**

   * 查看 log 方式

     如果是 yarn-client 模式提交，可以直接看 log，可以在 log 中找到当前运行到第几个 stage。

   * 查看 Spark Web UI 方式

     不论是 Spark 作业是以何种方式提交（yarn-client 还是 yarn-cluster），可以在 Spark Web UI 上查看当前运行到第几个 stage，而且还可以查看到当前 stage 各 task 分配的数据量，从而可以进一步确定是不是 task 分配的数据不均匀导致数据倾斜。

     **示例：**

     倒数第三列显示了每个 task 的运行时间。明显可以看到，有的 task 运行特别快，只需要几秒钟就可以运行完；而有的 task 运行特别慢，需要几分钟才能运行完，此时单从运行时间上看就已经能够确定发生数据倾斜了。此外，倒数第一列显示了每个 task 处理的数据量，明显可以看到，运行时间特别短的 task 只需要处理几百KB的数据即可，而运行时间特别长的 task 需要处理几千KB的数据，处理的数据量差了 10倍。此时更加能够确定是发生了数据倾斜。

     ![spark-job-detail](C:\Users\yue_zhou\Desktop\images\spark-job-detail.png)

2. **根据 stage 划分原理，推算发生倾斜的代码在那一部分。**

   想要精准推断 stage 与代码的对应关系，需要对 Spark 源码有深入的理解。此处介绍一个简单实用的方法：看到 Spark 代码中出现了 shuffle 类算子或 Spark SQL 的 SQL 语句出现导致 shuffle 的语句，可以判断，以此处为界，划分出了前后两个 stage。

   **示例：**

   以 wordCount 代码为例，代码如下：

   ```scala
   val conf = new SparkConf()
   val sc = new SparkContext(conf)
    
   val lines = sc.textFile("hdfs://...")
   val words = lines.flatMap(_.split(" "))
   val pairs = words.map((_, 1))
   val wordCounts = pairs.reduceByKey(_ + _)
    
   wordCounts.collect().foreach(println(_))
   ```

   在整个代码中，只有一个 reduceByKey 是会发生 shuffle 的算子，因此就可以认为，以这个算子为界限，会划分出前后两个 stage。

   * stage0，主要是执行从 textFile 到 map 操作，以及执行 shuffle write 操作。shuffle write 操作，我们可以简单理解为对 pairs RDD 中的数据进行分区操作，每个task处理的数据中，相同的 key 会写入同一个磁盘文件内。
   * stage1，主要是执行从 reduceByKey 到collect操作，stage1 的各个 task 一开始运行，就会首先执行shuffle read操作。执行 shuffle read 操作的task，会从 stage0 的各个 task 所在节点拉取属于自己处理的那些key，然后对同一个 key 进行全局性的聚合或join等操作，在这里就是对key的value值进行累加。stage1 在执行完 reduceByKey 算子之后，就计算出了最终的 wordCounts RDD，然后会执行 collect 算子，将所有数据拉取到 driver上，供我们遍历和打印输出。



#### 5.4.2 某个 task 内存溢出

这种情况下去定位出问题的代码就比较容易了。我们建议直接看 yarn-client 模式下本地log的异常栈，或者是通过 YARN 查看 yarn-cluster 模式下的 log 中的异常栈。一般来说，通过异常栈信息就可以定位到你的代码中哪一行发生了内存溢出。然后在那行代码附近找找，一般也会有 shuffle 类算子，此时很可能就是这个算子导致了数据倾斜。

但是需要注意的是，不能单纯靠偶然的内存溢出就判定发生了数据倾斜。因为自己编写的代码的 bug，以及偶然出现的数据异常，也可能会导致内存溢出。因此还是要按照上面所讲的方法，通过 Spark Web UI 查看报错的那个stage 的各个 task 的运行时间以及分配的数据量，才能确定是否是由于数据倾斜才导致了这次内存溢出。



#### 5.4.3 查看导致数据倾斜的 key 的数据分布情况

知道了数据倾斜发生在哪里之后，通常需要分析一下那个执行了 shuffle 操作并且导致了数据倾斜的 RDD/Hive表，查看一下其中 key 的分布情况。这主要是为之后选择哪一种技术方案提供依据。针对不同的 key 分布与不同的 shuffle 算子组合起来的各种情况，可能需要选择不同的技术方案来解决。

此时根据你执行操作的情况不同，可以有很多种查看key分布的方式：

1. 如果是 Spark SQL 中的 group by、join 语句导致的数据倾斜，那么就查询一下 SQL 中使用的表的 key 分布情况。 

2. 如果是对 Spark RDD 执行 shuffle 算子导致的数据倾斜，那么可以在 Spark 作业中加入查看 key 分布的代码，比如 RDD.countByKey()。然后对统计出来的各个key出现的次数，collect/take 到客户端打印一下，就可以看到 key 的分布情况。

**示例：**

```scala
val sampledPairs = pairs.sample(false, 0.1)
val sampledWordCounts = sampledPairs.countByKey()
sampledWordCounts.foreach(println(_))
```



### 5.5 数据倾斜的解决方案

#### 5.5.1 使用 Hive ETL 预处理数据

**适用场景：**导致数据倾斜的是 Hive 表。如果该Hive表中的数据本身很不均匀（比如某个 key 对应了 100 万数据，其他 key 才对应了 10 条数据），而且业务场景需要频繁使用 Spark 对 Hive 表执行某个分析操作，那么比较适合使用这种技术方案。

**实现思路：**此时可以评估一下，是否可以通过Hive来进行数据预处理（即通过 Hive ETL 预先对数据按照 key 进行聚合，或者是预先和其他表进行 join），然后在Spark作业中针对的数据源就不是原来的 Hive 表了，而是预处理后的 Hive 表。此时由于数据已经预先进行过聚合或 join 操作了，那么在 Spark 作业中也就不需要使用原先的 shuffle 类算子执行这类操作了。

**实现原理：**这种方案从根源上解决了数据倾斜，因为彻底避免了在 Spark 中执行 shuffle 类算子，那么肯定就不会有数据倾斜的问题了。但是这里也要提醒一下大家，这种方式属于治标不治本。因为毕竟数据本身就存在分布不均匀的问题，所以 Hive ETL 中进行 group by 或者 join 等 shuffle 操作时，还是会出现数据倾斜，导致 Hive ETL 的速度很慢。我们只是把数据倾斜的发生提前到了 Hive ETL 中，避免 Spark 程序发生数据倾斜而已。

**方案优点：**实现起来简单便捷，效果还非常好，完全规避掉了数据倾斜，Spark 作业的性能会大幅度提升。

**方案缺点：**治标不治本，Hive ETL 中还是会发生数据倾斜。

**实践经验：**在一些 Java 系统与 Spark 结合使用的项目中，会出现 Java 代码频繁调用 Spark 作业的场景，而且对 Spark 作业的执行性能要求很高，就比较适合使用这种方案。将数据倾斜提前到上游的 Hive ETL，每天仅执行一次，只有那一次是比较慢的，而之后每次 Java 调用 Spark 作业时，执行速度都会很快，能够提供更好的用户体验。



#### 5.5.2 过滤少数导致倾斜的 key

**适用场景：**如果发现导致倾斜的 key 就少数几个，而且对计算本身的影响并不大的话，那么很适合使用这种方案。比如 99% 的 key 就对应 10 条数据，但是只有一个 key 对应了100万数据，从而导致了数据倾斜。

**实现思路：**如果我们判断那少数几个数据量特别多的 key，对作业的执行和计算结果不是特别重要的话，那么干脆就直接过滤掉那少数几个 key 。比如，在 Spark SQL 中可以使用 where 子句过滤掉这些 key 或者在 Spark Core 中对 RDD 执行 filter 算子过滤掉这些key。如果需要每次作业执行时，动态判定哪些key的数据量最多然后再进行过滤，那么可以使用 sample 算子对 RDD 进行采样，然后计算出每个 key 的数量，取数据量最多的 key 过滤掉即可。

**实现原理：**将导致数据倾斜的key给过滤掉之后，这些 key 就不会参与计算了，自然不可能产生数据倾斜。

**方案优点：**实现简单，而且效果也很好，可以完全规避掉数据倾斜。

**方案缺点：**适用场景不多，大多数情况下，导致倾斜的key还是很多的，并不是只有少数几个。

**实践经验：**在项目中我们也采用过这种方案解决数据倾斜。有一次发现某一天 Spark 作业在运行的时候突然 OOM了，追查之后发现，是Hive表中的某一个 key 在那天数据异常，导致数据量暴增。因此就采取每次执行前先进行采样，计算出样本中数据量最大的几个 key 之后，直接在程序中将那些 key 给过滤掉。



#### 5.5.3 提高 shuffle 操作的并行度

**适用场景：**如果我们必须要对数据倾斜迎难而上，那么建议优先使用这种方案，因为这是处理数据倾斜最简单的一种方案。

**实现思路：**在对 RDD 执行 shuffle 算子时，给 shuffle 算子传入一个参数，比如 reduceByKey(1000)，该参数就设置了这个 shuffle 算子执行时 shuffle read task 的数量。对于 Spark SQL 中的 shuffle 类语句，比如 group by、join 等，需要设置一个参数，即 spark.sql.shuffle.partitions，该参数代表了 shuffle read task 的并行度，该值默认是 200，对于很多场景来说都有点过小。

**实现原理：**增加 shuffle read task 的数量，可以让原本分配给一个 task 的多个 key 分配给多个 task，从而让每个 task 处理比原来更少的数据。举例来说，如果原本有 5 个 key，每个 key 对应 10 条数据，这 5 个 key 都是分配给一个 task 的，那么这个 task 就要处理 50 条数据。而增加了 shuffle read task 以后，每个 task 就分配到一个 key，即每个 task 就处理 10 条数据，那么自然每个 task 的执行时间都会变短了。具体原理如下图所示。

![spark-shuffle-parallel](https://gitee.com/struggle3014/picBed/raw/master/spark-shuffle-parallel.png)

**方案优点：**实现起来比较简单，可以有效缓解和减轻数据倾斜的影响。

**方案缺点：**只是缓解了数据倾斜而已，没有彻底根除问题，根据实践经验来看，其效果有限。

**实践经验：**该方案通常无法彻底解决数据倾斜，因为如果出现一些极端情况，比如某个 key 对应的数据量有 100万，那么无论你的 task 数量增加到多少，这个对应着 100 万数据的 key 肯定还是会分配到一个 task 中去处理，因此注定还是会发生数据倾斜的。所以这种方案只能说是在发现数据倾斜时尝试使用的第一种手段，尝试去用最简单的方法缓解数据倾斜而已，或者是和其他方案结合起来使用。



#### 5.5.4 两阶段聚合（局部聚合+全局聚合）

**适用场景：**对 RDD 执行 reduceByKey 等聚合类 shuffle 算子或者在 Spark SQL 中使用 group by 语句进行分组聚合时，比较适用这种方案。

**实现思路：**这个方案的核心实现思路就是进行两阶段聚合。第一次是局部聚合，先给每个 key 都打上一个随机数，比如 10 以内的随机数，此时原先一样的 key 就变成不一样的了，比如 (hello, 1) (hello, 1) (hello, 1) (hello, 1)，就会变成 (1_hello, 1) (1_hello, 1) (2_hello, 1) (2_hello, 1)。接着对打上随机数后的数据，执行 reduceByKey 等聚合操作，进行局部聚合，那么局部聚合结果，就会变成了 (1_hello, 2) (2_hello, 2)。然后将各个key的前缀给去掉，就会变成 (hello,2)(hello,2)，再次进行全局聚合操作，就可以得到最终结果了，比如 (hello, 4)。

**实现原理：**将原本相同的 key 通过附加随机前缀的方式，变成多个不同的 key，就可以让原本被一个 task 处理的数据分散到多个 task 上去做局部聚合，进而解决单个 task 处理数据量过多的问题。接着去除掉随机前缀，再次进行全局聚合，就可以得到最终的结果。具体原理见下图。

![spark-tow-stage-join](https://gitee.com/struggle3014/picBed/raw/master/spark-tow-stage-join.png)

**方案优点：**对于聚合类的 shuffle 操作导致的数据倾斜，效果是非常不错的。通常都可以解决掉数据倾斜，或者至少是大幅度缓解数据倾斜，将 Spark 作业的性能提升数倍以上。

**方案缺点：**仅仅适用于聚合类的 shuffle 操作，适用范围相对较窄。如果是 join 类的 shuffle 操作，还得用其他的解决方案。

**示例：**

```scala
// 第一步，给RDD中的每个key都打上一个随机前缀。
JavaPairRDD<String, Long> randomPrefixRdd = rdd.mapToPair(
        new PairFunction<Tuple2<Long,Long>, String, Long>() {
            private static final long serialVersionUID = 1L;
            @Override
            public Tuple2<String, Long> call(Tuple2<Long, Long> tuple)
                    throws Exception {
                Random random = new Random();
                int prefix = random.nextInt(10);
                return new Tuple2<String, Long>(prefix + "_" + tuple._1, tuple._2);
            }
        });
  
// 第二步，对打上随机前缀的key进行局部聚合。
JavaPairRDD<String, Long> localAggrRdd = randomPrefixRdd.reduceByKey(
        new Function2<Long, Long, Long>() {
            private static final long serialVersionUID = 1L;
            @Override
            public Long call(Long v1, Long v2) throws Exception {
                return v1 + v2;
            }
        });
  
// 第三步，去除RDD中每个key的随机前缀。
JavaPairRDD<Long, Long> removedRandomPrefixRdd = localAggrRdd.mapToPair(
        new PairFunction<Tuple2<String,Long>, Long, Long>() {
            private static final long serialVersionUID = 1L;
            @Override
            public Tuple2<Long, Long> call(Tuple2<String, Long> tuple)
                    throws Exception {
                long originalKey = Long.valueOf(tuple._1.split("_")[1]);
                return new Tuple2<Long, Long>(originalKey, tuple._2);
            }
        });
  
// 第四步，对去除了随机前缀的RDD进行全局聚合。
JavaPairRDD<Long, Long> globalAggrRdd = removedRandomPrefixRdd.reduceByKey(
        new Function2<Long, Long, Long>() {
            private static final long serialVersionUID = 1L;
            @Override
            public Long call(Long v1, Long v2) throws Exception {
                return v1 + v2;
            }
        });
```



#### 5.5.5 将 reduce join 转为 map join

**适用场景：**在对 RDD 使用 join 类操作，或者是在 Spark SQL 中使用 join 语句时，而且 join 操作中的一个 RDD 或表的数据量比较小（比如几百M或者一两G），比较适用此方案。

**实现思路：**不使用 join 算子进行连接操作，而使用 broadcast 变量与 map 类算子实现 join 操作，进而完全规避掉 shuffle 类的操作，彻底避免数据倾斜的发生和出现。将较小 RDD 中的数据直接通过 collect 算子拉取到 driver端的内存中来，然后对其创建一个 broadcast 变量；接着对另外一个 RDD 执行 map 类算子，在算子函数内，从 broadcast 变量中获取较小 RDD 的全量数据，与当前 RDD 的每一条数据按照连接 key 进行比对，如果连接 key 相同的话，那么就将两个 RDD 的数据用你需要的方式连接起来。

**实现原理：**普通的 join 是会走 shuffle 过程的，而一旦 shuffle，就相当于会将相同 key 的数据拉取到一个 shuffle read task 中再进行 join，此时就是 reduce join。但是如果一个 RDD 是比较小的，则可以采用广播小 RDD 全量数据 + map 算子来实现与 join 同样的效果，也就是 map join，此时就不会发生 shuffle 操作，也就不会发生数据倾斜。具体原理如下图所示。

![spark-reducejoin2mapjoin](https://gitee.com/struggle3014/picBed/raw/master/spark-reducejoin2mapjoin.png)

**方案优点：**对 join 操作导致的数据倾斜，效果非常好，因为根本就不会发生 shuffle，也就根本不会发生数据倾斜。

**方案缺点：**适用场景较少，因为这个方案只适用于一个大表和一个小表的情况。毕竟我们需要将小表进行广播，此时会比较消耗内存资源，driver 和每个 executor 内存中都会驻留一份小 RDD 的全量数据。如果我们广播出去的 RDD 数据比较大，比如 10G 以上，那么就可能发生内存溢出了。因此并不适合两个都是大表的情况。

**示例：**

```scala
// 首先将数据量比较小的RDD的数据，collect到Driver中来。
List<Tuple2<Long, Row>> rdd1Data = rdd1.collect()
// 然后使用Spark的广播功能，将小RDD的数据转换成广播变量，这样每个Executor就只有一份RDD的数据。
// 可以尽可能节省内存空间，并且减少网络传输性能开销。
final Broadcast<List<Tuple2<Long, Row>>> rdd1DataBroadcast = sc.broadcast(rdd1Data);
  
// 对另外一个RDD执行map类操作，而不再是join类操作。
JavaPairRDD<String, Tuple2<String, Row>> joinedRdd = rdd2.mapToPair(
        new PairFunction<Tuple2<Long,String>, String, Tuple2<String, Row>>() {
            private static final long serialVersionUID = 1L;
            @Override
            public Tuple2<String, Tuple2<String, Row>> call(Tuple2<Long, String> tuple)
                    throws Exception {
                // 在算子函数中，通过广播变量，获取到本地Executor中的rdd1数据。
                List<Tuple2<Long, Row>> rdd1Data = rdd1DataBroadcast.value();
                // 可以将rdd1的数据转换为一个Map，便于后面进行join操作。
                Map<Long, Row> rdd1DataMap = new HashMap<Long, Row>();
                for(Tuple2<Long, Row> data : rdd1Data) {
                    rdd1DataMap.put(data._1, data._2);
                }
                // 获取当前RDD数据的key以及value。
                String key = tuple._1;
                String value = tuple._2;
                // 从rdd1数据Map中，根据key获取到可以join到的数据。
                Row rdd1Value = rdd1DataMap.get(key);
                return new Tuple2<String, String>(key, new Tuple2<String, Row>(value, rdd1Value));
            }
        });
  
// 这里得提示一下。
// 上面的做法，仅仅适用于rdd1中的key没有重复，全部是唯一的场景。
// 如果rdd1中有多个相同的key，那么就得用flatMap类的操作，在进行join的时候不能用map，而是得遍历rdd1所有数据进行join。
// rdd2中每条数据都可能会返回多条join后的数据。
```



#### 5.5.6 采样倾斜 key 并分拆 join 操作

**适用场景：**两个 RDD/Hive 表进行 join 的时候，如果数据量都比较大，无法采用“解决方案五”，那么此时可以看一下两个 RDD/Hive 表中的 key 分布情况。如果出现数据倾斜，是因为其中某一个 RDD/Hive 表中的少数几个 key 的数据量过大，而另一个 RDD/Hive 表中的所有 key 都分布比较均匀，那么采用这个解决方案是比较合适的。

**实现思路：**

* 对包含少数几个数据量过大的 key 的那个 RDD，通过 sample 算子采样出一份样本来，然后统计一下每个 key的数量，计算出来数据量最大的是哪几个 key。
* 然后将这几个 key 对应的数据从原来的 RDD 中拆分出来，形成一个单独的 RDD，并给每个 key 都打上 n 以内的随机数作为前缀，而不会导致倾斜的大部分 key 形成另外一个 RDD。
* 接着将需要 join 的另一个 RDD，也过滤出来那几个倾斜 key 对应的数据并形成一个单独的RDD，将每条数据膨胀成 n 条数据，这n条数据都按顺序附加一个 0~n 的前缀，不会导致倾斜的大部分 key 也形成另外一个 RDD。
* 再将附加了随机前缀的独立 RDD 与另一个膨胀 n 倍的独立 RDD 进行 join，此时就可以将原先相同的 key 打散成 n 份，分散到多个 task 中去进行 join 了。
* 而另外两个普通的 RDD 就照常  join即可。
* 最后将两次 join 的结果使用 union 算子合并起来即可，就是最终的 join 结果。

**实现原理：**对于 join 导致的数据倾斜，如果只是某几个 key 导致了倾斜，可以将少数几个 key 分拆成独立 RDD，并附加随机前缀打散成 n 份去进行 join，此时这几个 key 对应的数据就不会集中在少数几个 task 上，而是分散到多个 task 进行 join 了。具体原理见下图。

![spark-split-join](https://gitee.com/struggle3014/picBed/raw/master/spark-split-join.png)

**方案优点：**对于 join 导致的数据倾斜，如果只是某几个 key 导致了倾斜，采用该方式可以用最有效的方式打散 key 进行 join 。而且只需要针对少数倾斜 key 对应的数据进行扩容 n 倍，不需要对全量数据进行扩容。避免了占用过多内存。

**方案缺点：**如果导致倾斜的 key 特别多的话，比如成千上万个 key 都导致数据倾斜，那么这种方式也不适合。

**代码示例：**

```scala
// 首先从包含了少数几个导致数据倾斜key的rdd1中，采样10%的样本数据。
JavaPairRDD<Long, String> sampledRDD = rdd1.sample(false, 0.1);
  
// 对样本数据RDD统计出每个key的出现次数，并按出现次数降序排序。
// 对降序排序后的数据，取出top 1或者top 100的数据，也就是key最多的前n个数据。
// 具体取出多少个数据量最多的key，由大家自己决定，我们这里就取1个作为示范。
JavaPairRDD<Long, Long> mappedSampledRDD = sampledRDD.mapToPair(
        new PairFunction<Tuple2<Long,String>, Long, Long>() {
            private static final long serialVersionUID = 1L;
            @Override
            public Tuple2<Long, Long> call(Tuple2<Long, String> tuple)
                    throws Exception {
                return new Tuple2<Long, Long>(tuple._1, 1L);
            }     
        });
JavaPairRDD<Long, Long> countedSampledRDD = mappedSampledRDD.reduceByKey(
        new Function2<Long, Long, Long>() {
            private static final long serialVersionUID = 1L;
            @Override
            public Long call(Long v1, Long v2) throws Exception {
                return v1 + v2;
            }
        });
JavaPairRDD<Long, Long> reversedSampledRDD = countedSampledRDD.mapToPair( 
        new PairFunction<Tuple2<Long,Long>, Long, Long>() {
            private static final long serialVersionUID = 1L;
            @Override
            public Tuple2<Long, Long> call(Tuple2<Long, Long> tuple)
                    throws Exception {
                return new Tuple2<Long, Long>(tuple._2, tuple._1);
            }
        });
final Long skewedUserid = reversedSampledRDD.sortByKey(false).take(1).get(0)._2;
  
// 从rdd1中分拆出导致数据倾斜的key，形成独立的RDD。
JavaPairRDD<Long, String> skewedRDD = rdd1.filter(
        new Function<Tuple2<Long,String>, Boolean>() {
            private static final long serialVersionUID = 1L;
            @Override
            public Boolean call(Tuple2<Long, String> tuple) throws Exception {
                return tuple._1.equals(skewedUserid);
            }
        });
// 从rdd1中分拆出不导致数据倾斜的普通key，形成独立的RDD。
JavaPairRDD<Long, String> commonRDD = rdd1.filter(
        new Function<Tuple2<Long,String>, Boolean>() {
            private static final long serialVersionUID = 1L;
            @Override
            public Boolean call(Tuple2<Long, String> tuple) throws Exception {
                return !tuple._1.equals(skewedUserid);
            } 
        });
  
// rdd2，就是那个所有key的分布相对较为均匀的rdd。
// 这里将rdd2中，前面获取到的key对应的数据，过滤出来，分拆成单独的rdd，并对rdd中的数据使用flatMap算子都扩容100倍。
// 对扩容的每条数据，都打上0～100的前缀。
JavaPairRDD<String, Row> skewedRdd2 = rdd2.filter(
         new Function<Tuple2<Long,Row>, Boolean>() {
            private static final long serialVersionUID = 1L;
            @Override
            public Boolean call(Tuple2<Long, Row> tuple) throws Exception {
                return tuple._1.equals(skewedUserid);
            }
        }).flatMapToPair(new PairFlatMapFunction<Tuple2<Long,Row>, String, Row>() {
            private static final long serialVersionUID = 1L;
            @Override
            public Iterable<Tuple2<String, Row>> call(
                    Tuple2<Long, Row> tuple) throws Exception {
                Random random = new Random();
                List<Tuple2<String, Row>> list = new ArrayList<Tuple2<String, Row>>();
                for(int i = 0; i < 100; i++) {
                    list.add(new Tuple2<String, Row>(i + "_" + tuple._1, tuple._2));
                }
                return list;
            }
              
        });
 
// 将rdd1中分拆出来的导致倾斜的key的独立rdd，每条数据都打上100以内的随机前缀。
// 然后将这个rdd1中分拆出来的独立rdd，与上面rdd2中分拆出来的独立rdd，进行join。
JavaPairRDD<Long, Tuple2<String, Row>> joinedRDD1 = skewedRDD.mapToPair(
        new PairFunction<Tuple2<Long,String>, String, String>() {
            private static final long serialVersionUID = 1L;
            @Override
            public Tuple2<String, String> call(Tuple2<Long, String> tuple)
                    throws Exception {
                Random random = new Random();
                int prefix = random.nextInt(100);
                return new Tuple2<String, String>(prefix + "_" + tuple._1, tuple._2);
            }
        })
        .join(skewedUserid2infoRDD)
        .mapToPair(new PairFunction<Tuple2<String,Tuple2<String,Row>>, Long, Tuple2<String, Row>>() {
                        private static final long serialVersionUID = 1L;
                        @Override
                        public Tuple2<Long, Tuple2<String, Row>> call(
                            Tuple2<String, Tuple2<String, Row>> tuple)
                            throws Exception {
                            long key = Long.valueOf(tuple._1.split("_")[1]);
                            return new Tuple2<Long, Tuple2<String, Row>>(key, tuple._2);
                        }
                    });
 
// 将rdd1中分拆出来的包含普通key的独立rdd，直接与rdd2进行join。
JavaPairRDD<Long, Tuple2<String, Row>> joinedRDD2 = commonRDD.join(rdd2);
 
// 将倾斜key join后的结果与普通key join后的结果，uinon起来。
// 就是最终的join结果。
JavaPairRDD<Long, Tuple2<String, Row>> joinedRDD = joinedRDD1.union(joinedRDD2);
```



#### 5.5.7 使用随机前缀和扩容 RDD 进行 join

**适用场景：**如果在进行 join 操作时，RDD 中有大量的 key 导致数据倾斜，那么进行分拆 key 也没什么意义，此时就只能使用最后一种方案来解决问题了。

**实现思路：**

* 该方案的实现思路基本和“解决方案六”类似，首先查看 RDD/Hive 表中的数据分布情况，找到那个造成数据倾斜的 RDD/Hive 表，比如有多个 key 都对应了超过 1 万条数据。
* 然后将该 RDD 的每条数据都打上一个 n 以内的随机前缀。
* 同时对另外一个正常的 RDD 进行扩容，将每条数据都扩容成 n 条数据，扩容出来的每条数据都依次打上一个 0~n 的前缀。
* 最后将两个处理后的 RDD 进行 join 即可。

**实现原理：**将原先一样的 key 通过附加随机前缀变成不一样的 key，然后就可以将这些处理后的“不同 key”分散到多个 task 中去处理，而不是让一个 task 处理大量的相同 key。该方案与“解决方案六”的不同之处就在于，上一种方案是尽量只对少数倾斜 key 对应的数据进行特殊处理，由于处理过程需要扩容 RDD，因此上一种方案扩容 RDD 后对内存的占用并不大；而这一种方案是针对有大量倾斜 key 的情况，没法将部分 key 拆分出来进行单独处理，因此只能对整个 RDD 进行数据扩容，对内存资源要求很高。

**方案优点：**对 join 类型的数据倾斜基本都可以处理，而且效果也相对比较显著，性能提升效果非常不错。

**方案缺点：**该方案更多的是缓解数据倾斜，而不是彻底避免数据倾斜。而且需要对整个 RDD 进行扩容，对内存资源要求很高。

**代码示例：**

```scala
// 首先将其中一个key分布相对较为均匀的RDD膨胀100倍。
JavaPairRDD<String, Row> expandedRDD = rdd1.flatMapToPair(
        new PairFlatMapFunction<Tuple2<Long,Row>, String, Row>() {
            private static final long serialVersionUID = 1L;
            @Override
            public Iterable<Tuple2<String, Row>> call(Tuple2<Long, Row> tuple)
                    throws Exception {
                List<Tuple2<String, Row>> list = new ArrayList<Tuple2<String, Row>>();
                for(int i = 0; i < 100; i++) {
                    list.add(new Tuple2<String, Row>(0 + "_" + tuple._1, tuple._2));
                }
                return list;
            }
        });
  
// 其次，将另一个有数据倾斜key的RDD，每条数据都打上100以内的随机前缀。
JavaPairRDD<String, String> mappedRDD = rdd2.mapToPair(
        new PairFunction<Tuple2<Long,String>, String, String>() {
            private static final long serialVersionUID = 1L;
            @Override
            public Tuple2<String, String> call(Tuple2<Long, String> tuple)
                    throws Exception {
                Random random = new Random();
                int prefix = random.nextInt(100);
                return new Tuple2<String, String>(prefix + "_" + tuple._1, tuple._2);
            }
        });
  
// 将两个处理后的RDD进行join即可。
JavaPairRDD<String, Tuple2<String, Row>> joinedRDD = mappedRDD.join(expandedRDD);
```



#### 5.5.8 多种方案组合使用

在实践中发现，很多情况下，如果只是处理较为简单的数据倾斜场景，那么使用上述方案中的某一种基本就可以解决。但是如果要处理一个较为复杂的数据倾斜场景，那么可能需要将多种方案组合起来使用。比如说，我们针对出现了多个数据倾斜环节的Spark作业，可以先运用解决方案一和二，预处理一部分数据，并过滤一部分数据来缓解；其次可以对某些shuffle操作提升并行度，优化其性能；最后还可以针对不同的聚合或join操作，选择一种方案来优化其性能。大家需要对这些方案的思路和原理都透彻理解之后，在实践中根据各种不同的情况，灵活运用多种方案，来解决自己的数据倾斜问题。



## 6 Shuffle 调优

### 6.1 调优概述

大多数Spark作业的性能主要就是消耗在了 shuffle 环节，因为该环节包含了大量的磁盘 IO、序列化、网络数据传输等操作。因此，为了进一步提高作业的性能，需要对 shuffle 过程进行调优。**注意：shuffle 调优只是整个 Spark 性能调优中的一小部分，不要舍本逐末。**下面讲解 shuffle 的原理，及相关参数说明和各参数调优建议。



### 6.2 ShuffleManager 发展概述

在 Spark 的源码中，负责 shuffle 过程的执行、计算和处理的组件主要就是 ShuffleManager，也即 shuffle 管理器。而随着 Spark 的版本的发展，ShuffleManager 也在不断迭代，变得越来越先进。

* 在 Spark 1.2 以前，默认的 shuffle 计算引擎是 HashShuffleManager。该 HashShuffleManager 有着一个非常严重的弊端，就是会产生大量的中间磁盘文件，进而由大量的磁盘 IO 操作影响了性能。

* 在 Spark 1.2 以后的版本中，默认的 ShuffleManager 改成了 SortShuffleManager。SortShuffleManager 相较于 HashShuffleManager 来说，有了一定的改进。主要就在于，每个 task 在进行 shuffle 操作时，虽然也会产生较多的临时磁盘文件，但是最后会将所有的临时文件合并（merge）成一个磁盘文件，因此每个Task就只有一个磁盘文件。在下一个 stage 的 shuffle read task 拉取自己的数据时，只要根据索引读取每个磁盘文件中的部分数据即可。



### 6.3 HashShuffleManager 运行原理

#### 6.3.1 未经优化的 HashShuffleManager

下图说明了未经优化的 HashShuffleManager 的原理。这里我们先明确一个假设前提：每个 executor 只有1个CPU core，也就是说，无论这个 executor 上分配多少个 task 线程，同一时间都只能执行一个 task 线程。

![hashShuffleManager-principle](https://gitee.com/struggle3014/picBed/raw/master/hashShuffleManager-principle.png)

* **shuffle write 阶段**

  在一个 stage 结束计算之后所要做的事情。

  * 为了下一个 stage 可以执行 shuffle 类的算子（比如 reduceByKey），而将每个 task 处理的数据按 key 进行“分类”。所谓“分类”，就是对相同的 key 执行 hash 算法，从而将相同 key 都写入同一个磁盘文件中，而每一个磁盘文件都只属于下游 stage 的一个 task。在将数据写入磁盘之前，会先将数据写入内存缓冲中，当内存缓冲填满之后，才会溢写到磁盘文件中去。
  * 每个执行 shuffle write 的 task，要为下一个 stage 创建多少个磁盘文件呢？下一个 stage 的 task 有多少个，当前 stage 的每个 task 就要创建多少份磁盘文件。
    * **例如**：下一个 stage 总共有 100 个 task，那么当前 stage 的每个 task 都要创建 100 份磁盘文件。如果当前 stage 有 50 个 task，总共有 10 个 executor，每个 executor执行5个Task，那么每个executor 上总共就要创建500个磁盘文件，所有 executor 上会创建 5000 个磁盘文件。

* **shuffle read 阶段**

  一个 stage 刚开始时要做的事情。

  * 此时该 stage 的每一个 task 就需要将上一个 stage 的计算结果中的所有相同 key，从各个节点上通过网络都拉取到自己所在的节点上，然后进行 key 的聚合或连接等操作。由于 shuffle write 的过程中，task给下游 stage 的每个 task 都创建了一个磁盘文件，因此 shuffle read 的过程中，每个 task 只要从上游 stage 的所有 task 所在节点上，拉取属于自己的那一个磁盘文件即可。
  * shuffle read 的拉取过程是一边拉取一边进行聚合的。每个 shuffle read task 都会有一个自己的 buffer 缓冲，每次都只能拉取与 buffer 缓冲相同大小的数据，然后通过内存中的一个 Map 进行聚合等操作。聚合完一批数据后，再拉取下一批数据，并放到 buffer 缓冲中进行聚合操作。以此类推，直到最后将所有数据到拉取完，并得到最终的结果。



#### 6.3.2 优化后的 HashShufffleManager

下图说明了优化后的 HashShuffleManager 的原理。这里说的优化，是指我们可以设置一个参数，spark.shuffle.consolidateFiles。该参数默认值为 false，将其设置为 true 即可开启优化机制。通常来说，如果我们使用 HashShuffleManager，那么都建议开启这个选项。

开启 consolidate 机制之后，在 shuffle write 过程中，task 就不是为下游 stage 的每个 task 创建一个磁盘文件了。此时会出现 shuffleFileGroup 的概念，每个 shuffleFileGroup 会对应一批磁盘文件，磁盘文件的数量与下游 stage 的 task 数量是相同的。一个 executor 上有多少个 CPU core，就可以并行执行多少个 task。而第一批并行执行的每个 task 都会创建一个 shuffleFileGroup，并将数据写入对应的磁盘文件内。

当 executor 的 CPU core 执行完一批 task，接着执行下一批 task 时，下一批 task 就会复用之前已有的 shuffleFileGroup，包括其中的磁盘文件。也就是说，此时 task 会将数据写入已有的磁盘文件中，而不会写入新的磁盘文件中。因此，consolidate 机制允许不同的 task 复用同一批磁盘文件，这样就可以有效将多个 task 的磁盘文件进行一定程度上的合并，从而大幅度减少磁盘文件的数量，进而提升 shuffle write 的性能。

假设第二个 stage 有 100 个 task，第一个 stage 有 50 个 task，总共还是有 10个 executor，每个 executor 执行5 个 task。那么原本使用未经优化的 HashShuffleManager 时，每个 executor 会产生 500 个磁盘文件，所有 executor 会产生 5000 个磁盘文件的。但是此时经过优化之后，每个 executor 创建的磁盘文件的数量的计算公式为：CPU core 的数量 * 下一个 stage 的 task 数量。也就是说，每个 executor 此时只会创建 100 个磁盘文件，所有 executor 只会创建 1000 个磁盘文件。

![optimize-hashShuffleManager](https://gitee.com/struggle3014/picBed/raw/master/optimize-hashShuffleManager.png)



### 6.4 SortShuffleManager 运行原理

SortShuffleManager 的运行机制主要分成两种，一种是普通运行机制，另一种是 bypass 运行机制。当 shuffle read task 的数量小于等于 spark.shuffle.sort.bypassMergeThreshold 参数的值时（默认为200），就会启用 bypass 机制。

#### 6.4.1 普通运行机制

下图说明了普通的 SortShuffleManager 的原理。在该模式下，数据会先写入一个内存数据结构中，此时根据不同的 shuffle 算子，可能选用不同的数据结构。如果是 reduceByKey 这种聚合类的 shuffle 算子，那么会选用 Map 数据结构，一边通过 Map 进行聚合，一边写入内存；如果是 join 这种普通的 shuffle 算子，那么会选用 Array 数据结构，直接写入内存。接着，每写一条数据进入内存数据结构之后，就会判断一下，是否达到了某个临界阈值。如果达到临界阈值的话，那么就会尝试将内存数据结构中的数据溢写到磁盘，然后清空内存数据结构。

在溢写到磁盘文件之前，会先根据key对内存数据结构中已有的数据进行排序。排序过后，会分批将数据写入磁盘文件。默认的 batch 数量是 10000 条，也就是说，排序好的数据，会以每批1万条数据的形式分批写入磁盘文件。写入磁盘文件是通过 Java 的 BufferedOutputStream 实现的。BufferedOutputStream是 Java 的缓冲输出流，首先会将数据缓冲在内存中，当内存缓冲满溢之后再一次写入磁盘文件中，这样可以减少磁盘 IO 次数，提升性能。

一个 task 将所有数据写入内存数据结构的过程中，会发生多次磁盘溢写操作，也就会产生多个临时文件。最后会将之前所有的临时磁盘文件都进行合并，这就是 merge 过程，此时会将之前所有临时磁盘文件中的数据读取出来，然后依次写入最终的磁盘文件之中。此外，由于一个 task 就只对应一个磁盘文件，也就意味着该 task 为下游stage 的 task 准备的数据都在这一个文件中，因此还会单独写一份索引文件，其中标识了下游各个 task 的数据在文件中的 start offset 与 end offset。

SortShuffleManager 由于有一个磁盘文件 merge 的过程，因此大大减少了文件数量。比如第一个 stage 有 50个task，总共有 10 个 executor，每个 executor 执行 5 个 task，而第二个 stage 有 100 个 task。由于每个 task 最终只有一个磁盘文件，因此此时每个 executor 上只有 5 个磁盘文件，所有 executor只有50个磁盘文件。

![oridinary-sortShuffleManager](https://gitee.com/struggle3014/picBed/raw/master/oridinary-sortShuffleManager.png)

#### 6.4.2 bypass 运行机制

下图说明了 bypass SortShuffleManager 的原理。bypass 运行机制的触发条件如下： 

* shuffle map task 数量小于 spark.shuffle.sort.bypassMergeThreshold 参数的值。 
* 不是聚合类的 shuffle 算子（比如reduceByKey）。

此时 task 会为每个下游 task 都创建一个临时磁盘文件，并将数据按 key 进行 hash 然后根据 key 的 hash 值，将 key 写入对应的磁盘文件之中。当然，写入磁盘文件时也是先写入内存缓冲，缓冲写满之后再溢写到磁盘文件的。最后，同样会将所有临时磁盘文件都合并成一个磁盘文件，并创建一个单独的索引文件。

该过程的磁盘写机制其实跟未经优化的 HashShuffleManager 是一模一样的，因为都要创建数量惊人的磁盘文件，只是在最后会做一个磁盘文件的合并而已。因此少量的最终磁盘文件，也让该机制相对未经优化的HashShuffleManager 来说，shuffle read 的性能会更好。

而该机制与普通 SortShuffleManager 运行机制的不同在于：

* 第一，磁盘写机制不同；
* 第二，不会进行排序。也就是说，启用该机制的最大好处在于，shuffle write过程中，不需要进行数据的排序操作，也就节省掉了这部分的性能开销。



![bypass-sortShuffleManager](C:\Users\yue_zhou\Desktop\images\bypass-sortShuffleManager.png)

### 6.5 shuffle 相关参数调优

#### 6.5.1 spark.shuffle.file.buffer

- 默认值：32k
- 参数说明：该参数用于设置 shuffle write task 的 BufferedOutputStream 的 buffer 缓冲大小。将数据写到磁盘文件之前，会先写入 buffer 缓冲中，待缓冲写满之后，才会溢写到磁盘。
- 调优建议：如果作业可用的内存资源较为充足的话，可以适当增加这个参数的大小（比如64k），从而减少 shuffle write 过程中溢写磁盘文件的次数，也就可以减少磁盘 IO 次数，进而提升性能。在实践中发现，合理调节该参数，性能会有 1%~5% 的提升。

#### 6.5.2 spark.reducer.maxSizeInFlight

- 默认值：48m
- 参数说明：该参数用于设置 shuffle read task 的 buffer 缓冲大小，而这个 buffer 缓冲决定了每次能够拉取多少数据。
- 调优建议：如果作业可用的内存资源较为充足的话，可以适当增加这个参数的大小（比如96m），从而减少拉取数据的次数，也就可以减少网络传输的次数，进而提升性能。在实践中发现，合理调节该参数，性能会有 1%~5% 的提升。

#### 6.5.3 spark.shuffle.io.maxRetries

- 默认值：3
- 参数说明：shuffle read task 从 shuffle write task 所在节点拉取属于自己的数据时，如果因为网络异常导致拉取失败，是会自动进行重试的。该参数就代表了可以重试的最大次数。如果在指定次数之内拉取还是没有成功，就可能会导致作业执行失败。
- 调优建议：对于那些包含了特别耗时的 shuffle 操作的作业，建议增加重试最大次数（比如60次），以避免由于 JVM 的 full gc 或者网络不稳定等因素导致的数据拉取失败。在实践中发现，对于针对超大数据量（数十亿~上百亿）的 shuffle 过程，调节该参数可以大幅度提升稳定性。

#### 6.5.4 spark.shuffle.io.retryWait

- 默认值：5s
- 参数说明：具体解释同上，该参数代表了每次重试拉取数据的等待间隔，默认是 5s。
- 调优建议：建议加大间隔时长（比如60s），以增加 shuffle 操作的稳定性。

#### 6.5.5 spark.shuffle.memoryFraction

- 默认值：0.2
- 参数说明：该参数代表了 executor 内存中，分配给 shuffle read task 进行聚合操作的内存比例，默认是20%。
- 调优建议：在资源参数调优中讲解过这个参数。如果内存充足，而且很少使用持久化操作，建议调高这个比例，给 shuffle read 的聚合操作更多内存，以避免由于内存不足导致聚合过程中频繁读写磁盘。在实践中发现，合理调节该参数可以将性能提升 10% 左右。

#### 6.5.6 spark.shuffle.manager

- 默认值：sort
- 参数说明：该参数用于设置 ShuffleManager 的类型。Spark 1.5 以后，有三个可选项：hash、sort和tungsten-sort。HashShuffleManager 是 Spark 1.2 以前的默认选项，但是 Spark 1.2 以及之后的版本默认都是 SortShuffleManager 了。tungsten-sort 与 sort 类似，但是使用了 tungsten 计划中的堆外内存管理机制，内存使用效率更高。
- 调优建议：由于 SortShuffleManager 默认会对数据进行排序，因此如果你的业务逻辑中需要该排序机制的话，则使用默认的 SortShuffleManager 就可以；而如果你的业务逻辑不需要对数据进行排序，那么建议参考后面的几个参数调优，通过 bypass 机制或优化的 HashShuffleManager 来避免排序操作，同时提供较好的磁盘读写性能。这里要注意的是，tungsten-sort 要慎用，因为之前发现了一些相应的 bug。

#### 6.5.7 spark.shuffle.sort.bypassMergeThreshold

- 默认值：200

- 参数说明：当 ShuffleManager 为 SortShuffleManager 时，如果 shuffle read task 的数量小于这个阈值（默认是200），则 shuffle write 过程中不会进行排序操作，而是直接按照未经优化的 HashShuffleManager 的方式去写数据，但是最后会将每个 task 产生的所有临时磁盘文件都合并成一个文件，并会创建单独的索引文件。

- 调优建议：当你使用 SortShuffleManager时，如果的确不需要排序操作，那么建议将这个参数调大一些，大于shuffle read task的数量。那么此时就会自动启用bypass机制，map-side就不会进行排序了，减少了排序的性能开销。但是这种方式下，依然会产生大量的磁盘文件，因此shuffle write性能有待提高。



#### 6.5.8 spark.shuffle.consolidateFiles

- 默认值：false
- 参数说明：如果使用 HashShuffleManager，该参数有效。如果设置为 true，那么就会开启 consolidate 机制，会大幅度合并shuffle write的输出文件，对于 shuffle read task 数量特别多的情况下，这种方法可以极大地减少磁盘IO开销，提升性能。
- 调优建议：如果的确不需要 SortShuffleManager 的排序机制，那么除了使用 bypass 机制，还可以尝试将 spark.shffle.manager 参数手动指定为 hash，使用 HashShuffleManager，同时开启 consolidate 机制。在实践中尝试过，发现其性能比开启了 bypass 机制的 SortShuffleManager 要高出 10%~30%。



# 总结

本文介绍了开发过程中的优化原则，运行前的资源参数设置调优，运行中的数据倾斜的解决方案及 shuffle 调优。

记住这些性能调优的原则以及方案，在 Spark 作业开发、测试以及运行的过程中多尝试，这样，我们才能不断提高 Spark 作业的性能。



# 参考文献

[1] [Spark 调优，官方文档](http://spark.apache.org/docs/latest/tuning.html)

[2] [Spark SQL 调优指南，官方文档]()

[3] [美团技术团队—Spark 性能优化指南基础篇](https://tech.meituan.com/2016/04/29/spark-tuning-basic.html)

[4] [美团技术团队—Spark 性能优化指南高级篇](https://tech.meituan.com/2016/05/12/spark-tuning-pro.html)