<div align="center"><img src="https://gitee.com/struggle3014/picBed/raw/master/name_code.png"></div>

# 导读

本文对 Spark 常见算子进行总结，方便后续回顾。

***持续更新中~***



# 目录

<nav>
<a href='#导读' style='text-decoration:none;font-weight:bolder'>导读</a><br/>
<a href='#目录' style='text-decoration:none;font-weight:bolder'>目录</a><br/>
<a href='#正文' style='text-decoration:none;font-weight:bolder'>正文</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1 创建类算子' style='text-decoration:none;${border-style}'>1 创建类算子</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#sc.parallelize' style='text-decoration:none;${border-style}'>sc.parallelize</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#sc.makeRDD' style='text-decoration:none;${border-style}'>sc.makeRDD</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#sc.textFile' style='text-decoration:none;${border-style}'>sc.textFile</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2 转换类算子' style='text-decoration:none;${border-style}'>2 转换类算子</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.1 映射类' style='text-decoration:none;${border-style}'>2.1 映射类</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#map' style='text-decoration:none;${border-style}'>map</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#flatMap' style='text-decoration:none;${border-style}'>flatMap</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#mapPartitions' style='text-decoration:none;${border-style}'>mapPartitions</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#mapPartitionsWithIndex' style='text-decoration:none;${border-style}'>mapPartitionsWithIndex</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#mapValues' style='text-decoration:none;${border-style}'>mapValues</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#flatMapValues' style='text-decoration:none;${border-style}'>flatMapValues</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#zipWithIndex' style='text-decoration:none;${border-style}'>zipWithIndex</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#zipWithUniqueId' style='text-decoration:none;${border-style}'>zipWithUniqueId</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.2 过滤类' style='text-decoration:none;${border-style}'>2.2 过滤类</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#filter' style='text-decoration:none;${border-style}'>filter</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.3 排序类' style='text-decoration:none;${border-style}'>2.3 排序类</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#sortBy' style='text-decoration:none;${border-style}'>sortBy</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#sortByKey' style='text-decoration:none;${border-style}'>sortByKey</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.4 聚合类' style='text-decoration:none;${border-style}'>2.4 聚合类</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#combineByKey' style='text-decoration:none;${border-style}'>combineByKey</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#reduceByKey' style='text-decoration:none;${border-style}'>reduceByKey</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#groupByKey' style='text-decoration:none;${border-style}'>groupByKey</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#aggregateByKey' style='text-decoration:none;${border-style}'>aggregateByKey</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#distinct' style='text-decoration:none;${border-style}'>distinct</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.5 连接类' style='text-decoration:none;${border-style}'>2.5 连接类</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#cogroup' style='text-decoration:none;${border-style}'>cogroup</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#join' style='text-decoration:none;${border-style}'>join</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.6 数据集操作类' style='text-decoration:none;${border-style}'>2.6 数据集操作类</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#cartesian' style='text-decoration:none;${border-style}'>cartesian</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#intersection' style='text-decoration:none;${border-style}'>intersection</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#subtract' style='text-decoration:none;${border-style}'>subtract</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#union' style='text-decoration:none;${border-style}'>union</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.7 高级算子' style='text-decoration:none;${border-style}'>2.7 高级算子</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#sample' style='text-decoration:none;${border-style}'>sample</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3 行动类算子' style='text-decoration:none;${border-style}'>3 行动类算子</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#foreach' style='text-decoration:none;${border-style}'>foreach</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#foreachPartitions' style='text-decoration:none;${border-style}'>foreachPartitions</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#collect' style='text-decoration:none;${border-style}'>collect</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#take' style='text-decoration:none;${border-style}'>take</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#count' style='text-decoration:none;${border-style}'>count</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#saveAsTextFile' style='text-decoration:none;${border-style}'>saveAsTextFile</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4 控制类算子' style='text-decoration:none;${border-style}'>4 控制类算子</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4.1 缓存' style='text-decoration:none;${border-style}'>4.1 缓存</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#persist' style='text-decoration:none;${border-style}'>persist</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#cache' style='text-decoration:none;${border-style}'>cache</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#unpersist' style='text-decoration:none;${border-style}'>unpersist</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#checkpoint' style='text-decoration:none;${border-style}'>checkpoint</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4.2 重分区' style='text-decoration:none;${border-style}'>4.2 重分区</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#coalesce' style='text-decoration:none;${border-style}'>coalesce</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#repartition' style='text-decoration:none;${border-style}'>repartition</a><br/>
<a href='#总结' style='text-decoration:none;font-weight:bolder'>总结</a><br/>
<a href='#参考文献' style='text-decoration:none;font-weight:bolder'>参考文献</a><br/>
</nav>

# 正文

## 1 创建类算子

### sc.parallelize

* **函数**

  ```scala
  def parallelize[T: ClassTag](
        seq: Seq[T],
        numSlices: Int = defaultParallelism): RDD[T] = withScope {
      new ParallelCollectionRDD[T](this, seq, numSlices, Map[Int, Seq[String]]())
    }
  ```

* **功能**

  分发本地 Scala 集合生成 RDD

* **使用场景**

* **拓展**

  **parallelize 是懒操作**。如果 seq 是可变集合并且在该 RDD 上调用第一个 action 操作之前如果该集合发生改变，那么相关的 RDD 也会反映该修改的集合。可以传入参数的副本防止上述情况。

