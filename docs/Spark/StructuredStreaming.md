<div align="center"><img src="https://gitee.com/struggle3014/picBed/raw/master/name_code.png"></div>

# 导读

本文介绍 Structured Streaming 相关的内容。

***持续更新中~***



# 目录

<nav>
<a href='#导读' style='text-decoration:none;font-weight:bolder'>导读</a><br/>
<a href='#目录' style='text-decoration:none;font-weight:bolder'>目录</a><br/>
<a href='#正文' style='text-decoration:none;font-weight:bolder'>正文</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1 简介' style='text-decoration:none;${border-style}'>1 简介</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2 快速例子' style='text-decoration:none;${border-style}'>2 快速例子</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3 编程模型' style='text-decoration:none;${border-style}'>3 编程模型</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3.1 基础概念' style='text-decoration:none;${border-style}'>3.1 基础概念</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3.2 处理事件时间（event-time）和延迟数据' style='text-decoration:none;${border-style}'>3.2 处理事件时间（event-time）和延迟数据</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3.3 容错语义' style='text-decoration:none;${border-style}'>3.3 容错语义</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4 使用 Datasets 和 DataFrame 的 API' style='text-decoration:none;${border-style}'>4 使用 Datasets 和 DataFrame 的 API</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4.1 创建流式 DataFrames 和流式  Datasets' style='text-decoration:none;${border-style}'>4.1 创建流式 DataFrames 和流式  Datasets</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4.2 流 DataFrames/Datasets 上的操作' style='text-decoration:none;${border-style}'>4.2 流 DataFrames/Datasets 上的操作</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4.3 开始流式查询' style='text-decoration:none;${border-style}'>4.3 开始流式查询</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4.4 管理流式查询' style='text-decoration:none;${border-style}'>4.4 管理流式查询</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4.5 监视流式查询' style='text-decoration:none;${border-style}'>4.5 监视流式查询</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4.6 使用检查点从故障中恢复' style='text-decoration:none;${border-style}'>4.6 使用检查点从故障中恢复</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4.7 流式查询更改后的恢复语义' style='text-decoration:none;${border-style}'>4.7 流式查询更改后的恢复语义</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5 连续处理' style='text-decoration:none;${border-style}'>5 连续处理</a><br/>
<a href='#参考文献' style='text-decoration:none;font-weight:bolder'>参考文献</a><br/>
</nav>

# 正文

## 1 简介

