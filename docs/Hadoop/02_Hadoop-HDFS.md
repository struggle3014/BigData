![name_code](https://gitee.com/struggle3014/picBed/raw/master/name_code.png)

# Hadoop HDFS

## 导读

Hadoop HDFS 围绕 What, Why, How 展开



## Hadoop 项目中 HDFS 存在的意义



## 理论知识点

### 存储模型

* 文件线性按字节切割成块（block），具有 offset，id

* 文件与文件的 block‘ 大小可以不一样

* 一个文件除最后一个 block，其他 block 大小一致

* block 的大小依据硬件的 I/O 的特性调整

* block 被分散存放在集群的节点中，具有 location

* block 具有副本（replication），没有主从概念，副本不能出现在同一节点上

* 副本是满足可靠性和性能的关键

* 文件上传可以指定 block 大小和副本数，上传后只能修改副本数

* 一次写入多次读取

* 不支持修改，支持追加数据

![image-20200802133011774](https://gitee.com/struggle3014/picBed/raw/master/block-replication.png)

<div align="center"><font size="2">HDFS 块的复制</font></div>

### 架构设计

* HDFS 是一个主从（Master/Slaves）架构
* 由一个 NameNode 和一些 DataNode 组成
* 面向文件包含：文件数据（data）和文件元数据（metadata）
* NameNode 负责存储和管理文件元数据，并维护一个层次型的文件目录树
* DataNode 负责存储文件数据（block 块），并提供 block 的读写
* DataNode 和 NameNode 维持心跳，并汇报自己持有的 block 信息
* Client 和 NameNode 交互文件元数据和 DataNode 交互文件 block 数据

![HDFS-Architecture](https://gitee.com/struggle3014/picBed/raw/master/HDFS-Architecture.png)

<div align="center"><font size="2">HDFS 架构</font></div>

### 角色功能

#### 补充知识

框架中的角色即 JVM 进程

#### 角色及其功能

NamNode

* 完全基于内存存储文件元数据、目录结构、文件 block 的映射
* 需要持久化方案保证数据可靠性
* 提供<font color="red">副本</font>放置策略

DataNode

* 基于本地磁盘存储 block（文件的形式），并保存 block 的校验和数据保证 block 的可靠性
* 与 NameNode 保持心跳，汇报 block 列表状态

SecondaryNameNode（SNN）

* 在非 HA 模式下，SNN 一般是单独的节点，周期性完成对 NameNode 的 EditLog 向 FsImage 合并，减少 EditLog 大小，减少 NameNode 启动时间。
* 根据配置文件设置的时间间隔 fs.checkpoint.period，默认为 3600 秒
* 根据配置文件设置 EditLog 大小 fs.checkpoint.size 规定 edits 文件的最大值，默认为 64M。

![image-20200802140011354](https://gitee.com/struggle3014/picBed/raw/master/HDFS-SecondaryNameNode.png)

<div align="center"><font size="2">HDFS-SencondaryNameNode 架构</font></div>

### 元数据持久化

#### 元数据持久化实现

* 任何对文件系统元数据产生修改的操作，NameNode 都会用一种称为 EditLog 的事务日志记录下来

* 使用 FsImage 存储内存所有的元数据状态
* 使用本地磁盘保存 EditLog 和 FsImage
* EditLog 具有完整性，数据丢失少，但恢复速度慢，并有体积膨胀风险
* FsImage 具有恢复速度快，体积与内存数据相当，但不能实时保存，数据丢失多
* NameNode 使用了 FsImage + EditLog 整合的方案：
  * 滚动将增量的 EditLog 更新到 FsImage，以保证更近时点的 FsImage 和更小体积的 EditLog

#### 补充知识

##### 数据持久化的实现方式

* **日志文件**。记录实时发生的增删改查的操作，如 mkdir, append 等。

  ​	<font color="red">优缺点：</font>

  * 完整性比较好
  * 加载恢复数据慢且占用空间。例如，NN 的内存为 4G，运行了 10 年，日志是多大？5年恢复，内存是否会溢出？

* **镜像、快照、dump、db、序列化**。	间隔（天，小时，分钟，秒），内存全量基于某一个时间点做的向磁盘的溢写。

  ​	<font color="red">优缺点：</font>

  * 恢复速度快过日志文件。
  * 由于间隔，容易丢失一部分数据。

##### FsImage 和 EditsLog 思考

EditsLog 日志。体积小，记录少，必然有优势

FsImage 镜像，快照。如果能更快地滚动更新时间点

获取 EditsLog 和 FsImage 的两者优势的方案：最近时点的 FsImage + 增量的 EditsLog

场景：如果现在是 10 点，FsImage：9 点的 FsImage + 9点到10点的增量的 EditsLog。

恢复步骤：

1）加载 FsImage

2）加载 EditsLog

3）内存就得到关机前的全量数据

问题：FsImage 时点是怎么滚动更新的？

解决方案：

1）由 NN 8点溢写一次，9点溢写一次...

该方案，没小时 NN 都会溢写一次，又 IO 较慢，会影响 NN 正常服务。

2）NN 只在第一次开机时，只写一次 FsImage，不妨假设开机时点为8点，到9点时，EditsLog 记录的是8点~9点的日志，只需要将8点~9点的日志记录更新到8点的 FsImage 中，那么 FsImage 的数据时点就变成了9点了。

思考2）方案，NN 没有必要做 FsImage 和 EditsLog 合并操作，只需要每个整点落一次 FsImage 即可。为了获得 FsImage 和 EditsLog 两者的优势，可以考虑寻求 SNN 来实现 FsImage 和 EditsLog 合并操作，减小 NN 的压力。

##### 数据持久化细节

NN 存储元数据，元数据信息包括：

* 文件属性，即文件的路径、名称、大小，用户和组，读写权限

  path /a/b/c.txt	32G	root:root	rwxrwxrwx

* 块信息，块在哪些 DN

  blk01	node01 node03

  blk02	node04 node07

在持久化时，文件属性会持久化，但文件的每一个块不会持久化，恢复时候，NN 会丢失块的位置信息。<font color="red">岂不是，出现了 Bug？</font>

HDFS 依然会这么去做，主要是为了解决数据一致性问题。假设，某些 DN 挂掉，如果持久化时，存储了块信息，会出现数据不一致问题。而 HDFS 解决该问题，会等待一段时间，因为 DN 会和 NN 建立心跳，汇报块信息。该过程被成为**安全模式**。

### 安全模式

* HDFS 搭建时会格式化，格式化操作会产生一个空的 FsImage。
* 当 NameNode 启动时，它从硬盘中读取 EditsLog 和 FsImage。
* 将所有 EditLog 中的事务作用在内存中的 FsImage 上，并将这个新版本的 FsImage 从内存中保存到本地磁盘上，然后删除旧的 EditsLog，因为该旧的 EditsLog 的事务都已经作用在 FsImage 上了。
* NameNode 启动后会进入一个成为安全模式的特殊状态。
* 处于安全模式的 NameNode 是不会进行数据块的复制的。
* NameNode 从所有的 DataNode 接收心跳信号和块状态报告。
* 每当 NameNode 检测确认某个数据块的副本数目达到这个最小值，那么该数据块就会被认为是副本安全（safely replicated）的。
* 在一定百分比（该参数可配置）的数据块被 NameNode 检测确认是安全之后（加上一个额外的 30 秒的等待时间），NameNode 将退出安全模式状态。
* 接下来它会确定还有哪些数据块的副本没有达到指定数目，并将这些数据块复制到其他 DataNode 上。



### 副本放置策略

第一个副本：放置在上传文件的 DN；如果是集群外提交，则随机选择一台磁盘不太满，CPU 不太忙的节点。

第二个副本：放置在与第一个副本不同的机架的节点上。

第三个副本：与第二个副本相同机架的节点。

更多副本：随机节点。



### 读写流程

#### HDFS 写流程

![HDFS写流程](https://gitee.com/struggle3014/picBed/raw/master/HDFS写流程.png)

<div align="center"><font size="2">HDFS 写流程</font></div>

* Client 与 NN 连接，创建文件元数据
* NN 判定元数据是否有效
* NN 触发副本放置策略，返回一个有序的 DN 列表
* Client 和 DN 建立 Pipeline 连接
* Client 将块 block 切分成**数据包** package（64KB），并使用 chunk（512B）+ chucksum（4B）填充。
* Client 将 packet 放入发送队列 dataqueue 中，并向第一个 DN 发送
* 第一个 DN 收到 packet 后本地保存并发送给第二个 DN
* 第二个 DN 收到 packet 后本地保存并发送给第三个 DN
* 该过程中，上游节点同时发送下一个 packet

<font color="red">生活中类比工厂的流水线，结论：流式其实也是变种的并行计算</font>

* 当 block 传输完成，DN 们各自向 NN 汇报，同时 Client 继续传输下一个 block

<font color="red">Client 的传输和 block 的汇报也是并行的。</font>



#### HDFS 写流程

![image-20200802141257212](https://gitee.com/struggle3014/picBed/raw/master/HDFS读流程.png)

<div align="center"><font size="2">HDFS 读流程</font></div>

* 为了降低整体的带宽消耗和读取延迟，HDFS 会尽量让读取程序读取离它最近的副本。
* 如果在读取程序的同一个机架上有一个副本，那么就读取该副本。
* 如果一个 HDFS 集群跨多个数据中心，那么客户端也将首先读取本地数据中心的副本。

语义：

（1）下载一个文件

* Client 和 NN 交互文件元数据获取 FileBlockLocation
* NN 会按距离策略排序返回
* Client 尝试下载 block 并校验数据完整性

（2）下载一个文件其实是获取文件的所有 block 元数据，那么子集获取某些 block 应该成立

* <font color="red">HDFS 支持 Client 给出文件的 offset 自定义连接哪些 block 的 DN，自定义获取数据，这个是支持计算层的分治、并行计算的核心</font>



## HDFS HA

### HDFS 架构优缺点剖析

主从集群：结构相对简单，主从协调工作

优点：主是单点，数据一致性好掌握

问题：

* 单点故障，集群整体不可用
* 压力过大，内存受限

HDFS 解决方案

* 单点故障：
  * 高可用方案：HA（Hig Avaiable），多个 NN，主备切换
* 压力过大，内存首先：
  * 联邦机制：Federation（元数据分片）
  * 多个 NN，管理不同的元数据
* Hadoop 2.x 只支持 HA 的一主一备

### HDFS-HA 解决方案

#### 补充知识

##### CPA 原则（定理）

![CAP原则](https://gitee.com/struggle3014/picBed/raw/master/CAP原则.png)

<div align="center"><font size="2">CAP 定理</font></div>

Consistency：一致性

Availability：可用性

Partition tolerance：分区容错性



##### Paxos 算法

Paxos 算法是莱斯利.兰伯特于1990年提出的一种基于消息传递的<font color="red">一致性</font>算法。

该算法被认为是类似算法中最有效的，覆盖全部场景的一致性。每种技术会根据自己技术的特征选择简化算法实现。

传递：NN 之间通过一种可靠的传输技术，最终数据能同步就可以。我们一般假设网络等因素是稳定的，类似一种带存储能力的消息系统。



#### 简化思路

* 分布式节点是否明确

* 节点权重是否明确

* 强一致性破坏可用性

* 过半通过可以中和一致性和可用性

![无主态到主从态](https://gitee.com/struggle3014/picBed/raw/master/无主态到主从态.png)

<div align="center"><font size="2">无主状态->主从状态</font></div>

最简单的自我协调实现：主从

主的选举：明确节点数量和权重

主从的职能：

* 主：增删盖茶
* 从：查询，增删改传递给主
* 主与从：过半数同步数据

#### HA 方案

![image-20200802152002686](https://gitee.com/struggle3014/picBed/raw/master/HDFS-HA解决方案.png)

<div align="center"><font size="2">HDFS-HA 解决方案</font></div>

多态 NN 主备模式，Active 和 Standby 状态

​	Active 对外提供服务

增加 JournalNode 角色（>3台），负责同步 NN 的 EditsLog

​	最终一致性

增加 zkfc 角色（与 NN 同台），通过 Zookeeper 集群协调 NN 的主从选举和切换

​	事件回调机制

DN 同时向 NNs 汇报 block 清单



### HDFS-Federation 解决方案

![image-20200802154802875](https://gitee.com/struggle3014/picBed/raw/master/HDFS-Federation.png)

<div align="center"><font size="2">HDFS-Federation 解决方案</font></div>

NN 的压力过大，内存首先问题：

* 元数据分治，复用 DN 存储
* 元数据访问隔离性
* DN 目录隔离 block



## HDFS 权限管理

