![name_code](https://gitee.com/struggle3014/picBed/raw/master/name_code.png)

# 导读

本文主要介绍 HBase 的**读写**流程。

和写流程相比，读流程是一个更加复杂的操作过程，主要是由于两方面的原因：

* HBase 存储引擎基于 LSM-Like 树实现，一次查询可能会涉及到多个分片，多块缓存甚至多个存储文件。
* HBase 中更新操作以及删除实现都很简单。
  * 更新操作没有更新原有数据，而是使用时间戳属性实现了多版本。
  * 删除操作也并没有真正删除原有数据，只是插入了一条打上“deleted”标签的数据，而真正的数据删除发生在系统异步执行 Major_Compact 的时候。上述套路极大地简化了数据更新、删除流程，但对于数据查询却意味着套上了层层枷锁，读取过程需要根据版本进行过滤，同时对已经标记删除的数据也要进行过滤。

# 目录

<nav>
<a href='#导读' style='text-decoration:none;font-weight:bolder'>导读</a><br/>
<a href='#目录' style='text-decoration:none;font-weight:bolder'>目录</a><br/>
<a href='#正文' style='text-decoration:none;font-weight:bolder'>正文</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#Client-Server 交互逻辑' style='text-decoration:none;${border-style}'>Client-Server 交互逻辑</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#HBase 读流程' style='text-decoration:none;${border-style}'>HBase 读流程</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#简化版本' style='text-decoration:none;${border-style}'>简化版本</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#详细版本' style='text-decoration:none;${border-style}'>详细版本</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#构建 Scanner 体系（一组施工队）' style='text-decoration:none;${border-style}'>构建 Scanner 体系（一组施工队）</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#Scan 查询（一层层盖楼）' style='text-decoration:none;${border-style}'>Scan 查询（一层层盖楼）</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#HBase 写流程' style='text-decoration:none;${border-style}'>HBase 写流程</a><br/>
<a href='#总结' style='text-decoration:none;font-weight:bolder'>总结</a><br/>
<a href='#参考文献' style='text-decoration:none;font-weight:bolder'>参考文献</a><br/>
</nav>

# 正文

## Client-Server 交互逻辑

1，客户端根据配置文件中 Zookeeper 地址连接 Zookeeper，并读取 /<hbase-rootdir>/meta-region-server 节点信息。

* 该节点存储 HBase 元数据表（hbase:meta）表所在的 RegionServer 地址以及访问端口等信息。
* 可以通过 Zookeeper 命令（get /<hbase-rootdir>/meta-region-server）查看节点信息。

2，根据 hbase:mea 所在 RegionServer 访问信息，客户端会将元数据表加载到本地进行缓存，然后再表中确定检索 rowkey 所在的 RegionServer 信息。

3，根据数据所在 RegionServer 的访问信息，客户端会向该 RegionServer 发送真正的数据请求。服务器接收到请求后，再进行处理。

![HBase的Client-Server交互逻辑](https://gitee.com/struggle3014/picBed/raw/master/HBase的Client-Server交互逻辑.png)

<div align="center"><font size="2">Client-Server交互逻辑</font></div>

## HBase 读流程

### 简化版本

1，客户端从 Zookeeper 中获取元数据表（hbase:meta）所在的 RegionServer 节点信息。

2，客户端访问元数据表（hbase:meta）所在的 RegionServer，获取 Region 所在的 RegionServer 信息。

3，客户端具体的 Region 所在的 RegionServer，找到对应的 Region 及 Store，并返回给客户端。

* 首先从 MemStore 中读取数据，若读取到直接返回给客户端；若没有，则去 BlockCache 读取数据。
* 若从 BlockCache 中读取到数据，则直接返回给客户端；若没有，遍历 StoreFile 文件，查找数据。
* 若从 StoreFile 中读取到数据，则将数据缓存在 BlockCache 中，再返回给客户端；若没有，则返回客户端为空。

其中，BlockCache 是内存空间，如果缓存的数据较多，满了之后会采用 LRU 策略，将较老的数据进行删除。

### 详细版本

RegionServer 接收到 get/scan 请求后，先后做了两件事：

* 构建 scanner 体系（实质是 scan 前的准备工作）。
* 在此体系上一行一行检索。

可以把 scan 数据的过程类比成开发商盖房子，同样是分成两步：

* 组建施工队体系，明确每个工人的职责。
* 一层一层盖楼

#### 构建 Scanner 体系（一组施工队）

Scanner 体系的核心在于三层 Scanner：RegionScanner，StoreScanner 和 StoreFileScanner。

三者是层级关系，一个 RegionScanner 由多个 StoreScanner 构成，一张表由多个列族组成，就有多少个 StoreScanner 负责该列族的数据扫描。一个 StoreScanner 又是由多个 StoreFileScanner 组成。每个 Store 的数据由内存中的 MemStore 和磁盘上的 StoreFile 文件组成，与此相对应的，StoreScanner 对象会雇佣一个 MemStoreScanner 和 N 个 StoreFileScanner 来进行实际的数据读取，每个 StoreFile 文件对应一个 StoreFileScanner。

**注意：StoreFileScanner 和 MemSotreScanner 是整个 scan 的最终执行者。**

对应建楼项目，一栋楼通常由好几个单元楼构成（每个单元楼对应一个 Store），每个单元楼会请一个监工（StoreScanner）负责该单元楼的建造。而监工一般不做具体事情，他负责招募很多工人（StoreFileScanner），这些工人是建楼的主体。

![scanner体系构建](https://gitee.com/struggle3014/picBed/raw/master/scanner体系构建.png)

<div align="center"><font size="2">scanner体系构建过程</font></div>



***1，RegionScanner 根据列族构建 StoreScanner，有多少列族就构建多少 StoreScanner，用于负责该列族的数据检索。***

**1.1，构建 StoreFileScanner**

每个 StoreScanner 会为当前该 Store 中每个 HFile 构建一个 StoreFileScanner，用于实际执行对应文件的索引。同时会为对应 MemStore 构建一个 MemStoreScanner，用于执行该 Store 中 MemStore 的数据检索。

该步骤对应监工在人才市场招募捡建楼所需的各种类型工匠。

**1.2，过滤淘汰的 StoreFileScanner**

根据 Time Range 及 RowKey Range 对 StoreFileScanner 以及 MemStoreScanner 进行过滤，淘汰肯定不存在待检索结果的 Scanner。上图中 StoreFile3 因为检查 RowKeyRange 不存在待检索 Rowkey，所以被淘汰。

该步骤针对具体建楼方案，裁剪掉部分不需要的工匠，比如这栋楼不需要地暖安装，对应的工匠就可以被撤掉。

**1.3，Seek Rowkey**

所有 StoreFileScanner 开始做准备工作，在负责的 HFile 中定位到满足条件的起始 Row。

<u>工匠也开始准备自己的构建工具，建造材料，找到自己的工作地点，等待一声命令。</u>

就像所有重要项目的准备工作很核心一样，Seek 过程（此处省略 Lazy Seek 优化）也是很核心的步骤，主要包含三步：

* 定位 Block Offset

  在 BlockCache 中读取该 HFile 的树荫树结构，根据索引树检索对应的 Rowkey 所在的 Block Offset 和 Block Size。

* Load Block

  根据 Block Offset 首先在 Block Cache 中查找 Data Block，如果不在缓存，再在 HFile 中加载。

* Seek Rowkey

  在 Data Block 内部通过二分查找的方式定位具体的 Rowkey。

Seek 整体流程细节参见<font color="red">《HBase 原理-探索HFile索引机制》</font>，文中详细说明了 HFile 索引结构以及如何通过索引结构定位具体 Block 和 Rowkey。

**1.4，StoreFileScanner 合并构建最小堆**

将该 Store 中所有 StoreFileScanner 和 MemStoreScanner 合并行程一个 heap（最小堆），所谓 heap 是一个优先级队列，队列中元素是所有 scanner，排序规则按照 scanner seek 到的 KeyValue 大小由大到小排列。

此处需要重点关注三个问题：

* 为什么这些 Scanner 需要从小到大排列？

  最直接的解释是 scan 的结果需要从小到大输出给用户，当然，这并不全面，最合理的解释是只有从小打到排列才能使得 scan 效率最高。

  例如，HBase 支持数据多版本，假设用户指向获取最新版本，那只需要将这些数据由最新到最旧进行排列，然后取队首元素返回即可。如果不排序，则需要遍历所有元素，查看符不符合用户的查询条件。

  <u>工匠们也需要排列，先做地板的排前面，做墙体的次之，最后是做门窗的。做墙体的内部还需要再排序，做内墙的排前面，做外迁的排后面，这样，加入设计师临时决定不做外墙的话，则可以直接跳过外墙部分工作。很显然，如果不排序的话，是没有办法临时做决定的，因为这部分工作可能已经做掉了。</u>

* KeyValue 的结构是啥？

  HBase 的 KeyValue 不是简单的 KV 键值对，而是一个具有复杂元素的结构体，其中 Key 由 RowKey，ColumnFamily，Qualifier，TimeStamp，KeyType 等组成，Value 是一个简单的二进制数据。Key 中元素 KeyType 表示该 KeyValue 的类型，取值分别为 Put/Delete/Delete Column/Delete Family 等。

  ![KeyValue结构](https://gitee.com/struggle3014/picBed/raw/master/KeyValue结构.png)

  <div align="center"><font size="2">KeyValue结构</font></div>

* KeyValue 的大小是如何确定的？

  KeyValue 中 Key 由 Rowkey，ColumnFamily，Qualifier，TimeStamp，KeyType 组成。HBase 设定 Key 大小首先比较 Rowkey，Rowkey 越小 Key 越小；Rowkey 相同看 CF，CF 越小 Key 越小；CF 相同，看 Qualifier，Qualifier 越小 Key 越小；Qualifier 相同，看 TimeStamp，TimeStamp 越大表示时间越新，对应的 Key 越小。若 TimeStamp 还相同，看 KeyType，KeyType 按照 DeleteFamily -> DeleteColumn -> Delete -> Put 顺序对应的 Key 越来越大。

***2，StoreScanner体系构建 Scanner 合并构建最小堆：上文讨论的是一个监工如何构建自己的工匠师团队以及工匠师如何做准备工作、排序工作。***

实际上，监工也需要进行排序，比如一单元的监工排前面，二单元的监工排之后...

StoreScanner 一样，列族小的 StoreScanner 排前面，列族大的 StoreScanner 排后面。

#### Scan 查询（一层层盖楼）

构建 Scanner 体系是为了更好地执行 scan 操作，scan 查询总是一行一行查询的，先查第一行的所有数据，再查第二行的所有数据，但每一行的查询流程却没有本质区别。所以实际上我们只需要关注其中一行数据是如何查询的就可以了。

对于一行数据的查询，又可以分解为多个列族的查询，比如 Rowkey=row1 的一行数据查询，首先查询列族1上该行的数据集合，再查询列族2上该行的数据集合。所以我们也只需要关注某一行某个列族的数据是如何查询就可以了。

Scanner 体系构建的最终结果由一个 StoreFileScanner 和 MemStoreScanner 组成的 heap（最小堆）。下图是一张表的逻辑视图，该表有两个列族cf1和cf2（我们只关注cf1），cf1只有一个列name，表中有5行数据，其中每个cell基本都有多个版本。cf1的数据假如实际存储在三个区域，memstore中有r2和r4的最新数据，hfile1中是最早的数据。现在需要查询RowKey=r2的数据，按照上文的理论对应的Scanner指向就如图所示：

![HBase逻辑视图](https://gitee.com/struggle3014/picBed/raw/master/HBase逻辑视图.png)

<div align="center"><font size="2">HBase逻辑视图</font></div>

这三个 Scanner 组成的 heap 为<MemstoreScanner，StoreFileScanner2, StoreFileScanner1>，Scanner 由小到大排列。查询的时候首先 pop 出 heap 的堆顶元素，即 MemstoreScanner，得到 keyvalue = r2:cf1:name:v3:name23 的数据，拿到这个keyvalue之后，需要进行如下判定：

1. 检查该 KeyValue 的 KeyType 是否是 Deleted/DeletedCol 等，如果是就直接忽略该列所有其他版本，跳到下列（列族）。
2. 检查该 KeyValue 的 Timestamp 是否在用户设定的 Timestamp Range 范围，如果不在该范围，忽略。
3. 检查该 KeyValue 是否满足用户设置的各种 filter 过滤器，如果不满足，忽略。
4. 检查该 KeyValue 是否满足用户查询中设定的版本数，比如用户只查询最新版本，则忽略该 cell 的其他版本；反正如果用户查询所有版本，则还需要查询该cell的其他版本。

现在假设用户查询所有版本而且该 KeyValue 检查通过，此时当前的堆顶元素需要执行 next 方法去检索下一个值，并重新组织最小堆。即图中 MemstoreScanner 将会指向 r4，重新组织最小堆之后最小堆将会变为<StoreFileScanner2, StoreFileScanner1, MemstoreScanner>，堆顶元素变为 StoreFileScanner2，得到keyvalue＝r2:cf1:name:v2:name22，进行一系列判定，再 next，再重新组织最小堆…

不断重复这个过程，直至一行数据全部被检索得到。继续下一行…

## HBase 写流程



# 总结



# 参考文献

[1] [HBase 官网](http://hbase.apache.org/)

[2] [个人网站，范欣欣，HBase 数据读取流程简述](http://hbasefly.com/2016/12/21/hbase-getorscan/)

[3] [个人网站，范欣欣，HBase 数据读取流程部分细节阐述](http://hbasefly.com/2017/06/11/hbase-scan-2/)

[4] [个人网站，范欣欣，HBase 原理-探索 HFile 索引机制](http://hbasefly.com/2016/04/03/hbase_hfile_index/)