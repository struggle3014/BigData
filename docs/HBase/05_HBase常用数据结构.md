<div align="center"><img src="https://gitee.com/struggle3014/picBed/raw/master/name_code.png"></div>

# 导读

本文介绍 HBase 常用的数据结构，包括 LSM 树，跳跃表，布隆过滤器等。

***持续更新中~***



# 目录

<nav>
<a href='#导读' style='text-decoration:none;font-weight:bolder'>导读</a><br/>
<a href='#目录' style='text-decoration:none;font-weight:bolder'>目录</a><br/>
<a href='#正文' style='text-decoration:none;font-weight:bolder'>正文</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1 LSM 树' style='text-decoration:none;${border-style}'>1 LSM 树</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#LSM 树的由来' style='text-decoration:none;${border-style}'>LSM 树的由来</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#LSM 的设计思想和原理' style='text-decoration:none;${border-style}'>LSM 的设计思想和原理</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#LSM 树的设计思想' style='text-decoration:none;${border-style}'>LSM 树的设计思想</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#LSM 树原理' style='text-decoration:none;${border-style}'>LSM 树原理</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2 跳跃表' style='text-decoration:none;${border-style}'>2 跳跃表</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3 布隆过滤器' style='text-decoration:none;${border-style}'>3 布隆过滤器</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#使用布隆过滤器的原因' style='text-decoration:none;${border-style}'>使用布隆过滤器的原因</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#更新模式' style='text-decoration:none;${border-style}'>更新模式</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#添加开销' style='text-decoration:none;${border-style}'>添加开销</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#布隆过滤器优缺点' style='text-decoration:none;${border-style}'>布隆过滤器优缺点</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#选择布隆过滤器' style='text-decoration:none;${border-style}'>选择布隆过滤器</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#布隆过滤器的使用场景' style='text-decoration:none;${border-style}'>布隆过滤器的使用场景</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#布隆过滤器的使用原则' style='text-decoration:none;${border-style}'>布隆过滤器的使用原则</a><br/>
<a href='#总结' style='text-decoration:none;font-weight:bolder'>总结</a><br/>
<a href='#参考文献' style='text-decoration:none;font-weight:bolder'>参考文献</a><br/>
</nav>

# 正文

## 1 LSM 树

### LSM 树的由来

在了解 LSM 树之前，大家需要对 hash 表和 B+树有所了解。

hash 存储方式支持增删改查以及随机读取操作，但不支持顺序扫描，对应的存储系统为 key-value 的存储系统。对于 key-value 的插入以及查询，哈希表的复杂度都是 O(1)，明显比树的操作 O(n)快，如果有不需要遍历的数据，哈希表就是最佳选择。

B+树不仅支持单条记录的增删改操作，还支持顺序扫描（B+树的叶子节点之间的指针），对应的存储系统就是关系数据库（MySQL 等）。但是删除和更新比较麻烦。

正是基于以上结构的分析，LSM 树应运而生。

LSM 树（Log-Structured Merge Tree）存储引擎和 B 树存储引擎一样，同样支持增、删、改、查、顺序扫描操作。而且通过批量存储技术规避磁盘随机写入问题。当然凡事有利有弊，LSM 树和 B+ 树相比，LSM 树牺牲了部分读性能，用来大幅度提高写性能。

### LSM 的设计思想和原理

#### LSM 树的设计思想

将对数据的修改增量保存在内存中，达到指定的大小限制后将这些修改操作批量写入磁盘，不过读取的时候稍麻烦，需要合并磁盘中历史数据和内存中最近修改操作，所以写入性能大大提升，读取时可能需要先看是否命中内存，否则需要访问较多的磁盘文件。极端地说，基于 LSM 树实现的 HBase 的写性能比 MySQL 高一个数量级，读性能低一个数量级。



#### LSM 树原理

把一棵大树拆分成 N 棵小树，它首先写入内存中，随着小树越来越大，内存中的小树会 flush 到磁盘中，磁盘中的树定期可以做 merge 操作，合并成一颗大树，以优化读性能。

在 HBase 中 LSM 的应用流程梳理：

1，因为小树先写到内存中，为了防止内存数据丢失，写内存的同时需要暂时缓存到磁盘，对应 HBase 的 MemStore 和 HLog。

2，MemStore 上的树达到一定大小之后，需要 flush 到 HRegion 磁盘上（一般是 Hadoop DataNode），这样 MemStore 就变成了 DataNode 上的磁盘文件 StoreFile，定期 HRegionServer 对 DataNode 的数据做 merge 操作，彻底删除无效空间，多棵小树在这个时机合并成大树，来增强读性能。