* **案例**

### sc.makeRDD

* **函数**

  ```scala
  def makeRDD[T: ClassTag](
        seq: Seq[T],
        numSlices: Int = defaultParallelism): RDD[T] = withScope {
      parallelize(seq, numSlices)
  }
  ```

* **功能**

  分发本地 Scala 集合生成 RDD，底层实现是 parallelize。

### sc.textFile

* **函数**

  ```scala
  def textFile(
        path: String,
        minPartitions: Int = defaultMinPartitions): RDD[String] = withScope {
      hadoopFile(path, classOf[TextInputFormat], classOf[LongWritable], classOf[Text],
        minPartitions).map(pair => pair._2.toString).setName(path)
  }
  ```

* **功能**

  从 HDFS 文件系统读取文本文件，本地文件（所有节点都可以获取），或任意 Hadoop 支持的文件 URI，返回 RDD[String]。

* **使用场景**

* **案例**

## 2 转换类算子

### 2.1 映射类

#### map

* **函数**

  ```scala
  def map[T](f: T => U): RDD[U] = withScope {
      new MapPartitionsRDD[U, T](this, (context, pid, iter) => iter.map(cleanF))
  }
  ```

* **功能**

  返回一个应用 f 函数在该 RDD 上每个**元素**的新 RDD

* **使用场景**

  一进一出的情形

* **拓展**

* **案例**

#### flatMap

* **函数**

  ```scala
  def map[U: ClassTag](f: T => TraversableOnce[U]): RDD[U] = withScope {
      new MapPartitionsRDD[U, T](this, (context, pid, iter) => iter.flatMap(cleanF))
  }
  ```

* **功能**

  返回一个应用 f 函数在该 RDD 上每个**元素**的新 RDD

* **使用场景**

  一进多出的情形，扁平化处理

* **拓展**

  由于是一进多出的情形，可以做 map 和 filter 的逻辑。

* **案例**

#### mapPartitions

* **函数**

  ```scala
  def mapPartitions[U: ClassTag](
        f: Iterator[T] => Iterator[U],
        preservesPartitioning: Boolean = false): RDD[U] = withScope {
      new MapPartitionsRDD(
        this,
        (context: TaskContext, index: Int, iter: Iterator[T]) => cleanedF(iter),
        preservesPartitioning)
  }
  ```

* **功能**

  返回一个应用 f 函数在该 RDD 上每个**分区**的新 RDD

