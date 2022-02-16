# Java程序日志打印规范

> 摘录自拉勾教育。

## 一、日志技术介绍

### 1.1 日志技术框架一览

- **JUL：**JDK中的日志记录工具，也常称为`JDKLog`、`jdk-logging`。

- **LOG4J1：**一个具体的日志实现框架。

- **LOG4J2：**一个具体的日志实现框架，是`LOG4J1`的下一个版本。

- **LOGBACK：**一个具体的日志实现框架，但其性能更好。

- **JCL：**一个日志门面，提供统一的日志记录接口，也常称为`commons-logging`。

- **SLF4J：**一个日志门面，与`JCL`一样提供统一的日志记录接口，可以方便地切换看具体的实现框架。

  > 注意：JUL、LOG4J1、LOG4J2、LOGBACK是**日志实现框架**，而JCL、SLF4J是**日志实现门面（接口）**。

### 1.2 日志系统切换

&emsp;&emsp;在实际的日志转换过程中，`SLF4J`其实是充当一个**中介**的角色。例如当一个项目原来是使用`LOG4J`进行日志记录，但是要换成`LogBack`进行日志记录。

&emsp;&emsp;此时需要先将`LOG4J`转换成`SLF4J`日志系统，再从`SLF4J`日志系统转成`LogBack`日志系统。

- 从日志框架转SLF4J（如下图）

  **jul-to-slf4j：**`jdk-logging`到`slf4j`的桥梁
  **log4j-over-slf4j：**`log4j1`到`slf4j`的桥梁
  **jcl-over-slf4j：**`commons-logging`到`slf4j`的桥梁

- 从`SLF4J`转具体的日志框架（如下图）

  **slf4j-jdk14：**`slf4j`到`jdk-logging`的桥梁
  **slf4j-log4j12：**`slf4j`到`log4j1`的桥梁
  **log4j-slf4j-impl：**`slf4j`到`log4j2`的桥梁
  **logback-classic：**`slf4j`到`logback`的桥梁
  **slf4j-jcl：**`slf4j`到`commons-logging`的桥梁

&emsp;&emsp;例如一开始使用的是 `Log4J` 日志框架，现在希望转成 `LogBack` 框架，那么首先需要加入 `log4j-over-slf4j.jar` 将 `Log4J` 转成 `SLF4J`，之后再加入 `logback-classic.jar` 将 `SLF4J` 转成 `LogBack`。
![在这里插入图片描述](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20211101204700.png)

## 二、Java常用的日志框架案例

### 2.1 JDKLog:日志小刀

>（功能太简单，不支持占位符，扩展差，很少有人用）

```java
Logger logger = Logger.getLogger("JDKLog");
logger.info("Hello World");
12
```

### 2.2 Log4J:日志大炮

>（分为1.X和2.X两个版本，功能强大，扩展好，性能有待提升）
>
>在前几年的系统中经常使用log4j，所以现在很多公司需要将log4j切换到logback。肯定是不可能一行行修改的，需要通过下面的slf4j适配器进行转换。

**使用姿势：**

1. 使用 `Log4J` 框架首先需要引入依赖的包：

```xml
<!-- Log4J -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.6.2</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.6.2</version>
</dependency>
1234567891011
```

2. 增加配置文件 log4j2.xml 放在 resource 目录下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<Configuration status="WARN"> 
  <Appenders> 
    <Console name="Console" target="SYSTEM_OUT"> 
      <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/> 
    </Console> 
  </Appenders>  
  <Loggers> 
    <Root level="info"> <!-- 其中节点的 level 属性表示输出的最低级别 -->
      <AppenderRef ref="Console"/> 
    </Root> 
  </Loggers> 
</Configuration>
```

3. 最后编写一个测试类：

```java
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

/****
 ** Log4J Demo
 **/
public class Log4jLog {
    public static void main(String[] args) {
        Logger logger = LogManager.getLogger(Log4jLog.class);
        logger.debug("Debug Level");
        logger.info("Info Level");
        logger.warn("Warn Level");
        logger.error("Error Level");
    }
}
```

4. 运行测试类输出结果：

```shell
10:16:08.279 [main] INFO com.chanshuyi.Log4jLog - Info Level
10:16:08.280 [main] WARN com.chanshuyi.Log4jLog - Warn Level
10:16:08.280 [main] ERROR com.chanshuyi.Log4jLog - Error Level
```

5. 如果没有配置 log4j2.xml 配置文件，那么LOG4J将自动启用类似于下面的的配置文件：

```xml
<?xml version="1.0" encoding="utf-8"?>

<Configuration status="WARN"> 
  <Appenders> 
    <Console name="Console" target="SYSTEM_OUT"> 
      <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/> 
    </Console> 
  </Appenders>  
  <Loggers> 
    <Root level="error"> 
      <AppenderRef ref="Console"/> 
    </Root> 
  </Loggers> 
