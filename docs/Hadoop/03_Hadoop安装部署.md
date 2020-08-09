![name_code](https://gitee.com/struggle3014/picBed/raw/master/name_code.png)#



# 导读

本文介绍 Hadoop 安装部署具体内容。

# 目录

<nav>
<a href='#导读' style='text-decoration:none;font-weight:bolder'>导读</a><br/>
<a href='#目录' style='text-decoration:none;font-weight:bolder'>目录</a><br/>
<a href='#正文' style='text-decoration:none;font-weight:bolder'>正文</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#基础环境' style='text-decoration:none;${border-style}'>基础环境</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#应用搭建、使用及验证' style='text-decoration:none;${border-style}'>应用搭建、使用及验证</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#单节点集群' style='text-decoration:none;${border-style}'>单节点集群</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#伪分布式集群' style='text-decoration:none;${border-style}'>伪分布式集群</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#分布式集群' style='text-decoration:none;${border-style}'>分布式集群</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#HA 集群' style='text-decoration:none;${border-style}'>HA 集群</a><br/>
<a href='#参考文献' style='text-decoration:none;font-weight:bolder'>参考文献</a><br/>
</nav>

# 正文

## 基础环境

环境：

| 软件   | 版本  |
| ------ | ----- |
| CentOS | 6.5   |
| JDK    | 1.8   |
| Hadoop | 2.6.5 |

1. 设置网路

2. 设置 IP

   查看 vm 的编辑->虚拟网络编辑器->观察 NAT模式的地址

   vi /etc/sysconfig/network-scripts/ifcfg-eth0

   ```shell
   DEVICE=eth0
   #HWADDR=00:0C:29:42:15:C2
   TYPE=Ethernet
   ONBOOT=yes
   NM_CONTROLLED=yes
   BOOTPROTO=static
   IPADDR=192.168.150.11
   NETMASK=255.255.255.0
   GATEWAY=192.168.150.2
   DNS1=223.5.5.5
   DNS2=114.114.114.114
   ```

3. 设置主机名

   vi /etc/sysconfig/network

   ```shell
   NETWORKING=yes
   HOSTNAME=node01
   ```

4. 设置本机 IP 到主机名的映射关系

   vi /etc/hosts

   ```shell
   192.168.150.11 node01
   192.168.150.12 node02
   ```

5. 关闭防火墙

   service iptables stop
   chkconfig iptables off

6. 关闭 selinux

   vi /etc/selinux/config

   ```shell
   SELINUX=disabled
   ```

7. 做时间同步

   yum install ntp  -y

   vi /etc/ntp.conf

   ```shell
   server ntp1.aliyun.com
   ```

   service ntpd start
   chkconfig ntpd on

8. 安装 JDK

   rpm -i   jdk-8u181-linux-x64.rpm

   *有一些软件只认：/usr/java/default

   vi /etc/profile

   ```shell
   export  JAVA_HOME=/usr/java/default
   export PATH=$PATH:$JAVA_HOME/bin
   ```

   source /etc/profile   |  .    /etc/profile

9. ssh 免密

   ssh  localhost	1,验证自己还没免密 2,被动生成了  /root/.ssh

   ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa

   cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

   如果 A 想免密的登陆到 B：

   ​	A：ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa

   ​	B：cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

   结论：B包含了A的公钥，A就可以免密的登陆。形象化解释：你去陌生人家里得撬锁，去女朋友家里：拿钥匙开门。

## 应用搭建、使用及验证

### 单节点集群

#### 规划路径

```shell
mkdir /opt/bigdata
tar xf hadoop-2.6.5.tar.gz
mv hadoop-2.6.5  /opt/bigdata/
pwd
	/opt/bigdata/hadoop-2.6.5
vi /etc/profile	
    export  JAVA_HOME=/usr/java/default
    export HADOOP_HOME=/opt/bigdata/hadoop-2.6.5
    export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
source /etc/profile   
```



#### 配置 Hadoop 角色

```shell
cd $HADOOP_HOME/etc/hadoop
	必须给hadoop配置javahome，要不ssh过去找不到
vi hadoop-env.sh
    export JAVA_HOME=/usr/java/default
    给出NN角色在哪里启动
vi core-site.xml
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://node01:9000</value>
	</property>
	配置hdfs  副本数为1.。。。
vi hdfs-site.xml
	<property>
		<name>dfs.replication</name>
		<value>1</value>
	</property>
	<property>
		<name>dfs.namenode.name.dir</name>
		<value>/var/bigdata/hadoop/local/dfs/name</value>
	</property>
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>/var/bigdata/hadoop/local/dfs/data</value>
	</property>
	<property>
		<name>dfs.namenode.secondary.http-address</name>
		<value>node01:50090</value>
	</property>
	<property>
		<name>dfs.namenode.checkpoint.dir</name>
		<value>/var/bigdata/hadoop/local/dfs/secondary</value>
	</property>
	配置DN这个角色再那里启动
vi slaves
	node01
```

#### 格式化及启动

hdfs namenode -format
		创建目录
		并初始化一个空的fsimage
		VERSION
			CID

start-dfs.sh
		第一次：datanode和secondary角色会初始化创建自己的数据目录

http://node01:50070
		修改windows： C:\Windows\System32\drivers\etc\hosts
			192.168.150.11 node01
			192.168.150.12 node02
			192.168.150.13 node03
			192.168.150.14 node04

#### 使用验证

cd   /var/bigdata/hadoop/local/dfs/name/current
		观察 editlog的id是不是再fsimage的后边

cd /var/bigdata/hadoop/local/dfs/secondary/current
		SNN 只需要从NN拷贝最后时点的FSimage和增量的Editlog


```shell
hdfs dfs -put hadoop*.tar.gz  /user/root
cd  /var/bigdata/hadoop/local/dfs/data/current/BP-281147636-192.168.150.11-1560691854170/current/finalized/subdir0/subdir0
for i in `seq 100000`;do  echo "hello hadoop $i"  >>  data.txt  ;done
hdfs dfs -D dfs.blocksize=1048576  -put  data.txt 
cd  /var/bigdata/hadoop/local/dfs/data/current/BP-281147636-192.168.150.11-1560691854170/current/finalized/subdir0/subdir0
检查data.txt被切割的块，他们数据什么样子
```



### 伪分布式集群

在一个节点启动所有的角色： NN,DN,SNN



### 分布式集群

#### 规划 Hadoop 角色

1）角色在哪里启动
			NN： core-site.xml:  fs.defaultFS  hdfs://node01:9000
			DN:  slaves:  node01
			SNN: hdfs-siet.xml:  dfs.namenode.secondary.http.address node01:50090
2) 角色启动时的细节配置：
			dfs.namenode.name.dir
			dfs.datanode.data.dir