* **使用场景**

  针对比较重的操作可以使用 mapPartition，如数据库的连接。

  当然，由于为每个分区创建一个连接，同一分区的所有数据复用同一个连接，该方式仍然会造成一定的资源浪费。最理想的方式是所有分区、所有数据始终复用同一连接池，亦即整个 Spark 应用的每个 Executor 对应每一连接（连接池）。可用 Scala lazy 关键字和 Spark broadcast 功能实现，具体查看 [SparkCore#广播变量](./SparkCore.md)。

* **拓展**

  注意 f 函数内部要**正确使用迭代器模式**，否则会造成数据挤压，可能会造成内存溢出。

* **案例**

  ```scala
  // 不正确使用迭代器模式，致命~！！！
  data.mapPartitionsWithIndex(
      (pIndex, pIter) => {
          // TODO ... 创建 MySQL 连接
          val lb = new ListBuffer[String] // 致命的！！！Spark 源码中发现 Spark 是一个 pipline，迭代器的嵌套模式
          // TODO ...
          while(pIter.hasNext) {
              val value: Int = pIter.next()
              lb += (value + "selected~")
          }
          // TODO ... 关闭连接
          lb.iterator
      }
  )
  
  // 正确使用迭代器模式的方式~！！！
  data.mapPartitionsWithIndex(
      (pIndex, pIter) => {
          new Iterator[String] {
              // TODO ... 创建 MySQL 连接
              override def hasNext = pIter.hasNext =
              if(pIter.hasNext == false) {
                  // TODO ... 关闭连接
                  false;
              } else {
                  true;
              }
              override def next(): Int = {
                  val value: Int = pIter.next();
                  value+"selected~!"
              }
          }
      }
  )
  ```

  

#### mapPartitionsWithIndex

* **函数**

  ```scala
  def mapPartitionsWithIndexInternal[U: ClassTag](
        f: (Int, Iterator[T]) => Iterator[U],
        preservesPartitioning: Boolean = false): RDD[U] = withScope {
      new MapPartitionsRDD(
        this,
        (context: TaskContext, index: Int, iter: Iterator[T]) => f(index, iter),
        preservesPartitioning)
  }
  ```

* **功能**

  返回一个应用 f 函数在该 RDD 上每个**分区**的新 RDD，会追踪原始的分区索引

* **使用场景**

  观察统计 RDD 的分区数据情况，如是否分区数据量过少（使用 coalesce 减少分区数），是否发生数据倾斜等

* **拓展**

* **案例**

  ```scala
  # 创建 RDD
  val rdd1 = sc.parallelize(Array(("A", 1), ("B", 2), ("C", 3)))
  # 使用 mapPartitionsWithIndex 遍历
  rdd1.mapPartitionsWithIndex((index, iter) => {
    println(s"partitionId: $index")
      while(iter.hasNext) {
          print(iterator.next)
      }
      iter
  })
  ```

  

#### mapValues

* **函数**

  ```scala
  def flatMapValues[U](f: V => TraversableOnce[U]): RDD[(K, U)] = self.withScope {
      new MapPartitionsRDD[(K, U), (K, V)](self,
        (context, pid, iter) => iter.flatMap { case (k, v) =>
          cleanF(v).map(x => (k, x))
        },
        preservesPartitioning = true)
  }
  ```

  

* **功能**

  将 key-value pair RDD 的每个 value 传给一个 map 函数，不改变 keys。同时**保留原始 RDD 的分区**。

* **使用场景**

  * 若 key 未发生变化，分区器未发生变化，分区数未发生变化，且是 KV 类型的 RDD，那么建议使用 mapValues 或 flatMapValues，可以**避免不必要的 Shuffle 操作**。因为后续再使用 combineKey 类算子时，会判断分区器是否相同，若相同，则不走 Shuffle 逻辑，否则走 Shuffle 逻辑。

    ![mapValues血缘关系](https://gitee.com/struggle3014/picBed/raw/master/mapValues血缘关系.png)

    <div align="center"><font size="2">mapValues血缘关系</font></div>

* **拓展**

* **案例**

  ```scala
  // 使用 mapValues 或 flatMapValues 算子优化，避免不必要的 shuffle 操作
  val data: RDD[String] = sc.parallelize(List(
  			"hello world",
  			"hello spark",
  			"hello flink",
  			"hello world",
  			"hello hadoop"
  		))
  val words: RDD[String] = data.flatMap(_.split(" "))
  val kv: RDD[(String, Int)] = words.map((_, 1))
  val res: RDD[(String, Int)] = kv.reduceByKey(_+_)
  val res01: RDD[(String, Int)] = res.map(x => (x._1, x._2*10))
  // val res01: RDD[(String, Iterable)] = res.mapValues(x => x*10) // 优化方法
  val res02: RDD[(String, Iterable[Int])] = res01.groupByKey()
  res02.foreach(println)
  ```

#### flatMapValues

* **函数**

  ```scala
  def flatMapValues[U](f: V => TraversableOnce[U]): RDD[(K, U)] = self.withScope {
      new MapPartitionsRDD[(K, U), (K, V)](self,
        (context, pid, iter) => iter.flatMap { case (k, v) =>
          cleanF(v).map(x => (k, x))
        },
        preservesPartitioning = true)
  }
  ```

* **功能**

  将 key-value pair RDD 的每个 value 传给一个 flatMap 函数，不改变 keys。同时**保留原始 RDD 的分区**。

* **使用场景**

  与 mapValues 类似

* **拓展**

* **案例**

#### zipWithIndex

* **函数**

  ```scala
  def zipWithIndex(): RDD[(T, Long)] = withScope {
      new ZippedWithIndexRDD(this)
  }
  ```

* **功能**

  使用该 RDD 的元素的索引 zip 该RDD。排序首先基于分区索引，然后基于每个分区内的条目的顺序。因此，第一个条目是第一个分区的0号索引所在的元素，最后一个条目是最后一个分区的最大索引所在的元素。

* **使用场景**

  为 RDD 中所有分区的每个元素有序创建唯一索引。**如果分区数超过1，会触发 Spark 作业**。

* **案例**

#### zipWithUniqueId

* **函数**

  ```scala
  def zipWithUniqueId(): RDD[(T, Long)] = withScope {
      val n = this.partitions.length.toLong
      this.mapPartitionsWithIndex { case (k, iter) =>
          Utils.getIteratorZipWithIndex(iter, startIndex = 0L).map { case (item, i) =>
          	(item, i * n + k)
          }
      }
  }
  ```

* **功能**

  使用生成的唯一 Long 类型的 ids 来zip 该 RDD。在第 k 个分区的条目将获得此类 ids，即 k， n+k， 2*n+k， ... 其中 n 是分区数。

* **使用场景**

  为 RDD 的所有分区的每个元素创建唯一索引。**不会触发 Spark 作业，区别于 zipWithIndex**。

* **案例**

### 2.2 过滤类

#### filter

* **函数**

  ```scala
  def filter(f: T => Boolean): RDD[(T)] = withScope {
      new MapPartitionsRDD[T, T] (
          this,
          (context, pid, iter) => iter.filter(cleanF),
          preservesPartitioning = true)
  }
  ```

* **功能**

  返回一个新 RDD，仅仅包含满足函数 f 的元素

* **使用场景**

  过滤出符合条件的元素。**如果 filter 之后，RDD 每个分区数据量较少，可以使用 coalesce 对 RDD 分区进行合并**。

* **案例**

### 2.3 排序类

#### sortBy

* **函数**

  ```scala
  def sortBy[K](
        f: (T) => K,
        ascending: Boolean = true,
        numPartitions: Int = this.partitions.length)
        (implicit ord: Ordering[K], ctag: ClassTag[K]): RDD[T] = withScope {
      this.keyBy[K](f)
          .sortByKey(ascending, numPartitions)
          .values
  }
  ```

* **功能**

  返回基于给定的 key 函数排序的 RDD

* **使用场景**

* **案例**

#### sortByKey

* **函数**

  ```scala
  def sortByKey(ascending: Boolean = true, numPartitions: Int = self.partitions.length)
        : RDD[(K, V)] = self.withScope {
      val part = new RangePartitioner(numPartitions, self, ascending)
      new ShuffledRDD[K, V, V](self, part)
        .setKeyOrdering(if (ascending) ordering else ordering.reverse)
  }
  ```

* **功能**

  基于 key 排序 RDD，因此每个分区包含一个排序好范围的元素。

* **使用场景**

* **案例**

### 2.4 聚合类

#### combineByKey

* **函数**

  ```scala
  def combineByKey[C](
        createCombiner: V => C, // 为每一个分区中每一个分组进行初始化，如何初始化？将createCobiner 函数作用在这个分组的第一个元素上
        mergeValue: (C, V) => C, // 按照 mergeValue 的聚合逻辑对每一个分区中每一个分组进行聚合
        mergeCombiners: (C, C) => C, // reduce 端的大聚合
        partitioner: Partitioner,
        mapSideCombine: Boolean = true,
        serializer: Serializer = null): RDD[(K, C)] = self.withScope {
      combineByKeyWithClassTag(createCombiner, mergeValue, mergeCombiners,
        partitioner, mapSideCombine, serializer)(null)
  }
  ```

* **功能**

  使用一组自定义聚合函数组合每个键的元素。

* **使用场景**

* **拓展**

  combineByKey 是 Spark 调优的关键，可以使用 combineByKey 实现自定义聚合逻辑。

* **案例**

  ```scala
  // sum, count, min, max, avg
  // sum 求和操作
  val data:  RDD[(String, Int)] = null
  val sum: RDD[(String, Int)] = data.reduceByKey(_+_)
  sum.foreach(println)
  // max 求最大值
  val data:  RDD[(String, Int)] = null
  val max: RDD[(Streing, Int)] = data.reduceByKey((oldV, newV) => if(oldV > newV) oldV else newV)
  max.foreach(println)
  // min 求最小值
  val data:  RDD[(String, Int)] = null
  val min: RDD[(Streing, Int)] = data.reduceByKey((oldV, newV) => if(oldV > newV) newV else oldV)
  min.foreach(println)
  // count 计数
  val data:  RDD[(String, Int)] = null
  val count: RDD[(Streing, Int)] = data.mapValues(e => 1).reduceByKey(_+_)
  count.foreach(println)
  // avg 计算
  val tmp: RDD[(String, (Int, Int))] = sum.join(count)
  val avg = tmp.mapValues(x => x.1 / x.2)
  avg.foreach(println)
  // avg 优化，avg 的操作涉及对同一数据集做两次计算的操作得到 sum 和 join，其次解决了 sum 和 count 数据相遇的问题，再计算平均值。如何只计算一次，就可以做到相同的效果？优化武器 combineByKey
  val tmp = data.combineByKey(
      (value: Int) => (value, 1), // createCombiner: V => C	第一条记录的 value 如何放入 hashMap 中
      (oldValue: (Int, Int), newValue: Int) => (oldValue._1 + newValue, oldValue._2 + 1), // mergeValue: (C, V) => C	如果有第二条记录，第二条及以后的 value 如何放入 hashMap，其中 key 为加总和，value 为元素个数
      (v1: (Int, Int), v2: (Int, Int)) => (v1._1 + v2._1, v1._2 + v2._2)) // mergeCombiners: (C, C) => C	合并溢写结果的函数
  tmp.mapValues(_.1/_.2).foreach(println)
  ```

#### reduceByKey

* **函数**

  ```scala
  def reduceByKey(partitioner: Partitioner, func: (V, V) => V): RDD[(K, V)] = self.withScope {
      combineByKeyWithClassTag[V]((v: V) => v, func, func, partitioner)
  }
  // 传入分区数，使用 HashPartitioner 分区器
  def reduceByKey(func: (V, V) => V, numPartitions: Int): RDD[(K, V)] = self.withScope {
      reduceByKey(new HashPartitioner(numPartitions), func)
  }
  // 使用默认分区器 defaultPartitioner
  def reduceByKey(func: (V, V) => V): RDD[(K, V)] = self.withScope {
      reduceByKey(defaultPartitioner(self), func)
  }
  ```

* **功能**

  使用关联和交换 reduce 函数合并每个键的值，会执行 map 端聚合。

* **使用场景**

  统计词频

* **拓展**

  reduceByKey 可由 combineByKey 生成，具体见上述源码实现。

* **案例**

#### groupByKey

* **函数**

  ```scala
  // groupByKey 不应该使用 map 端聚合，因为 map 端聚合并不能减少 shuffle 的数据量，同时会将数据插入 hash 表，导致更多对象进入老年代
  def groupByKey(partitioner: Partitioner): RDD[(K, Iterable[V])] = self.withScope {
      val createCombiner = (v: V) => CompactBuffer(v)
      val mergeValue = (buf: CompactBuffer[V], v: V) => buf += v
      val mergeCombiners = (c1: CompactBuffer[V], c2: CompactBuffer[V]) => c1 ++= c2
      val bufs = combineByKeyWithClassTag[CompactBuffer[V]](
        createCombiner, mergeValue, mergeCombiners, partitioner, mapSideCombine = false)
      bufs.asInstanceOf[RDD[(K, Iterable[V])]]
  }
  // 传入分区数，使用 HashPartitioner 分区器
  def groupByKey(numPartitions: Int): RDD[(K, Iterable[V])] = self.withScope {
      groupByKey(new HashPartitioner(numPartitions))
  }
  // 使用默认分区器 defaultPartitioner
  def groupByKey(): RDD[(K, Iterable[V])] = self.withScope {
      groupByKey(defaultPartitioner(self))
  }
  ```

* **功能**

  将 RDD 中每个键的值分组到单个序列中。

* **使用场景**

  分组操作

* **拓展**

  groupByKey 可由 combineByKey 生成，具体见上述源码实现。

* **案例**

#### aggregateByKey

* **函数**

  ```scala
  // 使用给定的combine函数和中立的“零值”聚合每个键的值。这个函数可以返回与RDD V中值类型不同的结果类型U。因此，我们需要一个操作来合并一个V到一个U，以及一个操作来合并两个U，如scala.TraversableOnce中所述。前者操作用于合并分区内的值，后者用于合并分区之间的值。为了避免内存分配，这两个函数都允许修改并返回它们的第一个参数，而不是创建一个新的U
  def aggregateByKey[U: ClassTag](zeroValue: U, partitioner: Partitioner)(seqOp: (U, V) => U,
        combOp: (U, U) => U): RDD[(K, U)] = self.withScope {
      // Serialize the zero value to a byte array so that we can get a new clone of it on each key
      val zeroBuffer = SparkEnv.get.serializer.newInstance().serialize(zeroValue)
      val zeroArray = new Array[Byte](zeroBuffer.limit)
      zeroBuffer.get(zeroArray)
  
      lazy val cachedSerializer = SparkEnv.get.serializer.newInstance()
      val createZero = () => cachedSerializer.deserialize[U](ByteBuffer.wrap(zeroArray))
  
      // We will clean the combiner closure later in `combineByKey`
      val cleanedSeqOp = self.context.clean(seqOp)
      combineByKeyWithClassTag[U]((v: V) => cleanedSeqOp(createZero(), v),
        cleanedSeqOp, combOp, partitioner)
      // TODO ... DELETE 上述操作的本质
      // combineByKey(V => U, (U, V) => U, (U, U) => U)
    }
  ```

* **功能**

  使用给定的combine函数和中立的“零值”聚合每个键的值。

* **使用场景**

  每个分组只会创建一次 zeroValue，同一分组的后续数据会复用该 zeroValue，避免重复创建。

* **拓展**

  aggregateByKey 可由 combineByKey 生成，具体见上述源码实现。

* **案例**

  ```scala
  // 获取用户访问过去重后的所有站点，
  // 元组的第一个元素为用户 ID，元组的第二个用户访问网站
  val userAccesses = sc.sc.parallelize(Array("u1", "site1"), ("u2", "site1"), ("u1", "site1"), ("u2", "site3"), ("u2", "site4")))
  // groupByKey 运算量大，可选择 reduceByKey 和 aggregateByKey
  // reduceByKey，RDD 的每个值都创建一个 Set，对垃圾回收造成压力
  val mapedUserAccess = userAccesses.map(userSite => (userSite._1, Set(userSite._2)))
  val distinctSite = mapedUserAccess.reduceByKey(_++_)
  // 使用 aggregateByKey 优化
  val zeroValue = collection.mutable.set[String]()
  val aggregated = userAccesses.aggregateByKey(zeroValue)((set, v) => set += v, (setOne, setTwo) => setOne ++= setTwo)
  ```

#### distinct

* **函数**

  ```scala
  def distinct(numPartitions: Int)(implicit ord: Ordering[T] = null): RDD[T] =  withScope {
      map(x => (x, null)).reduceByKey((x, y) => x, numPartitions).map(_._1)
  }
  def distinct(): RDD[T] = withScope {
      distinct(partitions.length)
  }
  ```

* **功能**

  返回当前 RDD 去重元素后的新 RDD。

* **使用场景**

* **拓展**

* **案例**

### 2.5 连接类

#### cogroup

* **函数**

  ```scala
  def cogroup[W](other: RDD[(K, W)], partitioner: Partitioner)
        : RDD[(K, (Iterable[V], Iterable[W]))] = self.withScope {
      if (partitioner.isInstanceOf[HashPartitioner] && keyClass.isArray) {
        throw new SparkException("HashPartitioner cannot partition array keys.")
      }
      val cg = new CoGroupedRDD[K](Seq(self, other), partitioner)
      cg.mapValues { case Array(vs, w1s) =>
        (vs.asInstanceOf[Iterable[V]], w1s.asInstanceOf[Iterable[W]])
      }
  }
  ```

* **功能**

  对于 'this' 和 ‘other’ 中的每个 key，返回一个结果 RDD，其中包含 'this' 和 'other' 中该键值列表的元组。

* **使用场景**

* **拓展**

  cogroup 可实现自定义的关联逻辑，例如 join。

* **案例**

  ```scala
  import org.apache.spark.rdd.RDD
  val kv1: RDD[(String, Int)] = spark.sparkContext.parallelize(List(
        ("zhangsan", 11),
        ("zhangsan", 12),
        ("lisi", 13),
        ("wangwu", 14)
      ))
  val kv2: RDD[(String, Int)] = sc.parallelize(List(
        ("zhangsan", 21),
        ("zhangsan", 22),
        ("lisi", 23),
        ("zhaoliu", 28)
      ))
  val cogroup: RDD[(String, (Iterable[Int], Iterable[Int]))] = kv1.cogroup(kv2)
  cogroup.foreach(println)
  /* 结果
  (zhangsan, (CompactBuffer(11, 12), CompactBuffer(21, 22)))
  (wangwu, (CompactBuffer(14), CompactBuffer()))
  (zhaoliu, (CompactBuffer(), CompactBuffer(28)))
  (lisi, (CompactBuffer(13), CompactBuffer(23)))
  */
  ```

#### join

* **函数**

  ```scala
  def join[W](other: RDD[(K, W)], partitioner: Partitioner): RDD[(K, (V, W))] = self.withScope {
      this.cogroup(other, partitioner).flatMapValues( pair =>
        for (v <- pair._1.iterator; w <- pair._2.iterator) yield (v, w)
      )
  }
  // 传入分区数，使用 HashPartitioner 分区器
  def join[W](other: RDD[(K, W)], numPartitions: Int): RDD[(K, (V, W))] = self.withScope {
      join(other, new HashPartitioner(numPartitions))
  }
  // 使用默认分区器 defaultPartitioner
  def join[W](other: RDD[(K, W)]): RDD[(K, (V, W))] = self.withScope {
      join(other, defaultPartitioner(self, other))
  }
  ```

* **功能**

  返回一个包含所有在 'this' 和 'other' 匹配上的 key 的元素的 pair 的 RDD。RDD 中的元素的每个 pair 将作为一个 (k, (v1, v2)) 元组返回，其中 (k, v1) 在 'this' 中，(k, v2) 在 'other' 中。

* **使用场景**

* **拓展**

  join 的底层实现是 cogroup。

* **案例**

### 2.6 数据集操作类

#### cartesian

* **函数**

  ```scala
  def catesian[U: ClassTag](other: RDD[U]): RDD[(T, U)] = withScope {
      new CartesianRDD(sc, rdd1 = this, other)
  }
  ```

* **功能**

  返回该 RDD 和 other RDD 笛卡尔积的结果，即所有元素 (a, b) 的 RDD，其中 a 在 'this' 中，b 在 ‘other’中。

* **使用场景**

* **拓展**

* **案例**

#### intersection

* **函数**

  ```scala
  def intersection(other: RDD[T]): RDD[T] = withScope {
      this.map(v => (v, null)).cogroup(other.map(v => (v, null)))
  	    .filter { case (_, (leftGroup, rightGroup)) => leftGroup.nonEmpty && rightGroup.nonEmpty }.keys
  }
  ```

* **功能**

  返回该 RDD 和 other RDD 的交集。输出将不包含任何元素，即使输入的 RDDs 有重复元素。

* **使用场景**

* **拓展**

* **案例**

#### subtract

* **函数**

  ```scala
  def subtract(
  	ohter: RDD[T],	
  	p: Partitioner)(implicit ord: Ordering[T] = null): RDD[T] = withScope {
      if(partitioner == Some(p)) {
          // 我们的 partitioner 知道如何处理 T 类型（因为我们有一个 partitioner 是真正的 (K, V) 类型），因此创建一个新的用于分离我们假的 tuple 的 partitioner
          val p2 = new Partitioner() {
              override def numPartitions:Int = p.numPartitions
              override def getPartition(k: Any): Int = p.getPartition(k.asInstanceOf[(Any, _)]._1)
          }
          // 不幸得是，由于我么创建一个新的 p2，所以无论如何我们都会得到 SHuffleDependencies，并且当调用 .keys 时，还没有 partitioner 集合，即使 SubstractedRDD 将由于 p2 的分离 tuple（de-tupled） 的分区，已经通过正确的/真实的 keys 分区了。
          this.map(x => (x, null)).substractByKey(other.map((_, null)), p2).keys
      } else {
          this.map(x => (x, null)).substractByKey(other.map((_, null)), p).keys
      }
  }
  // 传入分区数，使用 HashPartitioner 分区器
  def subtract(other: RDD[T], numPartitions: Int): RDD[T] = withScope {
      subtract(other, new HashPartitioner(numPartitions))
  }
  // 使用当前 RDD 的分区器，如果没有则创建 HashPartitioner
  def subtract(other: RDD[T]): RDD[T] = withScope {
      subtract(other, partitioner.getOrElse(new HashPartitioner(partitions.length)))
  }
  
  // 返回一个 RDD，其 pairs 来自 this 中，但不在 other 中
  def subtractByKey[W: ClassTag](other: RDD[(K, W)], p: Partitioner): RDD[(K, V)] = self.withScope {
      new SubtractedRDD[K, V, W](self, other, p)
  }
  ```

* **功能**

  返回一个 RDD 包含所有在 'this' 但不在 'other' 的元素。

* **使用场景**

* **拓展**

* **案例**

#### union

* 函数

  ```scala
  def union(): RDD[T] = withScope {
      sc.union(first = this, other)
  }
  ```

* **功能**

  返回该 RDD 和另一个 RDD 的并集。任何相同的元素可能会出现多次（可以使用 distinct() 来消除）。

* **使用场景**

* **拓展**

* **案例**

### 2.7 高级算子

#### sample

* **函数**

  ```scala
  def sample(
        withReplacement: Boolean, // 是否将数据放回重新抽取
        fraction: Double, // 抽样的数据所占百分比
        seed: Long = Utils.random.nextLong // 种子
  ): RDD[T] = {
      withScope {
        if (withReplacement) {
          new PartitionwiseSampledRDD[T, T](this, new PoissonSampler[T](fraction), true, seed)
        } else {
          new PartitionwiseSampledRDD[T, T](this, new BernoulliSampler[T](fraction), true, seed) // 伯努利概型
        }
      }
    }
  ```

* **功能**

  返回该 RDD 抽样的子集。

* **使用场景**

  * 抽样计数统计导致数据倾斜的 key
  * 抽样入表数据，对 HBase 表进行预分区

* **拓展**

* **案例**

  ```scala
  val data: RDD[Int] = sc.parallelize(1 to 100)
  data.sample(true, 0.1,).foreach(println)
  ```

## 3 行动类算子

### foreach

* **函数**

  ```scala
  def foreach(f: T => Unit): Unit = withScope {
      sc.runJob(rdd = this, (iter: Iterator[T]) => iter.foreach(cleanF))
  }
  ```

* **功能**

  在 RDD 的每个元素上应用一个 f 函数。该操作会触发一个 job，并返回一个值给用户程序。

* **使用场景**

* **拓展**

* **案例**

### foreachPartitions

* **函数**

  ```scala
  def foreachPartition(f: Iterator[T] => Unit): Unit = withScope {
      sc.runJob(rdd = this, (iter:Iterator[T]) => cleanF(iter))
  }
  ```

* **功能**

  在 RDD 的每个分区上应用一个 f 函数。

* **使用场景**

* **拓展**

  与 mapPartitionis 类似，需要注意 f 函数内部要**正确使用迭代器模式~！！！**。

* **案例**

### collect

* **函数**

  ```scala
  def collect(): Array[T] = withScope {
      val results = sc.runJob(rdd = this, (iter: Iterator[T]) => iter.toArray)
      Array.concat(results: _*)
  }
  ```

* **功能**

  返回一个包含该 RDD 所有元素的数组。

* **使用场景**

  由于 collect 操作会将所有 Executor 进程中的 tasks 的结果全部收集到 Drvier 进程中，**易出现内存溢出的情况**。

* **拓展**

* **案例**

### take

* **函数**

  ```scala
  def take(num: Int): Array[T] = withScope {
      val scaleUpFactor = Math.max(conf.getInt("spark.rdd.limit.scaleUpFactor", 4), 2)
      if(num == 0) {
          new Array[T](0)
      } else {
          val buf = new ArrayBuffer[T]
          val totalParts = this.partitions.length
          var partsScanned = 0
          while(buf.size < num && partsScanned < totalParts) {
              val numPartsToTry = 1L
              if(partsScanned > 0) {
                  if(buf.isEmpty) {
                      numPartsToTry = partsScanned * scaleUpFactor
                  } else {
                      numPartsToTry = Math.max((1.5 * num * partsScanned / buf.size).toInt - partsScanned, 1)
                      numsPartsToTry = Math.min(numsPartsToTry, partsScanned * scaleUpFactor)
                  }
              }
              val left = num - buf.size
              val p = partsScanned.until(math.min(partsScanned + numPartsToTry, totalParts).toInt)
              // 触发 spark 作业
              val res = sc.runJob(rdd = this, (it: Iterator[T]) => it.take(left).toArray, p)
              res.foreach(buff ++= _.take(num - buf.size))
              partsScanned += p.size
          }
      }
  }
  ```

* **功能**

  获取该 RDD 的前 num 个元素。它是如下工作方式，首先扫描一个分区，然后使用分区的结果来估计还需多少额外的分区数来满足 num 需求。

* **使用场景**

  结果数组要尽可能小，因为所有的数据都会被加载到 Driver 的内存中。

* **拓展**

* **案例**

### count

* **函数**

  ```scala
  def cout(): Long = sc.runJob(rdd = this, Utils.getIteratorSize _).sum
  ```

* **功能**

  返回该 RDD 元素的数量。

* **使用场景**

* **拓展**

* **案例**

### saveAsTextFile

* **函数**

  ```scala
  def saveAsTextFile(path: String): Unit = withScope {
      val nullWritableClassTag = implicitly[ClassTag[NullWritable]]
      val textClassTag = implicitly[ClassTag[Text]]
      val r = this.mapPartitions {
          val text = new Text()
          iter.map { x =>
              text.set(x.toString)
              (NullWritable.get(), text)
          }
      }
      RDD.rddToPaairRDDFunctions(r)(nullWritableClassTag, textClassTag, ord = null).saveAsHadoopFile[TextOutputFormat[NullWritable, Text]](path)
  }
  ```

* **功能**

  将该 RDD 保存成 text 文件，使用 string 来代表元素。

* **使用场景**

* **拓展**

* **案例**

## 4 控制类算子

### 4.1 缓存

#### persist

* **函数**

  ```scala
  def persist(newLevel: StorageLevel): this.type = {
      if (isLocallyCheckpointed) {
        // 意味着如果用户之前调用了 localCheckpoint()，它应该已经将此 RDD 标记为持久化。这里，我们应该用用户显示请求的存储级别覆盖旧的存储级别（在调整为磁盘之后）。
        persist(LocalRDDCheckpointData.transformStorageLevel(newLevel), allowOverride = true)
      } else {
        persist(newLevel, allowOverride = false)
      }
  }
  ```

* **功能**

  设置 RDD 的存储级别，以便**第一次计算之后**跨操作持久保存它的值。只有在 RDD 未设置存储级别的情况下，才可以使用它来分配新的存储级别。

* **使用场景**

  多次使用同一 RDD，可进行缓存操作。

* **拓展**

  persist 的存储级别选择：MEMORY_ONLY， MEMORY_ONLY_SER，MEMORY_AND_DISK，不要使用 DISK_ONLY 和带 _2 的存储级别，因为效率偏低。

* **案例**

#### cache

* **函数**

  ```scala
  def cache(): this.type = persist()
  def persist(): this.type = persist(StorageLevel.MEMORY_ONLY)
  ```

* **功能**

  使用默认的存储级别 MEMORY_ONLY 保存该 RDD。

* **使用场景**

  多次使用同一 RDD，可进行缓存操作。

* **拓展**

* **案例**

#### unpersist

* **函数**

  ```scala
  def unpersist(blocking: Boolean = true): this.type = {
      sc.unpersistRDD(id, blocking)
      storageLevel = StorageLevel.NONE
  }
  ```

* **功能**

  设置该 RDD 非缓存，并从内存和磁盘上移除它所有的块（blocks）。

* **使用场景**

  缓存 RDD 后续不使用时，为资源消耗考虑，可以设置非缓存。

* **拓展**

* **案例**

#### checkpoint

* **函数**

  ```scala
  def checkpoint(): Unit = RDDCheckpointData.synchronized {
      if(context.checkpointDir.isEmpty) {
          throw new SparkException("Checkpoint directory has not been set in the SparkContext")
      } else if(checkpointData.isEmpty) {
          checkpointData = Some(new ReliableRDDCheckpointData(this))
      }
  }
  ```

* **功能**

  标记该 RDD 为 checkpointing，它将会被保存在 checkpoint 目录（由 SparkContext#setCheckpointDir 设置）的某个文件中，**其父 RDD 的引用将全被移除**。该方法必须在任何在该 RDD 上执行的任何 job 之前调用。**<font color='red'>强烈建议</font>将 RDD 缓存在内存中，否则将其保存在文件上将会重新计算**。

* **使用场景**

  * 数据血缘关系链太长，且数据获取不易，需要保存在可靠的介质上，供后续使用。
  
* **拓展**

  checkpoint 不会立即触发作业，action 算子触发 runJob 后再触发 checkpoint，而 checkpoint 会单独触发 runJob（**痛点：代码逻辑会重复计算**），因此使用 checkpoint 时需结合 persist 算子使用。

  checkpoint 还是 persist 都有一个共同点，就是那些被重复利用的数据。

* **案例**

### 4.2 重分区

#### coalesce

* **函数**

  ```scala
  def coalesce(numPartitions: Int, shuffle: Boolean = false,
                 partitionCoalescer: Option[PartitionCoalescer] = Option.empty)
                (implicit ord: Ordering[T] = null)
        : RDD[T] = withScope {
      if (shuffle) {
          // 从随机分区开始，在输出分区中均匀地分布元素
        val distributePartition = (index: Int, items: Iterator[T]) => {
          var position = (new Random(index)).nextInt(numPartitions)
          items.map { t =>
              // 注意，key 的散列值仅仅是 key 本身。HashPartitioner 将使用总分区数对它进行取模操作
            position = position + 1
            (position, t)
          }
        } : Iterator[(Int, T)]
          
        // 包含一个 shuffle 步骤，使得上游任务仍然是分散的
        new CoalescedRDD(
          new ShuffledRDD[Int, T, T](mapPartitionsWithIndex(distributePartition),
          new HashPartitioner(numPartitions)),
          numPartitions,
          partitionCoalescer).values
      } else {
        new CoalescedRDD(this, numPartitions, partitionCoalescer)
      }
    }
  ```

* **功能**

  返回一个 reduce 到 numPartitions 个分区的新 RDD。

* **使用场景**

  * 抽样计数统计导致数据倾斜的 key
  * 抽样入表数据，对 HBase 表进行预分区

* **拓展**

  默认情况下，将产生一个窄依赖，因为不会发生 shuffle 过程。例如，RDD 分区数从1000 调整到 100，那么100个新分区中的每个分区将占用10个当前分区。**如果传入一个更大的分区数，那么将保持当前分区数**。

  然而，如果做一个剧烈的合并，将可能导致计算发生在比你期望的更少的节点上。例如，numPartitions = 1 时，计算节点只有一个。为了避免上述情况，可以传入 shuffle = true。

  若 shuffle = true，数据将分散到更多的分区中。例如 RDD 只有少量分区，调用 coalesce(1000, shuffle = true) 将产生 1000 个分区。

  **分布式情况下数据的移动方式**：

  * IO 移动：分区的数据直接读取，不区分对待。
  * Shuffle 移动：每个元素去往的分区可能不同，需要计算分区号，将数据发送到指定分区。

* **案例**

#### repartition

* **函数**

  ```scala
  def repartition(numPartitions: Int) (implicit ord: Ordering[T] = null): RDD[T] = withScope {
      coalesce(numPartitions, shuffle = true)
  }
  ```

* **功能**

  返回有确切 numPartitions 个分区数的 RDD。

* **使用场景**

  RDD 分区数设置不合理的情况下，可以通过 repartition 来调整分区数。

* **拓展**

  如果要减少 RDD 的分区数，考虑使用 coalesce 算子，可以避免执行 shuffle 操作。

* **案例**



# 总结



# 参考文献

[1] [RDD 编程指南，Spark 官网](http://spark.apache.org/docs/latest/rdd-programming-guide.html)