</Configuration>
1234567891011121314
```

6. 使用默认配置文件的输出结果：
   `11:40:07.377 [main] ERROR com.chanshuyi.Log4jLog - Error Level`
   从上面的使用步骤可以看出 Log4J 的使用稍微复杂一些，但是条理还是很清晰的。而且因为 Log4J 有多个分级`（DEBUG/INFO/WARN/ERROR）`记录级别，所以可以很好地记录不同业务问题。因为这些优点，所以在几年前几乎所有人都使用 Log4J 作为日志记录框架，群众基础可谓非常深厚。

&emsp;&emsp;但 `Log4J` 本身也存在一些缺点，比如不支持使用占位符，不利于代码阅读等缺点。但是相比起 `JDKLog`，`Log4J` 可以说是非常好的日志记录框架了。

### 2.3 LogBack：日志火箭

&emsp;&emsp;`LogBack` 其实可以说是 `Log4J` 的进化版，因为它们两个都**是同一个人（Ceki Gülcü）设计的开源日志组件**。`LogBack` 除了具备 `Log4j` 的所有优点之外，还解决了 `Log4J` 不能使用占位符的问题。



使用 `LogBack` 需要首先引入依赖：

```xml
<!-- LogBack -->
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-classic</artifactId>
  <version>1.1.7</version>
</dependency>
```



配置 `logback.xml` 配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <Pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</Pattern>
        </layout>
    </appender>
    <logger name="com.chanshuyi" level="TRACE"/>
 
    <root level="debug">
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```



`LogBack` 的**日志级别区分可以细分到类或者包**，这样就可以使日志记录变得更加灵活。之后在类文件中引入Logger类，并进行日志记录：

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
/****
 ** LogBack Demo
 **/
public class LogBack {
    static final Logger logger = LoggerFactory.getLogger(LogBack.class);
    public static void main(String[] args) {
        logger.trace("Trace Level.");
        logger.debug("Debug Level.");
        logger.info("Info Level.");
        logger.warn("Warn Level.");
        logger.error("Error Level.");
    }
}
```



输出结果：

```shell
14:34:45.747 [main] TRACE com.chanshuyi.LogBack - Trace Level.
14:34:45.749 [main] DEBUG com.chanshuyi.LogBack - Debug Level.
14:34:45.749 [main] INFO  com.chanshuyi.LogBack - Info Level.
14:34:45.749 [main] WARN  com.chanshuyi.LogBack - Warn Level.
14:34:45.749 [main] ERROR com.chanshuyi.LogBack - Error Level.
```



&emsp;&emsp;`LogBack` 解决了 `Log4J` 不能使用占位符的问题，这使得阅读日志代码非常方便。除此之外，`LogBack` 比 `Log4J` 有更快的运行速度，更好的内部实现。并且 `LogBack` 内部集成了 `SLF4J` 可以更原生地实现一些日志记录的实现。

## 三、SLF4J：适配器

&emsp;&emsp;因为在实际的项目应用中，有时候可能会**从一个日志框架切换到另外一个日志框架的需求**，这时候往往需要在代码上进行很大的改动。为了**避免切换日志组件时要改动代码**，这时候一个叫做 `SLF4J（Simple Logging Facade for Java，即Java简单日志记录接口集）`的东西出现了。如下图中的日志接口：
![在这里插入图片描述](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20211101204654.png)
&emsp;&emsp;**SLF4J（Simple Logging Facade for Java，即Java简单日志记录接口集)是一个日志的接口规范**，它对用户**提供了统一的日志接口**，屏蔽了不同日志组件的差异。这样我们**在编写代码的时候只需要看 SLF4J 这个接口文档即可**，不需要去理会不同日之框架的区别。而当需要更换日志组件的时候，只需要更换一个具体的日志组件Jar包就可以了。

### 3.1 SLF4J+JDKLog



`SLF4J` + `JDKLog` 需要在 `Maven` 中导入以下依赖包：

```xml
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>1.7.21</version>
</dependency>
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-jdk14</artifactId>
  <version>1.7.21</version>
</dependency>
```



编写测试类：

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
/****
 ** SLF4J + JDKLog
 **/
public class Slf4jJDKLog {
    final static Logger logger = LoggerFactory.getLogger(Slf4jJDKLog.class);
    public static void main(String[] args) {
        logger.trace("Trace Level.");
        logger.info("Info Level.");
        logger.warn("Warn Level.");
        logger.error("Error Level.");
    }
}
```



输出结果：

```shell
七月 15, 2016 3:30:02 下午 com.chanshuyi.slf4j.Slf4jJDKLog main
信息: Info Level.
七月 15, 2016 3:30:02 下午 com.chanshuyi.slf4j.Slf4jJDKLog main
警告: Warn Level.
七月 15, 2016 3:30:02 下午 com.chanshuyi.slf4j.Slf4jJDKLog main
严重: Error Level.
```

