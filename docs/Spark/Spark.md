<div align="center"><img src="https://gitee.com/struggle3014/picBed/raw/master/name_code.png"></div>

# 导读

本文是 Spark 的导航页，介绍 Spark 的相关知识。

***持续更新中~***



# 目录

<nav>
<a href='#导读' style='text-decoration:none;font-weight:bolder'>导读</a><br/>
<a href='#目录' style='text-decoration:none;font-weight:bolder'>目录</a><br/>
<a href='#正文' style='text-decoration:none;font-weight:bolder'>正文</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1 Spark简介' style='text-decoration:none;${border-style}'>1 Spark简介</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2 Spark Core' style='text-decoration:none;${border-style}'>2 Spark Core</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3 Spark Streaming' style='text-decoration:none;${border-style}'>3 Spark Streaming</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4 Spark SQL' style='text-decoration:none;${border-style}'>4 Spark SQL</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5 Structured Streaminig' style='text-decoration:none;${border-style}'>5 Structured Streaminig</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6 Spark 优化' style='text-decoration:none;${border-style}'>6 Spark 优化</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#7 超越批处理的流处理' style='text-decoration:none;${border-style}'>7 超越批处理的流处理</a><br/>
<a href='#总结' style='text-decoration:none;font-weight:bolder'>总结</a><br/>
<a href='#参考文献' style='text-decoration:none;font-weight:bolder'>参考文献</a><br/>
</nav>

# 正文

## 1 [Spark简介](./Spark简介.md)

* 简介
* 特点
* 集群架构
* 核心组件



## 2 [Spark Core](./SparkCore.md)

* RDD
  * RDD 简介
  * RDD 特点
  * RDD 创建
  * RDD 操作
* 共享变量
  * 广播变量
  * 累加器



## 3 [Spark Streaming](./SparkStreaming)

* 原理概述
* 使用技巧
* 使用限制
* 项目实战



## 4 [Spark SQL](./SparkSQL.md)

* Spark SQL 简介
* DataFrame，DataSet，RDD
* Structured API 使用
* 外部数据源
* Spark SQL 常用函数
* Spark SQL 运行原理



## 5 [Structured Streaminig](./SparkStreaming.md)

* 简介
* 快速例子
* 编程模型
* 使用 Datasets 和 DataFrame 的 API
* 连续处理



## 6 [Spark 优化](./Spark优化.md)

![Spark调优](https://gitee.com/struggle3014/picBed/raw/master/Spark调优.png)

<div align="center"><font size="2"><a href="../MindMapping/Spark调优.xmind"/>Spark调优</a></font></div>



## 7 [Spark 算子](./Spark算子)

![Spark算子](https://gitee.com/struggle3014/picBed/raw/master/Spark算子.png)

<div align="center"><font size="2"><a href="../MindMapping/Spark算子.xmind"/>Spark算子</a></font></div>



## 8 [超越批处理的流处理](./超越批处理的流处理.md)

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