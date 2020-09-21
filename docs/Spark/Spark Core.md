<div align="center"><img src="https://gitee.com/struggle3014/picBed/raw/master/name_code.png"></div>

# 导读

本文主要简介了 Spark Core 相关的内容。主要从 RDD 和 共享变量两个点出发。在 RDD 部分，我们从 RDD 简介，RDD 特点，RDD 创建及 RDD 操作四个方面做介绍；在共享变量部分，我们介绍了广播变量和累加器。开启 Spark Core 之旅吧。

***持续更新中~***



# 目录

[TOC]

# 正文

## 1 RDD

### 1.1  RDD 简介

Spark 围绕弹性分布式数据集（RDD）的概念展开，RDD 是一组以并行操作的元素的容错集合。

RDD 是弹性分布式数据集（Resilient Distributed Datasets）简写，是分布式内存抽象，表示一个只读的记录分区的集合。RDD 支持丰富的转换操作，因而，RDDs 之间具有互相依赖关系，基于此依赖关系，RDDs 会行程一个有向无环图 DAG，该 DAG 描述了整个计算的过程。同时，由于 RDDs 之间是存在`血缘（lineage）`关系，所以即使出现分区丢失的情况，也可以通过血缘关系重建分区。



### 1.2 RDD 特点

RDD 表示只读的分区的数据集，对 RDD 进行改动，只能通过RDD的转换操作，由一个 RDD
得到一个新的 RDD，新的 RDD 包含了从其他 RDD 衍生所必需的信息。
RDDs之间存在依赖，RDD 的执行是按照血缘关系延时计算的。**如果血缘关系较长，可以
通过持久化 RDD 来切断血缘关系**。

RDD 具有如下特点：

* 分区
* 只读
* 依赖
* 缓存
* checkpoint

#### 1.2.1 分区