### 3.2 SLF4J+LOG4J

需要依赖的 `Jar `包：`slf4j-api.jar`、`slf4j-412.jar`、`log4j.jar`，导入`Maven`依赖：

```xml
<!-- 2.SLF4J + Log4J -->
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>1.7.21</version>
</dependency>
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
  <version>1.7.21</version>
</dependency>
<dependency>
  <groupId>log4j</groupId>
  <artifactId>log4j</artifactId>
  <version>1.2.17</version>
</dependency>
```



配置` log4j.xml `文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
 
<log4j:configuration xmlns:log4j='http://jakarta.apache.org/log4j/' >
 
    <appender name="myConsole" class="org.apache.log4j.ConsoleAppender">
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern"
                   value="[%d{dd HH:mm:ss,SSS\} %-5p] [%t] %c{2\} - %m%n" />
        </layout>
        <!--过滤器设置输出的级别-->
        <filter class="org.apache.log4j.varia.LevelRangeFilter">
            <param name="levelMin" value="debug" />
            <param name="levelMax" value="error" />
            <param name="AcceptOnMatch" value="true" />
        </filter>
    </appender>
 
    <!-- 根logger的设置-->
    <root>
        <priority value ="debug"/>
        <appender-ref ref="myConsole"/>
    </root>
</log4j:configuration>
123456789101112131415161718192021222324
```

我们还是用上面的代码，无需做改变，运行结果为：

```xml
[15 16:04:06,371 DEBUG] [main] slf4j.SLF4JLog - Debug Level.
[15 16:04:06,371 INFO ] [main] slf4j.SLF4JLog - Info Level.
[15 16:04:06,371 WARN ] [main] slf4j.SLF4JLog - Warn Level.
[15 16:04:06,371 ERROR] [main] slf4j.SLF4JLog - Error Level.
```

### 3.3 SLF4J+LogBack



导入依赖：

```xml
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>1.7.21</version>
</dependency>
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-classic</artifactId>
  <version>1.1.7</version>
</dependency>
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-core</artifactId>
  <version>1.1.7</version>
</dependency>
```



配置 `logback.xml` 文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <Pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</Pattern>
        </layout>
    </appender>
    <logger name="com.chanshuyi" level="TRACE"/>
 
    <root level="warn">
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```

还是用上面的代码，无需做改变，运行结果为：

```shell
16:08:01.040 [main] TRACE com.chanshuyi.slf4j.SLF4JLog - Trace Level.
16:08:01.042 [main] DEBUG com.chanshuyi.slf4j.SLF4JLog - Debug Level.
16:08:01.043 [main] INFO  com.chanshuyi.slf4j.SLF4JLog - Info Level.
16:08:01.043 [main] WARN  com.chanshuyi.slf4j.SLF4JLog - Warn Level.
16:08:01.043 [main] ERROR com.chanshuyi.slf4j.SLF4JLog - Error Level.
```

## 四、LogBack日志框架

> 那么在实际使用中到底选择哪种日志框架合适呢？

现在最流的日志框架解决方案莫过于`SLF4J + LogBack`。原因有下面几点：

- `LogBack` 自身实现了 `SLF4J` 的日志接口，不需要 `SLF4J` 去做进一步的适配。
- `LogBack` 自身是在 `Log4J` 的基础上优化而成的，其运行速度和效率都比 `LOG4J` 高。
- `SLF4J` + `LogBack` 支持占位符，方便日志代码的阅读，而 `LOG4J` 则不支持。

从上面几点来看，**SLF4J + LogBack**是一个较好的选择。



LogBack 被分为3个组件：*`logback-core`、`logback-classic `和 `logback-access`*

- **logback-core** ：提供了` LogBack `的核心功能，是另外两个组件的基础。

- **logback-classic** ：则实现了 `SLF4J` 的`API`，所以当想配合 `SLF4J` 使用时，需要将 `logback-classic` 引入依赖中。

- **logback-access** ：是为了集成Servlet环境而准备的，可提供`HTTP-access`的日志接口。

  

&emsp;&emsp;`LogBack`的日志记录数据流是从 `Class（Package）`到 `Logger`，再从`Logger`到`Appender`，最后从`Appender`到具体的输出终端。

![img](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20211101204654.png)

&emsp;&emsp;`LogBack`配置文件可以分为几个节点，其中 `Configuration` 是根节点，`Appender`、`Logger`、`Root`是`Configuration`的子节点。

### 4.1 appender节点

&emsp;&emsp;负责写日志的组件。`appender`有两个必要属性`name`、`class` 。

