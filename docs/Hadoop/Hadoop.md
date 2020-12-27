<div align="center"><img src="https://gitee.com/struggle3014/picBed/raw/master/name_code.png"></div>

# 导读

本文起，开始进入大数据的世界。我们深入大数据的基石项目 Hadoop。



# 目录

<nav>
<a href='#导读' style='text-decoration:none;font-weight:bolder'>导读</a><br/>
<a href='#目录' style='text-decoration:none;font-weight:bolder'>目录</a><br/>
<a href='#正文' style='text-decoration:none;font-weight:bolder'>正文</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1 大数据启蒙' style='text-decoration:none;${border-style}'>1 大数据启蒙</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2 Hadoop HDFS' style='text-decoration:none;${border-style}'>2 Hadoop HDFS</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3 Hadoop 安装部署' style='text-decoration:none;${border-style}'>3 Hadoop 安装部署</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4 Hadoop MapReduce' style='text-decoration:none;${border-style}'>4 Hadoop MapReduce</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5 MapReduce 源码分析' style='text-decoration:none;${border-style}'>5 MapReduce 源码分析</a><br/>
<a href='#总结' style='text-decoration:none;font-weight:bolder'>总结</a><br/>
<a href='#参考文献' style='text-decoration:none;font-weight:bolder'>参考文献</a><br/>
</nav>

# 正文

## 1 [大数据启蒙](./01_大数据启蒙.md)

* 分治思想
* 单机处理大数据问题
* 集群分布式处理大数据的辩证
* 结论
* Hadoop 项目介绍

## 2 [Hadoop HDFS](./02_Hadoop-HDFS.md)

* Hadoop 项目中 HDFS 存在的意义

* 理论知识点

  * 架构模型

  * 架构设计

  * 角色功能

  * 元数据持久化

  * 安全模式

  * 副本放置策略

  * 读写流程

* HDFS HA
  * HDFS 架构优缺点剖析
  * HDFS HA 解决方案
    * CAP 原则
    * Paxos 算法
    * HA 方案
  * HDFS Federation 解决方案
* HDFS 权限管理

## 3 [Hadoop 安装部署](./03_Hadoop安装部署.md)

* 基础环境设置
* 应用搭建，使用及验证
  * 单节点集群
  * 伪分布式集群
  * 分布式集群
  * HA 集群

## 4 [Hadoop MapReduce](./04_Hadoop-MapReduce.md)

* MapReduce 过程
  * 粗粒度过程
  * 细粒度过程
* MapReduce 开发

## 5 [MapReduce 源码分析](./05_MapReduce源码分析.md)

* Client
* MapTask
  * Input
  * Map
  * Output
    * 环形缓冲区
* ReduceTask
  * Input
  * Reduce
  * Output



# 总结



# 参考文献

[1] [Apache Hadoop 官网](http://hadoop.apache.org/)

[2] [Process On Hadoop ](https://www.processon.com/view/link/5f27b76ae401fd181aeb1496)