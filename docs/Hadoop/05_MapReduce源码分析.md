![name_code](https://gitee.com/struggle3014/picBed/raw/master/name_code.png)

# 导读

源码分析不是为了能写出一个 MR 框架，而是为了更好地使用和更充分地理解框架。



![MapReduce细粒度过程](https://gitee.com/struggle3014/picBed/raw/master/MapReduce细粒度过程.png)

<div align="center"><font size="2">MapReduce细粒度过程</font></div>

做减法，跳过 YARN 的资源管理这一层，只研究计算层的实现，MR 计算层分为三个环节：

1，Client 提交阶段

2，MapTask

3，ReduceTask



研究源码的过程中，关注分布式计算追求的目标：

* 分治，并行计算

* 计算向数据移动

* 数据本地化



# 目录

<nav>
<a href='#导读' style='text-decoration:none;font-weight:bolder'>导读</a><br/>
<a href='#目录' style='text-decoration:none;font-weight:bolder'>目录</a><br/>
<a href='#正文' style='text-decoration:none;font-weight:bolder'>正文</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#Client' style='text-decoration:none;${border-style}'>Client</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#MapTask' style='text-decoration:none;${border-style}'>MapTask</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#Input' style='text-decoration:none;${border-style}'>Input</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#Output' style='text-decoration:none;${border-style}'>Output</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#ReduceTask' style='text-decoration:none;${border-style}'>ReduceTask</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#Reducer 的文档' style='text-decoration:none;${border-style}'>Reducer 的文档</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#Reducer run 方法详解' style='text-decoration:none;${border-style}'>Reducer run 方法详解</a><br/>
<a href='#参考文献' style='text-decoration:none;font-weight:bolder'>参考文献</a><br/>
</nav>

# 正文

## Client

Client 侧，没有计算发生。Client 支持了计算向数据移动和计算的并行度。

1，Checking the input and output specifications of the job.
2，Computing the InputSplits for the job.  // split  ->并行度和计算向数据移动就可以实现了
3，Setup the requisite accounting information for the DistributedCache of the job, if necessary.
4，Copying the job's jar and configuration to the map-reduce system directory on the distributed file-system.
5，Submitting the job to the JobTracker and optionally monitoring it's status



MR 框架默认的输入格式化类：TextInputFromat < FileInputFormat < InputFormat

getSplits()

minSize = 1

maxSize = Long.Max

blockSize = file

splitSize = Math.max(minSize, Math.min(maxSize, blockSize)); // 默认 split 大小等于 block 大小

​	切片 split 是一个窗口机制：（调大 split 改小，调小 split 该大）

if(blkLocations[i].getOffset() <= offset < blkLocations[i].getOffset() + blkLocations[i].getLength())

split：解耦存储层和计算层

​	1，file

​	2，offset

​	3 ，length

​	4，hosts // 支撑计算向数据移动



## MapTask

input -> map -> output

![MapTask核心流程](https://gitee.com/struggle3014/picBed/raw/master/MapTask核心流程.png)

<div align="center"><font size="2">MapTask核心流程</font></div>

### Input

input (split + format) 该部分属于通用的知识，后续 Spark 底层也是来自于输入格式化类给我们实际返回的记录读取器对象

TextInputFormat -> LineRecordReader

​	split: file, offset, length

​	init():

​		in = fs.open(file).seek(offset)

​		除了第一个切片对应的 map，之后的 map 都在 init 环节，从切片包含的数据中，让出第一行，并把切片的起始更新为切片的第二行。换言之，前一个 map 会多读取一行，来弥补 hdfs 把数据切割的问题~！

​	nextKeyValue():

​		1,读取数据中的一条记录，对 key，value 赋值

​		2，返回布尔值

​	getCurrentKey():

​	getCurrentValue():

![解决HDFS数据被分割到不同的blk问题](https://gitee.com/struggle3014/picBed/raw/master/解决HDFS数据被分割到不同的blk问题.png)

<div align="center"><font size="2">解决 HDFS 数据被分割到不同的 blk 问题</font></div>

### Output

#### 补充知识

##### 环形缓冲区

![MapReduce环形缓冲区](https://gitee.com/struggle3014/picBed/raw/master/MapReduce环形缓冲区.png)

<div align="center"><font size="2">MR环形缓冲区</font></div>

buffer 使用的是环形缓冲区，MR 底层源码实现是 MapOutputBuffer 类。

* 本质还是线性字节数组

* 赤道，两端方向分别放置 KV 和 Index 索引

* 索引为固定宽度：16Byte（4个int）

  * Partition
  * Key Start
  * Value Start
  * Value Length

* 如果数据填充到阈值80%，启动线程：

  * 快速排序80%数据，同时 map 输出的线程向剩余空间写
  * 快速排序的过程：比较 key 排序，但是移动的是索引

* 最终，溢写时只要按照排序的索引，卸下的文件中的数据就是有序的

  ​	<font color="red">注意</font>：<u>排序是二次排序（索引里有 P，排序先比较索引的 P 决定顺序，然后再比较相同的 P 中的 Key 顺序）。</u>

  ​		<u>分区内有序：最后 reduce 是按照分区进行拉取</u>

  ​		<u>分区内 key 有序：因为 reduce 计算时按照分组计算，分组的语义（相同的 Key 排在一起）</u>

#### Output 核心流程

NewOutputCollector

​	partitioner

​	collector

​		MapOutputBuffer:

​			map输出的KV会序列化成字节数组，算出P，最中是3元组：K,V,P



## ReduceTask

input -> reduce -> output

![image-20200806155850115](https://gitee.com/struggle3014/picBed/raw/master/ReduceTask核心流程.png)

### Reducer 的文档

​	1，Shuffle：	洗牌（相同的 key 被拉取到一个分区），拉取数据

​	2，Sort：		整个 MR 框架中只有 map 端是无序到有序的过程，使用的是快速排序

​									reducer 这里所谓的 sort 其实

​									你可以想成是一个对着 map 排好序的一堆小文件做归并排序

​	grouping comparator

​	1970-01-22	33	bj

​	1970-01-08	23	sh

​		排序比较器：年，月，温度，且温度倒叙

​		分区比较器：年，月

​	3，Reduce



### Reducer run 方法详解

Mapper 中的 run():	while (context.nextKeyValue())

​											一条记录调用一次map

Reducer 中的 run():	while (context.nextKey())

​											一组数据调用一次reduce



run():

​	rIter = shuffle。。//reduce拉取回属于自己的数据，并包装成迭代器~！真@迭代器

​		file(磁盘上)-> open -> readline -> hasNext() next()

​		时时刻刻想：我们做的是大数据计算，数据可能撑爆内存~！

​	comparator = job.getOutputValueGroupingComparator();

​		1，取用户设置的分组比较器

​		2，取getOutputKeyComparator();

​			1，优先取用户覆盖的自定义排序比较器

​		    2，保底，取key这个类型自身的比较器

​		#：分组比较器可不可以复用排序比较器

​			什么叫做排序比较器：返回值：-1,0,1

​			什么叫做分组比较器：返回值：布尔值，false/true

​			排序比较器可不可以做分组比较器：可以的

​			mapTask									reduceTask

​																1，取用户自定义的分组比较器

​			1，用户定义的排序比较器		2，用户定义的排序比较器

​			2，取key自身的排序比较器		3，取key自身的排序比较器

​			组合方式：

​				1）不设置排序和分组比较器

​					map：取key自身的排序比较器

​					reduce：取key自身的排序比较器

​				2）设置了排序

​					map：用户定义的排序比较器

​					reduce：用户定义的排序比较器

​				3）设置了分组

​					map：取key自身的排序比较器

​					reduce：取用户自定义的分组比较器

​				4）设置了排序和分组

​					map：用户定义的排序比较器

​					reduce：取用户自定义的分组比较器

​			做减法：结论，框架很灵活，给了我们各种加工数据排序和分组的方式



# 参考文献



