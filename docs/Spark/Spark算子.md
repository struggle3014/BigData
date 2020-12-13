<div align="center"><img src="https://gitee.com/struggle3014/picBed/raw/master/name_code.png"></div>

# 导读

本文对 Spark 常见算子进行总结，方便后续回顾。



# 目录



# 正文



# 总结



# 参考文献



Spark 算子及其原理


算子分类：
	功能划分：
		create - 创建类算子
			SparkContext.textFile
			SparkContext.parallelize
		transform - 转换类算子
			map, flatMap, filter
			排序类
				sortByKey
				sortBy
			聚合类
				reduceByKey
				combineByKey
				groupByKey			
			高级算子
				sample
		control - 控制类算子
			缓存
				cache
				persist
				checkpoint
			重分区
				repartition
				coalesce
		action - 行动类算子
			foreach
			collect
			take
			count

数据集：元素即单元素，K,V 元素，结构化、非结构化
面向数据集的操作：交，并，差，关联，笛卡尔积
面向数据集的 API 分类：基础 API 和复合 API

--------------------转换类算子（transform）---------------------
map
	函数：
		def map[U: ClassTag](f: T => U): RDD][U] = withScope {
			val cleanF = sc.clean(f)
			new MapParetitionsRDD[U, T](this, (context, pid, iter) => iter.map(cleanF))
		}
	功能：对数据做映射操作
	注意：1进1出
	案例：
	
flatMap
	函数：
		def flatMap[U: ClassTag](f: T => Traversable[U]): RDD[U] = withScope {
			val cleanF = sc.clean(f)
			new PartitionsRDD[U, T](this, (context, pid, iter) => iter.flatMap(cleanF))
		}
	功能：对数据做扁平化操作
	注意：1进多出
	案例：
	
filter
	函数：
		def filter(f: T => Boolean): RDD[T] = withScope {
			val cleanF = sc.clean(F)
			new PartitionsRDD[T, T](
				this,
				(context, pid, iter) => iter.filter(cleanF),
				preservesPartitioning = true)
		}
	功能：对数据做过滤操作
	注意：1进1出
	案例：

mapValues
	函数：
		def mapValues[U](f: V => U): RDD[(K, U)] = self.withScope {
			val cleanF = self.context.clean(f)
			new MapParetitionsRDD[(K, U), (K, V)](self,
				(context, pid, iter) => iter.map { case(k, v) => (k, cleanF(v1))},
				preservesPartitioning = true // 保留分区
			)
		}
	功能：
	注意：
	案例：

flatMapValues
	函数：
	功能：
	注意：
	案例：

cogroup	/ 基础 API
函数：def cogrup(other: RDD[(K, W)]): RDD[(K, (Iterable[V], Iterable[W]))]
功能：对于每一个在 this 和 other 中的 key k，返回的结果集 RDD 包含 tuple 的 this 的列和 other 值的列表。
使用：rdd1: RDD[(String, Int)].cogroup(rdd2: RDD[(String, Int)])\
注意：1，cogroup 会产生 shuffle，即相同的 key 为一组（分区器针对每条记录计算要去往的分区）
		2，方法内部，存在一个类似 HashMap 的数据结构，两个迭代器中的数据往里扔
案例：
	
	结果：
	(zhangsan, (CompactBuffer(11, 12), CompactBuffer(21, 22)))
	(wangwu, (CompactBuffer(14), CompactBuffer()))
	(zhaoliu, (CompactBuffer(), CompactBuffer(28)))
	(zhangsan, (CompactBuffer(13), CompactBuffer(23)))

cogroup 派生出的复合 API：
	join
		函数：
			def join(other: RDD[(K, W)], partitioner: Partitioner): RDD[(K, (V, W))] = self.withScope {
				this.cogroup(other, partitioner).flatMapValues( pair =>
					for(v <- pair._1.iterator; w <- pair._2.iterator) yield(v, w)
				)
			}
		功能：
		使用：
		案例：
	leftjoin
	rightjoin
	fulljoin