![rdd-partition](https://gitee.com/struggle3014/picBed/raw/master/rdd-partition.png)

RDD 逻辑上是分区的，每个分区的数据是抽象存在的，计算的时候会通过一个 `compute函数` 得到每个分区的数据。如果 RDD 是通过已有的文件系统构建，则 `compute 函数`是读取指定文件系统中的数据，如果RDD是通过其他RDD转换而来，则 `compute 函数` 是执行转换逻辑将其他 RDD 的数据进行转换。



#### 1.2.2 只读

![rdd-readonly](https://gitee.com/struggle3014/picBed/raw/master/rdd-readonly.png)

RDD 是**只读**的，要想改变 RDD 中的数据，只能在现有的 RDD 基础上创建新的 RDD。



#### 1.2.3 依赖

RDDs 通过操作算子进行转换，转换得到的新 RDD 包含了从其他 RDDs 衍生所必需的信息，
RDDs 之间维护着这种血缘关系，也称之为依赖。

##### 1.2.3.1 依赖分类

	* 宽依赖
	* 窄依赖

##### 1.2.3.2 宽窄依赖划分

从父 RDD 的 partition 被使用的个数来定义窄依赖和宽依赖。若父 RDD 的一个 Partition 被子RDD 的一个 partition 所使用就是窄依赖，否则的话就是宽依赖。

![rdd-narrow-wide](https://gitee.com/struggle3014/picBed/raw/master/rdd-narrow-wide.png)

![rdd-lineage](https://gitee.com/struggle3014/picBed/raw/master/rdd-lineage.png)

#### 1.2.4 缓存

Spark 中最重要的功能之一是跨操作在内存中的持久化（或缓存）数据集。当你持久化（persist）一个 RDD 时，每个节点在内存中存储它计算的任何分区，并在该数据集（或从中派生的数据集）的其他操作中重用它们。这使得将来的操作要快很多（通常超过10倍）。缓存是迭代算法和快速交互使用的关键工具。

可以使用 `persist()` 或 `cache()` 方法将 RDD 标记为持久化的。在第一次操作中国计算它时，将它保存在节点的内存中。Spark 的缓存是容错的—如果一个 RDD 的分区丢失了，它将使用最初创建它的转换自动重新计算。

此外，每个持久化的 RDD 都可以使用不同的存储级别存储。例如，允许你将数据集持久化到磁盘上，将数据集持久化到内存上，但作为序列化的 Java 对象（为了节省空间），可以跨节点复制数据集。通过传递一个 StorageLevel 对象给 persist()。cache() 方法是使用偶人存储级别的简写，即 StorageLevel.MEMORY_ONLY（将反序列化的对象存储在内存中）。完整的存储级别的设置为：

| 存储级别                                | 含义                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| MEMORY_ONLY                             | 将 RDD 作为反序列化的 Java 对象存储在 JVM 中。如果内存容纳不下 RDD，那么一些分区不会被缓存，而是在需要它们时动态地重新计算。这是默认级别。 |
| MEMORY_AND_DISK                         | 将 RDD 作为反序列化的 Java 对象存储在 JVM 中。如果 RDD 不适合存储，那么将内存容纳不下的分区存储在磁盘上，并在需要时从磁盘上读取。 |
| MEMORY_ONLY_SER<br>(Java and Scala)     | 将 RDD 存储为序列化的 Java 对象（每个分区一个字节数组）。这通常比反序列化对象更节省空间，特别是使用快速序列化器的时，但读取时需要更多 CPU。 |
| MEMORY_AND_DISK_SER<br>(Java and Scala) | 类似于 MEMORY_ONLY_SER，但将内存不能容纳的分区溢写（spill）到磁盘上，而不是每次都需要动态地计算。 |
| DISK_ONLY                               | 将 RDD 分区仅存储在磁盘上。                                  |
| MEMORY_ONLY_2,<br>MEMORY_AND_DISK_2 等  | 与上面的级别类似，但是在两个集群节点上复制每个分区。         |
| OFF_HEAP（试验）                        | 类似于 MEMORY_ONLY_SER，但将数据储存在堆外内存。这需要启用堆外内存。 |

**注意：** *在 Python 中，存储的对象将始终使用 Pickle 库进行序列化，因此是否选择序列化级别并不重要。Python 中可用的存储级别包括 `MEMORY_ONLY、MEMORY_ONLY_2、MEMORY_AND_DISK、MEMORY_AND_DISK_2、DISK_ONLY`和 `DISK_ONLY_2`*。

甚至在没有用户调用 `persist` 的情况下，Spark 也会自动持久化一些洗牌操作中的中间数据（例如 `reduceByKey`）。这样做是为了避免在节点转移期间重新计算整个输入。我们仍然建议用户在计划重用结果 RDD 时调用 persist。

##### 1.2.4.1 选用哪种存储等级

Spark 的存储级别意味着在内存使用和 CPU 效率之间提供不同的权衡。我们建议通过以下步骤来选择一个：

* 如果在使用默认存储级别（`MEMORY_ONLY`），内存能够很好地容纳 RDDs ，保持这种方式。这是 CPU 效率最高的选项，允许 RDDs 的操作尽可能快的运行。

* 如果不行，尝试使用 `MEMORY_ONLY_SER` 并[选择一个快速序列化库](http://spark.apache.org/docs/latest/tuning.html)，使得对象更节省空间，但访问速度仍然相当快。（Java 和 Scala）

* 除非计算数据集的函数非常昂贵，或者它们过滤了大量数据，否则不要溢写（spill）到磁盘。否则，重新计算分区的速度可能与从磁盘读取分区的速速一样快。

* 如果需要快速地故障恢复，请使用复制的存储级别（例如，如果使用 Spark 为来自 web 应用程序的请求提供服务）。通过重新计算丢失的数据，所有存储级别都提供了完整的容错能力，但是复制的存储级别允许你在 RDD 上继续运行任务，而不必等待重新计算丢失的分区。


##### 1.2.4.2 移除数据

Spark 自动监视每个节点上的缓存使用情况，并以最近最少使用（LRU）的方式删除旧的数据分区。如果你希望手动删除一个 RDD，而不是等待它从缓存中小时，那么可以使用 RDD.unpersist() 方法。

##### 1.2.4.3 缓存例子

如果应用程序中多次使用到同一个 RDD，可以将该 RDD 缓存起来，该 RDD 只有在第一次
计算的时候会根据血缘关系得到分区的数据，在后续其他地方用到该 RDD 时，会直接从
缓存处取而不用再根据血缘关系计算，这样就加速后期的重用。

![rdd-cache](https://gitee.com/struggle3014/picBed/raw/master/rdd-cache.png)

如上图所示，RDD-1 经过一系列的转换后得到 RDD-n 并将结果保存到 HDFS。RDD-1 在这个过程中会有一个中间结果，如果将其缓存到内存，那么随后的 RDD-1 转换到 RDD-m 这一过程，就不会计算其之前的 RDD-0 了。

##### 1.2.5 checkpoint

RDD 的血缘关系天然地可以<font color="red">实现容错</font>，当 RDD 的某个分区数据失败或丢失，可以
通过血缘关系重建。但是对于长时间迭代型应用来说，随着迭代的进行，RDDs 之
间的血缘关系会越来越长，一旦后续迭代过程出错，需要通过非常长的血缘关系
取重建，势必会影响性能。
为此 RDD 支持 checkpoint 将数据保存到持久化的存储中，这样就可以切断之前的
血缘关系，因为 checkpoint 后的 RDD 不需要知道它的父 RDDs，它可以从 checkpoint
 处拿数据。



### 1.3 RDD 创建

创建 RDD 有两种方式：

* 在 driver 程序中 `并行化（parallelizing）` 现有的集合。

* 引用外部存储系统中的数据源，如共享文件系统，HDFS，HBase 或者任何其他 Hadoop InputFormat 的数据源。

#### 1.3.1 并行化集合

我们以 Scala 代码为例，演示 RDD 的创建过程。

`并行化（parallelizing）` 现有的集合

可以通过使用 `SparkContext` 的 `parallelize` 方法在 driver 程序创建并行化集合。

```scala
# 数字 1-5 的数组
val data = Array(1, 2, 3, 4, 5)
# 依据现有的数组，创建并行化的集合
val distData = sc.parallelize(data)
```

在创建完成后，可以进行后续并行化操作。例如，调用 `disData.reduce((a, b) => a+ b)`。Spark 将会在集群的每个分区运行一个任务。正常，Spark 会基于集群自动地设置分区数量。同样，我们也可以手动设置，如

```scala
sc.parallelize(data, 10)
```



#### 1.3.2 外部 Datasets

我们以 Scala 代码为例，演示 RDD 的创建过程。

引用外部存储系统中的数据源

Text file RDDs 可以使用 `SparkContext` 的 `textFile` 方法创建。该方法使用 URI （或者是本机地址，或者 hdfs://，s3a:// 等）作为参数，将所有行读成集合。

```scala
scala> val distFile = sc.textFile("data.txt")
distFile:org.apache.spark.rdd.RDD[String] = data.txt.MapPartitionsRDD[10] at textFile at <console>:26
```

使用 Spark 读文件注意点：

  - 如果使用本地文件系统上的路径，则该文件必须在所有 worker 节点相同路径可以访问。要么，将文件复制到所有的工作节点，或者，使用挂载在网络上的共享文件系统。
  - Spark 所有基于文件的输入方法（包括 textFile）都在目录、压缩文件和通配符上运行。
  - textFile 方法接收一个可选的第二参数，用于控制文件的分区数量。默认情况下，Spark 为文件的每块创建一个分区（在 HDFS 中，块的默认大小为 128MB），但是你也可以通过传递更大的值来请求更多的参数。注意，分区不能少于块。



除了 text 文件，Spark Scala API 也支持一些其他的数据格式：

* `SparkContext.wholeTextFiles` 允许你读取包含很多小 text 文件的目录，返回  (文件名, 内容) 对。这与 textFile 相反，textFile在每个文件中每行返回一条记录。分区数由数据位置决定，在某些情况下，数据位置可能导致分区太少。对于这些情况，wholeTextFiles 提供了一个可选的第二个参数，用于控制最小的分区数量。
* 对于 [SequenceFiles](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/mapred/SequenceFileInputFormat.html)，使用 SparkContext 的 sequenceFile[K, V] 方法，其中 K 和 V 是文件中的键和值的类型。这些应该是 Hadoop 的 [Writable](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/io/Writable.html) 接口的子类，如 [IntWritable](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/io/IntWritable.html) 和 [Text](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/io/Text.html)。此外，Spark 允许你为一些常见的 Writables 指定本地类型。例如， `sequenceFile[Int, String]` 将自动读取 IntWritable 和文本。
* 对于其他 Hadoop 输入格式，可以使用 `SparkContext.hadoopRDD` 方法。
* `RDD.saveAsObjectFile` 和 `SparkContext.objectFile` 支持一种由序列化的 Java 对象组成的简单格式保存的 RDD。虽然这不如 Avro这样的专门格式有效，但它提供了一种简单的方法来保存 RDD。



### 1.4 RDD 操作

RDDs 支持两种类型的操作：

* 转换操作

  从现有数据集创建新数据集

* 行动操作

  在数据集上进行计算并将结果返回 driver 进程。

Spark 中所有的转换操作都是惰性的，它们不会立即计算结果。只有当需要将结果返回给驱动程序时，才会计算转换。

默认情况下，你对每个转换后的 RDD 进行操作时，都可以重新计算它。但是，也可以使用 persist（或 cache） 方法将 RDDD 持久化到内存中。这样，Spark 会将元素保存在集群中，以便下一次查询时候更快地访问它。同时，还支持在磁盘上存储 RDDs，或跨多个节点复制 RDDs。



#### 1.4.1 基础

为了说明 RDD 基础，考虑下面简单的代码：

```scala
// 定义 RDD。基于外部文件。该数据集没有加载到内存中，也没有在其他地方执行： lines 只是指向文件的指针。
val lines = sc.textFile("data.txt")
// 将 lineLengths 定义为映射转换的结果。同样，由于惰性，lineLengths 不会立即执行。
val lineLengths = lines.map(s => s.length)
// 执行 reduce，该操作是行动操作。此时，Spark 将计算分解为在不同的机器上运行的结果，每台机器都运行其部分映射和局部约简，只向驱动程序返回答案。
val totalLength = lineLengths.reduce((a, b) => a+b)
// 如果后面我们想再次使用 lineLengths，我们可以在其后加上 lineLengths.persist()
```



#### 1.4.2 给 Spark 传递方法

Spark API 严重依赖驱动程序（driver program）中传递的在集群上运行的函数。有两种推荐的方法：

* [匿名函数语法](http://docs.scala-lang.org/tour/basics.html#functions)，用于短代码片

* 全局代理对象的静态方法。例如，你可以定义 object Functions，然后传递 MyFunctions.func1：

  ```scala
  object MyFunctions {
      def func1(s: String): String = {...}
  }
  
  myRdd.map(MyFunctions.func1)
  ```

注意，虽然也可以将引用传递给类实例中的方法（与单例对象相反），但这需要同时发送包含该类的对象和方法。例如，考虑：

```scala
class MyClass {
    def func1(s: String): String = {...}
    def doStuff(rdd: RDD[String]): RDD[String] = {rdd.map(func1)}
}
```

这里，如果我们创建一个新的 MyClass 实例并在其上调用 doStuff，那么其中的映射将引用 MyClass 实例的 fun1 方法，因此需要将整个对象发送到集群。这类似于编写 rdd.map(x => this.func1(x))。

以类似的方式，访问外部对象的字段将引用整个对象：

```scala
class MyClass {
    val field = "Hello"
    def doStuff(rdd: RDD[String]): RDD[String] = { rdd.map(x => field + x)}
}
```

等同于编写 rdd.map(x => this.filed + x)，这引用了所有这些。为了避免这个问题，最简单的方法是将字段复制到一个局部变量中，而不是从外部访问它：

```scala
def doStuff(rdd: RDD[String]): RDD[String] = {
    val field_ = this.field
    rdd.map(x => field_ + x)
}
```



#### 1.4.3 理解闭包

Spark 的难点之一是理解跨集群执行代码时变量和方法的范围和声明周期。在范围之外修改变量的 RDD 操作可能经常引起混淆。在下面的示例中，我们将查看使用 foreach() 来增加计数器的代码，但是其他操作也可能出现类似的问题。

**例子**

考虑下面简单的 RDD 元素 sum，它的行为可能会根据是否在相同的 JVM 中执行而有所不同。一个常见的例子是，在本地模式下运行 Spark（--master = local[n]） 与 Spark 应用程序部署到集群（例如，通过 Spark -submit to YARN）：

```scala
var counter = 0
var rdd = sc.parallelize(data)

// 错误做法
rdd.foreach(x => counter += x)

println("Counter value:" + counter)
```



**本地模式和集群模式**

上述代码的行为是未定义的，可能无法按预期工作。为了执行作业，Spark 将 RDD 操作的处理分解为任务，每个任务由执行程序执行。在执行之前，Spark 计算任务的闭包。闭包是哪些执行程序在 RDD 上执行其计算时必须可见的变量和方法（在本例中为 foreach()）。这个闭包被序列化并发送给每个执行器。

闭包中发送给每个执行器的变量现在都是副本，因此，当在 foreach 函数中引用 **counter ** 时，它不再是驱动节点上的计数器。在驱动节点的内存中仍然有一个计数器，但它对执行器不再可见！执行器只看到来自序列化闭包的副本。因此，**counter** 的最终值仍然是零，因为 **counter** 上的所有操作都引用了闭包的副本。因此，**counter** 的最终值仍然为零，因为  **counter ** 上的所有操作都引用额序列化闭包中的值。

在本地模式下，在某些情况下，foreach 的函数实际上会在与驱动程序相同的 JVM 中执行，并引用相同的原始计数器，并可能实际更新它。

为了确保这类场景中定义良好的行为，应该使用累加器。Spark 中的累加器专门用于提供一种机制，以便在集群中的工作节点之间执行分割时安全地更新变量。在累加器部分更加仔细讨论了这些。

一般来说，像循环或局部定义方法这样的闭包结构不应该用来改变全局变量。Spark 不定义或保证闭包外部引用的对象的突变行为。一些这样做得代码可能在本地模式下工作，但那只是偶然的，而且这样的代码在分布式模式下不会像预期的那样工作。如果需要全局聚合，则使用累加器。

**打印 RDD 元素**

另一个常见的习惯用法是尝试使用 `rdd.foreach(println)` 或 `rdd.map(println)` 打印出 RDD 的元素。在一台机器上，这将生成预期的输出并打印出所有的 RDD 元素。但是，在集群模式下，执行器调用的 stdout 输出现在是写入执行器的 stdout，而不是写入驱动程序上的 stout，所以驱动程序上的 stout 不会显示这些！要打印驱动程序上的所有元素，可以使用 `collect()` 方法首先将 RDD 带到驱动程序节点，如下所示：`rdd.collect().foreach(println)`。但是，这可能导致驱动程序耗尽内存，因为 collect() 将整个 RDD 提到一台机器上；如果只打印 RDD 的几个元素，更安全的方式是使用 `take()`:`rdd.take(100).foreach(println)`。



#### 1.4.4 使用 Key-Value 对

虽然大多数 Spark 操作在包含任何类型对象的 RDDs 上的工作，但是只有少数操作在键值对的 RDDs 上可用。最常见的是分布式“洗牌”操作，如按键分组或聚合元素。

在 Scala 中，这些操作在包含 Tuple2 对象的 RDDs 上自动可用（Tuple2 对象是语言中内置的元祖，只需编写 (a, b) 即可创建）。键值对操作在 PairRDDFunctions 类中是可用的，它自动包装元组的 RDD。

例如，使用键值对的 reduceByKey 操作来计算在一个文件中每行文本出现的次数：

```scala
val lines = sc.textFilee("data.txt")
val pairs = lines.map(s => (s, 1))
val counts = pairs.reduceByKey((a, b) => a + b)
```

我们也可以使用 counts.sortByKey()，例如，按字母顺序排序键值对，最后使用 counts.collect() 来将他们作为对象数组返回到驱动程序。

**注意：**在键值对操作中使用自定义对象作为键时，必须确保自定义 equals() 方法附带匹配的 hashCode() 方法。更多细节，查看 [Object.hashCode()](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#hashCode--)



#### 1.4.5 转换操作

下表是 Spark 支持的常见的转化操作。更多细节，查看 RDD API（[scala](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.rdd.RDD)，[Java](http://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/api/java/JavaRDD.html), [Python](http://spark.apache.org/docs/latest/api/python/pyspark.html#pyspark.RDD), [R](http://spark.apache.org/docs/latest/api/R/index.html)），pair RDD 函数文档（[scala](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.rdd.PairRDDFunctions), [Java](http://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/api/java/JavaPairRDD.html)）。

| 转换操作                                                     | 含义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **map**(func)                                                | 通过函数 func 传递 source 的每个元素，返回一个新的分布式数据集。 |
| **filter**(func)                                             | 通过函数 func 返回 true 的 source 元素返回一个新的数据集。   |
| **flatMap**(func)                                            | 与 map 类似，但是每个输入项可以映射到 0 或多个输出项（因此 func 应该返回一个 Seq，而不是单个 item）。 |
| **mapPartitions**(func)                                      | 与 map 类似，但是在 RDD 的每个分区（块）上单独运行，因此 func 在类型为 T 的 RDD 上运行时必须是类型 Iterator<T> => Iterator<U>。 |
| **mapPartitionsWithIndex**(func)                             | 与 mapPartitions 类似，但也提供了一个表示分区索引的整数值的 **func**，所以当运行在类型为 T 的 RDD 上时，**func** 的类型必须是 (Int, Iterator<T>) => Iterator<U>。 |
| **sample**(withReplacement, fraction, seed)                  | 使用给定的随机数生成器种子队数据进行一部分采用。             |
| **union**(otherDataset)                                      | 返回一个新数据集，该数据集包含源数据中的元素和参数的并集。   |
| **intersection**(otherDataset)                               | 返回一个新数据集，该数据集包含源数据中的元素和参数的交集。   |
| **distinct**([numPartitions])                                | 返回包含源数据去除去重元素的新数据集。                       |
| **groupByKey**([numPartitions])                              | 当调用一个 (K, V) 对的数据集，返回一个 (K, Iterable<V>) 对数据集。<br/>**注意：**如果你是为了对每个键执行聚合（例如 sum 或 average）而进行分组，那么可以使用 reduceByKey 或 aggregateByKey 将产生更好的性能。<br/>**注意：**默认情况下，输出中的并行度取决于父 RDD 的分区数。可以传递一个可选的 numPartitions 参数来设置不同的任务。 |
| **reduceByKey**(func, [numPartitions])                       | 当调用 (K, V) 对的数据集，返回一个 (K, V) 对数据集，当使用给定的 reduce 函数 **func** 时候，每个 key 对应的 values 值会被聚合，必须是 (V, V) 类型 => 类型 V。类似于 `groupByKey`，reduce 任务的数量可以通过第二个参数来设置。 |
| **aggregateByKey**(zeroValue)(seqOp, combOp, [numPartitions]) | 当在 (K, V) 对数据集调用时，返回一个 (K, U) 的数据集，其中每个键的值使用给定的 combine 函数和一个中性的“0”值进行聚合。允许与输入值类型不同的聚合值类型，同时避免不必要的分配。类似于 `groupByKey`，reduce 任务的数量可以通过第二个参数来设置。 |
| **sortByKey**([ascending], [numPartitions])                  | 当在调用 (K, V) 对数据集调用，返回按键（keys）升序或降序排序的 (K, V)对数据集。 |
| **join**(otherDataset, [numPartitions])                      | 当在类型 (K, V) 和 (K, W) 的数据集上调用时，返回一个 (K, (V, W)) 对的数据集，其中包含每个键的所有对元素。外部链接有 leftOuterJoin、rightOuterJoin 和 fullOuterJoin 支持。 |
| **cogroup**(otherDataset, [numPartitions])                   | 当调用 (K, V) 和 (K, W) 类型的数据集时，返回一个 (K, (Iterable, Iterable)) 元组的数据集。该操作也被成为 groupWith。 |
| **cartesian**(otherDataset)                                  | 笛卡尔积操作。当调用 T 和 U 类型的数据集，返回 (T, U) 数据集。 |
| **pipe**(command, [envVars])                                 | 通过 shell 命令（例如 Perl 或 bash 脚本）对 RDDD 的每个分区进行管道传输。将 RDD 元素写到进程的 stdin，并将输出到其 stdout 的行作为字符串的 RDD 返回。 |
| **coalesce**(numPartitions)                                  | 将 RDD 中的分区数量减少到 numPartitions，适用于过滤大型数据集后更有效地运行操作。 |
| **repartition**(numPartitions)                               | 随机地重新 shuffle RDD 中的数据，以创建更多或更少的分区，并在它们之间进行平衡。这总是通过网络打乱所有的数据。 |
| **repartitionAndSortWithinPartitions**(partitioner)          | 根据给定的分区程序重新划分 RDD，并在每个结果分区中根据键对记录进行排序。这比调用重划分然后对每个分区内排序更加有效，因为它可以将排序向下推到洗牌机制中。 |



#### 1.4.6 行动操作

下表是 Spark 支持的常见的行动操作。更多细节，查看 RDD API（[scala](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.rdd.RDD)，[Java](http://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/api/java/JavaRDD.html), [Python](http://spark.apache.org/docs/latest/api/python/pyspark.html#pyspark.RDD), [R](http://spark.apache.org/docs/latest/api/R/index.html)），pair RDD 函数文档（[scala](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.rdd.PairRDDFunctions), [Java](http://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/api/java/JavaPairRDD.html)）。

| 行动操作                                         | 含义                                                         |
| ------------------------------------------------ | ------------------------------------------------------------ |
| **reduce**(func)                                 | 使用函数 func （接收两个参数，返回一个参数）聚合数据集的元素。 |
| **collect**()                                    | 在 driver 程序中以数组的形式返回数据集的所有元素。           |
| **count**()                                      | 返回数据集中元素的数量。                                     |
| **first**()                                      |                                                              |
| **take**(n)                                      | 返回数据集的前 n 个元素的数组。                              |
| **takeSample**(withReplacement, num, [seed])     | 返回一个数组，其中包含数据集的随机 num 元素样本，可以替换也可以不替换，还可以预先指定一个随机种子。 |
| **takeOrdered**(n, [ordering])                   | 使用 RDD 的自然顺序或自定义比较器返回 RDD 的前 n 个元素。    |
| **saveAsTextFile**(path)                         | 将数据集的元素作为一个文本文件（或一组文本文件）写入本地文件系统、HDFS 或任何其他 Hadoop 支持的文件熊。Spark 将对每个元素调用 toString，将其转换为文件中的一行文本。 |
| **saveAsSequenceFile**(path)<br>(Java and Scala) | 将数据集的元素作为 Hadoop SequenceFile 写入本地文件系统、HDFS 或任何其他 Hadoop 支持的文件系统的给定路径中。这在实现 Hadoop 的 Writable 接口的键-值对的 RDDs 上是可用的。在 Scala 中，它也可用于隐式转换为可写的类型（Spark 包括对 Int，Double，String 等基本类型的转换）。 |
| **saveAsObjectFile**(path)<br>(Java and Scala)   | 使用 Java 序列化以简单的格式编写数据集的元素，然后可以使用 `SparkContext.objectFile()` 加载这些元素。 |
| **countByKey**()                                 | 仅在类型 (K, V) 的 RDDs 上可用。返回 带有每个键的计数的 (K, Int) 对的 hashmap。 |
| **foreach**(func)                                | 在数据集的每个元素上运行函数 func。这通常是为了解决诸如更新累加器或与外部存储系统交互之类的副作用。 |

Spark RDD API 还暴露了一些行动操作的异步版本，如 foreachAsync，它立即向调用者返回 FutureAction，而不是在操作完成时阻塞。这可以用于管理或等待行动操作的异步执行。



#### 1.4.7 Shuffle 操作

某些操作在 Spark 触发事件成为 shuffle。shuffle 是 Spark 用于重新分配数据的极值，以便在分区之间以不同的方式进行分组。这通常涉及到跨机器和机器复制数据，使 shuffle 成为一个复杂而昂贵的操作。

**背景**

为了理解在 shuffle 期间会发生什么，我们考虑 [reduceByKey](http://spark.apache.org/docs/latest/rdd-programming-guide.html#ReduceByLink) 操作的例子。reduceByKey 操作生成一个新的 RDD，其中单个键的所有值都被组合成一个元组—(键， 对与该键关联的所有值执行 reduce 函数的结果)。挑战在于，单个键的值不一定都位于相同的分区，甚至也不一定位于同一台机器上，但它们必须位于同一位置才能计算结果。

在 Spark 中，数据通常不会跨分区分布到特定操作所需的位置。在计算期间，单个任务将在单个分区上进行操作—因此，为了阻止单个 `reduceByKey` reduce 任务要执行的所有数据，Spark 需要执行 all-to-all 操作。它必须从所有分区中读取所有键的值，然后将各个分区的值放在一起，以计算每个键的最终结果—上述过程被成为 **shuffle**。

尽管新打乱的数据的每个分区中的元素集时确定的，分区本身的排序也是确定的，但是这些元素的排序是不确定的。如果在 shuffle 后得到可预测的有序数据，可以使用：

* `mapParitions` 来对每个分区进行排序，例如 .sorted

* `repartitionAndSortWithinPartitions` 可以有效地对分区进行排序，同时进行重新分区。

* `sortBy` 创建一个全局有序的 RDD。

可能导致 shuffle 的操作包括 <font color="red">`repartition 操作`</font>（如 repartition 重分区和 coalesce 合并）、<font color="red">`ByKey 操作`</font>（除 counting 外）（如 groupByKey 和 reduceByKey）以及<font color="red">`连接操作`</font>（如 cogroup 和 join）。

**性能影响**

**Shuffle** 是一项昂贵的操作，涉及磁盘 I/O，数据序列化和网路 I/O。为 shuffle 组织数据，Spark 生成任务集—map 任务组织数据，而 reduce 任务用于聚合数据。这个属于来自于 MapReduce，与 Spark 的 map 和 reduce 操作没有直接关系。

在内部，来自单个 map 任务的结果被保存在内存中，直到它们不能适应容纳为止。然后，根据目标分区对它们进行排序，并将它们写入单个文件中。在 reduce 端，任务读取相关的已排序的块。

某些 shuffle 操作会消耗大量堆内存，因为它们使用内存中的数据结果来组合传输之前和滞后的记录。具体来说，`reduceByKey` 和  `aggregateByKey` 在 map 端创建这些结果，而 `'ByKey` 操作在 reduce 端生成这些结构。当内存容纳不下数据时，Spark 会将这些表 spill 到磁盘，导致磁盘 I/O 的额外开销和增加的垃圾收集。

Shuffle 还会在磁盘上生成大量的中间文件。从 Spark 1.3 开始，这些文件将一直保留到不再使用相应的 RDDs 并进行垃圾回收。这样做是为了在重新计算血缘关系（linage）时候不需要重新创建 shuffle 文件。如果应用程序保留对这些 RDDs 的引用，或者 GC 不经常启动，那么垃圾收集可能只会在很长一段时间之后才发生。这意味着长时间的 Spark 作业可能会消耗大量磁盘空间。当配置 Spark contex 时，临时存储目录由 spark.local 配置参数指定。

Shuffle 行为可以通过各种配置参数来调整洗牌行为。可以参考 [Spark 配置指南](http://spark.apache.org/docs/latest/configuration.html)中的 Shuffle 行为。



## 2 共享变量

通常，当传递给 Spark 操作（如 map 或 reduce）的函数在远程集群节点上执行时，它会在函数中使用的所有变量的单独副本上工作。这些变量被复制到每台机器上，而对远程机器上的变量的更新不会传播会 driver 程序。在任务之间支持通用的读写共享变量是低效的。但是，Spark 两种常见的使用模式提供了两种有限的共享变量类型：广播变量和累加器。

### 2.1 广播变量

广播变量允许程序开发人员在每台机器上缓存一个只读变量，而不是将其副本与任务一起发送。例如，可以使用它们以最有效的方式为每个节点提供一个大型输入数据集的副本。Spark 还尝试使用使用有效的广播算法来分发广播变量，以降低通信成本。

Spark 操作通过一组 stages 执行，被分布式 “shuffle” 操作分割。Spark 自动广播每个 stage 中 tasks 所需的公共数据。以这种方式广播的数据以序列化的形式缓存，并在运行每个任务之前反序列化。这意味着，只有当跨多个 stages 的 tasks 需要相同的数据，或者以反序列化形式缓存数据很重要时，显示地创建广播变量才有用。

广播变量是通过 SparkContext.brodacast(v) 方式创建。广播变量是 v 的包装器，它的值可以通过 .value 来访问。下面的代码显示如下：

```scala
scala> val brodacastVar = sc.broadcast(Array(1, 2, 3))
broadcastVar: org.apache.spark.broadcast.Broadcast[Array[Int]] = Broadcast(0)

scala> brodacastVar.value
res0: Array[Int] = Array(1, 2, 3)
```

创建广播变量后，应该使用它而不是集群上运行的任何函数中的值 v，这样 v 就不会被多次发送到节点。此外，对象 v 在广播后不应进行修改，以确保所有节点得到广播变量的相同值。



### 2.2 累机器

累加器是只能通过 associative 和 commutative 操作 ”添加“ 的变量，因此可以有效地并行支持。它们可以用于实现计数器。Spark 本身支持数字类型的累加器，程序开源人员可以添加对新的类型的支持。

作为用户，你可以创建已命名或未命名的累加器。如下图所示，一个指定的累加器（在实例 counter 中）将显示在 web 界面上，对于修改该累加器的 stage。Spark 在 “Tasks“ 表中显示由任务修改的每个累加器的值。

![spark-webui-accumulators](https://gitee.com/struggle3014/picBed/raw/master/spark-webui-accumulators.png)

在 UI 中跟踪累加器对于理解运行阶段的进度是很有用的（**注意：**python 现在还不支持）。

可以通过调用 SparkContext.LongAccumulator() 或 SparkContext.doubleAccumulator() 来分别累加 Long 或 Double 类型的值来创建数值累加器。运行在集群上的任务可以使用 add 方法添加。然而，它们无法读取到它的值。只有 driver 程序可以读取到累加器的值，使用 value 方法。

下面是一个累加器，用于将数组中的元素相加：

```scala
scala> val accum = sc.longAccumulator("My Accumulator")
accum: org.apache.spark.util.LongAccumulator = LongAccumulator(id: 0, name: Some(My Accumulator), value: 0)

scala> sc.parallelize(Array(1, 2, 3, 4)).foreach(x => accum.add(x))
...
10/09/29 18:41:08 INFO SparkContext: Tasks finished in 0.317106 s

scala> accum.value
res2: Long = 10
```

虽然这段代码使用了内置的 Long 类型的累加器，但是程序开发人员可以通过子类 AccumulatorV2 来创建自定义类型。AccumlatorV2 抽象类有几个必须重写的方法：reset（将累加器重置为零）、add（将另一个值添加到累加器中）、merge（将另一个相同类型的累加器合并到这个累加器中）。例如，我们有一个表示数学向量的 MyVector 类，我们可以这样写：

```scala
class VectorAccumulatorV2 extends AccumlatorV2[MyVector, MyVector] {
    
    private val myVector:MyVector = MyVector.createZeroVector
    
    def reset(): Unit = {
        myVector.reset()
    }
    
    def add(v: MyVector): Unit = {
        myVector.add(v)
    }
    ...    
}

// 创建该类型的累加器
val myVectorAcc = new VectorAccumulatorV2
// 然后，将其注册到 spark context 上
sc.register(myVectorAcc, "MyVectorAcc1")
```

注意，当程序开发人员定义他们自己的 AccumulatorV2 类型时，得到的类型可能与添加的元素的类型不同。

对于仅在**行动操作**内部执行的累加器更新，Spark 保证每个任务对累加器的更新只应用一次，即重新启动的任务不会更新值。在转换操作中，用户应该知道，如果重新执行任务或作业阶段，每个任务的更新可能会应用多次。

累加器不改变 Spark 的惰性评价模型。如果它们在 RDD 上的操作被更新，那么它们的值只在计算 RDD 作为行动操作的一部分时更新一次。因此，不能保证在 `map()` 这样的延迟转换中执行累加器更新。下面的代码片段演示了这个属性：

```scala
val accum = sc.longAccumlator
data.map { x => accum.add(x); x }
// 这里，accum 仍然是 0，因为没有行动操作导致 map 操作的计算。
```



#  参考文献

[1] [Spark 官方文档](http://spark.apache.org/docs/latest/)

[2] [Spark 官方文档— Scala API 文档](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.package)