- **name指定：**`appender`的名称。
- **class指定：**`appender`的全限定名`class`，主要包括：
  - `ch.qos.logback.core.ConsoleAppender`： 控制台输出
  - `ch.qos.logback.core.FileAppender`：文件输出
  - `ch.qos.logback.core.RollingFileAppender`： 文件滚动输出

```xml
<?xml version="1.0" encoding="utf-8"?> 
<configuration debug="true" scan="true" scanPeriod="2">
    <!-- conf consoel out -->
    <appender name ="console_out" class="ch.qos.logback.core.ConsoleAppender">
    </appender>

    <!-- conf file out -->
    <appender name="file_out" class="ch.qos.logback.core.FileAppender">
    </appender>
    
    <!-- conf file out -->
    <appender name="file_out" class="ch.qos.logback.core.RollingFileAppender">
    </appender>

    <root></root>
    <logger></logger>
</configuration>
```



#### 4.1.1 ConsoleAppender

把日志添加到控制台，有如下节点：

- `encoder` : 对日志进行格式化。
- `` : 字符串System.out 或者 System.err, 默认 System.out;

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <!-- conf consoel out -->
  <appender name ="console_out" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%date [%thread] %-5level %logger - %message%newline</pattern>
        </encoder>
  </appender>

  <root level="INFO">             
    <appender-ref ref="console_out" />   
  </root>     
</configuration>
```



#### 4.1.2 FileAppender

把日志添加到文件，有如下节点：

- `file`：被写入的文件名,可以是相对目录 , 也可以是绝对目录 , 如果目录不存在则会自动创建。
- ``：如果是true , 日志被追加到文件结尾 , 如果是false,清空现存文件 , 默认是true。
- `pattern`：对日志进行格式化 [具体的转换符说明请参见官网.]

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <appender name="file_out" class="ch.qos.logback.core.FileAppender">
        <file>logs/debug.log</file>
        <encoder>
            <pattern>%date [%thread] %-5level %logger - %message%newline</pattern>
        </encoder>
    </appender>
</configuration>
```



#### 4.1.3 rollingFileAppender

滚动纪录文件，先将日志记录到指定文件，当符合某种条件时，将日志记录到其他文件，有如下节点：

- ``：被写入的文件名，可以是相对目录，也可以解决目录，如果目录不存在则自动创建。
- ``：如果是true，日志被追加到文件结尾，如果是false，清空现存文件，默认是true。
- ``：对日志进行格式化。
- ``：当发生滚动时，决定 RollingFileAppender 的行为，涉及文件移动和重命名。

##### 4.1.3.1 rollingPolicy

- `TimeBaseRollingPolicy `：最常用的滚动策略，根据时间来制定滚动策略，即负责滚动也负责触发滚动。有如下节点；
  - `fileNamePattern`：必要节点，包含文件及`“%d”` 转换符，`“%d”`可以包含一个`java.text.SimpleDateFormat` 制定的时间格式，如：`%d{yyyy-MM}`,如果直接使用`%d` ，默认格式是 `yyyy－MM－dd`。
  - `minIndex/maxIndex`：可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件，假设设置每个月滚动，且 是 6，则只保存最近6个月的文件，删除之前的旧文件，注意：删除旧文件是哪些为了归档而创建的目录也会被删除。
  - `fileNamePattern`：必须包含`“%i” `例如：设置最小值，和最大值分别为1和2，命名模式为 `log%i.log`,会产生归档文件`log1.log`和`log2.log`，还可以指定文件压缩选项，例如：`log％i.log.gz` 或者 `log%i.log.zip`
- `triggeringPolicy`：告知`RollingFileAppender`，激活`RollingFileAppender`滚动。

```xml
<!-- 03:conf errorAppender out -->
<appender name="errorAppender" class="ch.qos.logback.core.RollingFileAppender">
    <file>logs/error.log</file>
    <!-- 设置滚动策略 -->
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
        <!--设置日志命名模式--> 
        <fileNamePattern>errorFile.%d{yyyy-MM-dd}.log</fileNamePattern>
        <!--最多保留30天log-->
        <maxHistory>30</maxHistory>
    </rollingPolicy>
    <!-- 超过150MB时，触发滚动策略 -->
    <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
        <maxFileSize>150</maxFileSize>
    </triggeringPolicy>
    <encoder>
        <pattern>%d [%p] %-5level %logger - %msg%newline</pattern>
    </encoder>
</appender>
```



### 4.2 logger节点

&emsp;&emsp;`logger`的子节点，来设置某一个包或者具体的某一个类的日志打印级别，以及指定。`logger`仅有一个`name`属性，两个可选属性 `level／addtivity`。

- `name`：用来指定受此loger约束的某一个包或者具体的某一个类。
- `level`：用来设置打印级别，大小写无关。可选值有`TRACE`、`DEBUG`、`INFO`、`WARN`、`ERROR`、`ALL`和`OFF`。还有一个特俗值`INHERITED` 或者 同义词`NULL`，代表强制执行上级的级别。如果未设置此属性，那么**当前`logger`将会继承上级的级别。**
- `addtivity`：是否向上级`logger`传递打印信息，默认为`true`；