-----------------------------------------
聚合类算子（aggregator）
combineByKey	调优的关键
	函数：def combineByKey[C](
		createCombiner: V => C, // 第一个元素如何操作
		mergeValues: (C, V) => C, // 后续元素如何操作
		mergeCombiners: (C, C) => C // 多次溢写的结果如何合并（由于内存有限，数据会有写入磁盘现象）
		): RDD[(K, C)] = self.withScope {
		 combineByKeyWithClassTag(createCombiner, mergeValues, mergeCombiners)(null)
	}
	
	功能：相同的 key 聚合在一起

reduceByKey
	函数：
	功能：

groupByKey
	函数：
	功能：相同 key 的记录聚合在一起，列转行的过程
	注意：其中会涉及 shuffle 操作
	案例：
		val data: RDD[(String, Int)] = sc.parallelize(List(("zhangsan", 234), ("zhangsan", 567), ("lisi", 44)));
		// 1，相同的 key 为一组，多列转行
		val group: RDD[(String, Iterable[Int])] = data.groupByKey()
		group.foreach(println)
		// (zhangsan, CompactBuffer(234, 567))
		// (lisi, CompactBuffer(44))
		// 2，列转行，即 groupByKey 的逆过程；一进多出
		val res: RDD[(String, Int)] = group.flatMap(e => e._2.map(x => (e.1, x)))
		// 3，针对 group 中的 values 做扁平化操作，功能与步骤2的功能相同
		group.flatMapValues()

reduceByKey 和 groupByKey 的对比
	相同点：底层都调用 combineByKey
	不同点：reduceByKey 默认会进行 map 端的聚合，可以减少磁盘IO 和网络 IO。

sum，count，min，max，avg
	sum 
		功能：加和操作
		案例：
			val data:  RDD[(String, Int)] = null
			val sum: RDD[(String, Int)] = data.reduceByKey(_+_)
			sum.foreach(println)
	max
		功能：求最大值
		案例：
			val data:  RDD[(String, Int)] = null
			val max: RDD[(Streing, Int)] = data.reduceByKey((oldV, newV) => if(oldV > newV) oldV else newV)
			max.foreach(println)
	min
		功能：求最小值
		案例：
			val data:  RDD[(String, Int)] = null
			val min: RDD[(Streing, Int)] = data.reduceByKey((oldV, newV) => if(oldV > newV) newV else oldV)
			min.foreach(println)
	count
		功能：计数
		案例：
			val data:  RDD[(String, Int)] = null
			val count: RDD[(Streing, Int)] = data.mapValues(e => 1).reduceByKey(_+_)
			count.foreach(println)
	avg
		功能：计算平均值
		案例：
			val tmp: RDD[(String, (Int, Int))] = sum.join(count)
			val avg = tmp.mapValues(x => x.1 / x.2)
			avg.foreach(println)
		注意：avg 的操作涉及对同一数据集做两次计算的操作得到 sum 和 join，其次解决了 sum 和 count 数据相遇的问题，再计算平均值
		优化：如何使得只计算一次，就可以做到对 sum 和 count 的操作。
		val tmp = data.combineByKey(
			// createCombiner: V => C	第一条记录的 value 如何放入 hashMap 中
			(value: Int) => (value, 1),
			// mergeValue: (C, V) => C	如果有第二条记录，第二条及以后的 value 如何放入 hashMap，其中 key 为加总和，value 为元素个数。
			(oldValue: (Int, Int), newValue: Int) => (oldValue._1 + newValue, oldValue._2 + 1),
			// mergeCombiners: (C, C) => C	合并溢写结果的函数
			(v1: (Int, Int), v2: (Int, Int)) => (v1._1 + v2._1, v1._2 + v2._2)
		)
		tmp.mapValues(_.1/_.2).foreach(println)

-----------------------------------------
分区类算子（partition）

