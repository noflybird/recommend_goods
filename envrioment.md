# 基于spark商品推荐系统

一、集群搭建
## 版本：
	centos7
	java jdk8
	hadoop 3.1.3
	Hive 3.1.2
	zookeeper 3.4.4
	kafka 2.12
	scala 2.12.12
	flume 1.8.0
	Hbase 2.1.1
	spark 2.4.0-hadoop 2.7
	storm 1.2.2
## 集群由五台linux虚拟机构建

## 系统说明:
节点		ip
centos1	192.168.199.150
centos2	192.168.199.151
centos3	192.168.199.152
centos4 192.168.199.153
centos5 192.168.199.154
网关:192.168.199.1
子网掩码:255.255.255.0
## .

二、Hadoop 搭建
主要修改两个文件：core-site.xml 和 hdfs-site.xml
另外还要修改 hadoop-env.sh，这个主要配置Jdk位置

etc/hadoop/core-site.xml

<property>
<name>fs.defaultFS</name>
<value>hdfs://集群名称</value>
</property>
<property>
<name>hadoop.tmp.dir</name>
<value>临时目录地址</value>
</property>
<property>
<name>ha.zookeeper.quorum</name>
<value>zookeeper集群</value>
</property>
etc/hadoop/hdfs-site.xml

<property>
<name>dfs.nameservices</name>
<value>集群名称</value>
</property>
<property>
<name>dfs.namenode.name.dir</name>
<value>namenode地址</value>
</property>
<property>
<name>dfs.datanode.data.dir</name>
<value>datanode地址</value>
</property>
<property>
<name>dfs.replication</name>
<value>副本数</value>
</property>
<property>

namenode 机器 别名
<name>dfs.ha.namenodes.ns</name>
<value>nn1,nn2</value>
</property>
<property>

namenode nn1 http地址
<name>dfs.namenode.http-address.ns.nn1</name>
<value>centos1:50070</value>
</property>
<property>

namenode nn2 http地址
<name>dfs.namenode.http-address.ns.nn2</name>
<value>centos2:50070</value>
</property>

namenode nn1 rpc地址
<property>
<name>dfs.namenode.rpc-address.ns.nn1</name>
<value>centos1:50080</value>
</property>

namenode nn2 rpc地址
<property>
<name>dfs.namenode.rpc-address.ns.nn2</name>
<value>centos2:50080</value>
</property>

journalnode 写入地址
<property>
<name>dfs.namenode.shared.edits.dir</name>
<value>qjournal://centos1:8485;centos2:8485;centos3:8485/ns</value>
</property>
journalnode 存储目录
<property>
<name>dfs.journalnode.edits.dir</name>
<value>/journalnode</value>
</property>
fencing 方式
<property>
<name>dfs.ha.fencing.methods</name>
<value>sshfence</value>
</property>
公钥地址
<property>
<name>dfs.ha.fencing.ssh.private-key-files</name>
<value>/home/hadoop/.ssh/id_rsa</value>
</property>
自动切换类
<property>
<name>dfs.client.failover.proxy.provider.ns</name>
<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
</property>
是否自动切换
<property>
<name>dfs.ha.automatic-failover.enabled</name>
<value>true</value>
</property>
不做权限验证
<property>
<name>dfs.permissions.enabled</name>
<value>false</value>
</property>
Hdfs 初始化
hadoop-daemon.sh start journalnode //在各台机启动journalnode
hdfs namenode -format //格式化 namenode
hadoop-daemon.sh start namenode // 启动 namenode
hdfs namenode -bootstrapStandby // 在另一台机同步namenode数据
hdfs zkfc -formatZK // 格式化zkfc
hadoop-daemon.sh start datanode //在各台机启动datanode
三、Yarn 搭建（这里是搭建 Yarn 高可用）
主要修改：yarn-site.xml

etc/hadoop/yarn-site.xml

是否实现 HA
<property>
<name>yarn.resourcemanager.ha.enabled</name>
<value>true</value>
</property>
Yarn HA id
<property>
<name>yarn.resourcemanager.cluster-id</name>
<value>yarn-ha</value>
</property>
Yan resourcemanager 机器别名
<property>
<name>yarn.resourcemanager.ha.rm-ids</name>
<value>rm1,rm2</value>
</property>
Yan resourcemanager host
<property>
<name>yarn.resourcemanager.hostname.rm1</name>
<value>centos4</value>
</property>
Yan resourcemanager host
<property>
<name>yarn.resourcemanager.hostname.rm2</name>
<value>centos5</value>
</property>
Yan resourcemanager web地址
<property>
<name>yarn.resourcemanager.webapp.address.rm1</name>
<value>centos4:8088</value>
</property> 
Yan resourcemanager web地址
<property>
<name>yarn.resourcemanager.webapp.address.rm2</name>
<value>centos5:8088</value>
</property>
zookeeper 地址
<property>
<name>yarn.resourcemanager.zk-address</name>
<value>centos3:2181,centos4:2181,centos5:2181</value>
</property>

是否自动切换
<property>
<name>yarn.resourcemanager.recovery.enabled</name>
<value>true</value>
</property>
HA 状态存放地址
<property>
<name>yarn.resourcemanager.store.class</name>
<value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
</property>
把rm和nm启动之后，使用以下命令运行wordcount，看是否hadoop和yarn运行成功 (/test 是测试文件)

bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar wordcount /test /output
四、Spark 搭建
conf/Spark-env.sh 增加以下内容，并根据自己情况填写

