# 二、KafKa安装和部署

## 1.前置环境部署

### 1.1 Java环境准备

**1、上传解压安装包**

```shell
#解压上传的压缩包
tar -zxvf jdk-8u261-linux-x64.rpm
#修改安装包名字
mv 解压后的名字  新名字 
```

**2、配置环境变量并验证环境**

```shell
#配置环境变量
export JAVA_HOME=/root/opt/java
export PATH=$JAVA_HOME/bin:$PATH
# 生效环境变量
source /etc/profile
# 验证环境变量
[root@node01 opt]# java -version
java version "1.8.0_231"
Java(TM) SE Runtime Environment (build 1.8.0_231-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.231-b11, mixed mode)
```



### 1.2 Zookeeper的安装配置  

**1、上传解压安装zookeeper**

```shell
#解压安装包
tar -zxf zookeeper-3.4.14.tar.gz
#改名
mv 解压后的名字  新名字 
```

**2、修改配置文件**

```shell
#进入安装包下修改zookeeper配置文件
cd /root/opt/zookeeper/conf
# 复制zoo_sample.cfg命名为zoo.cfg
cp zoo_sample.cfg zoo.cfg
# 编辑zoo.cfg文件
vim zoo.cfg
#修改Zookeeper保存数据的目录，dataDir
dataDir=指定Zookeeper保存数据的目录路径
```

**3、环境变量配置**

```shell
#设置环境变量ZOO_LOG_DIR，指定Zookeeper保存日志的位置
export ZOO_LOG_DIR=/root/opt/zookeeper/log
#ZOOKEEPER_PREFIX指向Zookeeper的解压目录
export ZOOKEEPER_PREFIX=/root/opt/zookeeper
#将Zookeeper的bin目录添加到PATH中
export PATH=$PATH:$ZOOKEEPER_PREFIX/bin

#使环境变量生效
 source /etc/profile
```

**4、启动zookeeper验证**

```shell
#启动zookeeper
[root@node01 ~]# zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /root/opt/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
#查看zookeeper状态，这里是启动了3台zookeeper，所以这里这台状态是follower角色
[root@node01 js]# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /root/opt/zookeeper/bin/../conf/zoo.cfg
Mode: follower
```



## 2.KafKa安装和部署

### 2.1单机版安装部署

**1、上传解压安装kafka**

```shell
#解压
tar -zxf kafka_2.12-1.0.2.tgz
```

**2、配置环境变量**

```shell
#配置
export KAFKA_HOME=/root/opt/kafka
export PATH=$PATH:$KAFKA_HOME/bin
#使环境变量生效
source /etc/profile
```

**3、配置安装目录下/config中的server.properties文件**  

**配置Kafka连接Zookeeper的地址**

此处使用本地启动的Zookeeper实例，连接地址是localhost:2181，后面的 myKafka 是Kafka在Zookeeper中的根节点路径：  

```shell
############################# Zookeeper #############################

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=node01:2181/myKafKa
```

**配置Kafka存储持久化数据的目录**  

首先需要预先创建好相应的目录

```shell
#创建Kafka存储持久化数据的目录
mkdir /root/opt/kafka/kafka-logs
```

配置：

```shell
############################# Log Basics #############################

# A comma seperated list of directories under which to store log files
log.dirs=/root/opt/kafka/kafka-logs
```

**完整的配置文件server.properties**

还有其他配置

```shell
[root@node01 config]# cat server.properties 
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# see kafka.server.KafkaConfig for additional details and defaults

############################# Server Basics #############################

# The id of the broker. This must be set to a unique integer for each broker.
broker.id=001

############################# Socket Server Settings #############################

# The address the socket server listens on. It will get the value returned from 
# java.net.InetAddress.getCanonicalHostName() if not configured.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
#listeners=PLAINTEXT://:9092

# Hostname and port the broker will advertise to producers and consumers. If not set, 
# it uses the value for "listeners" if configured.  Otherwise, it will use the value
# returned from java.net.InetAddress.getCanonicalHostName().
advertised.listeners=PLAINTEXT://node01:9092

# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
#listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL

# The number of threads that the server uses for receiving requests from the network and sending responses to the network
num.network.threads=3

# The number of threads that the server uses for processing requests, which may include disk I/O
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=102400

# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=102400

# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600


############################# Log Basics #############################

# A comma seperated list of directories under which to store log files
log.dirs=/root/opt/kafka/kafka-logs

# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across
# the brokers.
num.partitions=3

# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
num.recovery.threads.per.data.dir=1

############################# Internal Topic Settings  #############################
# The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
# For anything other than development testing, a value greater than 1 is recommended for to ensure availability such as 3.
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

############################# Log Flush Policy #############################

# Messages are immediately written to the filesystem but by default we only fsync() to sync
# the OS cache lazily. The following configurations control the flush of data to disk.
# There are a few important trade-offs here:
#    1. Durability: Unflushed data may be lost if you are not using replication.
#    2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush.
#    3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to exceessive seeks.
# The settings below allow one to configure the flush policy to flush data after a period of time or
# every N messages (or both). This can be done globally and overridden on a per-topic basis.

# The number of messages to accept before forcing a flush of data to disk
#log.flush.interval.messages=10000

# The maximum amount of time a message can sit in a log before we force a flush
#log.flush.interval.ms=1000

############################# Log Retention Policy #############################

# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.

# The minimum age of a log file to be eligible for deletion due to age
#数据在磁盘保存的时间，单位小时
log.retention.hours=168

# A size-based retention policy for logs. Segments are pruned from the log unless the remaining
# segments drop below log.retention.bytes. Functions independently of log.retention.hours.
#log.retention.bytes=1073741824

# The maximum size of a log segment file. When this size is reached a new log segment will be created.
#允许保存数据的大小
log.segment.bytes=1073741824

# The interval at which log segments are checked to see if they can be deleted according
# to the retention policies
log.retention.check.interval.ms=300000

############################# Zookeeper #############################

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=node01:2181/myKafKa

# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=6000


############################# Group Coordinator Settings #############################

# The following configuration specifies the time, in milliseconds, that the GroupCoordinator will delay the initial consumer rebalance.
# The rebalance will be further delayed by the value of group.initial.rebalance.delay.ms as new members join the group, up to a maximum of max.poll.interval.ms.
# The default value for this is 3 seconds.
# We override this to 0 here as it makes for a better out-of-the-box experience for development and testing.
# However, in production environments the default value of 3 seconds is more suitable as this will help to avoid unnecessary, and potentially expensive, rebalances during application startup.
group.initial.rebalance.delay.ms=0
```

### 2.2启动测试

**1、启动zookeeper**

```shell
zkServer.sh start
```

**2、查看zookeeper状态是否启动成功**

```shell
zkServer.sh status
```

**3、启动KafKa**

```shell
#命令：[root@node01 config]# kafka-server-start.sh 配置文件的路径
[root@node01 config]# kafka-server-start.sh server.properties
```

后台方式启动：加上 -daemon

```shell
[root@node01 config]# kafka-server-start.sh  -daemon server.properties
```

查看Kafka的后台进程：  

```shell
ps aux | grep kafka
```

停止后台方式启动：

```shell
[root@node01 config]# kafka-server-stop.sh 
```

