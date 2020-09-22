<div align="center"><img src="https://gitee.com/struggle3014/picBed/raw/master/name_code.png"></div>

# 导读

由于大数据组件种类繁多，对于组件的安装、部署、监控对于企业来说都是一件极具挑战的事情。市面上便应运而生了相关的技术，其中比较流行的是 Cloudera 公司的 Hadoop 发行版，CDH（Cloudera Distribution Hadoop）。本文将介绍 CDH 集群的部署及简易使用教程。

***持续更新中~***



# 目录

<nav>
<a href='#导读' style='text-decoration:none;font-weight:bolder'>导读</a><br/>
<a href='#目录' style='text-decoration:none;font-weight:bolder'>目录</a><br/>
<a href='#正文' style='text-decoration:none;font-weight:bolder'>正文</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1 Cloudera Manager 简介' style='text-decoration:none;${border-style}'>1 Cloudera Manager 简介</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1.1 Cloudera Manager 介绍' style='text-decoration:none;${border-style}'>1.1 Cloudera Manager 介绍</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1.2 Cloudera Manager 功能' style='text-decoration:none;${border-style}'>1.2 Cloudera Manager 功能</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#1.3 Cloudera Manager 架构' style='text-decoration:none;${border-style}'>1.3 Cloudera Manager 架构</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2 Cloudera Manager 安装部署' style='text-decoration:none;${border-style}'>2 Cloudera Manager 安装部署</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.1 环境介绍' style='text-decoration:none;${border-style}'>2.1 环境介绍</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.2 基础环境搭建' style='text-decoration:none;${border-style}'>2.2 基础环境搭建</a><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='#2.3 Cloudera Manager 离线安装部署' style='text-decoration:none;${border-style}'>2.3 Cloudera Manager 离线安装部署</a><br/>
<a href='#总结' style='text-decoration:none;font-weight:bolder'>总结</a><br/>
<a href='#参考文献' style='text-decoration:none;font-weight:bolder'>参考文献</a><br/>
</nav>

# 正文

## 1 Cloudera Manager 简介

