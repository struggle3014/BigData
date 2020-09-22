<div align="center"><img src="https://gitee.com/struggle3014/picBed/raw/master/name_code.png"></div>

# 导读

本文介绍 Spark Streaming 内容。

***持续更新中~***



# 目录

<nav>
<a href='#导读' style='text-decoration:none;font-weight:bolder'>导读</a><br/>
<a href='#目录' style='text-decoration:none;font-weight:bolder'>目录</a><br/>
<a href='#正文' style='text-decoration:none;font-weight:bolder'>正文</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1 原理概述' style='text-decoration:none;${border-style}'>1 原理概述</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2 项目实战' style='text-decoration:none;${border-style}'>2 项目实战</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3 使用技巧' style='text-decoration:none;${border-style}'>3 使用技巧</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3.1 优雅关闭' style='text-decoration:none;${border-style}'>3.1 优雅关闭</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3.2 监控预警' style='text-decoration:none;${border-style}'>3.2 监控预警</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4 使用限制' style='text-decoration:none;${border-style}'>4 使用限制</a><br/>
<a href='#总结' style='text-decoration:none;font-weight:bolder'>总结</a><br/>
<a href='#参考文献' style='text-decoration:none;font-weight:bolder'>参考文献</a><br/>
</nav>

# 正文

## 1 原理概述