伪分布式到分布式，需要对角色重新规划：

| **HOST** | **NN**                     | **NN**                     | **SNN** | **DN** | **ZKFC** | **ZK** |
| -------- | -------------------------- | -------------------------- | ------- | ------ | -------- | ------ |
| node01   | <font color="red">*</font> |                            |         |        | *        |        |
| node02   |                            | <font color="red">*</font> | *       | *      | *        | *      |
| node03   |                            |                            |         | *      |          | *      |
| node04   |                            |                            |         | *      |          | *      |

免密配置

	node01:
			stop-dfs.sh
	ssh 免密是为了什么 ：  启动start-dfs.sh：  在哪里启动，那台就要对别人公开自己的公钥
		这一台有什么特殊要求吗： 没有
	node02~node04:
		rpm -i jdk....
	node01:
		scp /root/.ssh/id_dsa.pub  node02:/root/.ssh/node01.pub
		scp /root/.ssh/id_dsa.pub  node03:/root/.ssh/node01.pub
		scp /root/.ssh/id_dsa.pub  node04:/root/.ssh/node01.pub
	node02:
		cd ~/.ssh
		cat node01.pub >> authorized_keys
	node03:
		cd ~/.ssh
		cat node01.pub >> authorized_keys
	node04:
		cd ~/.ssh
		cat node01.pub >> authorized_keys

