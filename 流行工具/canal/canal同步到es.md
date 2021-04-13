# Canal

[官方文档](https://github.com/alibaba/canal/wiki)

## 一、简介

&emsp;&emsp;`canal`主要用途是对`MySQL`数据库增量日志进行解析，提供增量数据的订阅和消费，简单说就是可以对`MySQL`的增量数据进行实时同步，支持同步到`MySQL`、`Elasticsearch`、`HBase`等数据存储中去。



## 二、工作原理

&emsp;&emsp;`canal`会模拟`MySQL`主库和从库的交互协议，从而伪装成`MySQL`的从库，然后向`MySQL`主库发送`dump`协议，`MySQL`主库收到`dump`请求会向`canal`推送`binlog`，`canal`通过解析`binlog`将数据同步到其他存储中去。

![img](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20210319114001.png)

## 三、组件介绍

### 3.1 Canal-server(canal-deploy)

&emsp;&emsp;可以直接监听`MySQL`的`binlog`，把自己伪装成`MySQL`的从库，只负责接收数据，并不做处理。



### 3.2 Canal-adapter

&emsp;&emsp;相当于`canal`的客户端，会从`canal-server`中获取数据，然后对数据进行同步，可以同步到`MySQL`、`Elasticsearch`和`HBase`等存储中去。



### 3.3 Cannal-admin

&emsp;&emsp;为`canal`提供整体配置管理、节点运维等面向运维的功能，提供相对友好的`WebUI`操作界面，方便更多用户快速和安全的操作。

> 由于不同版本的`MySQL`、`Elasticsearch`和`canal`会有兼容性问题，所以我们先对其使用版本做个约定。

![img](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20210319114529)





## 四、使用

&emsp;&emsp;以`MySQL`实时同步数据到`Elasticsearch`为例。
&emsp;&emsp;需要下载`canal`的各个组件`canal-server、canal-adapter、canal-admin`，下载地址：[https://github.com/alibaba/canal/releases](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Falibaba%2Fcanal%2Freleases)



### 4.1 配置Mysql

&emsp;&emsp;由于`canal`是通过订阅`MySQL`的`binlog`来实现数据同步的，所以我们需要开启`MySQL`的`binlog`写入功能，并设置`binlog-format`为`ROW`模式

```sh
[mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=101 
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql  
## 开启二进制日志功能
log-bin=mall-mysql-bin  
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M  
## 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=row  
## 二进制日志过期清理时间。默认值为0，表示不自动清理。
expire_logs_days=7  
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062  
```



**检查配置是否生效：**

1. 通过如下命令查看`binlog`是否启用

   ```sql
   show variables like '%log_bin%'
   ```

   ```sqlite
   +---------------------------------+-------------------------------------+
   | Variable_name                   | Value                               |
   +---------------------------------+-------------------------------------+
   | log_bin                         | ON                                  |
   | log_bin_basename                | /var/lib/mysql/mall-mysql-bin       |
   | log_bin_index                   | /var/lib/mysql/mall-mysql-bin.index |
   | log_bin_trust_function_creators | OFF                                 |
   | log_bin_use_v1_row_events       | OFF                                 |
   | sql_log_bin                     | ON                                  |
   +---------------------------------+-------------------------------------+
   ```

2. 查看`mysql`的`binlog`模式

   ```sql
   show variables like 'binlog_format%'; 
   ```

   ```sqlite
   +---------------+-------+
   | Variable_name | Value |
   +---------------+-------+
   | binlog_format | ROW   |
   +---------------+-------+
   ```

   

**为从库配置权限**

1. 创建账号为`canal:canal`

   ```sql
   CREATE USER canal IDENTIFIED BY 'canal';
   GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
   FLUSH PRIVILEGES;
   ```

2. 创建测试数据库`canal-test`，之后创建一张商品表`product`

   ```sql
   CREATE TABLE `product`  (
     `id` bigint(20) NOT NULL AUTO_INCREMENT,
     `title` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
     `sub_title` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
     `price` decimal(10, 2) NULL DEFAULT NULL,
     `pic` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
     PRIMARY KEY (`id`) USING BTREE
   ) ENGINE = InnoDB AUTO_INCREMENT = 2 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
   ```

   



### 4.2 canal-server配置

**server的文件目录：**

```shell
├── bin
│   ├── restart.sh
│   ├── startup.bat
│   ├── startup.sh
│   └── stop.sh
├── conf
│   ├── canal_local.properties
│   ├── canal.properties
│   └── example
│       └── instance.properties
├── lib
├── logs
│   ├── canal
│   │   └── canal.log
│   └── example
│       ├── example.log
│       └── example.log
└── plugin
```



**修改`conf/example/instance.properties`，主要是数据库相关配置**

```properties
# 需要同步数据的MySQL地址
canal.instance.master.address=127.0.0.1:3306
canal.instance.master.journal.name=
canal.instance.master.position=
canal.instance.master.timestamp=
canal.instance.master.gtid=
# 用于同步数据的数据库账号
canal.instance.dbUsername=canal
# 用于同步数据的数据库密码
canal.instance.dbPassword=canal
# 数据库连接编码
canal.instance.connectionCharset = UTF-8
# 需要订阅binlog的表过滤正则表达式
canal.instance.filter.regex=.*\\..*
```



**使用`startup.sh`脚本启动服务**

```shell
# 启动脚本
sh bin/startup.sh
# 停止脚本
sh bin/stop.sh
```



**打印日志检测是否成功**

1. 查看服务日志信息

   ```shell
   tail -f logs/canal/canal.log
   ```

   ```shell
   2021-03-19 13:20:33.162 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## set default uncaught exception handler
   2021-03-19 13:20:33.203 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## load canal configurations
   2021-03-19 13:20:33.224 [main] INFO  com.alibaba.otter.canal.deployer.CanalStarter - ## start the canal server.
   2021-03-19 13:20:33.306 [main] INFO  com.alibaba.otter.canal.deployer.CanalController - ## start the canal server[192.168.1.95(192.168.1.95):11111]
   2021-03-19 13:20:34.789 [main] INFO  com.alibaba.otter.canal.deployer.CanalStarter - ## the canal server is running now ......
   ```

2. 查看`instance`日志信息

   ```shell
   tail -f logs/example/example.log 
   ```

   ```shell
   2021-03-19 13:20:33.876 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [example/instance.properties]
   2021-03-19 13:20:34.146 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [canal.properties]
   2021-03-19 13:20:34.147 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [example/instance.properties]
   2021-03-19 13:20:34.713 [main] INFO  c.a.otter.canal.instance.spring.CanalInstanceWithSpring - start CannalInstance for 1-example 
   2021-03-19 13:20:34.724 [main] WARN  c.a.o.canal.parse.inbound.mysql.dbsync.LogEventConvert - --> init table filter : ^.*\..*$
   2021-03-19 13:20:34.724 [main] WARN  c.a.o.canal.parse.inbound.mysql.dbsync.LogEventConvert - --> init table black filter : ^mysql\.slave_.*$
   2021-03-19 13:20:34.737 [main] INFO  c.a.otter.canal.instance.core.AbstractCanalInstance - start successful....
   2021-03-19 13:20:34.856 [destination = example , address = /127.0.0.1:3306 , EventParser] WARN  c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - ---> begin to find start position, it will be long time for reset or first position
   2021-03-19 13:20:34.856 [destination = example , address = /127.0.0.1:3306 , EventParser] WARN  c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - prepare to find start position just show master status
   2021-03-19 13:20:36.059 [destination = example , address = /127.0.0.1:3306 , EventParser] WARN  c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - ---> find start position successfully, EntryPosition[included=false,journalName=wym-mysql-bin.000001,position=4,serverId=101,gtid=<null>,timestamp=1616118647000] cost : 1186ms , the next step is binlog dump
   ```



### 4.3 canal-adapter配置

**文件目录:**

```shell
├── bin
│   ├── adapter.pid
│   ├── restart.sh
│   ├── startup.bat
│   ├── startup.sh
│   └── stop.sh
├── conf
│   ├── application.yml
│   ├── es6
│   ├── es7
│   │   ├── biz_order.yml
│   │   ├── customer.yml
│   │   └── product.yml
│   ├── hbase
│   ├── kudu
│   ├── logback.xml
│   ├── META-INF
│   │   └── spring.factories
│   └── rdb
├── lib
├── logs
│   └── adapter
│       └── adapter.log
└── plugin
```



**修改配置文件：**

1. 修改`conf/application.yml`：主要是修改canal-server配置、数据源配置和客户端适配器配置

   ```yml
   canal.conf:
     mode: tcp # 客户端的模式，可选tcp kafka rocketMQ
     flatMessage: true # 扁平message开关, 是否以json字符串形式投递数据, 仅在kafka/rocketMQ模式下有效
     zookeeperHosts:    # 对应集群模式下的zk地址
     syncBatchSize: 1000 # 每次同步的批数量
     retries: 0 # 重试次数, -1为无限重试
     timeout: # 同步超时时间, 单位毫秒
     accessKey:
     secretKey:
     consumerProperties:
       # canal tcp consumer
       canal.tcp.server.host: 127.0.0.1:11111 #设置canal-server的地址
       canal.tcp.zookeeper.hosts:
       canal.tcp.batch.size: 500
       canal.tcp.username:
       canal.tcp.password:
   
     srcDataSources: # 源数据库配置
       defaultDS:
         url: jdbc:mysql://127.0.0.1:3306/canal_test?useUnicode=true
         username: canal
         password: canal
     canalAdapters: # 适配器列表
     - instance: example # canal实例名或者MQ topic名
       groups: # 分组列表
       - groupId: g1 # 分组id, 如果是MQ模式将用到该值
         outerAdapters:
         - name: logger # 日志打印适配器
         - name: es7 # ES同步适配器
           hosts: 127.0.0.1:9200 # ES连接地址
           properties:
             mode: rest # 模式可选transport(9300) 或者 rest(9200)
             # security.auth: test:123456 #  only used for rest mode
             cluster.name: docker-cluster # ES集群名称
   ```

2. 添加`canal-adapter/conf/es7/product.yml`：配置`MySQL`中的表与`Elasticsearch`中索引的映射关系

   ```yml
   dataSourceKey: defaultDS # 源数据源的key, 对应上面配置的srcDataSources中的值
   destination: example  # canal的instance或者MQ的topic
   groupId: g1 # 对应MQ模式下的groupId, 只会同步对应groupId的数据
   esMapping:
     _index: canal_product # es 的索引名称
     _id: _id  # es 的_id, 如果不配置该项必须配置下面的pk项_id则会由es自动分配
     sql: "SELECT
            p.id AS _id,
            p.title,
            p.sub_title,
            p.price,
            p.pic
           FROM
            product p"        # sql映射
     etlCondition: "where a.c_time>={}"   #etl的条件参数
     commitBatch: 3000   # 提交批大小
   ```



**启动服务：**

```shell
# 启动服务
sh bin/startup.sh
# 停止服务
sh bin/stop.sh
```

- 检查是否启动成功

  ```shell
  tail -f logs/adapter/adapter.log
  ```

  ```shell
  2021-03-19 13:45:53.487 [main] INFO  c.a.o.canal.adapter.launcher.loader.CanalAdapterLoader - Load canal adapter: logger succeed
  2021-03-19 13:45:53.756 [main] INFO  c.a.o.c.client.adapter.es.core.config.ESSyncConfigLoader - ## Start loading es mapping config ... 
  2021-03-19 13:45:53.825 [main] INFO  c.a.o.c.client.adapter.es.core.config.ESSyncConfigLoader - ## ES mapping config loaded
  2021-03-19 13:45:54.093 [main] INFO  c.a.o.canal.adapter.launcher.loader.CanalAdapterLoader - Load canal adapter: es7 succeed
  2021-03-19 13:45:54.104 [main] INFO  c.alibaba.otter.canal.connector.core.spi.ExtensionLoader - extension classpath dir: /Users/wangyiming/MyProject/canal/canal.adapter-1.1.5-SNAPSHOT/plugin
  2021-03-19 13:45:54.123 [main] INFO  c.a.o.canal.adapter.launcher.loader.CanalAdapterLoader - Start adapter for canal-client mq topic: example-g1 succeed
  2021-03-19 13:45:54.123 [Thread-4] INFO  c.a.otter.canal.adapter.launcher.loader.AdapterProcessor - =============> Start to connect destination: example <=============
  2021-03-19 13:45:54.123 [main] INFO  c.a.o.canal.adapter.launcher.loader.CanalAdapterService - ## the canal client adapters are running now ......
  2021-03-19 13:45:54.129 [main] INFO  org.apache.coyote.http11.Http11NioProtocol - Starting ProtocolHandler ["http-nio-8081"]
  2021-03-19 13:45:54.143 [main] INFO  org.apache.tomcat.util.net.NioSelectorPool - Using a shared selector for servlet write/read
  2021-03-19 13:45:54.159 [main] INFO  o.s.boot.web.embedded.tomcat.TomcatWebServer - Tomcat started on port(s): 8081 (http) with context path ''
  2021-03-19 13:45:54.164 [main] INFO  c.a.otter.canal.adapter.launcher.CanalAdapterApplication - Started CanalAdapterApplication in 5.06 seconds (JVM running for 5.827)
  2021-03-19 13:45:54.452 [Thread-4] INFO  c.a.otter.canal.adapter.launcher.loader.AdapterProcessor - =============> Subscribe destination: example succeed <=============
  ```

  



### 4.4 canal-admin配置



**目录结构：**

```shell
├── bin
│   ├── restart.sh
│   ├── startup.bat
│   ├── startup.sh
│   └── stop.sh
├── conf
│   ├── application.yml
│   ├── canal_manager.sql
│   ├── canal-template.properties
│   ├── instance-template.properties
│   ├── logback.xml
│   └── public
│       ├── avatar.gif
│       ├── index.html
│       ├── logo.png
│       └── static
├── lib
└── logs
```



1. 创建数据库`canal_manager`，执行脚本`/conf/canal_manager.sql`。

2. 修改配置文件`conf/application.yml`，主要配置数据库连接以及管理员账号

   ```yml
   server:
     port: 8089
   spring:
     jackson:
       date-format: yyyy-MM-dd HH:mm:ss
       time-zone: GMT+8
   
   spring.datasource:
     address: 127.0.0.1:3306
     database: canal_manager
     username: root
     password: root
     driver-class-name: com.mysql.jdbc.Driver
     url: jdbc:mysql://${spring.datasource.address}/${spring.datasource.database}?useUnicode=true&characterEncoding=UTF-8&useSSL=false
     hikari:
       maximum-pool-size: 30
       minimum-idle: 1
   
   canal:
     adminUser: admin
     adminPasswd: admin
   ```

3. 修改`server`的`conf/canal_local.properties`，修改完成后使用`sh bin/startup.sh local`启动服务。

   ```properties
   # register ip
   canal.register.ip =
   
   # canal admin config
   canal.admin.manager = 127.0.0.1:8089
   canal.admin.port = 11110
   canal.admin.user = admin
   canal.admin.passwd = 4ACFE3202A5FF5CF467898FC58AAB1D615029441
   # admin auto register
   canal.admin.register.auto = true
   canal.admin.register.cluster = 
   ```

4. 启动`canal-admin`服务

   ```shell
   sh bin/startup.sh
   ```

5. 校验服务是否成功

   ```shell
   tail -f logs/admin.log
   ```

   ```shell
   2021-03-19 14:33:40.023 [main] INFO  org.apache.coyote.http11.Http11NioProtocol - Starting ProtocolHandler ["http-nio-8089"]
   2021-03-19 14:33:40.040 [main] INFO  org.apache.tomcat.util.net.NioSelectorPool - Using a shared selector for servlet write/read
   2021-03-19 14:33:40.055 [main] INFO  o.s.boot.web.embedded.tomcat.TomcatWebServer - Tomcat started on port(s): 8089 (http) with context path ''
   2021-03-19 14:33:40.059 [main] INFO  com.alibaba.otter.canal.admin.CanalAdminApplication - Started CanalAdminApplication in 4.405 seconds (JVM running for 5.285)
   ```

6. 访问：[本地IP:端口(8089)](http://localhost:8089/)

   - 账号：admin
   - 密码：123456

## 五、数据同步演示

&emsp;&emsp;提供了本文`demo`可用的测试用例，测试结果可以自行查询`es`。

### 5.1 创建索引

```json
PUT canal_product
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "sub_title": {
        "type": "text"
      },
      "pic": {
        "type": "text"
      },
      "price": {
        "type": "double"
      }
    }
  }
}
```



### 5.2 插入数据

```sql
INSERT INTO product ( id, title, sub_title, price, pic ) VALUES ( 5, '小米8', ' 全面屏游戏智能手机 6GB+64GB', 1999.00, NULL);
```



### 5.3 修改数据

```sql
UPDATE product SET title='小米10' WHERE id=5;
```



### 5.4 删除数据

```sql
DELETE FROM product WHERE id=5;
```