export HADOOP_CONF_DIR=
export YARN_CONF_DIR=
export JAVA_HOME=
export SPARK_MASTER_HOST=centos4
export SPARK_MASTER_PORT=7077
export SPARK_WORKER_PORT=7078
export SPARK_LOG_DIR=
export SPARK_PID_DIR=
export SPARK_LOCAL_DIRS=
conf/spark-default.conf 增加以下内容，这是spark优化

spark.default.parallelism=20
spark.executor.memory=512m
spark.driver.memory=512m
spark.driver.extraClassPath=/home/spark/lib
spark.executor.extraClassPath=/home/spark/lib
spark.executor.cores=2
spark.storage.memoryFraction=0.3
spark.executor.instances=5
spark.yarn.jars=
spark.shuffle.memoryFraction=0.5
spark on yarn
执行语句 看spark是否能提交任务到yarn

spark-submit –master yarn -deploy-mode cluster  xx.jar
五、hive MetastoreServer搭建 （元数据存放于 mysql）
conf/hive-site.xml

jdbc 连接地址
<property>
<name>javax.jdo.option.ConnectionURL</name>
<value>jdbc:mysql://localhost:3306/hive?serverTimezone=GMT%2B8</value>
</property>
jdbc driver
<property>
<name>javax.jdo.option.ConnectionDriverName</name>
<value>com.mysql.cj.jdbc.Driver</value>
</property>
mysql 用户名
<property>
<name>javax.jdo.option.ConnectionUserName</name>
<value>hive</value>
</property>
mysql 密码
<property>
<name>javax.jdo.option.ConnectionPassword</name>
<value>123456</value>
</property>
hive 在 hdfs存储路径
<property>
<name>hive.metastore.warehouse.dir</name>
<value>/hive/warehouse</value>
</property>
六、hive hiveserver2搭建
conf/Hive-site.xml

<property>
<name>hive.metastore.uris</name>
<value>thrift://metastore地址:9083</value>
</property>
七、Flume 搭建
搭建flume 读取 nginx
channel 为 kafka

建立flume.properties 文件 （名字、存放位置自定）

nginx.sources = source_1
nginx.channels = channel_1
nginx.sinks = sink_1 #sink_2
set source
nginx.sources.source_1.type = exec
nginx.sources.source_1.command = tail -F /var/log/nginx/access.log
nginx.sources.source_1.channels = channel_1
set sink_1
nginx.sinks.sink_1.type = avro
nginx.sinks.sink_1.hostname = centos3
nginx.sinks.sink_1.port = 52020
set channel
nginx.sinks.sink_1.channel = channel_1
nginx.channels.channel_1.type = org.apache.flume.channel.kafka.KafkaChannel
nginx.channels.channel_1.kafka.bootstrap.servers = centos1:9092,centos2:9092,centos3:9092
nginx.channels.channel_1.kafka.topic = centos1_flume_channel
nginx.channels.channel_1.kafka.consumer.group.id = centos1_flume_channel_consumer
nginx.channels.channel_1.kafka.consumer.timeout.ms = 70000
nginx.channels.channel_1.kafka.consumer.request.timeout.ms = 80000
nginx.channels.channel_1.kafka.consumer.fetch.max.wait.ms=7000
nginx.channels.channel_1.kafka.consumer.offset.flush.interval.ms = 50000
nginx.channels.channel_1.kafka.consumer.session.timeout.ms = 70000
nginx.channels.channel_1.kafka.consumer.heartbeat.interval.ms = 60000
nginx.channels.channel_1.kafka.consumer.enable.auto.commit = false
#nginx.channels.memoryChannel.capacity = 100
搭建flume 写入hdfs
建立flume.properties文件 （名字、存放位置自定）

collect.sources = source_1
collect.channels = channel_1
collect.sinks = sink_1
set source
collect.sources.source_1.type = avro
collect.sources.source_1.bind = centos3
collect.sources.source_1.port = 52020
collect.sources.source_1.channels = channel_1
set sink
collect.sinks.sink_1.channel = channel_1
collect.sinks.sink_1.type = hdfs
collect.sinks.sink_1.hdfs.path = hdfs://ns/nginx-ori/%Y/%m/%d/%H
collect.sinks.sink_1.hdfs.useLocalTimeStamp = true
collect.sinks.sink_1.hdfs.writeFormat = Text
collect.sinks.sink_1.hdfs.fileType = DataStream
collect.sinks.sink_1.hdfs.filePrefix = access_log
set channel
collect.channels.channel_1.type = memory
collect.channels.channel_1.capacity = 100
八、zookeeper 搭建
配置 conf/zoo.cfg (把conf/zoo_sample.cfg改名)

server.3 = centos3:2888:3888
server.4 = centos4:2888:3888
server.5 = centos5:2888:3888
在 zookeeper data目录分别建立 myid，并把zookeeper各台机的序号填进去

九、kafka 搭建
配置 config/server.properties

填写 broker.id =
填写 zookeeper.connect =

十、hbase 搭建
配置hbase-site.xml (这里hbase master 没有使用高可用)

<property>
     <name>hbase.tmp.dir</name>
     <value>d:/xx1</value>
</property>
<property>
     <name>hbase.rootdir</name>
     <value>file:d:/xx2</value>
</property>
<property>
     <name>hbase.zookeeper.property.dataDir</name>
     <value>d:/xx3</value>
</property>
<property>
     <name>hbase.cluster.distributed</name>
     <value>false</value>
</property>

十一、 storm 搭建
配置conf/storm.yaml

填写

storm.zookeeper.servers:
nimbus.host:
supervisor.slots.ports:
6700
6701
6702
6703
启动

storm nimbus &

storm supervisor &
