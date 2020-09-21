

<div align="center"><img src="https://gitee.com/struggle3014/picBed/raw/master/name_code.png"></div>

# 导读

本文起，正式走进 HBase 的世界。

***持续更新中~***



# 目录

<nav>
<a href='#导读' style='text-decoration:none;font-weight:bolder'>导读</a><br/>
<a href='#目录' style='text-decoration:none;font-weight:bolder'>目录</a><br/>
<a href='#正文' style='text-decoration:none;font-weight:bolder'>正文</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1 HBase 简介' style='text-decoration:none;${border-style}'>1 HBase 简介</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2 HBase 搭建' style='text-decoration:none;${border-style}'>2 HBase 搭建</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3 Protobuf 简介' style='text-decoration:none;${border-style}'>3 Protobuf 简介</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4 HBase 基本操作' style='text-decoration:none;${border-style}'>4 HBase 基本操作</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5 HBase 常用数据结构' style='text-decoration:none;${border-style}'>5 HBase 常用数据结构</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#6 HBase 读取流程' style='text-decoration:none;${border-style}'>6 HBase 读取流程</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#7 HBase 优化' style='text-decoration:none;${border-style}'>7 HBase 优化</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#8 HBase 集群规划' style='text-decoration:none;${border-style}'>8 HBase 集群规划</a><br/>
<a href='#总结' style='text-decoration:none;font-weight:bolder'>总结</a><br/>
<a href='#参考文献' style='text-decoration:none;font-weight:bolder'>参考文献</a><br/>
</nav>

# 正文

## 1 [HBase 简介](./01_HBase简介.md)

* 关系型数据库与非关系型数据库
* HBase 简介
* HBase 数据模型
* HBase 机构
* HBase 读写流程

## 2 [HBase 搭建](./02_HBase搭建.md)
* Standalone HBase
* Fully-distributed HBase

## 3 [Protobuf 简介](./03_Protobuf.md)



## 4 [HBase 基本操作](./04_HBase基本操作.md)

* 通用命令
* DDL 操作
* namespace 操作
* DML 操作

## 5 [HBase 常用数据结构](./05_HBase常用数据结构.md)

* LSM 树
* 跳跃表
* 布隆过滤器

## 6 [HBase 读取流程](./06_HBase读写流程.md)

* Client-Server 交互逻辑
* HBase 度流程
  * 简化版本
  * 详细版本
* HBase 写流程

## 7 [HBase 优化]((./07_HBase优化设计.md))

* 表的设计
* 写表操作
* 读表操作

## 8 HBase 集群规划

![HBase集群规划](https://gitee.com/struggle3014/picBed/raw/master/HBase集群规划.png)

<div align="center"><font size="2"><a href="../MindMapping/HBase集群规划.xmind">HBase 集群规划</a></font></div>



# 总结



# 参考文献

[1] [HBase 官网](https://hbase.apache.org/)