![Hadoop生态圈](https://gitee.com/struggle3014/picBed/raw/master/Hadoop生态圈.png)

### 1.1 Cloudera Manager 介绍

Hadoop 是一个开源项目，所以很多公司在这个基础进行商业化，Cloudera 对 Hadoop 做了相应的改变。Cloudera 公司的发行版，我们将该版本称为 CDH（Cloudera Distribution Hadoop）。

CDH 是 Cloudera Manager 的安装包，本地或者云端，其中包括 Hadoop 的生态系统需要的所有组件，通过Cloudera Manager 统一管理和安装。

Cloudera Manager 是一个拥有集群自动化安装、中心化管理、集群监控、报警功能的一个平台，使得安装集群从几天的时间缩短在几个小时内，运维人员从数十人降低到几人以内，极大的提高集群管理的效率。



### 1.2 Cloudera Manager 功能

Cloudera Manager 提供的功能众多：

* **管理**：对集群进行管理，如添加、删除节点等操作。
* **监控**：监控集群的健康情况，对设置的各种指标和系统运行情况进行全面监控。
  * 系统监控指标
    * HDFS IO：5G 以上警戒值
    * 集群网络 IO：8G 以上警戒值
    * 集群网络 IO：4G 以上警戒值
    * 集群 CPU：100% 警戒值
    * Hadoop 各节点服务健康状况：服务节点宕机警报
  * HBase 监控指标
    * GC 信息：HBase 集群详细的 GC 情况，可观察故障时间点是否有大规模 GC 活动等。
    * Web 服务器响应：HBase 集群整体的服务请求响应时间，可观察故障时间是否飙升等。
    * 块缓存命中计数：HBase 读缓存的命中率情况，可观察是否因为缓存命中率低而导致的延迟。
    * 刷新队列大小：Memstore Flush 队列任务信息，可观察故障时间点是否有大量的flush写磁盘活动。
    * Memstore 大小：Memstore 内存使用情况，可观察故障时间点 Memstore 内存是否过载。
    * 缓慢获取操作：HBase 集群 get/scan 等获取操作响应过长的情况，可观察故障时间点是否有大量的缓慢获取操作。
* **诊断**：对集群出现的问题进行诊断，对出现的问题给出建议解决方案。
* **集成**：多组件进行整合。



![cloudera-manager-ui](https://gitee.com/struggle3014/picBed/raw/master/cloudera-manager-ui.png)



### 1.3 Cloudera Manager 架构

![cloudera-manager-architecture](https://gitee.com/struggle3014/picBed/raw/master/cloudera-manager-architecture.png)

* Server：负责软件安装，配置，启动和停止服务，管理服务运行的集群。

* Agent：安装在每台机器上。负责启动和停止过程，配置，监控主机。

* Management Service：由一组执行各种监控，警报和报告功能角色的服务。

* Database：存储配置和监视信息。

* Cloudera Repository：软件由 Cloudera 管理分布存储库。（类似于 Maven 的中心仓库）

* Clients：用于与服务器进行交互的接口（API 和 Admin Console）。



## 2 Cloudera Manager 安装部署

### 2.1 环境介绍

三台节点，均为 CentOS 7。

10.0.70.61

10.0.70.62

10.0.70.63



### 2.2 基础环境搭建

#### 2.2.1 主机名设置

hostnamectl 作用于 /ect/hostname 文件

```shell
hostnamectl set-hostname cdh61
hostnamectl set-hostname cdh75
hostnamectl set-hostname cdh76
```



#### 2.2.2 网络设置

nmtui 作用于 /etc/sysconfig/network-scripts/ifcfg-ens32 文件

```shell
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens32
UUID=6467b9d2-7ce7-4ea7-8b6d-e01ee078cd82
DEVICE=ens32
ONBOOT=yes
IPADDR=10.0.70.61
NETMASK=255.255.255.0
GATEWAY=10.0.70.254
DNS1=114.114.114.114
DNS2=8.8.8.8
```



#### 2.2.3 Hosts 设置

vim /etc/hosts

```shell
10.0.70.61 cdh61
10.0.70.75 cdh75
10.0.70.76 cdh76
```



#### 2.2.3 防火墙设置

```shell
firewall-cmd --state
systemctl start firewalld
systemctl stop firewalld
systemctl disable firewalld
```



#### 2.2.4 Selinux 设置

```shell
vi /etc/selinux/config
SELINUX=disabled
```



#### 2.2.5 时间同步设置

```shell
systemctl status chrony
yum -y install chrony

systemctl start chronyd
systemctl status chronyd

timedatectl

systemctl restart chronyd
```



#### 2.2.6 其他组件安装

```shell
yum -y install lrzsz
yum -y install wget
yum -y install net-tools
yum -y install httpd
yum -y install createrepo

# 启动httpd服务并设置开机自启动
systemctl start httpd
systemctl enable httpd
```



#### 2.2.7 重启

```shell
reboot
```



#### 2.2.8 免密登录设置

```shell
ssh-keygen -t rsa
cd ~/.ssh
ssh-copy-id -i cdh61
ssh-copy-id -i cdh75
ssh-copy-id -i cdh76
```



#### 2.2.9 JDK 安装部署

```shell
# JDK 环境设置
vim /etc/profile
export JAVA_HOME=
export PATH=$PATH:$JAVA_HOME/bin
```

在安装 Cloudera Manager 时报 JDK 版本不兼容，需要安装兼容版本，详细报错信息如下：

```
Error:Unable to find a compatible version of Java on this host,either because JAVA_HOME has not been set or because a compatible version of Java is not installed.
```



#### 2.2.10 数据库连接池设置

[MySQL 连接池下载地址](https://mvnrepository.com/artifact/mysql/mysql-connector-java/5.1.48)

```shell
mkdir -p /usr/share/java
mv mysql-connector-java-5.1.48.jar mysql-connector-java.jar
grep -rl '/usr/share/java/mysql-connector-java.jar' /opt/cloudera/*
```



#### 2.2.10 MySQL 安装部署

[MySQL 下载地址](https://downloads.mysql.com/archives/community/)

CentOS 7 已经将默认集成 Mariadb 而不是 MySQL，这对于多数还是依赖于 MySQL 的应用来说，需要手动的进行更新。

若使用默认的 Mariadb 可能会出现如下错误，换成 MySQL 即可。错误信息：error 2002 (hy000) mysql.sock /var/lib/mysql/mysql.sock。

* 卸载 Mariadb 相关模块

  * 查看 Mariadb 相关模块：rpm -qa | grep mariadb
  * 卸载：rpm -e mariadb-libs-5.5.64-1.el7.x86_64，rpm -e --nodeps mariadb-libs-5.5.64-1.el7.x86_64

* 安装 MySQL

  ```shell
  tar -xvf mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar
  rpm -ivh mysql-community-common-5.7.28-1.el7.x86_64.rpm
  rpm -ivh mysql-community-libs-5.7.28-1.el7.x86_64.rpm
  rpm -ivh mysql-community-client-5.7.28-1.el7.x86_64.rpm
  rpm -ivh mysql-community-server-5.7.28-1.el7.x86_64.rpm
  rpm -ivh mysql-community-libs-compat-5.7.28-1.el7.x86_64.rpm
  ```

* 配置 MySQL

  ```shell
  # 初始化 mysql 使 mysql 目录的拥有者为 mysql 用户
  mysqld --initialize --user=mysql
  # 最后一行将会有随机生成的密码
  cat /var/log/mysqld.log
  # 设置 mysql 服务自启
  systemctl start mysqld.service
  # 连接 mysql，出现如下错误：ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
  mysql -uroot –p’password’
  # 修复上一步错误
  ALTER USER 'root'@'localhost' IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER;
  flush privileges;
  ```

* MySQL 数据库及用户设置

  | 服务名                               | 数据库名  | 用户名 |
  | ------------------------------------ | --------- | ------ |
  | Cloudera Manager Server            | scm       | scm    |
  | Activity Monitor                   | amon      | amon   |
  | Reports Manager                    | rman      | rman   |
  | Hue                                | hue       | hue    |
  | Hive Metastore Server              | metastore | hive   |
  | Sentry Server                      | sentry    | sentry |
  | Cloudera Navigator Audit Server    | nav       | nav    |
  | Cloudera Navigator Metadata Server | navms     | navms  |
  | Oozie                              | oozie     | oozie  |

```sql
CREATE database cmserver DEFAULT charset utf8 collate utf8_general_ci;
GRANT ALL ON cmserver.* TO 'cmserveruser'@'%' IDENTIFIED BY '123456';
CREATE database metastore DEFAULT charset utf8 collate utf8_general_ci;
GRANT ALL ON metastore.* TO 'hiveuser'@'%' IDENTIFIED BY '123456';
CREATE database amon DEFAULT charset utf8 collate utf8_general_ci;
GRANT ALL ON amon.* TO 'amonuser'@'%' IDENTIFIED BY '123456';
CREATE database rman DEFAULT charset utf8 collate utf8_general_ci;
GRANT ALL ON rman.* TO 'rmanuser'@'%' IDENTIFIED BY '123456';
CREATE database oozie DEFAULT charset utf8 collate utf8_general_ci;
GRANT ALL ON oozie.* TO 'oozieuser'@'%' IDENTIFIED BY '123456';
CREATE database hue DEFAULT charset utf8 collate utf8_general_ci;
GRANT ALL ON hue.* TO 'hueuser'@'%' IDENTIFIED BY '123456';
FLUSH PRIVILEGES;
```



### 2.3 Cloudera Manager 离线安装部署

* MySQL（元数据）
* Cloudera Manager
* Parcel（HDFS，YARN，Hive，HBase，Zookeeper）

#### 2.3.1 Cloudera Manager 安装包下载

[Cloudera Manager 下载地址](https://archive.cloudera.com/cm6/)

```shell
# allkeys.asc
wget https://archive.cloudera.com/cm6/6.3.1/allkeys.asc -P ~/
# cloudera-manager.repo
wget https://archive.cloudera.com/cm6/6.3.1/redhat7/yum/cloudera-manager.repo -P ~/
# parcels

```

[CDH 下载地址](https://archive.cloudera.com/cdh6/)

```shell
# cdh parcel
wget https://archive.cloudera.com/cdh6/6.3.2/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel
# cdh parcel.sha1
wget https://archive.cloudera.com/cdh6/6.3.2/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel.sha1
# cdh parcel.sha256
wget 
https://archive.cloudera.com/cdh6/6.3.2/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel.sha256
# cdh manifest.json
wget https://archive.cloudera.com/cdh6/6.3.2/parcels/manifest.json
```



#### 2.3.2 Cloudera Manager yum 源配置

* 执行命令

```shell
mkdir -p /var/www/html/cloudera-repos
# 将 cm & allkeys.asc 放在上述目录
# 目录下执行命令 createrepo .
createrepo .
```

* 通过浏览器查看 RPM 包

![cloudera-repos](https://gitee.com/struggle3014/picBed/raw/master/cloudera-repos.png)

* 创建 CM 的 repo 文件

  ```shell
  cd /etc/yum.repos.d
  vim cloudera-manager.repo
  # 添加如下内容
  [cloudera-manager]
  name=Cloudera Manager 6.3.1
  baseurl=http://cdh61/cloudera-repos/
  gpgcheck=0
  enabled=1
  # 执行 yum clean all && yum makecache 命令
  yum clean all && yum makecache
  ```

  

#### 2.3.3 Cloudera Manager 安装

```shell
# 安装 cloudera manager
yum install cloudera-manager-daemons cloudera-manager-agent cloudera-manager-server
# 安装完 CM 后，在 /opt 下出现 cloudera 目录，将 parcel 包移动到该目录下
mv parcel /opt/cloudera/parcel-repo
# 重命名 sha1 为 sha 文件
mv /opt/cloudera/parcel-repo/xxx.sha1 /opt/cloudera/parcel-repo/xxx.sha
# 执行初始化脚本
/opt/cloudera/cm/schema/scm_prepare_database.sh mysql cmserver cmserveruser 123456
```



#### 2.3.4 Server 启动

```shell
# Server 启动命令
systemctl start cloudera-scm-server
# 观察日志
tail -200f /var/log/cloudera-scm-server/cloudera-scm-server.log
# 若无法正常启动且无日志，查看服务是否正常
systemctl status cloudera-scm-server.service 或 journalctl -xe
# 查看端口是否正常
netstat -an | grep 7180
# 打开浏览器访问，访问 http://cdh61:7180，账号密码都是 amdin
```



#### 2.3.5 Agent 安装

![cloudera-manager-anget-insatll](https://gitee.com/struggle3014/picBed/raw/master/cloudera-manager-anget-insatll.png)



#### 2.3.6 Parcels 安装

网络上提供的一些集群的参数优化，在安装的时候我们暂且可以忽略，安装完成之后根据CM的主机检查提供的建议再做优化不迟。

![cloudera-manager-parcel-install](https://gitee.com/struggle3014/picBed/raw/master/cloudera-manager-parcel-install.png)

# 总结

上述是 Cloudera Manager 的安装部署过程，从 Cloudera Manager 简介及 Cloudera Manager 安装部署两方面展开。更多细节可以参考官方文档中的内容。



# 参考文献

[1] [Cloudera 安装指南](https://www.cloudera.com/downloads/cdh/6-3-2.html)

[2] [Cloudera 官方文档](https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/introduction.html)