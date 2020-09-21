<div align="center"><img src="https://gitee.com/struggle3014/picBed/raw/master/name_code.png"></div>

# 导读

HBase 简介

***持续更新中~***



# 目录

<nav>
<a href='#导读' style='text-decoration:none;font-weight:bolder'>导读</a><br/>
<a href='#目录' style='text-decoration:none;font-weight:bolder'>目录</a><br/>
<a href='#正文' style='text-decoration:none;font-weight:bolder'>正文</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1 关系型数据库与非关系型数据库' style='text-decoration:none;${border-style}'>1 关系型数据库与非关系型数据库</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1）关系型数据库' style='text-decoration:none;${border-style}'>1）关系型数据库</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2）非关系型数据库' style='text-decoration:none;${border-style}'>2）非关系型数据库</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2 HBase 简介' style='text-decoration:none;${border-style}'>2 HBase 简介</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3 HBase 数据模型' style='text-decoration:none;${border-style}'>3 HBase 数据模型</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1）Rowkey' style='text-decoration:none;${border-style}'>1）Rowkey</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2）Column Family & Qualifier' style='text-decoration:none;${border-style}'>2）Column Family & Qualifier</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3）TimeStamp' style='text-decoration:none;${border-style}'>3）TimeStamp</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4）Cell' style='text-decoration:none;${border-style}'>4）Cell</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4 HBase 架构' style='text-decoration:none;${border-style}'>4 HBase 架构</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1）角色介绍' style='text-decoration:none;${border-style}'>1）角色介绍</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2）RegionServer 组件介绍' style='text-decoration:none;${border-style}'>2）RegionServer 组件介绍</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3）注意' style='text-decoration:none;${border-style}'>3）注意</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#5 HBase 读写流程' style='text-decoration:none;${border-style}'>5 HBase 读写流程</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1）读流程' style='text-decoration:none;${border-style}'>1）读流程</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2）写流程' style='text-decoration:none;${border-style}'>2）写流程</a><br/>
<a href='#总结' style='text-decoration:none;font-weight:bolder'>总结</a><br/>
<a href='#参考文献' style='text-decoration:none;font-weight:bolder'>参考文献</a><br/>
</nav>

# 正文

## 1 关系型数据库与非关系型数据库

### 1）关系型数据库