`Structured Streaming` 是一种基于 Spark SQL 引擎的可扩展且容错的流处理引擎。你可以像表达静态数据的批处理计算一样表达流式计算。Spark SQL 引擎将负责逐步和连续地运行它，并在流数据继续到达时更新最终结果。Spark SQL引擎将负责逐步和连续地运行它，并在流数据继续到达时更新最终结果。你可以在 Scala，Java，Python，R 中使用 [Dataset/DataFrame API](http://spark.apache.org/docs/latest/sql-programming-guide.html) 来表达流聚合，事件时间窗口，流到批的连接等。上述计算在同一个优化的 Spark SQL 引擎上执行。最后，系统通过检查点（`checkpointing`）和预写日志（`Write-Ahead Logs`）确保端到端（`end-to-end`）、准确一次（`exactly-once`）、容错（`fault-tolerance`）保证。*简言之，`Structured Streaming` 提供快速，可扩展，容错，端到端的精确一次流处理，而无需用户推理流式传输。*

在内部，默认情况下，结构化流式查询使用微批处理引擎进行处理，该引擎将数据流作为一系列小批量作业处理，从而实现降低 100 毫秒的端到端延迟和完全一次的容错保证。然而，自 Spark 2.3 依赖，引入了一种称为**连续处理（Continuous Processing）**的新型低延迟处理模式，它可以实现降低 1 毫秒的端到端延迟，并且具有至少一次保证。无需更改查询中的 Dataset/DataFrame，你就可以根据引用程序要求选择模式。

本指南中，将引导完成编程模型和 API。我们将主要使用默认的微批处理模型来解释这些概念，然后讨论连续处理模型。



## 2 快速例子

假设你想维护从侦听 TCP Socket 的数据服务器接收的文本数据的运行 word count。如何使用 Structured Streaming 实现。

```scala
// 导入必要的类
import org.apache.spark.sql.functions._
import org.apache.spark.sql.SparkSession

// 创建 SparkSession
val spark = SparkSession
.builder
.appName("StructuredNetWorkWordCount")
.getOrCreate()

import spark.implicits._
```

然后，创建一个流式 Streaming DataFrame 来代表不断从 `localhost:9999` 接收数据，并在该 DataFrame 上执行 transform 来计算 word counts。

```scala
// 创建流式 DataFrame，监听 localhost:9999 的服务器接收的文本数据，并转换 DataFrame 来做单词计数。
val lines = spark.readStream
.format("socket")
.option("host", "localhost")
.option("port", 9999)
.load()

// 将行分解为单词
val words = lines.as[String].flatMap(_.split(" "))

// 生成运行单词计数
val wordCounts = words.groupBy("value").count()
```

DataFrame  `line` **代表包含流文本数据的无界表**。该表包含一列名为 “value” 的字符串，并且**流数据里的每条数据成为表中的一行**。注意，由于我们只是建立了转换操作，并且尚未启动它，因此目前没有接收任何数据。接下来，我们使用 `.as[String]` 将 DataFrame 转换成一个字符串数据集，这样我们就可以应用 `flatMap` 操作将每一行分割成多个单词。生成的 `words` Dataset 包含所有单词。最后，我们定义了 `wordCounts` DataFrame，在 Dataset 中唯一值进行分组并计数。注意，这是表示流的单词计数的 streaming DataFrame。

我们已经设置了对流数据的查询，剩下的就是实际开始接收数据并计数。为此，我们将它设置（指定 `outputMode("complete")`）为在每次更新 counts 集时，将它们在控制台打印。然后使用 `start()` 启动流计算。

```scala
// 启动运行查询，在控制台打印运行 counts。
val query = wordCounts.writeStream
.outputMode("complete")
.format("console")
.start()

query.awaitTermination()
```

执行完代码后，流式计算将在后台启动。`query` 对象是该活动流式查询的句柄，我们决定使用 `.awaitTermination()` 等待查询终止，以防止进程在查询处于活动状态时退出。

执行此示例代码，可以在 Spark 引用程序中编译代码，或者只需要下载 Spark 后台运行该示例。我们不妨使用第二种。首先，需要使用 Netcat 作为数据服务器运行。

```shell
$ nc -lk 9999
```

然后，在不同的终端中，你可以使用启动示例：

```scala
$ ./bin/run-example org.apache.spark.examples.sql.streaming.StructuredNetworkWordCount localhost 9999
```

·然后，在运行netcat服务器的终端中键入的任何行将被计数并每秒在屏幕上打印

|                                                              |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| \# TERMINAL<br> 1: <br># Running Netcat<br> $ nc -lk 9999 apache spark<br>apache hadoop | \# TERMINAL 2: RUNNING StructuredNetworkWordCount<br>  $ ./bin/run-example org.apache.spark.examples.sql.streaming.StructuredNetworkWordCount localhost 9999<br> -------------------------------------------<br> Batch: 0<br> -------------------------------------------<br> +------+-----+<br>\| value\|count\|<br> +------+-----+ \|<br>\|apache\|    1\|<br>\| spark\|    1\|<br> +------+-----+<br> -------------------------------------------<br> Batch: 1<br> -------------------------------------------<br> +------+-----+<br> \| value\|count\|<br> +------+-----+<br> \|apache\|    2\|<br> \| spark\|    1\|<br> \|hadoop\|    1\|<br> +------+-----+<br> ... |



## 3 编程模型

Structured Streaming 的关键思想是**将实时流数据视为一个不断追加的表**。这导致了一个新的流处理模型，它非常类似于批处理。你可以将流计算表示为标准的类似批处理的查询，就像在静态表上一样，而 Spark 将它作为无界输入表（`unbounded input table`）上的增量查询（`incremental`）来运行。

### 3.1 基础概念

Structured Streaming 中的关键思想是将实时数据流视为` “输入表（Input Table）”`。达到流上的每个数据项就像一个新行被追加到输入表中。

![structured-streaming-stream-as-a-table](https://gitee.com/struggle3014/picBed/raw/master/structured-streaming-stream-as-a-table.png)

对于输入的查询将生成 `“结果表（Result Table）”` 。在每个触发间隔（比如，每隔1秒）都会向输入表追加新行，从而最终更新结果表。无论是否更新结果表，我们都希望将更改后的结果行写入外部接收器/接收器（external sink）。

![structured-streaming-model](https://gitee.com/struggle3014/picBed/raw/master/structured-streaming-model.png)

`“输出 Output”` 被定义成写入外部存储器的内容。输出可以被定义成不同的输出模式：

* **Complete Mode： **整个更新的结果表将写入外部存储器，由存储连接器决定如何处理整个表的写入。
* **Append Mode： **自上次触发后，只有结果表中附加的新行才会写入外部存储器。这仅适用于预计结果表中的现有行不会更改的查询。
* **Update Mode： **只有自上次触发后在结果表中更新的行才会写入外部存储。注意，这与 **Complete Mode** 的不同之处在于此模式仅输出自上次触发后已更改的行。如果查询不包含聚合，则它将等同于 **Append Mode**。

注意，每种模式适用于某些类型的查询。

为了说明该模型的使用，让我们在上面的**快速例子**的上下文中理解模型。

* 最开始的 DataFrame `lines`  是输入表

* 最后的  DataFrame  `wordCounts` 是结果表。

在流上执行的查询将 DataFrame `lines` 转换为 DataFrame `wordCounts` 与在静态 DataFrame 上执行的操作完全相同。当启动计算后，Spark 会不断从 socket 连接接收数据。如果有新的数据到达，Spark 将运行一个“增量”查询，将以前的 counts 与新的数据结合，以计算更新的 counts，如下所示：

![structured-streaming-example-model](https://gitee.com/struggle3014/picBed/raw/master/structured-streaming-example-model.png)

 **注意：Structured Streaming 不会物化整个表。**它从流数据源读取最新的可用数据，以增量方式处理它以更新结果，然后丢弃源数据。它只保留更新结果所需的最小中间状态数据（例如，前面示例中的中间计数）。

这种模式与其他许多流处理引擎有着显著差异。许多流处理引擎要求用户自己维护运行的状态，因此必须对容错和数据一致性（at-least-once, or at-most-once, or exactly-once）进行处理。在这个模型中，当有新数据时，Spark 负责更新结果表，从而减轻用户的工作。作为例子，让我们看下模型如何处理事件时间（event-time）和延迟到达数据。



### 3.2 处理事件时间（event-time）和延迟数据

事件时间（event-time） 是嵌入在数据中的时间。对于许多 application，你可能希望在 event-time 上进行操作。例如，如果要每分钟获取 IoT 设备生成的事件数，则会希望使用数据生成的时间（即嵌入在数据中的 event-time），而不是 Spark 接收到数据的时间。在该模型中 event-time 被非常自然地表达，来自设备的每个事件都是表中的一行，**event-time 是行中的一列**。这允许基于 window 的集合（例如每分钟的事件数）仅仅是 event-time 列上的特殊类型的分组（grouping）和聚合（aggregation）：每个事件窗口是一个组，并且每一行可以属于多个窗口/组。因此，可以在静态数据集和数据流上进行基于事件时间窗口（event-time-window-based）的聚合查询，从而使用户操作更加方便。

此外，该模型也可以自然的处理接收到的时间晚于 event-time 的数据。因为 Spark 一直在更新结果表，所以它可以完全控制更新旧的聚合数据，或清除旧的聚合以限制中间状态数据的大小。**自 Spark 2.1 起，开始支持 watermark 来允许用于指定数据的超时时间（即接收时间比 event-time 晚多少），并允许引擎相应地清理旧状态**。这在下文的“[窗口操作]()”小节中进一步说明。



### 3.3 容错语义

提供端到端的 exactly-once 语义是 Struectured Streaming 背后设计的关键目标之一。为了达到这点，设计了 Structured Streaming 的 sources（数据源）、sink（输出）以及执行引擎可靠的追踪确切的执行进度以便于通过重启或重新处理来处理任何类型的故障。对于每个具有偏移量（类似于 Kafka 偏移量或 Kinesis 序列号）的 streaming source。引擎使用 checkpoint 和 WAL 来记录每个 trigger 处理的 offset 范围。**streaming sinks 被设计为对重新处理是幂等的**。结合可以重放的 sources 和支持重复处理幂等的 sinks，不管发生什么故障 Structured Streaming 可以确保**端到端的 exactly-once 语义**。



## 4 使用 Datasets 和 DataFrame 的 API

**自 Spark 2.0 起，Spark 可以代表静态的、有限数据和流式的、无限数据**。与静态的 Datasets/DataFrames 类似， 你可以使用 SparkSession 基于 streaming sources 来创建 DataFrames/Datasets，并且与静态 DataFrames/Datasets 使用相同的操作。[DataFrame/Dataset 编程指南](http://spark.apache.org/docs/latest/sql-programming-guide.html)。



### 4.1 创建流式 DataFrames 和流式  Datasets

流式 DataFrames 可以通过 `SparkSession.readStream()` 返回的 `DataStreamReader` 接口创建。与创建静态 DataFrame 的 read 接口类似，你可以指定 source 的细节包括：data format、schema、option 等。



#### 4.1.1 输入源（Input Sources）

有一些内置的源。

* File source：以文件流的形式读取目录中写入的文件。支持的文件格式为text，csv，json，parquet。请注意，**文件必须以原子方式放置在给定的目录中，这在大多数文件系统中可以通过文件移动操作实现**。

* Kafka source：从 Kafka 拉取数据。兼容 Kafka 0.10.0 以及更高版本。该部分可以查看 [Kafka 集成指南](http://spark.apache.org/docs/latest/structured-streaming-kafka-integration.html)。

* Socket source（仅做测试用）：从 socket 读取 UTF-8 文本数据。监听的 server socket 在 driver 端。**注意**，这只能用于测试，因为它不提供端到端的容错。

* Rate source（仅做测试用）：以每秒指定的行数生成数据，每个输出行包含一个时间戳和值。

某些 source 不是容错的，因为它们不能保证在故障后可以重放数据。以下是 Spark 中所有 sources 的详细信息：

| Source            | 选项                                                         | 错误容忍 | 备注                                               |
| ----------------- | ------------------------------------------------------------ | -------- | -------------------------------------------------- |
| **File Source**   | path:输入目录的路径，所有格式通用。<br>maxFilesPerTrigger:**每次 trigger 最大文件数（默认无限大）**<br>latestFirst:**是否首先处理最新的文件，当有大量积压的文件时很有用（默认 false）**<br>fileNameOnly:是否仅根据文件名而不是完整路径检查新文件（默认 false）。将此设置为“true”，以下文件将被视为相同的文件，因为它们的文件名“dataset.txt”是相同的：`"file:///dataset.txt","s3://a/dataset.txt","s3n://a/b/dataset.txt"、"s3a://a/b/c/dataset.txt"` | Y        | 支持 glob 路径，但不支持多个逗号分隔的路径 /glob。 |
| **Socket Source** | host:要连接的 host, 必须指定。<br>port:要连接的 port, 必须指定。<br> | N        |                                                    |
| **Rate Source**   | rowsPerSecond<br>rampUpTime<br>rowPerSecond<br>numPartitions | Y        |                                                    |
| **Kafka Source**  | 详见 [Kafka 集成指南](https://spark.apache.org/docs/latest/structured-streaming-kafka-integration.html) | Y        |                                                    |

下面是一些例子：

```scala
val spark: SparkSession = ...

// 从 socket 读取 text
val socketDF = spark
.readStream
.format("socket")
.option("host", "localhost")
.option("port", 9999)
.load()

socketDF.isStreaming // 对于流式数据源的 DataFrame，返回 True。

socketDF.printSchema

// 读取一个目录中自动写入的所有 csv 文件
val userSchema = new StructType().add("name", "string").add("age", "integer")
val csvDF = spark
.readStream
.option("sep;")
.schema(userSchema) // 指定 csv 文件的 schema
.csv("/path/to/directory") // 等价于 format("csv").load("/path/to/directory")
```

这些示例生成的流 DataFrames 是 untyped 的，在编译时并不会进行类型检查，只在运行时进行检查。某些操作，比如 map、flatMap 等，需要在编译时就知道类型，这时你可以将 DataFrame 转换为 Dataset（使用与静态相同的方法）。相关的细节可以查看 [SQL 编程指南](http://spark.apache.org/docs/latest/sql-programming-guide.html)。



#### 4.1.2 流式 DataFrames/Datasets 的 Schema 推断与分区

默认情况下，基于 File Source 需要你自行指定 schema，而不是依靠 Spark 自动推断。这样的限制确保了 streaming query 会使用确切的 schema。你也可以通过将`spark.sql.streaming.schemaInference` 设置为 true 来重新启用 schema 推断。

**当子目录名为 `/key=value/` 时，会自动发现分区，并且对这些子目录进行递归发现**。如果这些列出现在提供的 schema 中，spark 会读取相应目录的文件并填充这些列。可以增加组成分区的目录，比如当 `/data/year=2015/` 存在是可以增加 `/data/year=2016/`；但修改分区目录是无效的，比如创建目录 `/data/date=2016-04-17/`。



### 4.2 流 DataFrames/Datasets 上的操作

你可以在流式 DataFrames/Datasets 上应用各种操作：从`无类型（untyped）`，类似 SQL 的操作（比如 select、where、groupBy），到`有类型（typed）`的类似 RDD 操作（比如 `map、filter、flatMap`）。让我们通过几个例子来看看。



#### 4.2.1 基础操作—选择（Selection），投影（Projection），聚合（Aggregation）

数据流支持在 DataFrame/Dataset 上大多数常见操作。本节后面讨论少数不支持的操作。

```scala
case class DeviceData(device: String, deviceType: String, signal: Double, time: DateTime)

val df: DataFrame = ... // 带有 { device: string, deviceType: string, signal: double, time: string } schema 的物联网（IOT）数据的流式 DataFrame。
val ds: Dataset[DeviceData] = df.as[DeviceData] // 物联网设备数据的流 Dataset

// 选择 signal 大于 10 的设备
df.select("device").where("signal > 10")
ds.filter(_.signal > 10).map(_.device)

// 计算每个设备类型的数量
df.groupBy("deviceType").count() // 使用 untyped API

// 为每一个设备类型计算平均的 signal
import org.apache.spark.sql.expressions.scalalang.thread
ds.groupByKey(_.deviceType).agg(typed.avg(_.signal)) // 使用 typed 类型
```

也可以将流式 DataFrame/Dataset 注册为一个临时视图，然后在其上应用 SQL 命令。

```scala
df.createOrReplaceTempView("updates")
spark.sql("select count(*) from updates") // 返回另一个 Streaming DF
```

**注意**，可以使用 df.isStreaming 来判别是否是流式 DataFrame/Dataset

```scala
df.isStreaming
```



#### 4.2.2 基于事件时间（event time）的窗口操作

使用 Structured Streaming 进行滑动的 event-time 窗口聚合是很简单的，与分组聚合非常类似。在分组聚合中，为用户指定的分组列中的每个唯一值维护一个聚合值（例如计数）。**在基于 window 的聚合的情况下，为每个 window 维护聚合（aggregate values），流式追加的行根据 event-time 落入相应的聚合**。让我们通过下图来理解。

想象下，我们的快速示例现在改成了包含数据生成的时间。现在我们想在 10 分钟的 window 内计算 word count，每 5 分钟更新一次。比如 `12:00 - 12:10, 12:05 - 12:15, 12:10 - 12:20` 等。`12:00 - 12:10` 是指数据在 12:00 之后 12:10 之前到达。现在，考虑一个 word 在 12:07 的时候接收到。该 word 应当增加 `12:00 - 12:10` 和 `12:05 - 12:15` 相应的 counts。所以 counts 会被分组的 key 和 window 分组。

结果表将如下所示：

![structured-streaming-window](https://gitee.com/struggle3014/picBed/raw/master/structured-streaming-window.png)

由于这里的 window 与 group 非常类似，在代码上，你可以使用 `groupBy` 和 `window` 来表达 window 聚合。例子如下：

```scala
import spark.implicits._

val words = ... // 带有 { timestamp: Timestamp, word: String } schema 的流 DataFrame

// 通过窗口和单词对数据进行分组，然后计算每组的数量
val windowedCounts = words.groupBy(window($"timestamp", "10 minutes", "5 minutes"),
  $"word")
.count()
```



#### 4.2.3  处理延迟数据和水位（watermark）

现在考虑一个数据延迟到达会怎么样。例如，一个在 12:04 生成的 word 在 12:11 被接收到。**application 会使用 12:04 而不是 12:11 去更新 `12:00 - 12:10`的 counts**。这在基于 window 的分组中很常见。Structured Streaming 会长时间维持部分聚合的中间状态，以便于后期数据可以正确更新旧 window 的聚合，如下所示：

![structured-streaming-late-data](https://gitee.com/struggle3014/picBed/raw/master/structured-streaming-late-data.png)



#### 4.2.3 连接（Join）操作

#### 4.2.4 流式去重（Streaming Deduplication）

#### 4.2.5 处理多水位(Watermark)策略

#### 4.2.6 任意有状态的操作

#### 4.2.7 不支持的操作

### 4.3 开始流式查询

#### 4.3.1 输出模式

#### 4.3.2 输出 Sinks

#### 4.3.3 触发器

### 4.4 管理流式查询

### 4.5 监视流式查询

#### 4.5.1 交互式读取指标

#### 4.5.2 使用异步 API 以编程方式报告指标

#### 4.5.3 使用 Dropwizard 报告指标

### 4.6 使用检查点从故障中恢复

### 4.7 流式查询更改后的恢复语义



## 5 连续处理



# 参考文献

[1] [Spark 官方文档之 Structured Streaming 编程指南](http://spark.apache.org/docs/latest/structured-streaming-programming-guide.html#additional-information)

[2] [O'REILLY Streaming 101：The world beyond](https://www.oreilly.com/radar/the-world-beyond-batch-streaming-101/)

[3] [O'REILLY Streaming 102：The world beyond](https://www.oreilly.com/radar/the-world-beyond-batch-streaming-102/)

[4] [Structured Streaming 编程指南](https://www.cnblogs.com/fenghuoliancheng/p/10790307.html)

[5] [GitHub—xy2953396112—Structured-Streaming—编程指南](https://github.com/xy2953396112/spark-sourcecodes-analysis/blob/master/structured-streaming/Structured-Streaming-编程指南.md)

[6] [简书—Structured Streaming 编程指南](https://www.jianshu.com/p/ae07471c1f8d)