可以包含零个或多个元素，表示这个`appender`将会添加到`logger`。

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <!-- conf consoel out -->
    <appender name ="console_out" class="ch.qos.logback.core.ConsoleAppender">
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 过滤掉非INFO级别 -->
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!--  conf infoAppender out -->
    <appender name="infoAppender" class="ch.qos.logback.core.RollingFileAppender">
        <file>logs/info.log</file>
        <!-- 设置滚动策略 -->
        <rollingPoliy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
            <!--设置日志命名模式--> 
            <fileNamePattern>infoFile.%d{yyyy-MM-dd}.log</fileNamePattern>
            <!--最多保留30天log-->
            <maxHistory>30</maxHistory>
        </rollingPoliy>
        <!-- 超过150MB时，触发滚动策略 -->
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <maxFileSize>150</maxFileSize>
        </triggeringPolicy>
        <encoder>
            <pattern>%d [%p] %-5level %logger - %msg%newline</pattern>
        </encoder>
    </appender>
 
    <!-- 添加两个appender节点 -->
    <logger name="logback.olf.log" level="info">
        <appender-ref ref = "console_out"/>
        <appender-ref ref = "infoAppender"/>
    </logger>
</configuration>
```

### 4.3 root节点

&emsp;&emsp;元素配置根`logger`。该元素有一个`level`属性，没有`name`属性，因为已经被命名为`root`。`Level`属性的值大小写无关，其值为下面其中一个字符串：`TRACE`、`DEBUG`、`INFO`、 `WARN`、`ERROR`、`ALL` 和 `OFF`。如果 `root` 元素没有引用任何 `appender`，就会失去所有 `appender`。

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <!-- conf consoel out -->
    <appender name ="console_out" class="ch.qos.logback.core.ConsoleAppender">
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 过滤掉非INFO级别 -->
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 01:conf infoAppender out -->
    <appender name="infoAppender" class="ch.qos.logback.core.RollingFileAppender">
        
        <file>logs/info.log</file>
        <!-- 设置滚动策略 -->
        <rollingPoliy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
            <!--设置日志命名模式--> 
            <fileNamePattern>infoFile.%d{yyyy-MM-dd}.log</fileNamePattern>
            <!--最多保留30天log-->
            <maxHistory>30</maxHistory>
        </rollingPoliy>
        <!-- 超过150MB时，触发滚动策略 -->
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <maxFileSize>150</maxFileSize>
        </triggeringPolicy>
        <encoder>
            <pattern>%d [%p] %-5level %logger - %msg%newline</pattern>
        </encoder>
    </appender>

    <!-- 02:conf debugAppender out -->
    <appender name="debugAppender" class="ch.qos.logback.core.RollingFileAppender">
        <file>logs/debug.log</file>
        <!-- 设置滚动策略 -->
        <rollingPoliy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
            <!--设置日志命名模式--> 
            <fileNamePattern>debugFile.%d{yyyy-MM-dd}.log</fileNamePattern>
            <!--最多保留30天log-->
            <maxHistory>30</maxHistory>
        </rollingPoliy>
        <!-- 超过150MB时，触发滚动策略 -->
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <maxFileSize>150</maxFileSize>
        </triggeringPolicy>
        <encoder>
            <pattern>%d [%p] %-5level %logger - %msg%newline</pattern>
        </encoder>
    </appender>

    <!-- 03:conf errorAppender out -->
    <appender name="errorAppender" class="ch.qos.logback.core.RollingFileAppender">
        <file>logs/error.log</file>
        <!-- 设置滚动策略 -->
        <rollingPoliy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
            <!--设置日志命名模式--> 
            <fileNamePattern>errorFile.%d{yyyy-MM-dd}.log</fileNamePattern>
            <!--最多保留30天log-->
            <maxHistory>30</maxHistory>
        </rollingPoliy>
        <!-- 超过150MB时，触发滚动策略 -->
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <maxFileSize>150</maxFileSize>
        </triggeringPolicy>
        <encoder>
            <pattern>%d [%p] %-5level %logger - %msg%newline</pattern>
        </encoder>
    </appender>
 
    <root level="ALL">
        <appender-ref ref="infoAppender"/>
        <appender-ref ref="debugAppender"/>
        <appender-ref ref="errorAppender"/>
    </root>
</configuration>
```



### 4.4 filter过滤节点

#### 4.4.1 级别过滤器（LevelFilter）

