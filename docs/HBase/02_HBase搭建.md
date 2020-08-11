![name_code](https://gitee.com/struggle3014/picBed/raw/master/name_code.png)

# 导读

本文介绍 HBase 的搭建和部署。

# 目录

<nav>
<a href='#导读' style='text-decoration:none;font-weight:bolder'>导读</a><br/>
<a href='#目录' style='text-decoration:none;font-weight:bolder'>目录</a><br/>
<a href='#正文' style='text-decoration:none;font-weight:bolder'>正文</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#Standalone HBase' style='text-decoration:none;${border-style}'>Standalone HBase</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1）搭建方式说明' style='text-decoration:none;${border-style}'>1）搭建方式说明</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2）搭建步骤' style='text-decoration:none;${border-style}'>2）搭建步骤</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#Fully-distributed HBase' style='text-decoration:none;${border-style}'>Fully-distributed HBase</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1）搭建方式说明' style='text-decoration:none;${border-style}'>1）搭建方式说明</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2）搭建步骤' style='text-decoration:none;${border-style}'>2）搭建步骤</a><br/>
<a href='#总结' style='text-decoration:none;font-weight:bolder'>总结</a><br/>
<a href='#参考文献' style='text-decoration:none;font-weight:bolder'>参考文献</a><br/>
</nav>

# 正文

## Standalone HBase

### 1）搭建方式说明

```
the setup of a single-node standalone HBase.A standalone instance has all HBase daemons — the Master,RegionServer, and Zookeeper — running in a single JVM persisting to the local filesystem.
```



### 2）搭建步骤

1，虚拟机中必须安装 JDK，JDK 版本建议使用 1.8。

2，官网中下载 HBase 对应安装包（2.0.5）。

3，理论上可以将 HBase 上传到任意一台虚拟机，但因为 HBase 需要 Zookeeper 担任分布式协作服务的角色，而 HBase 的安装包中包含了 Zookeeper，而我们在开启虚拟机之后，一般会将高可用的 Hadoop 集群准备好，因为此 Hadoop 集群中已包含 Zookeeper 服务。**因此，建议将单节点的 HBase 配置在没有安装 Zookeeper 的节点上。**

4，解压 HBase 安装包

```
tar -zxvf hbase-2.0.5-bin.tar.gz -C /opt/bigdata
cd hbase-2.0.5
```

5，在 /etc/profile 文件中配置 HBase 环境变量

```
export HBASE_HOME=/opt/bigdata/hbase-2.0.5
将 $HBASE_HOME 设置到 PATH 路径中
```

6，进入 /opt/bigdata/hbase-2.0.5/conf 目录，在 hbase-env.sh 文件中添加 JAVA_HOME

```
JAVA_HOME=/usr/java/jdk1.8.0_181-amd64
```

7，进入 /opt/bigdata/hbase-2.0.5/bin 目录中，在 hbase-site.xml 文件中添加 hbase 相关属性

```xml
<configuration>
    <property>
    	<name>hbase.rootdir</name>
        <value>file:///home/testuser/hbase</value>
    </property>
    <property>
    	<name>hbase.zookeeper.property.dataDir</name>
        <value>/home/testuser/zookeeper</value>
    </property>
    <property>
    	<name>hbase.unsafe.stream.capability.enforce</name>
        <value>false</value>
    </property>
    <description>
    	Controls whether HBase will check for stream capabilities(hflush/hsync).
        
        Disable this if you intend to run on LocalFileSystem,denoted by a rootdir with the 'file://' schema,but be mindful of the NOTE below.
        
        WARNING:Setting this to false blinds you to potential data loss and inconsistent system state in the event of process and/or node failures.If HBase is complaining of an inability to use hsync or hflush it's most likely not a false positive.
    </description>
</configuration>
```

8，在任意目录下运行 hbase shell 命令，进入 hbase 命令行进行相关操作。



## Fully-distributed HBase

### 1）搭建方式说明

```
By default,HBase runs in standalone mode.Both standalone mode and pseudo-distributed mode are provided for the purposes of small-scale testing.For a production environment,distributed mode is advised.In distributed mode,multiple instance of HBase daemons run on multiple servers in the cluster.
```



### 2）搭建步骤

1，将集群中的所有节点的 hosts 文件配置完成。

2，将集群中的所有节点的防火墙关闭。

3，将集群中所有节点的时间设置一致。

```
yum intall ntpdate
ntpdate ntp1.aliyun.com
```

4，将所有节点设置免密登录

```
ssh-keygen
ssh-copy-id -i /root/.ssh/id_rsa.pub node01（节点名称）
```

5，解压 hbase 安装包

```
tar -zxvf hbase-2.0.5-bin.tar.gz -C /opt.bigdata
cd hbase-2.0.5/
```

6，在 /etc/profile 文件中配置 HBase 环境变量

```
export HBASE_HOME=/opt/bigdata/hbase-2.0.5
将 $HBASE_HOME 设置到 PATH 路径中
```

7，进入 /opt/bigdata/hbase-2.0.5/conf 目录中，在 hbase-env.sh 文件中添加 JAVA_HOME。

```
# 设置 JAVA 环境变量
JAVA_HOME/usr/java/jdk1.8.0_181-amd64
# 设置是否使用自己的 Zookeeper
HBASE_MANAGES_ZK=false
```

8，进到 /top/bigdata/hbase-2.0.5/conf 目录中，在 hbase-site.xml 文件中添加 hbase 相关属性。

```xml
<configuration>
	<property>
    	<name>hbase.rootdir</name>
        <value>hdfs://mycluster/hbase</value>
    </property>
    <property>
    	<name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
    <property>
    	<name>hbase.zookpper.quorum</name>
        <value>node02,node03,node04</value>
    </property>
</configuration>
```

9，修改 regionservers 文件，设置 regionserver 分布在那几台节点

```
node02
node03
node04
```

10，如果需要配置 Master 的高可用，需要在 conf 目录下创建 backup-maters 文件，并添加好如下内容

```
node04
```

11，拷贝 hdfs-site.xml 文件到 conf 目录下

```
cp /opt/bigdata/hbase-2.6.5/ect/hadoop/hdfs-site.xml /opt/bigdata/hbase-2.0.5.conf
```

12，在任意目录下运行 hbase shell 命令，进入 hbase 命令行进行相关操作。



# 总结



# 参考文献

[1] [HBase 官网](http://hbase.apache.org/)