#### 配置 Hadoop 角色

	node01:
			cd $HADOOP/etc/hadoop
			vi core-site.xml    不需要改
			vi hdfs-site.xml
				    <property>
					<name>dfs.replication</name>
					<value>2</value>
				    </property>
				    <property>
					<name>dfs.namenode.name.dir</name>
					<value>/var/bigdata/hadoop/full/dfs/name</value>
				    </property>
				    <property>
					<name>dfs.datanode.data.dir</name>
					<value>/var/bigdata/hadoop/full/dfs/data</value>
				    </property>
				    <property>
					<name>dfs.namenode.secondary.http-address</name>
					<value>node02:50090</value>
				    </property>
				    <property>
					<name>dfs.namenode.checkpoint.dir</name>
					<value>/var/bigdata/hadoop/full/dfs/secondary</value>
				    </property>
			vi slaves
				node02
				node03
				node04
	分发：
			cd /opt
			scp -r ./bigdata/  node02:`pwd`
			scp -r ./bigdata/  node03:`pwd`
			scp -r ./bigdata/  node04:`pwd`
	
		格式化启动
			hdfs namenode -format
			start-dfs.sh

#### 格式化及启动


​		格式化
​			Fsimage
​			VERSION
​		start-dfs.sh
​			加载我们的配置文件
​			通过ssh 免密的方式去启动相应的角色

### HA 集群

HA模式下：有一个问题，你的NN是2台？在某一时刻，谁是Active呢？client是只能连接Active

core-site.xml
fs.defaultFs -> hdfs://node01:9000

#### 规划 Hadoop 角色

| **HOST** | **NN**                     | **NN**                     | **JNN** | **DN** | **ZKFC** | **ZK** |
| -------- | -------------------------- | -------------------------- | ------- | ------ | -------- | ------ |
| node01   | <font color="red">*</font> |                            | *       |        | *        |        |
| node02   |                            | <font color="red">*</font> | *       | *      | *        | *      |
| node03   |                            |                            | *       | *      |          | *      |
| node04   |                            |                            |         | *      |          | *      |

#### 配置 Hadoop 角色

```shell
	core-site.xml
		<property>
		  <name>fs.defaultFS</name>
		  <value>hdfs://mycluster</value>
		</property>

		 <property>
		   <name>ha.zookeeper.quorum</name>
		   <value>node02:2181,node03:2181,node04:2181</value>
		 </property>

	hdfs-site.xml
		#以下是  一对多，逻辑到物理节点的映射
		<property>
		  <name>dfs.nameservices</name>
		  <value>mycluster</value>
		</property>
		<property>
		  <name>dfs.ha.namenodes.mycluster</name>
		  <value>nn1,nn2</value>
		</property>
		<property>
		  <name>dfs.namenode.rpc-address.mycluster.nn1</name>
		  <value>node01:8020</value>
		</property>
		<property>
		  <name>dfs.namenode.rpc-address.mycluster.nn2</name>
		  <value>node02:8020</value>
		</property>
		<property>
		  <name>dfs.namenode.http-address.mycluster.nn1</name>
		  <value>node01:50070</value>
		</property>
		<property>
		  <name>dfs.namenode.http-address.mycluster.nn2</name>
		  <value>node02:50070</value>
		</property>

		#以下是JN在哪里启动，数据存那个磁盘
		<property>
		  <name>dfs.namenode.shared.edits.dir</name>
		  <value>qjournal://node01:8485;node02:8485;node03:8485/mycluster</value>
		</property>
		<property>
		  <name>dfs.journalnode.edits.dir</name>
		  <value>/var/bigdata/hadoop/ha/dfs/jn</value>
		</property>
		
		#HA角色切换的代理类和实现方法，我们用的ssh免密
		<property>
		  <name>dfs.client.failover.proxy.provider.mycluster</name>
		  <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
		</property>
		<property>
		  <name>dfs.ha.fencing.methods</name>
		  <value>sshfence</value>
		</property>
		<property>
		  <name>dfs.ha.fencing.ssh.private-key-files</name>
		  <value>/root/.ssh/id_dsa</value>
		</property>
		
		#开启自动化： 启动zkfc
		 <property>
		   <name>dfs.ha.automatic-failover.enabled</name>
		   <value>true</value>
		 </property>
```