![spark-streaming-theory](https://gitee.com/struggle3014/picBed/raw/master/spark-streaming-theory.png)

spark 程序是使用一个 spark 应用实例一次性对一批历史数据进行处理，spark streaming
是将持续不断输入的数据流转换成多个 batch 分片，使用一批 spark 应用实例进行处理。

## 2 项目实战

[rockfish 项目](http://gitlab.xinyan.com/bigdata/device-data-apply)



## 3 使用技巧

### 3.1 优雅关闭

Spark Streaming 支持三种方式：

* 人工介入

  sparkConf.set("spark.streaming.stopGracefullyOnShutdown","true")//优雅的关闭

  * 通过 Hadoop 8088 页面找到运行的程序
  * 打开 spark ui 的监控页面
  * 打开 executor 的监控页面
  * 登录 liunx 找到驱动节点（Driver）所在的机器 ip 以及运行的端口号
    * ps -ef |grep java |grep ApplicationMaster|grep applicationId
      kill -SIGTERM <AM-PID>发送 SIGTERM 信号

* 使用 HDFS 系统做消息通知

  ```scala
  /**
      * 通过标记文件关闭实时流
      *
      * @param ssc
      * @param paramBean 参数对象
      */
    def stopByMarkFile(ssc: StreamingContext)(implicit paramBean: ParamBean): Unit = {
      var isStop: Boolean = false
      val config = paramBean.nonSerParamBean.nonSerRealtimeParamBean.config
      val intervalMills = 10 * 1000L
      val hookPath = config.getProperty("spark.hook")
      if (StringUtils.isNotBlank(hookPath)) {
        isStop = ssc.awaitTerminationOrTimeout(intervalMills)
        if (!isStop && !HDFSUtil.isExist(new Path(hookPath))) {
          // 优雅关闭
          kafkaLogger.warn(s"RockfishRealtimeApp | stopByMarkFile | app is stopping at ${System.currentTimeMillis()}")
          ssc.stop(true, true)
        }
      }
    }
  ```

* 内部暴露 socket 或 http 端口来接收请求，等待触发关闭流程序

  ```scala
  /**
      * 负责守护启动的 jetty 服务
      * @param port 对外暴露的端口号
      * @param ssc Stream 上下文
      */
    def daemonHttpServer(port:Int, ssc: StreamingContext): Unit = {
      val server = new Server(port)
      val context = new ConextHandler();
      context.setContextPath("/close")
      context.setHandler(new CloseStreamHandler(ssc))
      server.setHandler(context)
      server.start()
    }
  
    /**
      * 负责接收 http 请求实现优雅关闭
      * @param ssc
      */
    class CloseStreamHandler(ssc: StreamingContext) extends AbstractHandler {
      override def handle(target: String, baseRequest: Request, request: HttpServletRequest, response: HttpServletResponse): Unit = {
        ssc.stop(true, true) // 优雅关闭
        response.setContentType("text/html;charset=utf-8")
        response.setStatus(HttpServletResponse.SC_OK)
        val out = response.getWriter
        out.println("close success")
        baseRequest.setHandled(true)
      }
    }
  ```

  

### 3.2 监控预警

#### 3.2.1监控指标

cpu, memory, disk, jvm, source metrics, sink metrics, inner metrics

#### 3.2.2 监控维度

*  stream data
*   delay
*   back pressure
*   throughput
*   state
*   checkpoint

#### 3.2.3 告警范围

*  应用异常
*  应用延迟
*  应用宕机
*  故障恢复

#### 3.3 调优之路

##### 3.3.1 DAG 静态定义（DStreamGraph）

算子的使用优化，和 Spark 调优内容是相同的。

* 重复使用的 `RDD` 进行 `cache`
* 使用高性能算子代替低性能的算子
  * `reduceByKey`/`aggerateByKey` 替代 `groupByKey`
  * 使用 `mappartition` 代替 `map`
  * 使用 `foreachPartition` 替代 `foreach`
* 使用 `Kyro` 序列化替代 `Java` 序列化
* filter  之后使用 `colaesce` 减少小任务 

##### 3.3.2 Job 动态生成

该部分主要涉及调优的是 `batch interval`，为了使程序不延迟地运行，设置合理的
`batch interval`。

##### 3.3.3 数据的产生与导入

该部分主要是针对数据的接收速度进行调优，如果接收速度大于处理速度，那么程序
会走向无限延迟，所以主要的调优在于限速。

* `recevier` 和 `direct approach` 方式都通用的
  `spark.streaming.backpressure.enabled=true`；框架会自动计算速度来控制数据的接受
  `速度，建议开启。
* receiver 方式
  `spark.streaming.receiver.maxRate` 来进行限速。
  `spark.streaming.blockInternal` 设置缓存在内存块的大小，防止内存被撑爆。
* direct approach方式
  `spark.streaming.kafka.maxRatePartition` 来对每个分区进行限速。

##### 3.3.4 容错

热备：默认开启数据备份数为2。
冷备：开启 `WAL`，将 log 保存到 `HDFS` 上，`executor` 挂掉后可以从 `HDFS` 上进行数据的恢复。
重放：对于数据源本身支持重放有效，如 `Kafka`，失效后可以通过 `offset` 值进行恢复。



## 4 使用限制

#### 4.1 高延迟

##### 4.1.1 流和微批（`Micro Batching`）

1. 微批（`Spark`）
   Micro-Batching 计算模式认为“流是批的特例”，流计算就是连续不断的批进行
   持续计算，如果批有足够小的延迟，就能满足绝大部分的实时计算场景。
2. `Natvie Streaming`（`Flink`）
   `Natvie Streaming` 计算认为“批是流的特例”，这个认知更贴近流的概念。`Natvie
    Streaming` 计算模式是每条数据到来都进行计算，这种计算更接近自然，延迟
   性能更低。

![micro-batching](https://gitee.com/struggle3014/picBed/raw/master/micro-batching.png)

![native-streaming](https://gitee.com/struggle3014/picBed/raw/master/native-streaming.png)

##### 4.1.2 延迟对比

| 计算引擎             | 延迟级别 |      |
| -------------------- | -------- | ---- |
| Spark Streaming      | 秒级     |      |
| Structured Streaming | 毫秒级   |      |
| Flink                | 亚秒级   |      |

#### 4.2 状态存储

`Spark Streaming`

* 内存数据大，快照不宜太频繁，快照会增加集群计算量。
* Spark 的状态管理交简单，仅支持 `updateStateByKey` 和 `mapWithState` 算子。



`Flink`

* `Flink` 支持文件、内存、`RocksDB` 三种存储状态，可对运行中的状态数据异步
  持久化。打快照的机制是给 source 节点的下一个节点发一条特殊的 `savepoint`
   或 `checkpoint` 消息，这条消息在每个算子之间流动，通过协调者机制对齐多
  个并行度的算子中的状态数据，把状态数据异步持久化。
* 支持局部恢复快照。
* 作业快照保存后，支持修改作业及 `DAG` 优化，并启动作业恢复快照。
* 支持增量快照。在面对内存超大状态数据，能够有效降低网络和磁盘开销。



# 总结



# 参考文献

