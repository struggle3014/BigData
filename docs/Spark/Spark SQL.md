<div align="center"><img src="https://gitee.com/struggle3014/picBed/raw/master/name_code.png"></div>

# 导读

本文主要简介 Spark SQL 相关的内容。首先从 Spark SQL 简介出发；接着，介绍了 DataFrame，DataSet 及 RDD；然后演示了 Structured API 的基本使用；后面，又介绍了 Spark SQL 的数据源；而后，又列举了 Spark SQL 常用函数；最后，介绍了 Spark SQL 运行原理。本文只是对 Spark SQL 的简单介绍，在实际工作中，有更多的细节需要处理，不过只要抓住 Spark SQL 的整体框架，即使遇到问题，我们也能够从容应对。

***持续更新中~***



# 目录

[TOC]

# 正文

## 1 Spark SQL 简介

Spark SQL 是 Spark 中的一个子模块，主要用于结构化数据。它具有以下特点：

* 能够将 SQL 查询与 Spark 程序无缝混合，允许使用 SQL 或 DataFrame API 对结构化数据进行查询；
* 支持多种开发语言；
* 支持多种数据源，包括 Hive，Avro，Parquet，ORC，JSON 和 JDBC 等；
* 支持 HiveQL 语法以及 Hive SerDes 和 UDF，允许你访问现有的 Hive 仓库；
* 支持标准 JDBC 和 ODBC 连接；
* 支持优化器，列式存储和代码生成等特性；
* 支持扩展并能保证容错。![sql-hive-arch](https://gitee.com/struggle3014/picBed/raw/master/sql-hive-arch.png)



## 2 DataFrame，DataSet，RDD

### 2.1 DataFrame

为了支持结构化数据的处理，Spark SQL 提供了新的数据结构 DataFrame。DataFrame 是一个具有列名组成的数据集。它在概念上等同于关系数据库中的表或 R/Python 语言中的 `data frame`。由于 Spark SQL 支持多种语言的开发，所以每种语言都定义了 `DataFrame` 的抽象，主要如下：

| 语言   | 主要抽象                                    |
| ------ | ------------------------------------------- |
| Scala  | Dataset[T] & DataFrame(Dataset[Row] 的别名) |
| Java   | Dataset[T]                                  |
| Python | DataFrame                                   |
| R      | DataFrame                                   |



### 2.2 Dataset

Dataset 是分布式的数据集合，于 Spark 1.6 版本被引入，集成了 RDD 和 DataFrame 的优点，具有强类型的特点，同时支持 Lambda 函数，但只能在 Scala 和 Java 语言中使用。在 Spark 2.0 后，为了方便开发者，Spark 将 DataFrame 和 Dataset 的 API 融合到一起，提供了结构化的 API（Structured API），用户可以通过一套标准的 API 就能够完成对两者的操作。

> **注意：**DataFrame 被标记为 Untyped API，而 Dataset 被标记为 Typed API。

![spark-unifed](https://gitee.com/struggle3014/picBed/raw/master/spark-unifed.png)



### 2.3 静态类型与运行时类型安全

静态类型（static-typing）与运行时类型安全（runtime type-safety）主要变现如下：

在实际使用中，如果你使用 Spark SQL 的查询语句，则直接运行时你才会发现有语法错误，而如果你用的是 DtaFrame 和 Dataset，则在编译时就可以发现错误（节省了开发时间和整体代价）。DataFrame 和 Dataset 主要区别在于：

在 DataFrame 中，当你调用 API 之外的函数，编译器就会报错，但如果你使用了一个不存在的字段名字，编译器依然无法发现。而 Dataset 的 API 都是用 Lambda 函数和 JVM 类型对象表示的，所有不匹配的类型参数在编译时就会被发现。

以上这些最终都被解释成关于类型安全图谱，对应开发中的语法和分析错误。在图谱中，Dataset 最严格，但对于开发者效率最高。

![spark-运行安全](https://gitee.com/struggle3014/picBed/raw/master/spark-运行安全.png)

**更加直观的例子：**

![](https://gitee.com/struggle3014/picBed/raw/master/20200324162812.png)

这里可能存在的疑惑是 DataFrame 命名是有确定的 Schema 结构（即列名、列字段类型都是已知的），但是为什么还是无法对列名进行推断和错误判断，这是因为 DataFrame 是 Untyped 的。



### 2.4 Untyped & Typed

前面介绍过 DataFrame API 被标记为 `Untyped API`，而 Dataset API 被标记为 `Typed API`。DataFrame 的 `Untyped` 是相对于语言或 API 层面而言，它确实有明确的 Schema 结构，即列名和列类型都是明确的。但这些信息完全由 Spark 来维护，Spark 只会在运行时检查这些类型和指定类型是否一致。这也是为什么在 Spark 2.0 之后，官方推荐把 DataFrame 看做是 `DataSet[Row]`，Row 是 Spark 中定义的一个 `trait`，其子类中封装了列字段的信息。

相对而言，DataSet 是 Typed 的，即强类型。如下面代码，DataSet 的类型由 Case Class（Scala）或者 Java Bean（Java）来明确指定，在这里即每一行数据代表一个 `Person`，这些信息由 JVM 来保证正确性，所以字段错误和类型错误在编译时候就会被 IDE 发现。

```scala
case class Person(name: String, age: Long)
val dataSet: Dataset[Person] = spark.read.json("people.json").as[Person]
```



### 2.5 DataFrame，Dataset，RDD 总结

1.  RDDs 适合非结构化数据的处理，而 DataFrame 和 DataSet 更适合结构化数据和变结构化数据的处理；
2. DataFrame 和 DataSet 可以通过统一的 Structured API 进行访问，而 RDDs 则更适合函数式编程的场景；
3. 相比于 DataFrame，DataSet 是强类型的（Typed），有着更为严格的静态类型检查；
4. DataSets、DataFrames、SQL 的底层都依赖 RDDs API，并对外提供结构化的访问接口。

![spark-structure-api](https://gitee.com/struggle3014/picBed/raw/master/spark-structure-api.png)

##  3 Structured API 基本使用

### 3.1 DataFrame 和 Dataset 创建

**DataFrame 创建**

Spark 中所有的功能入口点是 `SparkSession`，可以使用 `SparkSession.builder()` 创建。创建后，应用程序就可以从现有 RDD，Hive 表或  Spark 数据源创建 DataFrame。示例如下：

```scala
val spark = SparkSession.builder().appName("Spark-SQL").master("local[2]").getOrCreate()
val df = spark.read.json("/usr/file/json.emp.json")
df.show()

// 建议在进行 spark SQL 编程前导入隐式转换，因为 DataFrames 和 DataSets 中很多操作都依赖了隐式转换。
import spark.implicits._
```



**DataSet 创建**

Spark 支持由内部数据集和外部数据集来创建 DataSet，其创建方式分别如下：

**1. 由外部数据源创建**

```scala
// 1- 导入隐式转换
import spark.implicits._

// 2- 创建 case class，等价于 Java Bean
case class Emp(ename: String, comm: Double, deptno: Long, empno: Long,
              hiredate: String, job: String, mgr: Long, sal: Double)

// 3- 由外部数据集创建 DataSets
val ds = spark.read.json("/usr/file/emp.json").as[Emp]
ds.show()
```



**2. 由外部数据源创建**

```scala
// 1- 导入隐式转换
import spark.implicits._

// 2- 创建 case class，等价于 Java Bean
case class Emp(ename: String, comm: Double, deptno: Long, empno: Long,
              hiredate: String, job: String, mgr: Long, sal: Double)

// 3- 由内部数据集创建 DataSets
val caseClassDS = Seq(Emp("ALLEN", 300.0, 30, 7499, "1981-02-20 00:00:00", "SALESMAN", 7698, 1600.0),
                     Emp("JONES", 300.0, 30, 7499, "1981-02-20 00:00:00", "SALESMAN", 7698, 1600.0)).toDS()

caseClassDS.show()
```



**3. 由 RDD 创建 DataFrame**

Spark 支持两种方式把 RDD 转换为 DataFrame，分别是使用反射推断和指定 Schema 转换：

**使用反射推断**

```scala
// 1- 导入隐式转换
import spark.implicits._

// 2- 创建 case class，等价于 Java Bean
case class Emp(ename: String, comm: Double, deptno: Long, empno: Long,
              hiredate: String, job: String, mgr: Long, sal: Double)

// 3- 创建 RDD 并转换为 DataSet
val rddToDS = spark.sparkContext
.textFile("/usr/file/dept.txt")
.map(_.split("\t"))
.map(line => Dept(line(0).trim.toLong, line(1), line(2)))
.toDS() // 如果调用 toDF() 则转换为 DataFrame。
```



**指定 Schema**

```scala
import org.apache.spark.sql.Row
import org.apache.spark.sql.types._

// 1- 定义每个列的类型
val fileds = Array(StructField("deptno", LongType, nullable = true),
                  StructField("dname", StringType, nullable = true),
                  StructField("loc", StringType, nullable = true))

// 2- 创建 schema
val schema = StructType(fields)

// 3- 创建 RDD
val deptRDD = spark.sparkContext.textFile("/usr/file/dept.txt")
val rowRDD = deptRDD.map(_.split("\t")).map(line => Row(line(0).toLong, line(1), line(2)))

// 4- 将 RDD 转换为 dataFrame
val deptDF = spark.createDataFrame(rowRD, schema)
deptDF.show()
```



**4. DataFrame 和 DataSets 互相转换**

Spark 提供了非常方便的转换方法用于 DataFrame 与 DataSet 间互相转换，示例如下：

```scala
// DataFrame 转 DataSet
scala> df.as[Emp]
res1: org.apache.spark.sql.Dataset[Emp] = [COMM: double, DEPTNO: bigint ... 6 more fields]

// DataSet 转 DataFrame
scala> ds.toDF()
res2: org.apache.spark.sql.DataFrame = [COMM: double, DEPTNO: bigint ... 6 more fields]
```



### 3.2 Columns 列操作

**1 引用列**

Spark 支持多种方法来构造和引用列，最简单的是使用 `col()` 或 `column()` 函数。

```scala
col("colNmae")
column("colNmae")

// 对于 Scala 语言而言，还可以使用 $"myColumn" 和 'myColumn 这两种糖语法进行引用。
df.select($"ename", $"job").show()
df.select('ename, 'job).show()
```

**2 新增列**

```scala
// 基于已有列值新增列
df.withColumn("upSal", $"sal"+1000)
// 基于固定值新增列
df.withColumn("intCol", lit(1000))
```

**3 删除列**

```scala
// 支持删除多个列
df.drop("comm", "job").show()
```

**4 重命名列**

df.withColumnRenamed("comm", "common").show()

**说明：**新增，删除，重命名都会产生新的 DataFrame，原来的 DataFrame 不会被改变。



### 3.3 使用 Structured API 进行基本查询

```scalsa
// 1- 查询员工姓名及工作
df.select($"ename", $"job").show()

// 2- filter 查询工资大于 2000 的员工信息
df.filter($"sal" >2000).show()

// 3- orderBy 按照员工编号降序，工资降序进行查询
df.orderBy(desc("deptno"), asc("sal")).show()

// 4- limit 查询工资最高的 3 名员工的信息
df.orderBy(desc("sal")).limit(3).show()

// 5- distinct 查询所有部门编号
df.select("deptno").distinct().show()

// 6- groupBy 分组统计部门人数
df.groupBy("deptno").count().show()
```



### 3.4 使用 Spark SQL 进行基本查询

#### **3.4.1 Spark SQL 基本使用**

```scala
// 1- 首先将 DataFrame 注册为临时视图
df.createOrReplaceTempView("emp")

// 2- 查询员工姓名及工作
spark.sql("SELECT ename, job FROM emp").show()

// 3- 查询工资大于 2000 的员工信息
spark.sql("SELECT * FROM emp where sal > 2000").show()

// 4- orderBy 按照部门编号降序，工资升序查询
spark.sql("SELECT * FROM emp ORDER BY deptno DESC, sal ASC").show()

// 5- limit 查询工资最高的 3 名员工信息
spark.sql("SELECT * FROM ORDER BY sal DESC LIMIT 3").show()

// 6- distinct 查询所有部门编号
spark.sql("SELECT DISTINCT(deptno) FROM emp").show()

// 7- 分组统计部分人数
spark.sql("SELECT deptno, count(ename) FROM emp group by deptno").show()
```



#### **3.4.2 全局临时视图**

上面是使用 `createOrReplaceTempView` 创建的是会话临时视图，它的生命周期仅限于会话范围，会随会话的结束而结束。

你也可以使用 `createGlobalTempVeiw` 创建全局临时视图，全局临时视图可以在所有会话之间共享，并直到整个 Spark 应用程序终止后才消失。全局临时视图被定义在内置的 `global_temp`数据库下，需要使用限定名进行引用，如 `SELECT * FROM global_temp.view1`。

```scala
// 注册为全局临时视图
df.createGlobalTempView("gemp")

// 使用限定名称进行引用
spark.sql("SELECT ename, job FROM global_temp.gemp").show()
```



## 4 外部数据源

### 4.1 简介

#### **4.1.1 多数据源支持**

Spark 支持很多不同种类的数据源，包括 Parquet，ORC，JSON，Hive，JDBC，Avro 等。能够满足大部分使用场景。

#### **4.1.2 读数据格式**

读取 API 遵循以下调用格式：

```scala
// 格式
DataFrameReader.format(...).option("key", "value").schema(...).load()

// 示例
spark.read.format("csv")
	.option("mode", "FAILFAST") // 读取模式
	.option("inferSchema", "true") // 是否自动推断
	.option("path", "path/to/file(s)") // 文件路径
	.schema(someSchema) // 使用预定义的 schema
	.load()
```

读模式有以下三种可选项

| 读模式          | 解释                                                         |
| --------------- | ------------------------------------------------------------ |
| `permissive`    | 当遇到损坏的记录时，将其所有字段设置为 null，并将所有损坏的记录放在名为 _corruption_t_record 的字符串列中。 |
| `dropMalformed` | 删除格式不正确的行。                                         |
| `failFast`      | 遇到格式不正确的数据时立即失败。                             |

#### **4.1.3 写数据格式**

写入 API 遵循以下调用格式：

```scala
// 格式
DataFrameWriter.format(...).option(...).partitionBy(...).bucketBy(...).sortBy(...).save()

// 示例
dataframe.write.format("csv")
	.option("mode", "OVERWRITE") // 写模式
	.option("dateFromat", "yyyy-MM-dd") // 日期格式
	.option("path", "path/to/file(s)")
	.save()
```

写数据模式有以下四种可选项：

| Scala/Java               | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| `SaveMode.ErrorIfExists` | 如果给定的路径已经存在文件，则抛出异常，这是写数据默认的模式。 |
| `SaveMode.Append`        | 数据以追加的方式写入。                                       |
| `SaveMode.Overwrite`     | 数据以覆盖的方式写入。                                       |
| `SaveMode.Ignore`        | 如果给定的路径已经存在文件，则不做任何操作。                 |



### 4.2 Parquet 文件

Parquet 是一个开源的面向列的数据存储，它提供了多种存储优化，允许读取单独的列非整个文件，这不仅节省了存储空间而且提升了读取效率，它是 Spark 是默认的文件格式。

#### **4.2.1 读取 Parquet 文件**

```scala
spark.read.format("parquet").load("/usr/file/parquet/dept.parquet").show(5)
```

#### **4.2.2 写入 Parquet 文件**

```scala
df.write.format("parquet").mode("overwrite").save("/tmp/spark/parquet/dept")
```

#### **4.2.3 可选配置**

| 读写配置 | 配置项            | 可选值                                                       | 默认值                                        | 描述                                                         |
| -------- | ----------------- | ------------------------------------------------------------ | --------------------------------------------- | ------------------------------------------------------------ |
| Write    | compression/codec | None,<br/>uncompressed,<br/>bzip2,<br/>deflate, gzip,<br/>lz4, or snappy | None                                          | 压缩文件格式                                                 |
| Read     | mergeSchema       | true, false                                                  | 取决于配置项<br>spark.sql.parquet.mergeSchema | 当为真时，Parquet 数据源将所有数据文件收集的 Schema 合并在一起，否则将从摘要文件中选择 Schema，如果没有可用的摘要文件，则从随机数据文件中选择 Schema。 |



### 4.3 ORC 文件

ORC 是一种自描述的、类型感知的列文件格式，它针对大型数据的读写进行了优化，也是大数据中常用的文件格式。

#### **4.3.1 读 ORC 文件**

```scala
spark.read.format("orc").load("/usr/file/orc/dept.orc").show(5)
```

#### **4.3.2 写 ORC 文件**

```scala
csvFile.write.format("orc").mode("overwrite").save("/tmp/spark/orc/dept")
```



### 4.4 JSON 文件

#### **4.4.1 读 JSON 文件**

```scala
spark.read.format("json").option("mode","FAILFAST").load("/usr/file/json/dept.json").show(5)
```

**注意：**默认不支持一条数据记录跨越多行 (如下)，可以通过配置 `multiLine` 为 `true` 来进行更改，其默认值为 `false`。

```json
// 默认支持单行
{"DEPTNO": 10,"DNAME": "ACCOUNTING","LOC": "NEW YORK"}

//默认不支持多行
{
  "DEPTNO": 10,
  "DNAME": "ACCOUNTING",
  "LOC": "NEW YORK"
}
```

#### **4.4.2 写入 JSON 文件**

```scala
df.write.format("json").mode("overwrite").save("/tmp/spark/json/dept")
```



### 4.5 Hive 表

#### **4.5.1 读写 Hive 表**

```scala
import java.io.File

import org.apache.spark.sql.{Row, SaveMode, SparkSession}

case class Record(key: Int, value: String)

// warehouseLocation points to the default location for managed databases and tables
val warehouseLocation = new File("spark-warehouse").getAbsolutePath

val spark = SparkSession
  .builder()
  .appName("Spark Hive Example")
  .config("spark.sql.warehouse.dir", warehouseLocation)
  .enableHiveSupport()
  .getOrCreate()

import spark.implicits._
import spark.sql

sql("CREATE TABLE IF NOT EXISTS src (key INT, value STRING) USING hive")
sql("LOAD DATA LOCAL INPATH 'examples/src/main/resources/kv1.txt' INTO TABLE src")

// 读：Queries are expressed in HiveQL
sql("SELECT * FROM src").show()
// +---+-------+
// |key|  value|
// +---+-------+
// |238|val_238|
// | 86| val_86|
// |311|val_311|
// ...

// 写：Save DataFrame to the Hive managed table
val df = spark.table("src")
df.write.mode(SaveMode.Overwrite).saveAsTable("hive_records")
```



### 4.6 JDBC 连接其他数据库

Spark 同样支持与传统的关系型数据库进行数据读写。但是 Spark 程序默认是没有提供数据库驱动的，所以在使用前需要将对应的数据库驱动上传到安装目录下的 `jars` 目录中。下面示例使用的是 Mysql 数据库，使用前需要将对应的 `mysql-connector-java-x.x.x.jar` 上传到 `jars` 目录下。

#### **4.6.1 读数据**

```scala
// 读取全表数据示例如下，这里的 help_keyword 是 mysql 内置的字典表，只有 help_keyword_id 和 name 两个字段。
spark.read
.format("jdbc")
.option("driver", "com.mysql.jdbc.Driver")            //驱动
.option("url", "jdbc:mysql://127.0.0.1:3306/mysql")   //数据库地址
.option("dbtable", "help_keyword")                    //表名
.option("user", "root").option("password","root").load().show(10)

// 从查询结果读取数据
val pushDownQuery = """(SELECT * FROM help_keyword WHERE help_keyword_id <20) AS help_keywords"""
spark.read.format("jdbc")
.option("url", "jdbc:mysql://127.0.0.1:3306/mysql")
.option("driver", "com.mysql.jdbc.Driver")
.option("user", "root").option("password", "root")
.option("dbtable", pushDownQuery)
.load().show()

//输出
+---------------+-----------+
|help_keyword_id|       name|
+---------------+-----------+
|              0|         <>|
|              1|     ACTION|
|              2|        ADD|
|              3|AES_DECRYPT|
|              4|AES_ENCRYPT|
|              5|      AFTER|
|              6|    AGAINST|
|              7|  AGGREGATE|
|              8|  ALGORITHM|
|              9|        ALL|
|             10|      ALTER|
|             11|    ANALYSE|
|             12|    ANALYZE|
|             13|        AND|
|             14|    ARCHIVE|
|             15|       AREA|
|             16|         AS|
|             17|   ASBINARY|
|             18|        ASC|
|             19|     ASTEXT|
+---------------+-----------+
```

#### **4.6.2 写数据**

```scala
val df = spark.read.format("json").load("/usr/file/json/emp.json")
df.write
.format("jdbc")
.option("url", "jdbc:mysql://127.0.0.1:3306/mysql")
.option("user", "root").option("password", "root")
.option("dbtable", "emp")
.save()
```

#### **4.6.3 配置选项**

| 属性名                                  | 含义                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| url                                     | 要连接的 JDBC URL                                            |
| dbtable                                 | 读取或写入的 JDBC 表。                                       |
| query                                   | 用于将数据读入Spark的查询。                                  |
| driver                                  | 用于连接到此URL的JDBC驱动程序的类名。                        |
| partitionColumn, lowerBound, upperBound | 分区总数，上界，下界                                         |
| numPartitions                           | 可用于表读写并行性的最大分区数。如果要写的分区数量超过这个限制，那么可以调用 coalesce(numpartition) 重置分区数。 |
| queryTimeout                            | 驱动程序将等待语句对象执行到给定秒数的秒数。0表示没有限制。  |
| fetchsize                               | 每次往返要获取多少行数据。此选项仅适用于读取数据。           |
| batchsize                               | 每次往返插入多少行数据，这个选项只适用于写入数据。默认值是 1000。 |
| isolationLevel                          | 事务隔离级别：可以是 NONE，READ_COMMITTED, READ_UNCOMMITTED，REPEATABLE_READ 或 SERIALIZABLE，即标准事务隔离级别。<br/>默认值是 READ_UNCOMMITTED。这个选项只适用于数据读取。 |
| sessionInitStatement                    | 在将每个数据库会话打开到远程数据库并开始读取数据之前，此选项将执行一个自定义SQL语句(或PL/SQL块)。使用它来实现会话初始化代码。 |
| truncate                                | 这是一个与JDBC写入器相关的选项。当SaveMode。启用覆盖，此选项将导致Spark截断现有表，而不是删除并重新创建它。这可能更有效，并且可以防止删除表元数据(例如，索引)。但是，在某些情况下，例如当新数据具有不同的模式时，它将不起作用。它默认为false。此选项仅适用于写入。 |
| createTableOptions                      | 写入数据时自定义创建表的相关配置。                           |
| createTableColumnTypes                  | 写入数据时自定义创建列的列类型。                             |
| customSchema                            | 用于从JDBC连接器读取数据的自定义模式。例如，“id DECIMAL(38,0)， name STRING”。您还可以指定部分字段，其他字段使用默认的类型映射。此选项适用于读操作。 |



### 4.7 Avro 文件

Avro是一个数据序列化系统，设计用于支持大批量数据交换的应用。

它的主要特点有：支持二进制序列化方式，可以便捷，快速地处理大量数据；动态语言友好，Avro提供的机制使动态语言可以方便地处理Avro数据。

**读写数据**

```scala
val usersDF = spark.read.format("avro").load("examples/src/main/resources/users.avro")
usersDF.select("name","favorite_color").write.format("avro").save("namesAndFavColors.avro")
```



## 5 Spark SQL 常用函数

### 5.1 聚合函数

| 函数                                                    | 参数描述 | 返回值 | 作用                                                         | 示例 |
| ------------------------------------------------------- | -------- | ------ | ------------------------------------------------------------ | ---- |
| count(columnName: String)                               |          |        | 计数                                                         |      |
| countDistinct(columnName: String, columnNames: String*) |          |        | 去重计数                                                     |      |
| approx_count_distinct(columnName: String, rsd: Double)  |          |        | 近似值，适用于大型数据集。其中 columnName 为列名，rsd 为允许的最大误差。 |      |
| first(columnName: String)                               |          |        | 返回指定列的第一个值                                         |      |
| last(columnName: String)                                |          |        | 返回指定列的最后一个值                                       |      |
| min(columnName: String)                                 |          |        | 返回指定列的最小值                                           |      |
| max(columnName: String)                                 |          |        | 返回指定列的最大值                                           |      |
| sum(columnName: String)                                 |          |        | 返回指定列的所有元素的和                                     |      |
| sumDistinct(columnName: String)                         |          |        | 返回指定列所有元素去重后的元素的和                           |      |
| avg(columnName: String)                                 |          |        | 返回指定列的所有元素的平均值                                 |      |



### 5.2 集合函数

| 函数 | 参数描述 | 返回值 | 作用 | 示例 |
| ---- | -------- | ------ | ---- | ---- |
|      |          |        |      |      |

### 5.3 时间相关函数

| 函数 | 参数描述 | 返回值 | 作用 | 示例 |
| ---- | -------- | ------ | ---- | ---- |
|      |          |        |      |      |

### 5.4 数学计算函数

| 函数                                                 | 参数描述 | 返回值 | 作用                                                         | 示例 |
| ---------------------------------------------------- | -------- | ------ | ------------------------------------------------------------ | ---- |
| rand()                                               |          |        | 从U[0.0, 1.0]中生成一个具有独立且同分布(i.i.d)样本的随机列。 |      |
| sqrt(colName: String)                                |          |        | 计算指定 float 值的平方根                                    |      |
| var_pop(columnName: String)                          |          |        | 方差                                                         |      |
| var_samp(columnName: String)                         |          |        | 无偏方差                                                     |      |
| stddev_pop(columnName: String)                       |          |        | 总体标准差                                                   |      |
| stddev_samp(columnName: String)                      |          |        | 样本标准差                                                   |      |
| skewness(columnName: String)                         |          |        | 偏度                                                         |      |
| kurtosis(columnName: String)                         |          |        | 峰度                                                         |      |
| corr(columnName1: String, columnName2: String)       |          |        | 返回两列的皮尔逊相关系数（Pearson ）                         |      |
| covar_samp(columnName1: String, columnName2: String) |          |        | 返回两列的样本协方差                                         |      |
| covar_pop(columnName1: String, columnName2: String)  |          |        | 返回两列的总体协方差                                         |      |



### 5.5 杂七杂八函数

| 函数                          | 参数描述 | 返回值 | 作用                                                         | 示例 |
| ----------------------------- | -------- | ------ | ------------------------------------------------------------ | ---- |
| hash(cols: Column*)           |          |        | 计算给定列的 hash 值，并以 int 的方式返回。                  |      |
| md5(e: Column)                |          |        | 计算 binary 列的 MD5 摘要，返回 32 个十六进制字符串形式返回该值。 |      |
| sha1(e: Column)               |          |        | 计算 binary 列的 SHA-1 摘要并以 40 个十六进制字符串形式返回该值。 |      |
| sha2(e: Column, numBits: Int) |          |        | 计算 binary 列的 SHA-2 hash 函数族，并以十六进制字符串的形式返回该值。 |      |

### 5.6 非聚合函数

| 函数                            | 参数描述 | 返回值 | 作用                                                         | 示例 |
| ------------------------------- | -------- | ------ | ------------------------------------------------------------ | ---- |
| column(colName: String): Column |          |        | 根据给定的列返回一个列。                                     |      |
| isnan(e: Column): Column        |          |        | 如果列是 NaN，则返回 true。                                  |      |
| isnull(e: Column): Column       |          |        | 如果该列是 null，则返回 true。                               |      |
| expr(expr: String): Column      |          |        | 将表达式字符串解析为它所表示的列，类似于 Dataset#selectExpr。 |      |

### 5.7 排序函数

| 函数                                         | 参数描述 | 返回值 | 作用                                               | 示例                                          |
| -------------------------------------------- | -------- | ------ | -------------------------------------------------- | --------------------------------------------- |
| asc(columnName: String): Column              |          |        | 根据列的升序返回排序表达式。                       | df.sort(asc("dept"), desc("age"))             |
| asc_nulls_first(columnName: String): Column  |          |        | 根据列的升序返回排序表达式，空值在非空值之前返回。 | df.sort(asc_nulls_first("dept"), desc("age")) |
| asc_nulls_last(columnName: String): Column   |          |        | 根据列的升序返回排序表达式，空值出现在非空值之后。 | df.sort(asc_nulls_last("dept"), desc("age"))  |
| desc(columnName: String): Column             |          |        |                                                    |                                               |
| desc_nulls_first(columnName: String): Column |          |        |                                                    |                                               |
| desc_nulls_last(columnName: String): Column  |          |        |                                                    |                                               |

### 5.8 字符串函数

| 函数                                               | 参数描述 | 返回值 | 作用                                                         |
| -------------------------------------------------- | -------- | ------ | ------------------------------------------------------------ |
| base64(e: Column): Column                          |          |        | 计算 binary 列的 BASE64 编码并将其作为字符串列返回。这与unbase64正好相反。 |
| unbase64(e: Column): Column                        |          |        |                                                              |
| length(e: Column): Column                          |          |        | 计算给定字符串的字符长度或二进制字符串的字节数。             |
| split(str: Column, pattern: String)                |          |        | 使用正则模式拆分字符串                                       |
| substring(str: Column, pos: Int, len: Int): Column |          |        | 当str为字符串类型时，子字符串以pos开头，长度为len;当str为二进制类型时，返回以pos开头的字节数组片段，长度为len。 |
| trim(e: Column, trimString: String): Column        |          |        | 从指定字符串列的两端修剪指定的字符                           |
| trim(e: Column): Column                            |          |        | 为指定的字符串列从两端修剪空格                               |
| lower(e: Column): Column                           |          |        | 将字符串列转换为小写                                         |
| upper(e: Column): Column                           |          |        | 将字符串列转换为大写                                         |

### 5.9 UDF 函数

| 函数                                                    | 参数描述 | 返回值                                     | 作用 |
| ------------------------------------------------------- | -------- | ------------------------------------------ | ---- |
| udf(f: AnyRef, dataType: DataType): UserDefinedFunction |          | 使用Scala闭包定义确定性用户定义函数(UDF)。 |      |

```scala
// 1- 定义一个 udf，将多个字段合并为一个字段。
def allInOne(seq: Seq[Any], sep: String): String = seq.mkString(sep)

// 2- 在 sql 中使用
sqlContext.udf.register("allInOne", allInOne _)
//将col1,col2,col3三个字段合并，使用','分割
val sql =
    """
      |select allInOne(array(col1,col2,col3),",") as col
      |from tableName
    """.stripMargin
sqlContext.sql(sql).show()

// 在 DataFrame 中使用
import org.apache.spark.sql.functions.{udf，array,lit}
val myFunc = udf(allInOne _)
val cols = array("col1","col2","col3")
val sep = lit(",")
df.select(myFunc(cols,sep).alias("col")).show()
```



### 5.10 窗口函数

| 函数                 | 参数描述 | 返回值 | 作用                                               |
| -------------------- | -------- | ------ | -------------------------------------------------- |
| rank(): Column       |          |        | 窗口函数:返回窗口分区内的行秩                      |
| row_number(): Column |          |        | 窗口函数:在一个窗口分区中返回一个从1开始的序列号。 |



## 6 Spark SQL 运行原理

DataFrame、DataSet 和 Spark SQL 的执行流程是相同的：

1. DataFrame/DataSet/SQL 编码；
2. 如果代码无编译错误，Spark 将其转换为逻辑计划；
3. Spark 将次逻辑计划转换为物理计划，同时进行代码优化；
4. Spark 在集群上执行该物理计划（基于 RDD 操作）。



### 6.1 逻辑计划（Logical Plan）

执行的第一个阶段是将用户代码转换成一个逻辑计划。它首先将用户代码转换成 `unresolved logical plan`（未解决的逻辑计划），之所以这个计划是未解决的，是因为尽管你的代码在语法上是正确的，但是它引用的表或列可能不存在。Spark 使用 `analyzer`（分析器）基于 `catalog`（存储的所有表和 `DataFrames` 的信息）进行解析。解析失败则拒绝执行，解析成功则将结果传给 `Catalyst` 优化器（`Catalyst Optimizer`），优化器是一组规则的集合，用于优化逻辑执行，通过谓词下推等方式进行优化，最终输出优化后的逻辑执行计划。

![spark-Logical-Planning](https://gitee.com/struggle3014/picBed/raw/master/spark-Logical-Planning.png)



### 6.2 物理计划（Physical Plan）

得到优化后的逻辑计划后，Spark 就开始了物理计划过程。它通过生成不同的物理执行计划，并通过成本模型来对比它们，从而选择一个最优的物理计划在集群上main执行。物理计划的输出结果是一系列的 RDDs 和转换关系（transformations）。

![spark-Physical-Planning](https://gitee.com/struggle3014/picBed/raw/master/spark-Physical-Planning.png)



### 6.3 执行（Execute）

在选择一个物理计划后，Spark 运行其 RDDs 代码，并在运行时执行进一步的优化，生成本地 Java 字节码，最后将运行结果返回给用户。



# 参考文献

[1] [Spark 官方文档—Spark SQL Guide](http://spark.apache.org/docs/latest/sql-programming-guide.html)

[2] [Spark 官方文档—spark sql 函数 API](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.functions$)

[3] [GitHub—wangzhiwubigdata—大数据](https://github.com/wangzhiwubigdata/God-Of-BigData)

[4] [Apache Avro 官网](https://avro.apache.org/)