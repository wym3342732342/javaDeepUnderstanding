# kafka搭建

## 一、集群规划

| 序号 | 机器名称 |      IP       | 硬件资源 |       安装服务        |
| :--: | :------: | :-----------: | :------: | :-------------------: |
|  1   | kafka-1  | 192.168.1.113 |  4核 8G  | jdk、zookeeper、kafka |
|  2   | kafka-2  | 192.168.1.128 |  4核 8G  | jdk、zookeeper、kafka |
|  3   | kafka-3  | 192.168.1.129 |  4核 8G  | jdk、zookeeper、kafka |



### 1.1、集群设备修改机器名称

```shell
# 查看hostname
hostname
# 设置名称
hostnamectl set-hostname kafka-X
# 重启
reboot
```



### 1.2 集群设备免密登录

1. 配置映射

   ```shell
   # 编写本地host
   vim /etc/hosts
   
   # 格式说明:
   #	服务器1IP 服务器1的计算机名 别名
   
   192.168.1.113 kafka-1 kfk1
   192.168.1.128 kafka-2 kfk2
   192.168.1.129 kafka-3 kfk3
   ```

2. 生成密钥

   ```shell
   #1、生成-一路回车
   ssh-keygen -t rsa
   
   #2、将公钥发送到需要免密的机器上
   ssh-copy-id -i ~/.ssh/id_rsa.pub ‘用户名’@‘发送的机器的ip地址’
   
   #3、正确输入目标密码后即可免密登录
   ```

   

### 1.3 集群jdk安装

&emsp;&emsp;略



## 二、开始搭建



### 2.1 安装zookeeper集群