![关系型数据库](https://gitee.com/struggle3014/picBed/raw/master/关系型数据库.png)

<div align="center"><font size="2">关系型数据库</font></div>

关系型数据库中最典型的数据结构是表，由二维表机器之间的联系组成一个数据组织。

优点：

1，易于维护：都是使用表结构，格式一致。

2，使用方便：SQL 语言通用，可用于复杂查询。

3，复杂操作：支持 SQL，可用于一个表以及多个表之间复杂查询。

缺点：

1，读写性能较差，尤其是海量数据的高效率读写。

2，固定的表结构，灵活度稍欠。

3，高并发读写需求，传统关系型数据库，磁盘 I/O 是很大的瓶颈。

### 2）非关系型数据库

![关系型数据库](https://gitee.com/struggle3014/picBed/raw/master/非关系型数据库.png)

<div align="center"><font size="2">非关系型数据库</font></div>

非关系型数据库严格上不是一种数据库，应该是一种数据结构化存储方法的集合，可以是文档或键值对。

优点：

1，格式灵活：存储数据的格式可以使 key，value 形式、文档形式、图片形式等，使用灵活，应用场景广泛，而关系型数据库则只支持基础类型。

2，速度快：NoSQL 可以使用硬盘或随机存储器作为载体，而关系型数据库只能使用磁盘。

缺点：

1，不提供 SQL 支持，学习和使用成本较高。

2，无事务处理。

3，数据结构相对复杂，复杂查询方面稍欠。



## 2 HBase 简介

Apache HBase 官网对 HBase 的介绍：

```
Use Apache HBase™ when you need random, realtime read/write access to your Big Data. This project's goal is the hosting of very large tables -- billions of rows X millions of columns -- atop clusters of commodity hardware. Apache HBase is an open-source, distributed, versioned, non-relational database modeled after Google's Bigtable: A Distributed Storage System for Structured Data by Chang et al. Just as Bigtable leverages the distributed data storage provided by the Google File System, Apache HBase provides Bigtable-like capabilities on top of Hadoop and HDFS.
```

HBase 的全称是 Hadoop Database，是一个高可靠性，高性能，面向列，可伸缩，实时读写的分布式数据库。

利用 Hadoop HDFS 作为其文件存储系统，利用 Hadoop MapReduce 来处理 HBase 中的海量数据，利用 Zookeeper 作为其分布式协同服务。

主要用来存储非结构化和半结构化数据的松散数据（列存 NoSQL 数据库）。

注意：NoSQL 的全称是 Not Only SQL，泛指非关系型数据库。



## 3 HBase 数据模型

![hbase数据模型](https://gitee.com/struggle3014/picBed/raw/master/hbase数据模型.png)

<div align="center"><font size="2">HBase 数据模型</font></div>

### 1）Rowkey

1，决定一行数据，每条记录的唯一标识

2，按照字典序排序

3，Rowkey 只能存储 64K 的字节数据

### 2）Column Family & Qualifier

1，HBase 表中的每个列都属于某个列族，列族必须作为表模式（schema）定义的一部分预先给出。如 create "test", "course";

2，列名以列族作为前缀，每个列族 Column Family 都可以有多个列 Column。如 course:math，course:english。

3，权限控制、存储以及调优都是在列族层面进行的。

4，HBase 把统一列族中的数据存储在统一目录下，由几个文件保存。

### 3）TimeStamp

1，在 HBase 每个 cell 存储单元对同一份数据有多个版本，根据唯一的时间戳来区分每个版本之间的差异，不同版本的数据按照时间序列倒叙，最新的数据版本排在最前面。

2，时间戳的类型是 64 位整形。

3，时间戳可以由 HBase 在数据写入时自动赋值，此时间戳是精确到毫秒的当前系统时间。

4，时间戳也可以由客户显示赋值，如果应用程序要避免数据版本冲突，就必须自己生成唯一性的时间戳。

### 4）Cell

1，由行和列的坐标交叉决定

2，单元格是有版本的

3，单元格的内容是未解析的字节数组

​	1，由 {rowkey, column, version} 唯一确定的单元。

​	2，cell 中的数据是没有类型的，都是字节数组的形式存储。



## 4 HBase 架构

![hbase架构图](https://gitee.com/struggle3014/picBed/raw/master/hbase架构图.png)

<div align="center"><font size="2">HBase 架构</font></div>

### 1）角色介绍

#### Client

包含访问 HBase 的接口并维护 cache 来加快对 HBase 的访问。

#### Zookeeper

1，任何时候保证集群中只有一个活跃的 Master。

2，存储所有 region 的寻址入口。

3，实时监控 Region Server 的上线和下线信息，并实时通知 Master。

#### Master

1，为 Region Server 分配 Region。

2，负责 Region Server 的负载均衡。

3，发现实现的 Region Server，并重新分配其上的 Region。

4，管理用户对 table 的增删改查操作。

#### RegionServer

1，Region Server 维护 Region，处理这些 Region 的 I/O 请求。

2，Region Server 负责切分在运行过程中变得过大的 Region。

### 2）RegionServer 组件介绍

#### Region

1，HBase 自动把表水平划分成多个区域 Region，每个 Region 保存一个表中某段连续的数据。

2，每个表默认只有一个 Region，随着数据不断插入表，Region 不断增大，当增大到一个阈值的时，Region 会等分成两个新的 Region（裂变）。

3，当 table 中的行不断增多，就会有越来越多的 Region。这样一张完整的表被保存在多个 RegionServer。

#### Memsotre 和 StoreFile

1，一个 Region 由多个 Store，一个 Store 对应一个列族 CF。

2，Store 包括位于内存中的 memstore 和位于磁盘的 StoreFile 写操作先写入 Memstore，当 Memstore 中的数据达到某个阈值，HRegionServer 会启动 flashcash 线程写入 StoreFile，每次写入行程单独的一个 StoreFile。

3，当 StoreFile 文件的数量增长到一定阈值后，系统会进行合并（minor 和 major），在合并过程中会进行版本合并和删除工作（major），形成更大的 StoreFile。

4，当某个 Region 所有 StoreFile 的大小和数量超过一定阈值后，会把当前的 Region 分割成两个，并由 HMaster 分配相应的 RegionServer 服务器，实现负载均衡。

### 3）注意

1，HRegion 是 HBase 中分布式存储和负载均衡的最小单元。最小单元就表示图通的 HRegion 可以分布在不同的 HRegion Server 上。

2，HRegion 由一个或多个 Store 组成，每个 Store 保存一个 Column Family。

3，每个 Store 由一个 MemStore 和 0 至多个 StoreFile 组成。StoreFile 是以 HFile 格式保存在 HDFS 上。

![hbase架构图3](https://gitee.com/struggle3014/picBed/raw/master/hbase架构图3.png)

<div align="center"><font size="2">HBase 架构</font></div>



## 5 HBase 读写流程

### 1）读流程

1，客户端从 Zookeeper 中后去 meta 表所在的 RegionServer 节点信息。

2，客户端访问 meta 表所在的 RegionServer 节点，获取到 Region 所在的 RegionServer 信息。

3，客户端访问具体的 Region 所在的 RegionServer，找到对应的 Region 及 Store。

4，首先从 MetaStore 中读取数据，若读取到直接返回数据；若没有，则去 BlockCache 读取数据。

5，接着从 BlockCache 中读取，若读取到直接返回；若没有，则遍历 StoreFile 文件查找数据。

6，然后从 StoreFile 中读取数据，若读取先将数据缓存到 BlockCache 中（方便后续读取），然后返回；若没有，则给客户端返回空，

7，BlockCache 是内存空间，若缓存的数据较多，满了会采用 LRU 策略，将较老的数据删除。

### 2）写流程

1，客户端从 Zookeeper 中获取 meta 表所在的 RegionServer 节点信息。

2，客户端访问 meta 所在的 RegionServer 节点，获取到 Region 所在的 RegionServer 信息。

3，客户端访问具体 Region 所在的 RegionServer，找到对应的 Region 及 Store。

4，开始写数据，写数据的时候会先向 HLog 中写一份数据（方便 MetaStore 中数据丢失后能够根据 HLog 恢复数据，向 HLog 中写数据的时候会优先写入内存，后台会有一个线程定期异步刷写数据到 HDFS，若 HLog 的数据也写入失败，那么数据也就丢失了）。

5，HLog 写数据完成后，会先将数据写入到 MemStore，MemStore 默认大小是 64M，当 MemStore 满了之后会进行统一的溢写操作，将 MemStore 中的数据持久化到 HDFS 中。

6，频繁的溢写会导致产生很多小文件，因此会进行文件的合并，文件在合并的时候会有两种方式，即 Minor 和 Major。Minor 表示小范围文件的合并，Major 表示将所有 StoreFile 文件都合并成一个。



# 总结





# 参考文献

[1] [HBase 官网](http://hbase.apache.org/)