#### 流程梳理

基础设施
		ssh免密：
			1）启动start-dfs.sh脚本的机器需要将公钥分发给别的节点
			2）在HA模式下，每一个NN身边会启动ZKFC，
				ZKFC会用免密的方式控制自己和其他NN节点的NN状态
应用搭建
		HA 依赖 ZK  搭建ZK集群
		修改hadoop的配置文件，并集群同步
初始化启动
		1）先启动JN   hadoop-daemon.sh start journalnode 
		2）选择一个NN 做格式化：hdfs namenode -format   <只有第一次搭建做，以后不用做>
		3)启动这个格式化的NN ，以备另外一台同步  hadoop-daemon.sh start namenode 
		4)在另外一台机器中： hdfs namenode -bootstrapStandby
		5)格式化zk：   hdfs zkfc  -formatZK     <只有第一次搭建做，以后不用做>
		6) start-dfs.sh

使用

#### 实操

1）停止之前的集群
	2）免密：node01,node02
		node02: 
			cd ~/.ssh
			ssh-keygen -t dsa -P '' -f ./id_dsa
			cat id_dsa.pub >> authorized_keys
			scp ./id_dsa.pub  node01:`pwd`/node02.pub
		node01:
			cd ~/.ssh
			cat node02.pub >> authorized_keys
	3)zookeeper 集群搭建  java语言开发  需要jdk  部署在2,3,4
		node02:
			tar xf zook....tar.gz
			mv zoo...    /opt/bigdata
			cd /opt/bigdata/zoo....
			cd conf
			cp zoo_sample.cfg  zoo.cfg
			vi zoo.cfg
				datadir=/var/bigdata/hadoop/zk
				server.1=node02:2888:3888
				server.2=node03:2888:3888
				server.3=node04:2888:3888
			mkdir /var/bigdata/hadoop/zk
			echo 1 >  /var/bigdata/hadoop/zk/myid 
			vi /etc/profile
				export ZOOKEEPER_HOME=/opt/bigdata/zookeeper-3.4.6
				export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$ZOOKEEPER_HOME/bin
			. /etc/profile
			cd /opt/bigdata
			scp -r ./zookeeper-3.4.6  node03:`pwd`
			scp -r ./zookeeper-3.4.6  node04:`pwd`
		node03:
			mkdir /var/bigdata/hadoop/zk
			echo 2 >  /var/bigdata/hadoop/zk/myid
			*环境变量
			. /etc/profile
		node04:
			mkdir /var/bigdata/hadoop/zk
			echo 3 >  /var/bigdata/hadoop/zk/myid
			*环境变量
			. /etc/profile

​			node02~node04:
​			zkServer.sh start

4）配置hadoop的core和hdfs
5）分发配置
	给每一台都分发
6）初始化：
	1）先启动JN   hadoop-daemon.sh start journalnode 
	2）选择一个NN 做格式化：hdfs namenode -format   <只有第一次搭建做，以后不用做>
	3)启动这个格式化的NN ，以备另外一台同步  hadoop-daemon.sh start namenode 
	4)在另外一台机器中： hdfs namenode -bootstrapStandby
	5)格式化zk：   hdfs zkfc  -formatZK     <只有第一次搭建做，以后不用做>
	6) start-dfs.sh

#### 使用验证

​	1）去看jn的日志和目录变化：
​	2）node04
​		zkCli.sh 
​			ls /
​			启动之后可以看到锁：
​			get  /hadoop-ha/mycluster/ActiveStandbyElectorLock
​	3）杀死namenode 杀死zkfc
​		kill -9  xxx
​		a)杀死active NN
​		b)杀死active NN身边的zkfc
​		c)shutdown activeNN 主机的网卡 ： ifconfig eth0 down
​			2节点一直阻塞降级
​			如果恢复1上的网卡   ifconfig eth0 up  
​			最终 2 变成 active



# 参考文献

