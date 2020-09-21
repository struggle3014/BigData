<div align="center"><img src="https://gitee.com/struggle3014/picBed/raw/master/name_code.png"></div>

# 导读

本文是 Spark 的导航页，介绍 Spark 的相关知识。

***持续更新中~***



# 目录



# 正文

## 1 [Spark简介](./Spark简介.md)

* 简介
* 特点
* 集群架构
* 核心组件



## 2 [Spark Core](./Spark Core.md)

* RDD
  * RDD 简介
  * RDD 特点
  * RDD 创建
  * RDD 操作
* 共享变量
  * 广播变量
  * 累加器



## 3 [Spark Streaming](./Spark Streaming)

* 原理概述
* 使用技巧
* 使用限制
* 项目实战



## 4 [Spark SQL](./Spark SQL.md)

* Spark SQL 简介
* DataFrame，DataSet，RDD
* Structured API 使用
* 外部数据源
* Spark SQL 常用函数
* Spark SQL 运行原理



## 5 [Structured Streaminig](./Spark Streaming.md)

* 简介
* 快速例子
* 编程模型
* 使用 Datasets 和 DataFrame 的 API
* 连续处理



## 6 [Spark 优化](./Spark优化.md)

* Spark 开发优化
  * 避免创建重复 RDD
  * 极可能复用同一 RDD
  * 对多次使用的 RDD 进行持久化
  * 尽量避免使用 Shuffle 操作
  * 使用 map-side 预聚合的 Shuffle 操作
  * 使用高性能算子
  * 广播大便量
  * 使用 Kryo 优化序列化性能
  * 优化数据结构
* 资源调优
  * Spark 作业基本运行原理
  * 资源参数调优
* 数据倾斜调优
  * 数据倾斜发生时的现象
  * 数据倾斜发生的原理
  * 定位导致数据倾斜的代码
  * 数据倾斜的方案
    * 使用 Hive ETL 预处理数据
    * 过滤少数导致倾斜的 key
    * 提高 shuffle 操作的并行度
    * 两阶段操作（局部聚合+全局聚合）
    * 将 reduce join 转为 map join
    * 采用倾斜 key 并分拆 join 操作
    * 使用随机前缀和扩容 RDD 进行 join
    * 多种方案组合使用
* Shuffle 调优
  * ShuffleManager 发展概述
  * HashShuffleManager 运行原理
  * SortShuffleManager 运行原理
  * Shuffle 相关参数调优



## 7 [超越批处理的流处理](./超越批处理的流处理.md)

* 常见术语
  * 流是什么
  * 时域
  * 窗口（Window）
  * 水位线（Watermark）
  * 触发器（Trigger）
  * 容忍延迟（垃圾回收）
  * 堆积（Accumulation）
* 数据处理模式
  * 有界数据
  * 无界数据（批处理）
  * 无界数据（流处理）
* 确定能力边界
* 正确性如何实现
  * 有状态的流式处理
  * 状态管理
* 时间推理工具
  * What result are calculated?（计算了什么结果）
  * Where is event time are result calculated?（event time 在哪里计算）
  * When in processing time are result materialized?（在 processing time 中何时将结果物化）
  * How do refinements of results relate?（结果的细化是如何关联的）



# 总结



# 参考文献

[1] [Spark 官方文档]()

[2] [Spark 内核，设计与实现，GitHub，JerryLead](https://github.com/JerryLead/SparkInternals)