&emsp;&emsp;`LevelFilter` 根据记录级别对记录事件进行过滤。如果事件的级别等于配置的级别，过滤 器会根据 `onMatch` 和 `onMismatch` 属性接受或拒绝事件。下面是个配置文件例子:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <!-- conf consoel out -->
    <appender name ="console_out" class="ch.qos.logback.core.ConsoleAppender">
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 过滤掉非INFO级别 -->
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        
        <encoder>
            <pattern>%-4relative [%thread] %-5level %logger{30} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="DEBUG">
        <appender-ref ref="console_out" />
    </root>
</configuration>
```

#### 4.4.2 临界值过滤器（ThresholdFilter）

&emsp;&emsp;`ThresholdFilter`过滤掉低于指定临界值的事件。

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <!-- conf consoel out -->
    <appender name ="console_out" class="ch.qos.logback.core.ConsoleAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">  
　　　　　　　　<!-- 过滤掉TRACE和DEBUG级别的日志 -->
            <level>INFO</level> 
        </filter>
        
        <encoder>
            <pattern>%-4relative [%thread] %-5level %logger{30} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="DEBUG">
        <appender-ref ref="console_out" />
    </root>
</configuration>
```

#### 4.4.3 求值过滤器（EvaluatorFilter）

&emsp;&emsp;评估是否符合指定的条件

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <!-- conf consoel out -->
    <appender name ="console_out" class="ch.qos.logback.core.ConsoleAppender">
        <filter class="ch.qos.logback.classic.filter.EvaluatorFilter">  
             <evaluator>
             <!--过滤掉所有日志中不包含hello字符的日志-->
                <expression>
                    message.contains("hello")
                </expression>
                <onMatch>NEUTRAL</onMatch>
                <onMismatch>DENY</onMismatch>
             </evaluator>
        </filter>
        
        <encoder>
            <pattern>%-4relative [%thread] %-5level %logger{30} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="DEBUG">
        <appender-ref ref="console_out" />
    </root>
</configuration>
```

#### 4.4.4 匹配器（Matchers）

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <!-- conf consoel out -->
    <appender name ="console_out" class="ch.qos.logback.core.ConsoleAppender">
        <filter class="ch.qos.logback.classic.filter.EvaluatorFilter">  
             <evaluator> 
                <matcher>
                    <Name>odd</Name>
                    <!-- 过滤掉序号为奇数的语句-->
                    <regex>statement [13579]</regex>
                </matcher>
                <expression>odd.matches(formattedMessage)</expression>
                <onMatch>NEUTRAL</onMatch>
                <onMismatch>DENY</onMismatch>
             </evaluator>
        </filter>
        
        <encoder>
            <pattern>%-4relative [%thread] %-5level %logger{30} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="DEBUG">
        <appender-ref ref="console_out" />
    </root>
</configuration>
```



下面是一个常用的`logback.xml`配置文件，供大家参考：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<configuration debug="true" scan="true" scanPeriod="30 seconds"> 

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender"> 
    <!-- encoders are  by default assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder -->
    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss} [%level] - %m%n</pattern>
        
        <!-- 常用的Pattern变量,大家可打开该pattern进行输出观察 -->
        <!-- 
          <pattern>
              %d{yyyy-MM-dd HH:mm:ss} [%level] - %msg%n
              Logger: %logger
              Class: %class
              File: %file
              Caller: %caller
              Line: %line
              Message: %m
              Method: %M
              Relative: %relative
              Thread: %thread
              Exception: %ex
              xException: %xEx
              nopException: %nopex
              rException: %rEx
              Marker: %marker
              %n
              
          </pattern>
           -->
    </encoder>
  </appender>
  
  <!-- 按日期区分的滚动日志 -->
  <appender name="ERROR-OUT" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <file>logs/error.log</file>
      
    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss} [%class:%line] - %m%n</pattern>
    </encoder>
      
      <filter class="ch.qos.logback.classic.filter.LevelFilter">
      <level>ERROR</level>
      <onMatch>ACCEPT</onMatch>
      <onMismatch>DENY</onMismatch>
    </filter>
      
      <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!-- daily rollover -->
      <fileNamePattern>error.%d{yyyy-MM-dd}.log.zip</fileNamePattern>

      <!-- keep 30 days' worth of history -->
      <maxHistory>30</maxHistory>
    </rollingPolicy>
  </appender>
  
  <!-- 按文件大小区分的滚动日志 -->
  <appender name="INFO-OUT" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <file>logs/info.log</file>
      
    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss} [%class:%line] - %m%n</pattern>
    </encoder>
    
      <filter class="ch.qos.logback.classic.filter.LevelFilter">
      <level>INFO</level>
      <onMatch>ACCEPT</onMatch>
      <onMismatch>DENY</onMismatch>
    </filter>
      
      <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
      <fileNamePattern>info.%i.log</fileNamePattern>
      <minIndex>1</minIndex>
      <maxIndex>3</maxIndex>
    </rollingPolicy>
    
    <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
      <maxFileSize>5MB</maxFileSize>
    </triggeringPolicy>
    
  </appender>
  
  
  <!-- 按日期和大小区分的滚动日志 -->
  <appender name="DEBUG-OUT" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <file>logs/debug.log</file>

    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss} [%class:%line] - %m%n</pattern>
    </encoder>
      
      <filter class="ch.qos.logback.classic.filter.LevelFilter">
      <level>DEBUG</level>
      <onMatch>ACCEPT</onMatch>
      <onMismatch>DENY</onMismatch>
    </filter>
      
      <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!-- rollover daily -->
      <fileNamePattern>debug-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
      
      <timeBasedFileNamingAndTriggeringPolicy
            class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
        <!-- or whenever the file size reaches 100MB -->
        <maxFileSize>100MB</maxFileSize>
      </timeBasedFileNamingAndTriggeringPolicy>
      
    </rollingPolicy>
    
  </appender>
  
  
   <!-- 级别阀值过滤 -->
  <appender name="SUM-OUT" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <file>logs/sum.log</file>

    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss} [%class:%line] - %m%n</pattern>
    </encoder>
      
    <!-- deny all events with a level below INFO, that is TRACE and DEBUG -->
    <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
      <level>INFO</level>
    </filter>

      
      <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!-- rollover daily -->
      <fileNamePattern>debug-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
      
      <timeBasedFileNamingAndTriggeringPolicy
            class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
        <!-- or whenever the file size reaches 100MB -->
        <maxFileSize>100MB</maxFileSize>
      </timeBasedFileNamingAndTriggeringPolicy>
      
    </rollingPolicy>
    
  </appender>
  
  
  <root level="debug">
    <appender-ref ref="STDOUT" />
    <appender-ref ref="ERROR-OUT" />
    <appender-ref ref="INFO-OUT" />
    <appender-ref ref="DEBUG-OUT" />
    <appender-ref ref="SUM-OUT" />
  </root>