组 group 和分区 partition 是不同的概念
迭代器模式的正确使用~！
map
mapPartition
	功能：
	案例：
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
	案例分析：该案例未正确使用迭代器模式，会造成数据的积压，可能会造成内存溢出问题。
	案例优化：
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
	案例思考：上述案例优化中是一进一出的场景，如果是一进多出的场景呢？可以参考 Scala flatMap 源码，需要再额外创建迭代器。

------------------高级算子-----------------------

sample
	函数： 
		withReplacement 是否将数据放入重新抽取
		fraction 抽取的数据占百分比例
		seed 种子
		def sample(withReplacement: Boolean, fraction: Double, seed: Long = Utils.random.nextLong): RDD[T] = {
			withScope {
				if(withReplacement) {
					new PartitionwiseSampledRDD[T, T](this, new PoissonSampler[T](fraction), true, seed)
				} else {
					new PartitionwiseSampledRDD[T, T](this, new BernoulliSampler[T](fraction), true, seed)
				}
			}
		}
	功能：从 RDD 中随机抽取一批数据
	案例：
		val data: RDD[Int] = sc.parallelize(1 to 100)
		data.sample(true, 0.1,).foreach(println)

repartition
	函数：
		def repartition(numPartitions: Int)(implicit ord: Ordering[T] = null): RDD[T] = withScope {
			coalesce(numPartitions, shuffle = true)
		}
	功能：对 RDD 进行重分区操作
	注意：repartition 会产生 shuffle 操作。
	案例：
		val data: RDD[Int] = sc.parallelize(1 to 10,5)
		// 将数据对应的分区号给标记上
		val data1: RDD[(Int, Int)] = data.mapPartitionsWithIndex(
			(pIndex, pIter) => {
				pIter.map(e => (pIndex, pIter=))
			}
		)
		// 重分区操作
		val repartition: RDD[(Int, Int)] = data1.repartition(3)
		val result: RDD[(Int, Int)] = repartition.mapPartitionsWithIndex(
			(pIndex, pIter) => {
				pIter.map(e => (pIndex, pIter))
			}
		)
	
coalesce
	函数：
	功能：重分区（涉及到 Spark 优化，要充分利用内存和 CPU）
	案例：
		val data: RDD[Int] = sc.parallelize(1 to 10,5)
	    // 将数据对应的分区号给标记上
        val data1: RDD[(Int, Int)] = data.mapPartitionsWithIndex(
        	(pIndex, pIter) => {
        		pIter.map(e => (pIndex, pIter=))
        	}
        )
        // 重分区操作
        val repartition: RDD[(Int, Int)] = data1.coalesce(3, false)
        val result: RDD[(Int, Int)] = repartition.mapPartitionsWithIndex(
        	(pIndex, pIter) => {
        		pIter.map(e => (pIndex, pIter))
        	}
        )
	注意：1，若 RDD 的 coalesce 后的分区数大于 RDD 之前的分区数。存在 shuffle 过程，因为必然会涉及到对记录计算分区，并重新去到对应分区，否则无法获取需要去往的分区。
	2，若 RDD 的 coalesce 后的分区数小于 RDD 之前的分区数，且对应的 shuffle 设置为 false，则不会产生 shuffle 过程。
	分布式情况下数据的移动方式：
		1，IO 移动：分区的数据直接读取即可，不区分对待。
		2，Shuffle 移动： 如果分区中每条元素去往的后续分区不一样，那么对记录计算分区号，并将数据发送到指定分区。

------------------实战案例-----------------------
使用 mapValues 或 flatMapValues 算子优化，避免不必要的 shuffle 操作
	案例：
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
	注意：
		若 key 没有发生变化
		分区器没有发生变化
		分区数没有发生变化
		那么建议使用 mapValues 或 flatMapValues，可以 combineByKey 算子中会判断分区器是否相同，
			若分区器相同，则不走 shuffle 依赖，否则会走 shuffle 依赖。