3，LSM 的原理

关于 LSM Tree，对于最简单的 LSM Tree 而言，内存中的数据和磁盘中的数据 mrege 操作，如下图

![关系型数据库](https://gitee.com/struggle3014/picBed/raw/master/树合并.png)

<div align="center"><font size="2">LSM 树合并</font></div>

LSM Tree，理论上，可以使内存中树的一部分和磁盘中第一层树做 merge，对于磁盘中的树直接做 update 操作有可能会破坏 block 的连续性，但是实际应用中，一般 LSM 有多层，当磁盘中的小树合并成一颗大树的时候，可以重新排好顺序，使得 block 连续，优化读性能。

HBase 在实现中，是把整个内存在一定阈值后，flush 到 disk 中，形成一个 file，这个 file 的存储就是一个小的 B+ 树，因为 HBase 一般是部署在 HDFS 上，HDFS 不支持对文件的 update 操作，所以 HBase 整体内存 flush，而不是和磁盘中的小树 merge update。内存 flush 到磁盘上的小树，定期也会合并成一个大树。整体上 HBase 用了 LSM Tree 的思路。



## 2 跳跃表

实现 MemStore 模型的数据结构是 SkipList（跳表），跳表可以实现高效的查询，插入，删除操作，这些操作的期望复杂度都是 O(logN)。另外，跳表本质是由链表构成，理解和实现简单。这也是很多 KV 数据库（Redis，LevelDB）使用跳表实现有序数据集合的两个主要原因。

JDK 原生自带的跳表实现目前只有 ConcurrentSkipListMap（简称 CSLM，注意：ConcurrentSkipListSet 是基于 ConcurrentSkipListMap 实现的）。ConcurrentSkipListMap 是基于 JDK Map 的一种实现，本质上也是 Map，不过该 Map 中的元素是有序的，且通过跳表来实现。

ConcurrentSkipListMap 的结构如下：

![ConcurrentSkipListMap](https://gitee.com/struggle3014/picBed/raw/master/ConcurrentSkipListMap.png)

<div align="center"><font size="2">ConcurrentSkipListMap 结构</font></div>

基于 ConcurrentSkipListMap 基础数据结构，按最简单的思路看，如果写入一个 KeyValue 到 MemStore，写入步骤如下：

1，在 JVM 堆中为 KeyValue 对象申请一块内存区域。

2，调用 ConcurrentSkipListMap 的 put(K key, V value) 方法将该 KeyValue 对象作为参数传入。

![基于跳表实现的最基础MemStore模型](https://gitee.com/struggle3014/picBed/raw/master/基于跳表实现的最基础MemStore模型.png)

<div align="center"><font size="2">基于跳表实现的最基础MemStore模型</font></div>

该内存模型有很多问题，主要是内存使用和回收方面。

上述存储模型中，发现 MemStore 从内存管理上主要有两部分组成：

* 原生的 KeyValue 的内存管理
* ConcurrentSkipListMap 的内存管理

可以对两部分的内存管理进行优化。内存优化的具体内容参见：HBase内存管理之MemStore进化论<sup>[[4] ](# )</sup>



## 3 布隆过滤器

### 使用布隆过滤器的原因

根本原因在于 HBase 默认机制决定一个存储文件是否包含特定的受限于可用块索引的行键，同时这个索引又是相当粒度的，该索引值存储了文件包含块的开始键。

如，系统默认的 64K 作为块大小，那么 1G 存储文件被分成 16384 个块，与索引到的行键数相同。若进一步假设每个单元格的平均大小是 200 Byte，那么用户将存储在一个文件中的超 500 万个单元格。若用户随机查找一个行键，则该行键坑呢处在两个块开始键之间的位置。对于 HBase 来说，判断该键是否真实存在的唯一方法是加载该快，并且扫描它来查找该键。

#### 更新模式

用户通常会以一定速率更新数据，将导致内存中的数据刷写到磁盘上，并且之后系统会将它们合并成更大的存储文件。由于 minor 合并仅合并最近几个存储文件，知道合并后的文件达到配置的最大大小。最终系统中将会有很多存储文件，所有这些文件都是候选文件，其可能包含一些用户请求行键的单元格。

![使用布隆过滤器减少IO操作的数量](https://gitee.com/struggle3014/picBed/raw/master/使用布隆过滤器减少IO操作的数量.png)

<div align="center"><font size="2">使用布隆过滤器减少IO操作的数量</font></div>

这些文件都来自同一列族，所以它们行键的分布很相似。尽管只有几个文件包含特定行的更新，文件的块索引还是覆盖了整个行键范围。块索引能确定文件中是否包含某个行键。Region 服务器需要加载每一个块来检查该快中是否真的实际包含该行的单元格。

**若用户使用布隆过滤器，则可以立即判断一个文件是否包含特定的行键**。**布隆过滤器的特性**是：如果该文件不包含该行，则会有明确答复；如果文件包含该行时，答复却可能有误，即它生成文件包含这个数据而实际却并非如此。错误肯定答复的数量可以被调整，通常被设置为1%，这以为这过滤器中关于一个文件包含一个请求行的报告中有 1% 是错误的，因此可能会有一个块被错误地加载和检查。

可以从上面的例子中看出块加载数量大大减少，这在负载很重的系统中会产生很大的影响。为了提高效率，用户还必须使用一种**特定的更新模式**：如果用户定期修改所有行，那么大部分的存储文件都将包含用户查询行的数据。这种场景不适合使用布隆过滤器。但是，如果用户批量更新数据，使得一行数据每次只被写入到几个存储文件中，那么过滤器就能够为减少整个系统 I/O 操作的；数量发挥很大作用。

使用块缓存

#### 添加开销

决定是否使用布隆过滤器的另一个因素是使用它所添加的开销。每项会在过滤器中占用约 1Byte 的存储空间。

上面的例子，存储文件大小是 1G 时，假设用户只存储计数器类型数据（即8个 Byte 的 long 类型），并加上 KeyValue 信息（即它的坐标，或航主键，列族名，列限定词，时间戳和类型）的开销，那么每个单元格大概是 20Byte。此时布隆过滤器的大小将是存储文件的二十分之一，大概占用 51M 空间。

现假设用户的单元格的平均大小是 1K，那么过滤器只需要 1M。考虑到过滤器在今后需要发挥优化作用，与 1G 甚至更大的文件相比，一个几百 KB 的行级布隆过滤器的开销甚小，这种情况下使用过滤器是有用的。

### 布隆过滤器优缺点

* 优点
  * 减少I/O 操作数量
    * **布隆过滤器**可以快速判断文件块中是否包含特定键，从而减少 I/O 操作的数量，即块的加载数量。
  * 低缓存波动，高缓存命中率。
    * 加载更少的块将导致更少的缓存波动，缓存的命中率会响应提高。
    * 由于服务器在大部分时后加载的都是包含请求数据的块，相关的数据将有更大的机会留在块缓存中随后读取操作可以使用这些数据。

* 缺点
  * 部分存储空间的开销



### 选择布隆过滤器

存在两种类型的布隆过滤器：行级布隆过滤器，行加列级布隆过滤器。

![布隆过滤器的选择标准](https://gitee.com/struggle3014/picBed/raw/master/布隆过滤器的选择标准.png)

<div align="center"><font size="2">使用布隆过滤器减少IO操作的数量</font></div>

#### 布隆过滤器的使用场景

##### 行级布隆过滤器

* 只做行扫描
  * 如果用户只做行扫描，使用行加列级的布隆过滤器不会有任何帮助。
* 行加列读取
  * 即使用户使用行加列的读操作时，使用行级的布隆过滤器仍然可以减少需要检查的文件数量。
  * 相反，如果使用行加列级的布隆过滤器却不能为行扫描提供更好的性能。

##### 行加列级布隆过滤器

* 用户不能批量更新特定的一行，且所有的存储文件都包含该行的一部分
  * 此场景下，行加列级的过滤器能够识别出那些文件包含用户请求的数据。
  * 若用户总是加载整行，那么该过滤器也很难起到作用，因为 region 服务器无论如何都需要加载每个文件中匹配的块。

#### 布隆过滤器的使用原则

如果可能的话，用户尽量使用行级布隆过滤器，因为它在额外的空间开销和利用选择过滤存储文件提升之间获得了很好的平衡。只有当用户使用行级布隆过滤器没有性能提升时，再考虑使用开销更大的行加列级布隆过滤器。



# 总结



# 参考文献

[1] [HBase 官网](http://hbase.apache.org/)

[2] HBase 权威指南

[3] [论文，O'Neil，LSM 树](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.44.2782)

[4] [个人网站，范欣欣，HBase 内存管理之 MemStore 进化论](http://hbasefly.com/2019/10/18/hbase-memstore-evolution/)

[5] [HBase，ISSUES,CompactedConcurrentSkipListMap](https://issues.apache.org/jira/browse/HBASE-20312)