</configuration>
```



## 五、如何进行日志系统转换？

&emsp;&emsp;在实际的日志转换过程中，`SLF4J`其实是充当了一个中介的角色。例如当我们一个项目原来是使用`LOG4J`进行日志记录，但是我们要换成`LogBack`进行日志记录。

&emsp;&emsp;此时我们需要先将`LOG4J`转换成`SLF4J`日志系统，再从`SLF4J`日志系统转成`LogBack`日志系统。

## 六、推荐的日志编写方式

&emsp;&emsp;以下是错误方式。因为系统中如果存在大量类似的代码，同时系统只输出 `info` 及 `info` 以上级别的日志，那么，在输出 `debug` 日志时会**产生大量的字符串**，而并不会输出 `debug` 日志，最后造成字符串不停地拼接，浪费系统性能。此时，`SLF4J` 就可以使用占位符的功能编写日志，通过这样的形式，`SLF4J` 就可以根据日志等级判断，只对符合要求的日志进行数据拼接和打印。

```java
//错误方式
logger.debug("用户" + userId + "开始下单:" + orderNo + ",请求信息:" + Gson.toJson(req));
//推荐方式
logger.debug("用户{}开始下单:{},请求信息:", userId, orderNo,  Gson.toJson(req));
```

&emsp;&emsp;有些时候日志输出需要进行数值计算，或者 `JSON` 转换，此时就需要一定的计算任务。但方法调用如果被当作参数传递的话一样会被执行，所以 `Java8` 中 `SLF4J` 还可以通过 `Supplier` 来传递。如下所示：

```java
logger.debug("用户{}开始下单:{},请求信息:", userId, orderNo, () -> Gson.toJson(req))
```



**日志编写推荐2个方向：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201105161319594.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dvbmdoYWl5dQ==,size_16,color_FFFFFF,t_70#pic_center)

### 6.1 日志编写位置

1. 系统/应用**启动和参数变更**。当系统启动时，可以将相关的参数信息进行打印，以便出现问题时，更准确地查询原因；参数信息可能并不存储在本地，需要通过配置中心获取，而参数信息有变更时，也需要将变更后的内容输出在日志中。
2. **关键操作节点**。最典型的就是在关键位置添加日志，记录用户进行的某个操作。
3. **大型任务进度上报**。当系统在处理某个比较大型的任务时，可以在这个过程中增加相关的日志来**表明任务处理的进度**，防止因为长时间没有处理而无法得知程序执行的状态，比如在文件下载时，可以按照百分比来定时/定次地上报数据。
4. **异常。如果是通过 try-catch 处理，你需要注意日志编写的位置**。如果你需要将日志在本层抛出，则不需要进行日志记录，否则会出现日志重复的问题。如果你**除了异常以外还需要记录其他的内容，则可以通过定制异常信息来实现**。



### 6.2 写入性能

1. **日志编写位置：**日志编写的位置在程序中十分重要，**如果在 `for` 循环中编写，因为这个循环会持续很多次**，那么就会产生大量的日志记录。此时可以考虑一下，这个日志的记录是否有必要。
2. **日志数量：**如果你**大量地编写日志，那么日志的质量一定会降低。**
3. **日志编写等级：**我在上一节中讲过，日志等级很容易被滥用，不正确的日志等级**会导致我们查询问题的时间增加**。
4. **日志输出级别：**这里指的是对于配置日志输出级别的选择。在线上环境，**不建议使用 `debug` 级别**，因为线上一直有请求，`debug` 级别会输出大量的基础和请求信息，极其浪费资源，因此**建议开启 `info` 或者以上**。
5. **用输出参数：** **不对大字段、无用字段输出，因为这很影响程序执行效率和日志的内容**。比如大段的`Html`，或者大段的对象信息，甚至有见到整个图片流信息。



### 6.3 占位符

你为什么要用占位符：

1. **节约性能**：在生成较高级别的日志时，低级别的日志会不停叠加字符串而占用过多的`内存`、`CPU 资源`，导致性能浪费。
2. **便于编写**：先确认日志所想要表达的内容，再确认你所需要编写的参数，这样在写日志时，目的也会更加明确。
3. **便于查看**：在代码 `review` 时，更方便查看日志想表达的意思，而不会被日志的参数打乱。



### 6.4 可读性

我总结了几点在日志中容易遗漏的信息：

1. **会话标识：**当前操作的用户和与当前请求相关的信息，类似 `Session`。当出现问题/查看行为时，可以**根据这个值来快速识别到相关的日志。**
2. **请求标识：** **每个请求都拥有一个唯一的标识，这样在查看问题时，我们只需要查看这一个请求中的所有日志即可**。一般我们会配合链路追踪系统一同使用，因为后者可以实现跨应用的日志追踪，从而帮助我们过滤掉不相关的信息。例如`MDC`。
3. **参数信息：**在日志中增加参数信息能帮你了解到，是什么情况下产生的问题，这样你也很容易复现问题，以及辨别错误的原因。
4. **发生数据的结果：**和**参数信息相互对应，一个是执行前，一个是执行后**。发生数据的结果可以帮你了解程序执行的结果，出错时也是很重要的参考条件。



### 6.5 关键信息隐蔽

&emsp;&emsp;**对于关键的信息不显示或者进行掩码显示**，以免信息被盗取后出现数据内容泄漏。**推特**在 `2018` 年曾将用户的密码打印在日志中，这一行为泄露了 `3.3` 亿人的密码。



### 6.6 减少代码位置信息的输出

&emsp;&emsp;如果**不是必要，尽量不要在日志格式中输出当前日志所在的代码行和方法名称信息**。因为这是通过获取当前线程堆栈快照信息来进行实现的，这种实现方式**会极大地影响程序执行的效率**。
&emsp;&emsp;在 `log4j` 的文档中有这样一段话：“使用同步方式进行获取位置信息会慢 `1.3` 到 `5` 倍，如果是使用异步日志，因为会涉及跨线程获取位置信息，会慢 `30` 到 `100` 倍。



### 6.7 文件分类

&emsp;&emsp;将不同的业务逻辑按照不同的日志文件来分类。这个在微服务中不存在这个问题，但是针对同一个微服务，也**需要将正常日志和错误日志，业务日志和非业务日志区分开来（非必须）。**



### 6.8 日志 review

&emsp;&emsp;好的日志不是一次就能写好的，而是**通过代码评审、不断迭代等方式进行完善的**



### 6.9 日志格式

&emsp;&emsp;日志的格式布局会影响运维人员将这些日志内容收集与管理的效率。如果编写者和管理者能够通过协商，规定出一套完整的日志格式，这样就能在排查问题时事半功倍。

&emsp;&emsp;介绍几点在**日志编写时需要注意的事项**：

1. **系统之间格式保持一致：**每个应用在日志**格式上尽量保持统一**，这样，运维人员在进行日志收集时，就可以采用统一的日志模板来收集和格式化内容，减少双方的沟通成本。
2. **不编写多行的日志内容：**除了异常堆栈信息，不对日志内容进行多行的输出，**多行的内容十分不便于数据解析**，可能会出现生成多条日志的情况。
3. **不要使用日志中的常见内容来分割：**如果采用日志中常见的内容来分割，会在格式解析时出现问题，比如用户 `ID` 中的空格就是比较常见的内容。



### 6.10 日志归档

&emsp;&emsp;日志归档是一件很重要的事情。一般情况下，我们会**对日志按照日期来归档**，每天生成一个日志文件，这样在日志备份和恢复时，可以按照日期来进行。如果感觉天级别的日志仍然太大了，可以考虑按照小时细分。