1. **下载zookeeper**

   zookeeper官网地址：[点击进入](https://zookeeper.apache.org/)，其是一个分布式协调服务,管理我们的集群。
   官网提供配置说明：[点击进入](https://zookeeper.apache.org/doc/current/zookeeperStarted.html)

2. **执行命令安装：**

   ```shell
   # 下载
   wget --no-check-certificate https://dlcdn.apache.org/zookeeper/zookeeper-3.7.0/apache-zookeeper-3.7.0-bin.tar.gz
   # 解压
   tar -zxvf apache-zookeeper-3.7.0-bin.tar.gz
   # 重命名
   mv apache-zookeeper-3.7.0-bin zookeeper
   # 转移到指定路径
   cp -r zookeeper /usr/local/
   
   # 进入conf目录
   cd /usr/local/zookeeper/conf/
   
   # 复制配置文件
   cp zoo_sample.cfg zoo.cfg
   
   # 编辑文件[文件模板见下方]
   vim zoo.cfg
   
   # 创建myid
   mkdir -p /tmp/zookeeper
   echo 1 > /tmp/zookeeper/myid
   
   # 分发命令（也可以按照步骤重新在另外两台上走一边）,不要忘记修改myid
   scp -r zookeeper/ kfk2:$PWD
   scp -r zookeeper/ kfk3:$PWD
   
   # 启动所有zookeeper 并查看状态
   /usr/local/zookeeper/bin/zkServer.sh start
   /usr/local/zookeeper/bin/zkServer.sh status
   ```

   ```shell
   ### zoo.cfg 模板文件
   # The number of milliseconds of each tick
   
   # 时间单元，zk中的所有时间都是以该时间单元为基础，进行整数倍配置(单位是毫秒,下面配置的是2秒)
   tickTime=2000
   # The number of ticks that the initial
   # synchronization phase can take
   
   # follower在启动过程中，会从leader同步最新数据需要的最大时间。如果集群规模比较大，可以调大该参数
   initLimit=10
   # The number of ticks that can pass between
   # sending a request and getting an acknowledgement
   
   # leader与集群中所有机器进行心跳检查的最大时间。如果超出该时间，某follower没有回应，则说明该follower下线
   syncLimit=5
   # the directory where the snapshot is stored.
   # do not use /tmp for storage, /tmp here is just
   # example sakes.
   # 事务日志输出目录,myid也在这个里边找
   dataDir=/tmp/zookeeper
   # the port at which the clients will connect
   # 客户端连接端口
   clientPort=2181
   # the maximum number of client connections.
   # increase this if you need to handle more clients
   #maxClientCnxns=60
   #
   # Be sure to read the maintenance section of the
   # administrator guide before turning on autopurge.
   #
   # http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
   #
   # The number of snapshots to retain in dataDir
   # 需要保留文件数目，默认就是3个
   autopurge.snapRetainCount=3
   # Purge task interval in hours
   # Set to "0" to disable auto purge feature
   # 自动清理事务日志和快照文件的频率，这里是1个小时
   autopurge.purgeInterval=1
   
   
   ## Metrics Providers
   #
   # https://prometheus.io Metrics Exporter
   #metricsProvider.className=org.apache.zookeeper.metrics.prometheus.PrometheusMetricsProvider
   #metricsProvider.httpPort=7000
   #metricsProvider.exportJvmInfo=true
   #集群服务器配置，数字1/2/3需要与myid文件一致。右边两个端口，2888表示数据同步和通信端口；3888表示选举端口
   server.1=kfk1:2888:3888
   server.2=kfk2:2888:3888
   server.3=kfk3:2888:3888
   ```



### 2.2 安装kafka集群

1. **下载kafka**

   kafka官网：[点击进入](http://kafka.apache.org/)

   下载路径：[点击进入](http://kafka.apache.org/downloads)

2. **安装：**

   ```shell
   # 下载kafka
   wget --no-check-certificate https://dlcdn.apache.org/kafka/3.1.0/kafka_2.12-3.1.0.tgz
   # 解压并放到指定位置
   tar -zxvf kafka_2.12-3.1.0.tgz
   mv kafka_2.12-3.1.0 kafka
   cp -r kafka /usr/local/
   
   # 修改配置文件
   vim server.properties
   
   # 传输到另外两台服务器
    scp -r kafka/ kfk2:$PWD
    scp -r kafka/ kfk3:$PWD
   
   # 创建log.dirs
   mkdir -p /tmp/kafka-logs
   
   
   # 分别启动节点
   /usr/local/kafka/bin/kafka-server-start.sh -daemon /usr/local/kafka/config/server.properties
   # 停止
   /usr/local/kafka/bin/kafka-server-stop.sh
   ```

   ```shell
   ### server.properties 配置文件模板
   
   
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
   # 每个broker.id在集群中唯一，不能重复
   broker.id=0
   
   # 端口
   port=9092
   # broker 主机地址
   host.name=kfk1
   
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
   #advertised.listeners=PLAINTEXT://your.host.name:9092
   
   # Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
   #listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL
   
   # The number of threads that the server uses for receiving requests from the network and sending responses to the network
   # broker处理消息的线程数
   num.network.threads=3
   
   # The number of threads that the server uses for processing requests, which may include disk I/O
   # broker处理磁盘io的线程数
   num.io.threads=8
   
   # The send buffer (SO_SNDBUF) used by the socket server
   # socket发送数据缓冲区
   socket.send.buffer.bytes=102400
   
   # The receive buffer (SO_RCVBUF) used by the socket server
   # socket接收数据缓冲区
   socket.receive.buffer.bytes=102400
   
   # The maximum size of a request that the socket server will accept (protection against OOM)
   # socket接收请求最大值
   socket.request.max.bytes=104857600
   
   
   ############################# Log Basics #############################
   
   # A comma separated list of directories under which to store log files
   
   # kafka数据存放目录位置，多个位置用逗号隔开
   log.dirs=/tmp/kafka-logs
   
   # The default number of log partitions per topic. More partitions allow greater
   # parallelism for consumption, but this will also result in more files across
   # the brokers.
   
   # topic默认的分区数
   num.partitions=1
   
   # The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
   # This value is recommended to be increased for installations with data dirs located in RAID array.
   
   # 恢复线程数
   num.recovery.threads.per.data.dir=1
   
   ############################# Internal Topic Settings  #############################
   # The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
   # For anything other than development testing, a value greater than 1 is recommended to ensure availability such as 3.
   # 默认副本数
   offsets.topic.replication.factor=1
   transaction.state.log.replication.factor=1
   transaction.state.log.min.isr=1
   
   ############################# Log Flush Policy #############################
   
   # Messages are immediately written to the filesystem but by default we only fsync() to sync
   # the OS cache lazily. The following configurations control the flush of data to disk.
   # There are a few important trade-offs here:
   #    1. Durability: Unflushed data may be lost if you are not using replication.
   #    2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush.
   #    3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to excessive seeks.
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
   # 消息日志最大存储时间，这里是7天
   log.retention.hours=168
   
   # A size-based retention policy for logs. Segments are pruned from the log unless the remaining
   # segments drop below log.retention.bytes. Functions independently of log.retention.hours.
   #log.retention.bytes=1073741824
   
   # The maximum size of a log segment file. When this size is reached a new log segment will be created.
   # 每个日志段文件大小，1g
   log.segment.bytes=1073741824
   
   # The interval at which log segments are checked to see if they can be deleted according
   # to the retention policies
   # 消息日志文件大小检查间隔时间
   log.retention.check.interval.ms=300000
   
   ############################# Zookeeper #############################
   
   # Zookeeper connection string (see zookeeper docs for details).
   # This is a comma separated host:port pairs, each corresponding to a zk
   # server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
   # You can also append an optional chroot string to the urls to specify the
   # root directory for all kafka znodes.
   
   # zookeeper集群地址
   zookeeper.connect=kfk1:2181,kfk2:2181,kfk3:2181
   
   # Timeout in ms for connecting to zookeeper
   # 连接超时时间
   zookeeper.connection.timeout.ms=18000
   
   
   ############################# Group Coordinator Settings #############################
   
   # The following configuration specifies the time, in milliseconds, that the GroupCoordinator will delay the initial consumer rebalance.
   # The rebalance will be further delayed by the value of group.initial.rebalance.delay.ms as new members join the group, up to a maximum of max.poll.interval.ms.
   # The default value for this is 3 seconds.
   # We override this to 0 here as it makes for a better out-of-the-box experience for development and testing.
   # However, in production environments the default value of 3 seconds is more suitable as this will help to avoid unnecessary, and potentially expensive, rebalances during application startup.
   group.initial.rebalance.delay.ms=0
   ```

   