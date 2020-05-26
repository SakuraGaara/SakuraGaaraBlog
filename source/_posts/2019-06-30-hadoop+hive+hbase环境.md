---
title: hadoop+hive+hbase环境
categories: 大数据
tags:
  - hadoop
  - hive
  - hbase
abbrlink: 51709
date: 2019-06-30 00:00:00
---

> 相关软件准备 apache-hive-2.3.5-bin.tar.gz  hadoop-2.7.7.tar.gz  hbase-1.3.5-bin.tar.gz  mysql-5.7.22-linux-glibc2.12-x86_64.tar.gz  mysql-connector-java-5.1.42-bin.jar  zookeeper-3.4.14.tar.gz

### 环境介绍
1. 准备三台服务器，修改hostname主机名，分别为master, slave1, slave2,并将三台主机名添加到/etc/hosts中

> 10.27.214.15 slave1
  10.26.234.215 slave2
  10.26.108.150 master

2. 准备好相关的安装目录，mkdir /yibao/data/app (所有的项目都会安装在次目录下)
3. 配置三台服务器之间无密码登陆
<!--more-->

### 配置Java环境
三台都需要配置
```sh
JAVA_HOME=/usr/local/java/jdk1.8.0_60/
JAVA_BIN=/usr/local/java/jdk1.8.0_60/bin
JRE_HOME=/usr/local/java/jdk1.8.0_60/jre
PATH=$PATH:/usr/local/java/jdk1.8.0_60/bin:/usr/local/java/jdk1.8.0_60/jre/bin
CLASSPATH=/usr/local/java/jdk1.8.0_60/jre/lib:/usr/local/java/jdk1.8.0_60/lib:/usr/local/java/jdk1.8.0_60/jre/lib/charsets.jar
export  JAVA_HOME  JAVA_BIN JRE_HOME  PATH  CLASSPATH
```

### 安装hadoop集群
#### 解压hadoop到相关安装目录
```sh
tar zxvf hadoop-2.7.7.tar.gz -C /yibao/data/app/
```
#### 添加hadoop环境变量
/etc/profile
```sh
export HADOOP_HOME=/yibao/data/app/hadoop-2.7.7
```

