![name_code](https://gitee.com/struggle3014/picBed/raw/master/name_code.png)

# 导读



# 目录
<nav>
<a href='#导读' style='text-decoration:none;font-weight:bolder'>导读</a><br/>
<a href='#目录' style='text-decoration:none;font-weight:bolder'>目录</a><br/>
<a href='#正文' style='text-decoration:none;font-weight:bolder'>正文</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1 通用命令' style='text-decoration:none;${border-style}'>1 通用命令</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2 DDL 操作' style='text-decoration:none;${border-style}'>2 DDL 操作</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#3 namespace 操作' style='text-decoration:none;${border-style}'>3 namespace 操作</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#4 DML 操作' style='text-decoration:none;${border-style}'>4 DML 操作</a><br/>
<a href='#总结' style='text-decoration:none;font-weight:bolder'>总结</a><br/>
<a href='#参考文献' style='text-decoration:none;font-weight:bolder'>参考文献</a><br/>
</nav>


# 正文

## 1 通用命令

```shell
# 展示 regoinserver 的task 列表
hbase>processlist
# 展示集群状态
hbase>status
# table 命令的帮助手册
hbase>table_help
# 显示 hbase 的版本
hbase>version
# 展示当前 hbase 的用户
hbase>whoami
```

## 2 DDL 操作

```shell
# 修改表的属性
hbase >alter 't1', NAME => 'f1', VERSIONS=5
# 创建表
hbase>create 'test', 'cf'
# 查看表描述，只会展示列族的详细信息
hbase>describe 'test'
# 禁用表
hbase>disable 'test'
# 禁用所有表
hbase>disable_all
# 删除表
hbase>drop 'test'
# 删除所有表
hbase>drop_all
# 启用表
hbase>enable 'test'
# 启用所有表
hbase>enable_all
# 判断表是否存在
hbase>exists 'test'
# 获取表
hbase>get_table 'test'
# 判断表是否被禁用
hbase>is_enabled 'test'
# 展示所有表
hbase>list
# 展示表占用的 region
hbase>list_regions
# 定位某个 rowkey 所在的行在哪一个 region
hbase>locate_region
# 展示所有过滤器
hbase>show_filters
```

## 3 namespace 操作

```shell
# 修改命名空间的属性
hbase>alter_namespace 'my_ns', {METHOD => 'set', 'PROPERTY_NAME' => 'PROPERTY_VALUE'}
# 创建命名空间
hbase>create_namespace 'my_ns'
# 获取命名空间的描述信息
hbase>describe_namespace 'my_ns'
# 删除命名空间
hbase>drop_namespace 'my_ns'
# 展示所有的命名空间
hbase>list_namespace
# 展示某个命名空间下所有的表
hbase>list_namespace_tables 'my_ns'
```

## 4 DML 操作

```shell
# 向表中追加一个具体的值
hbase>append 't1', 'r1', 'c1', 'value', ATTRIBUTES => {'mykey' => 'myvalue'}
# 统计表的记录条数，默认一千条输出一次
hbase>count 'test'
# 删除表中某一个值
hbase>delete 't1', 'r1', 'c1', ts1
# 删除表的某一列的所有值
hbase>deleteall 't1', 'r1', 'c1'
# 获取表的一个列的值的个数
hbase>get_counter 't1', 'r1', 'c1'
# 获取表的切片
hbase>get_splits 't1'
# 增加一个 cell 对象的值
hbase>incr 't1', 'r1', 'c1'
# 向表中的某一个列插入值
hbase>put 't1', 'r1', 'c1', 'value', ts1
# 扫描表的全部数据
hbase>scan 't1'
# 清空表的所有数据
hbase>truncate
```

# 总结



# 参考文献

[1] [HBase 官网](http://hbase.apache.org/)