#### 配置hadoop中的配置文件
主要配置四个配置文件，在hadoop-2.7.7/etc/hadoop目录中，分别为  
core-site.xml  
hdfs-site.xml  
yarn-site.xml  
mapred-site.xml(由mapred-site.xml.template拷贝）  
slaves  
hadoop-env.sh  

1. 修改core-site.xml  
```xml
<configuration>
    <property>
    <name>hadoop.tmp.dir</name>
    <value>/yibao/data/app/hadoop-2.7.7/tmp</value>
  </property>
  <property>
     <name>fs.defaultFS</name>
     <value>hdfs://master:9000</value>
  </property>
</configuration>
```

2. 修改hdfs-site.xml  
```xml
<configuration>
        <property>
                <name>dfs.namenode.secondary.http-address</name>
                <value>master:9001</value>
        </property>
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>file:/yibao/data/app/hadoop-2.7.7/namenode</value>
        </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>file:/yibao/data/app/hadoop-2.7.7/datanode</value>
        </property>
        <property>
                <name>dfs.replication</name>
                <value>2</value>
        </property>
        <property>
                <name>dfs.webhdfs.enabled</name>
                <value>true</value>
        </property>
        <property>
                <name>dfs.permissions</name>
                <value>false</value>
        </property>
        <property>
                <name>dfs.web.ugi</name>
                <value>supergroup</value>
        </property>
</configuration>
```

3. 修改mapred-site.xml  
```xml
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.address</name>
    <value>master:10020</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>master:19888</value>
  </property>
</configuration>
```

4. 修改yarn-site.xml  
```xml
<configuration>
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
        <property>
                <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
                <value>org.apache.hadoop.mapred.ShuffleHandler</value>
        </property>
        <property>
                <name>yarn.resourcemanager.address</name>
                <value>master:8032</value>
        </property>
        <property>
                <name>yarn.resourcemanager.scheduler.address</name>
                <value>master:8030</value>
        </property>
        <property>
                <name>yarn.resourcemanager.resource-tracker.address</name>
                <value>master:8031</value>
        </property>
        <property>
                <name>yarn.resourcemanager.admin.address</name>
                <value>master:8033</value>
        </property>
        <property>
                <name>yarn.resourcemanager.webapp.address</name>
                <value>master:8078</value>
        </property>
</configuration>
```

5. slaves  
```sh
slave1
slave2
```
6. hadoop-env.sh
```sh
export JAVA_HOME=/usr/local/java/jdk1.8.0_60
export HADOOP_SSH_OPTS="-p 22222"  #由于三台主机的ssh端口都是22222,所以此处添加
```
7. 将目录cp到slave1,slave2两台服务器中
```sh
scp -r -P 22222 hadoop-2.7.7 slave1:/yibao/data/app/
scp -r -P 22222 hadoop-2.7.7 slave2:/yibao/data/app/
```
8. 在slave1,slave2中/etc/profile添加HADOOP_HOME环境变量
```sh
export HADOOP_HOME=/yibao/data/app/hadoop-2.7.7
source /etc/profile
```

#### 验证并启动hadoop
9. 验证，在master节点中初始化namenode节点,确认无误后，启动hadoop集群
```sh 
./bin/hadoop namenode -format #初始化namenode节点
./sbin/start-all.sh  #启动hadoop集群
```
10. 之后可以使用jps命令查看每台机器上的Java进程
master节点：
```sh
28368 Jps
11287 SecondaryNameNode
11534 ResourceManager
11055 NameNode
```
slave1节点:
```sh
32020 DataNode
13626 Jps
32142 NodeManager
```
slave2节点：
```sh
2608 NodeManager
2469 DataNode
21542 Jps
```
此时可查看NameNode进程端口50070,访问 http://master:50070,可以看到熟悉的Hadoop界面


### 安装mysql
#### 解压mysql并安装
```sh
sudo -i #使用root用户安装
tar zxvf mysql-5.7.22-linux-glibc2.12-x86_64.tar.gz -C /yibao/data/app/
cd /yibao/data/app
mv mysql-5.7.22-linux-glibc2.12-x86_64 mysql
cd mysql
groupadd mysql
groupadd mysql
useradd -r -g mysql -s /sbin/nologin mysql
chown -R mysql.mysql .
./bin/mysqld --initialize --user=mysql --basedir=/yibao/data/app/mysql --datadir=/yibao/data/app/mysql/data  #返回root@localhost密码
```
#### 创建mysql配置文件
vim /etc/my.cnf
```sh
[mysqld]
basedir=/yibao/data/app/mysql 
datadir=/yibao/data/app/mysql/data
port=3306
character_set_server=utf8
socket=/tmp/mysql.sock
#skip-grant-tables

#innodb_buffer_pool_size=1G
innodb_log_file_size=256M
max_allowed_packet=64M

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
```

#### mysql加入服务
```sh
cp support-files/mysql.server /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld 
chkconfig --add mysqld
chkconfig mysqld on
service mysqld start
```

### 安装hive
基于MySQL的本地模式安装（hive只需安装master即可）
#### 解压hive安装包
```sh
tar zxvf apache-hive-2.3.5-bin.tar.gz -C /yibao/data/app/
cd /yibao/data/app
mv apache-hive-2.3.5-bin hive-2.3.5
```
#### 配置hive
1. 将mysql驱动包放置hive-2.3.5/lib目录
```sh
cp mysql-connector-java-5.1.42-bin.jar hive-2.3.5/lib/
```
2. 登陆mysql创建hive链接需要的账号，和数据库
```sh
mysql -uroot -p -hlocalhost
mysql> GRANT ALL PRIVILEGES ON *.* TO 'hive'@'localhost' IDENTIFIED BY "hive_password";
mysql> FLUSH PRIVILEGES;
mysql> CREATE DATABASE hive;
```
3. 配置hive环境变量
将以下加入/etc/profile文件
```sh
export HIVE_HOME=/yibao/data/app/hive-2.3.5
PATH=$PATH:$HIVE_HOME/bin
```
4. 配置hive配置文件hive-site.xml(由hive-default.xml.template复制生成)  

```xml
<property>
	<name>hive.metastore.warehouse.dir</name>
	<value>/yibao/data/app/hivedata</value>
	<description>location of default database for the warehouse</description>
</property>
<property>
	<name>hive.server2.authentication</name>
	<value>NONE</value>
</property>

<property>
	<name>javax.jdo.option.ConnectionDriverName</name>
	<value>com.mysql.jdbc.Driver</value>
	<description>Driver class name for a JDBC metastore</description>
</property>
<property>
	<name>javax.jdo.option.ConnectionURL</name>
	<value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true&amp;useSSL=false</value>
	<description>
		JDBC connect string for a JDBC metastore.
		To use SSL to encrypt/authenticate the connection, provide database-specific SSL flag in the connection URL.
		For example, jdbc:postgresql://myhost/db?ssl=true for postgres database.
	</description>
</property>
<property>
	<name>javax.jdo.option.ConnectionUserName</name>
	<value>hive</value>
	<description>Username to use against metastore database</description>
</property>
<property>
	<name>javax.jdo.option.ConnectionPassword</name>
	<value>hive_password</value>
	<description>password to use against metastore database</description>
</property>

<property>
	<name>hive.exec.local.scratchdir</name>
	<value>/yibao/data/app/hive_tmp/HiveJobsLog</value>
	<description>Local scratch space for Hive jobs</description>
</property>
<property>
	<name>hive.downloaded.resources.dir</name>
	<value>/yibao/data/app/hive_tmp/resources</value>
	<description>Temporary local directory for added resources in the remote file system.</description>
</property>
<property>
	<name>hive.querylog.location</name>
	<value>/yibao/data/app/hive_tmp/HiveRunLog</value>
	<description>Location of Hive run time structured log file</description>
</property>
<property>
	<name>hive.server2.logging.operation.log.location</name>
	<value>/yibao/data/app/hive_tmp/OpertitionLog</value>
	<description>Top level directory where operation logs are stored if logging functionality is enabled</description>
</property>

```
5. 创建hivedata,hive_tmp文件夹
```sh
mkdir -p /yibao/data/app/hivedata /yibao/data/app/hive_tmp
```
6. 修改hive-env.sh文件（由hive-env.sh.template拷贝生成）
```sh
export JAVA_HOME=/usr/local/java/jdk1.8.0_60
export HADOOP_HOME=/yibao/data/app/hadoop-2.7.7
export HIVE_HOME=/yibao/data/app/hive-2.3.5
export HIVE_CONF_DIR=$HIVE_HOME/conf
export HIVE_AUX_JARS_PATH=$HIVE_HOME/lib
```

#### 初始化并启动
7. hive初始化元数据
```sh
bin/schematool -initSchema -dbType mysql
```
8. 使用hive命令启动hive
```sh
hive> show databases;
OK
default
Time taken: 7.743 seconds, Fetched: 1 row(s)
```

### 安装zookeeper集群
zookeeper集群三台都需要安装,先配置master,之后scp到slave

#### 解压zookeeper
```sh
tar zxvf zookeeper-3.4.14.tar.gz -C /yibao/data/app/
cd /yibao/data/app/zookeeper-3.4.14/conf
cp zoo_sample.cfg zoo.cfg
```
#### 配置zookeeper
1. 修改zoo.cfg
```sh
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/yibao/data/app/zookeeper-3.4.14/data
clientPort=2181
server.0=master:2888:3888
server.1=slave1:2888:3888
server.2=slave2:2888:3888
```
2. 创建zookeeper data目录
```sh
mkdir /yibao/data/app/zookeeper-3.4.14/data
echo 0 > /yibao/data/app/zookeeper-3.4.14/data/myid
```
3. 将zookeeper-3.4.14同步至slave
```sh
scp -r -P 22222 zookeeper-3.4.14 slave1:/yibao/data/app/
scp -r -P 22222 zookeeper-3.4.14 slave2:/yibao/data/app/
```
4. 修改zookeeper id
slave1 下/yibao/data/app/zookeeper-3.4.14/data/myid 改为1
slave2 下/yibao/data/app/zookeeper-3.4.14/data/myid 改为2

5. 配置zookeeper环境变量,加入/etc/profile
```sh
export ZOOKEEPER_HOME=/yibao/data/app/zookeeper-3.4.14
export PATH=$PATH:$ZOOKEEPER_HOME/bin
```

6. 启动zookeeper  
master: ./bin/zkServer.sh start  
slave1: ./bin/zkServer.sh start  
slave2: ./bin/zkServer.sh start  

7. 验证
master上执行
```sh
./bin/zkCli.sh
2019-06-30 15:30:12,558 [myid:] - INFO  [main:Environment@100] - Client 
.......................
WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2181(CONNECTED) 0] create /sakura Gaara
Created /sakura
[zk: localhost:2181(CONNECTED) 1] get /sakura
Gaara
...................
[zk: localhost:2181(CONNECTED) 2]
```
slave1, slave2上去验证
 ```sh
./bin/zkCli.sh
Connecting to localhost:2181
2019-06-30 15:33:03,582 [myid:] - INFO  [main:Environment@100] - Client 
.......................
WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2181(CONNECTED) 0] get /sakura
Gaara
...........
[zk: localhost:2181(CONNECTED) 1] 
```

### 安装hbase集群
#### 解压hbase
```sh
tar zxvf hbase-1.3.5-bin.tar.gz -C /yibao/data/app
cd hbase-1.3.5/conf
```
#### 配置hbase
1. 修改hbase-site.xml
```xml
<configuration>  
  <property>  
    <name>hbase.rootdir</name>  
    <value>hdfs://master:9000/yibao/data/app/hbase-1.3.5</value>  
  </property>  
  <property>  
     <name>hbase.cluster.distributed</name>  
     <value>true</value>  
  </property>  
  <property>  
      <name>hbase.master</name>  
      <value>master:60000</value>  
  </property>  
   <property>  
    <name>hbase.zookeeper.property.dataDir</name>  
    <value>/yibao/data/app/hbase-1.3.5/zookeeperdata</value>  
  </property>  
  <property>  
    <name>hbase.zookeeper.quorum</name>  
    <value>master,slave1,slave2</value>  
  </property>  
  <property>  
    <name>hbase.zookeeper.property.clientPort</name>  
    <value>2181</value>  
  </property>  
  <property>
    <name>hbase.tmp.dir</name>
    <value>/yibao/data/app/hbase-1.3.5/tmpdata</value>
  </property>
</configuration>
```
2. 在conf下创建backup-masters文件，设置slave1为Backup Masters  
```sh
echo "slave1" > backup-masters
```
3. 修改regionservers，设置slave1,slave2为Region Servers  
```sh
cat regionservers 
slave1
slave2
```
4. 修改hbase-env.sh
```sh
export HBASE_SSH_OPTS="-p 22222" # 由于ssh端口是22222，所以此处添加
export JAVA_HOME=/usr/local/java/jdk1.8.0_60
export HBASE_CLASSPATH=/yibao/data/app/hbase-1.3.5/conf
export HBASE_MANAGES_ZK=false
export HBASE_HOME=/yibao/data/app/hbase-1.3.5
export HADOOP_HOME=/yibao/data/app/hadoop-2.7.7
export HBASE_LOG_DIR=/yibao/data/app/hbase-1.3.5/logs
```
5. 创建hbase配置所需的文件夹
```sh
cd /yibao/data/app/hbase-1.3.5
mkdir -p tmpdata zookeeperdata
```
6. 将hbase-1.3.5同步到slave1,slave2
```sh
scp -r -P 22222 hbase-1.3.5 slave1:/yibao/data/app/
scp -r -P 22222 hbase-1.3.5 slave2:/yibao/data/app/
```
7. 三台服务器配置hbase环境变量
```sh
export HBASE_HOME=/yibao/data/app/hbase-1.3.5
export PATH=$PATH:$HBASE_HOME/bin
```

#### 启动hbase
8. master上启动hbase,启动之前确认ZK已经启动
```sh
cd $HBASE_HOME
bin/start-hbase.sh
```
9. 启动后jps验证
master: jps
启动的HMaster进程为hbase进程
```sh
25588 HMaster
4678 Jps
11287 SecondaryNameNode
6552 QuorumPeerMain
11534 ResourceManager
11055 NameNode
```
slave1: jps
启动的HMaster进程为配置slave1为Backup Masters，HRegionServer进程为Region Servers
```sh
24402 Jps
10402 HRegionServer
11267 QuorumPeerMain
32020 DataNode
31546 Bootstrap
10508 HMaster
32142 NodeManager
```
slave2: jps
启动的HRegionServer进程为Region Servers
```sh
2608 NodeManager
2388 Bootstrap
2469 DataNode
14135 QuorumPeerMain
18442 HRegionServer
32431 Jps
```

10. web界面访问master主机HMaster进程的端口16010
也可以看出详细的信息，显示Master，Region Servers，Backup Masters，Tables等更多的详细信息