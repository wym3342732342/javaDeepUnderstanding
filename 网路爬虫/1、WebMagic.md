# 网络爬虫框架Webmagic

## 一、浅谈网络爬虫

### 1.1 什么是网络爬虫

&emsp;&emsp;在大数据时代，信息采集是一项重要的工作，互联网中的数据是海量的，如果单纯靠人力进行信息采集，搜集的成本很高。而爬虫技术就是为了解决这些问题而生，自动高效地获取互联网中感兴趣的信息。

&emsp;&emsp;网络爬虫（web crawler）也叫做网络机械人，可以代替人们自动的在互联网进行数据采集和整理。它是一种按照一定的规则，自动地抓取万维网信息的程序或者脚本，可以自动采集所有其能够访问到的页面内容，以获取或更新这些网站的内容和检索方式。



### 1.2 网络爬虫可以做什么

- 可以实现搜索引擎；
- 大数据时代，可以让我们获取更多的数据源；
- 凯苏填充测试和运营数据；
- 为人工智能提供训练数据集；



## 二、网络爬虫常用技术（Java）

### 2.1 底层实现 HttpClient + Jsoup

&emsp;&emsp;`HttpClient`是`Apache Jakarta Common`下的子项目，用来提供高效的、最新的、功能丰富的支持`HTTP`协议的客户端编程工具包，并且它支持`Http`协议最新的版本和协议。`HttpClient`已经应用在很多的项目中，比如`Apache Jakarta`上很著名的另外两个开源项目`Cactus`和`HTMLUnit`都使用了`HttpClient`。[更多信息请关注](http://hc.apache.org)

&emsp;&emsp;`jsoup`是一款`Java`的`HTML`解析器，可直接解析某个`URL`地址、`HTML`文本内容。它提供了一套非常省力的`API`，可通过`DOM`,`CSS`以及类似于`JQuery`的操作方法来取出和操作数据。



### 2.2 开源框架WebMagic

&emsp;&emsp;`webmagic`是一个开源的`Java`爬虫框架，目标是简化爬虫的开发流程，让开发者专注于逻辑功能的开发。`webmagic`的核心非常简单，但是**覆盖爬虫的整个流程**，也是很好的学习爬虫的材料。

![image-20200326130238511](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200326130244.png)

#### 2.2.1 Web Magic的主要特色

- 完成模块化的设计，强大的可扩展性；
- 核心简单但是涵盖爬虫的全部流程，灵活而强大；
- 提供丰富的抽取页面API；
- 无配置，但是**可以通过POJO + 注解形式实现一个爬虫；**
- 支持多线程；
- 支持分布式；
- 支持爬取js动态渲染的页面；
- 无框架依赖，可以灵活的嵌入到项目中去；



## 三、爬虫框架Web Magic

### 3.1 架构解析

&emsp;&emsp;WebMagic项目代码分为核心和扩展两部分。核心部分(`webmagic-core`)是一个精简的、模块化的爬虫实现，而扩展部分则包括一些便利的、实用性的功能。扩展部门(`webmagic-extension`)提供了一些便捷的功能，例如注解模式编写爬虫等。同时内置了一些常用的组件，便于爬虫开发。

&emsp;&emsp;WebMagic的设计目标是尽量的模块化，并体现爬虫的功能特点。这部分提供非常简 单、灵活的API，在基本不改变开发模式的情况下，编写一个爬虫。

&emsp;**&emsp;WebMagic的结构分为Downloader、PageProcessor、Scheduler、Pipeline四大组件**，并由Spider将它们彼此组织起来。这四大组件对应爬虫生命周期中的下载、处理、管 理和持久化等功能。而**Spider则将这几个组件组织起来，让它们可以互相交互，流程化的执行**，可以认为Spider是一个大的容器，它也是WebMagic逻辑的核心。



### 3.2 组件分析

![image-20200326130941900](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200326130941.png)



**四大组件：**

- `Downloader`：Downloader负责从互联网上**下载页面**，以便后续处理。**WebMagic默认使用了ApacheHttpClient作为下载工具。**
- **`PageProcessor`：**PageProcessor负责解析页面，抽取有用信息，以及发现新的链接。**WebMagic使用Jsoup作为HTML解析工具，并基于其开发了解析XPath的工具Xsoup。【PageProcessor对于每个站点每个页面都不一样，是需要使用者定制的部分】**
- **`Scheduler`**：**Scheduler负责管理待抓取的URL，以及一些去重的工作。WebMagic默认提供了JDK的内存队列来管理URL，并用集合来进行去重。也支持使用Redis进行分布式管理。**
- `Pipeline`：Pipeline负责抽取结果的处理，包括**计算、持久化到文件、数据库**等。WebMagic默认提供 了**“输出到控制台”和“保存到文件”**两种结果处理方案。



### 3.3 API说明

#### 3.3.1 Spider的API说明

&emsp;&emsp;同时`Spider`的其他组件都可以通过set方法来进行设置。

| 方法                        | 说明                                           |
| --------------------------- | ---------------------------------------------- |
| `create(PageProcessor)`     | 创建Spider                                     |
| `addUrl(String...)`         | 添加初始的URL                                  |
| `thread(n)`                 | 开启n个线程                                    |
| `run()`                     | 启动，会阻塞当前线程执行                       |
| `start()/runAsync()`        | 异步启动，当前线程继续执行                     |
| `stop()`                    | 停止爬虫                                       |
| `addPipeline(Pipeline)`     | 添加一个Pipeline，一个Spider可以有多个Pipeline |
| `setScheduler(Scheduler)`   | 设置Scheduler，一个Spider只能有一个Scheduler   |
| `setDownloader(Downloader)` | 设置Download，一个Spider只能有一个Downloader   |
| `get(String)`               | 同步调用，并直接取得结果                       |
| `getAll(String...)`         | 同步调用，并直接取得一堆结果                   |

#### 3.3.2 Site的API说明

&emsp;&emsp;Site用于定义站点本身的一些配置信息，例如编码、HTTP头、超时时间、重试策略 等、代理等，都可以通过设置Site对象来进行配置。

| 方法                       | 说明                                      |
| -------------------------- | ----------------------------------------- |
| `setCharset(String)`       | 设置编码                                  |
| `setUserAgent(String)`     | 设置UserAgent                             |
| `setTimeOut(int)`          | 设置超时时间【毫秒】                      |
| `setRetryTimes(int)`       | 设置重试次数                              |
| `setCycleRetryTimes(int)`  | 设置重试间隔                              |
| `addCookie(String,String)` | 添加一条cookie                            |
| `setDomain(String)`        | 设置域名，需设置域名后，addCookie才可生效 |
| `addHeader(String,String)` | 添加一条请求头                            |
| `setHttpProxy(HttpHost)`   | 设置Http代理                              |



### 3.4 案例：爬取CSDN中的博客--Java的内容

#### 3.4.1 引入依赖

```xml
<!-- 网络爬爬 -->
<dependency>
  <groupId>us.codecraft</groupId>
  <artifactId>webmagic-core</artifactId>
  <version>0.7.3</version>
</dependency>
<dependency>
  <groupId>us.codecraft</groupId>
  <artifactId>webmagic-extension</artifactId>
  <version>0.7.3</version>
</dependency>
```

#### 3.4.2 实现简单的爬取案例

&emsp;&emsp;`Spider`是爬虫的入口，需要为它提供一个`PageProcessor`，然后在启动爬虫。此时打印了整个网页。

```kotlin
class MyProcessor : PageProcessor {
    /**
     * 处理方法
     * @param page
     */
    override fun process(page: Page) {
        println(page.html.toString())
    }

    /**
     * 爬取配置
     */
    override fun getSite(): Site {
        return Site
            .me()//创建新的site
            .setSleepTime(100)//设定两次处理间隔
            .setRetryTimes(3)//设置重试时间
    }

    companion object{
        @JvmStatic
        fun main(args: Array<String>) {
            Spider
                .create(MyProcessor())
                .addUrl("https://blog.csdn.net")
                .run()
        }
    }
}
```

- `Page`：代表了从**Downloader**下载到的一个页面——可能是HTML，也可能有JSON或者其他文本格式的内容。Page是WebMagic抽取过程的核心对象，它提供一些方法可供抽 取、结果保存等。



#### 3.4.3 爬取指定内容（XPath）

&emsp;&emsp;`XPath`，即为`XML`路径语言`(XMLPathLanguage)`，它是一种用来确定XML文档中某部分位置的语言。XPath使用路径表达式来选取 XML 文档中的节点或者节点集。这些路径 表达式和我们在常规的电脑文件系统中看到的表达式非常相似。**【语法可见下方】**

**更改处理方法：**

```kotlin
override fun process(page: Page) {
  println(
    page.html.xpath(
      """
                //*[@id="nav"]/div/div/ul/li[5]/a
            """.trimIndent()
    ).toString()
  )
}
```

**输出结果：**

```html
<a href="/nav/java">Java</a>
```



#### 3.4.4 添加目标地址

&emsp;&emsp;可以提取当前页面的所有地址，作为种子地址再爬取到更多的页面。

```kotlin
override fun process(page: Page) {
    //当前页面的所有链接添加到目标页面中
    page.addTargetRequests(page.html.links().all())
    println(
        page.html.xpath(
        """
            //*[@id="nav"]/div/div/ul/li[5]/a
        """.trimIndent()
        ).toString()
    )
}
```



### 3.5 案例：目标地址正则匹配——爬取文章详细页内容，并提取标题

```kotlin
class MyProcessor : PageProcessor {
    /**
     * 处理方法
     * @param page
     */
    override fun process(page: Page) {
        //当前页面的所有链接添加到目标页面中
        page.addTargetRequests(page.html.links().regex(
            "https://blog.csdn.net/[a-z 0-9 _]+/article/details/[0-9]{9}"
        ).all())
        println(
            page.html.xpath(
            """
                //*[@id="mainBox"]/main/div[1]/div[1]/div[1]/div[1]/h1/text()
            """.trimIndent()
            ).toString()
        )
    }
    /**
     * 爬取配置
     */
    override fun getSite(): Site {
        return Site
            .me()//创建新的site
            .setSleepTime(100)//设定两次处理间隔
            .setRetryTimes(3)//设置重试时间
    }
    companion object{
        @JvmStatic
        fun main(args: Array<String>) {
            Spider
                .create(MyProcessor())
                .addUrl("https://blog.csdn.net/nav/ai")
                .run()
        }
    }
}
```



### 3.6 Pipeline案例

#### 3.6.1 ConsolePipeline控制台输出

```kotlin
class MyProcessor : PageProcessor {
    /**
     * 处理方法
     * @param page
     */
    override fun process(page: Page) {
      
        page.addTargetRequests(page.html.links().regex(
            "https://blog.csdn.net/[a-z 0-9 _]+/article/details/[0-9]{9}"
        ).all())
        //存入字段
        page.putField("title",
            page.html.xpath(
            """
                //*[@id="mainBox"]/main/div[1]/div[1]/div[1]/div[1]/h1/text()
            """.trimIndent()
            ).toString()
        )
    }

    /**
     * 爬取配置
     */
    override fun getSite(): Site {
        return Site
            .me()//创建新的site
            .setSleepTime(100)//设定两次处理间隔
            .setRetryTimes(3)//设置重试时间
    }

    companion object{
        @JvmStatic
        fun main(args: Array<String>) {
            Spider
                .create(MyProcessor())
                .addUrl("https://blog.csdn.net/nav/ai")
                .addPipeline(ConsolePipeline())
                .run()
        }
    }
}
```



#### 3.6.2 FilePipeline文件保存

```kotlin
//此种方式会保存为html形式
companion object{
    @JvmStatic
    fun main(args: Array<String>) {
        Spider
            .create(MyProcessor())
            .addUrl("https://blog.csdn.net/nav/ai")
            .addPipeline(FilePipeline("/Users/wangyiming/aaa"))//保存到此文件夹
            .run()
    }
}
```



#### 3.6.3 JsonFilePipeline

```kotlin
companion object{
  @JvmStatic
  fun main(args: Array<String>) {
    Spider
    .create(MyProcessor())
    .addUrl("https://blog.csdn.net/nav/ai")
    .addPipeline(JsonFilePipeline("/Users/wangyiming/aaa"))
    .run()
  }
}
```



#### 3.6.4 定制Pipeline

&emsp;&emsp;可以通过定制与其他框架结合，存取到数据库等位置。

1. 创建`MyPipeline`实现接口`Pipeline`：

   ```kotlin
   class MyPipeline : Pipeline {
       override fun process(resultItems: ResultItems?, task: Task?) {
           val title: String = resultItems?.get<String>("title")?:return
           println("标题 ：${title}")
       }
   }
   ```

2. 修改`main`方法：

   ```kotlin
   companion object{
       @JvmStatic
       fun main(args: Array<String>) {
           Spider
               .create(MyProcessor())
               .addUrl("https://blog.csdn.net/nav/ai")
               .addPipeline(MyPipeline())
               .run()
       }
   }
   ```

   

### 3.7 Scheduler案例

&emsp;&emsp;以上的案例每次运行可能会爬取重复的页面，这样做是没有任何意义的。此时可以使用`Scheduler(URL管理)`，其最基本的功能是实现对已经爬取`URL`进行标示。可实现`URL`增量去重。

**Scheduler的三种实现方式：**

- **内存队列：**`QueueScheduler`
- **文件队列：**`FileCacheQueueScheduler`
- **Redis队列：**`RedisScheduler`



#### 3.7.1 内存队列

```kotlin
companion object{
    @JvmStatic
    fun main(args: Array<String>) {
        Spider
            .create(MyProcessor())
            .addUrl("https://blog.csdn.net/nav/ai")
            .setScheduler(QueueScheduler())
            .run()
    }
}
```

#### 3.7.2 文件队列

&emsp;&emsp;使用文件保存抓取`URL`，可以在关闭程序并下次启动时，**从之前抓取的URL继续抓取。**

1. 创建文件夹`/Users/wangyiming/scheduler`

2. 修改代码

   ```kotlin
   companion object{
       @JvmStatic
       fun main(args: Array<String>) {
           Spider
               .create(MyProcessor())
               .addUrl("https://blog.csdn.net/nav/ai")
               .setScheduler(FileCacheQueueScheduler("/Users/wangyiming/scheduler"))//设置文件队列
               .run()
       }
   }
   ```

3. 效果【文件夹下生成2个文件】

   - `blog.csdn.net.urls.txt`
   - `blog.csdn.net.cursor.txt`

#### 3.7.3 Redis队列

```kotlin
companion object{
    @JvmStatic
    fun main(args: Array<String>) {
        Spider
            .create(MyProcessor())
            .addUrl("https://blog.csdn.net/nav/ai")
            .setScheduler(RedisScheduler("127.0.0.1"))//设置Redis队列
            .run()
    }
}
```

**此时出现两个集合：**

- `set_blog.csdn.net`
-  `queue_blog.csdn.net`



## 四、案例：结合boot项目

### 4.1 引入依赖[结合jpa]

```xml
<!-- 网络爬爬 -->
<dependency>
    <groupId>us.codecraft</groupId>
    <artifactId>webmagic-core</artifactId>
    <version>0.7.3</version>
    <exclusions>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j.log4j12</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>us.codecraft</groupId>
    <artifactId>webmagic-extension</artifactId>
    <version>0.7.3</version>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
  <exclusions>
    <exclusion>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
    </exclusion>
  </exclusions>
  <!--<exclusions>-->
  <!--    <exclusion>-->
  <!--        <groupId>org.springframework.boot</groupId>-->
  <!--        <artifactId>spring-boot-starter-logging</artifactId>-->
  <!--    </exclusion>-->
  <!--    <exclusion>-->
  <!--        <groupId>org.slf4j</groupId>-->
  <!--        <artifactId>slf4j-log4j12</artifactId>-->
  <!--    </exclusion>-->
  <!--</exclusions>-->
</dependency>
<!--<dependency>-->
<!--    <groupId>org.springframework.boot</groupId>-->
<!--    <artifactId>spring-boot-starter-log4j</artifactId>-->
<!--</dependency>-->

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

### 4.2 log4j配置

```properties
log4j.rootLogger=info,error,CONSOLE,DEBUG
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
log4j.appender.CONSOLE.layout.ConversionPattern=%d{yyyy-MM-dd-HH-mm} [%t] [%c] [%p] - %m%n

log4j.logger.info=info
log4j.appender.info=org.apache.log4j.DailyRollingFileAppender
log4j.appender.info.layout=org.apache.log4j.PatternLayout
log4j.appender.info.layout.ConversionPattern=%d{yyyy-MM-dd-HH-mm} [%t] [%c] [%p] - %m%n
log4j.appender.info.datePattern='.'yyyy-MM-dd
log4j.appender.info.Threshold = info
log4j.appender.info.append=true
#log4j.appender.info.File=d://springboot3/logs/api_services_info.log

log4j.logger.error=error
log4j.appender.error=org.apache.log4j.DailyRollingFileAppender
log4j.appender.error.layout=org.apache.log4j.PatternLayout
log4j.appender.error.layout.ConversionPattern=%d{yyyy-MM-dd-HH-mm} [%t] [%c] [%p] - %m%n
log4j.appender.error.datePattern='.'yyyy-MM-dd
log4j.appender.error.Threshold = error
log4j.appender.error.append=true
#log4j.appender.error.File=d://springboot3/logs/error/api_services_error.log
log4j.logger.DEBUG=DEBUG
log4j.appender.DEBUG=org.apache.log4j.DailyRollingFileAppender
log4j.appender.DEBUG.layout=org.apache.log4j.PatternLayout
log4j.appender.DEBUG.layout.ConversionPattern=%d{yyyy-MM-dd-HH-mm} [%t] [%c] [%p] - %m%n
log4j.appender.DEBUG.datePattern='.'yyyy-MM-dd
log4j.appender.DEBUG.Threshold = DEBUG
log4j.appender.DEBUG.append=true
#log4j.appender.DEBUG.File=d://springboot3/logs/debug/api_services_debug.log
```

### 4.3 SpringBoot启动类

```kotlin
@SpringBootApplication
@EnableScheduling//启动定时任务
class CrawlerApplication {
    @Value("\${spring.redis.host}")//yml写好host
    private lateinit var redis_host: String

    /**
     * redis排程器
     */
    @Bean
    fun redisScheduler():RedisScheduler{
        return RedisScheduler(redis_host)
    }
  
    companion object{
        @JvmStatic
        fun main(args: Array<String>) {
            runApplication<CrawlerApplication>()
        }
    }
}
```

### 4.4 配置PageProcessor

```kotlin
//爬取类
@Component
class ArticleProcessor : PageProcessor {
    override fun getSite(): Site {
        return Site.me()
                .setRetryTimes(3000)
                .setSleepTime(100)
    }

    override fun process(page: Page) {
        page.addTargetRequests(page.html.links().regex(
                "https://blog.csdn.net/[a-z 0-9 _]+/article/details/[0-9]{9}"
        ).all())

        val title = page.html.xpath(
                """
                //*[@id="mainBox"]/main/div[1]/div[1]/div[1]/div[1]/h1/text()
            """.trimIndent()
        ).get() ?: page.setSkip(true)//跳过

        val content = page.html.xpath(
                """
                    //*[@id="content_views"]
                """.trimIndent()
        ).get() ?: page.setSkip(true)

        val url = page.url.toString()
        if (title is String && content is String) {
            page.putField("title", title)
            page.putField("content", content)
            page.putField("url", url)
        }
    }

}
```

### 4.5 配置Pipeline

```kotlin
//存入数据库
@Component
class ArticleDbPipeline:Pipeline {
    @Autowired
    private lateinit var articleDao: ArticleDao
    @Autowired
    private lateinit var idWorker: IdWorker

    var channelId: String = ""

    override fun process(resultItems: ResultItems, task: Task?) {
        val title: String = resultItems.get<String>("title")
        val content: String = resultItems.get<String>("content")
        val article: Article = Article()
        article.id = "${idWorker.nextId()}"
        article.channelid = channelId
        article.title = title
        article.content = content
        articleDao.save(article)
    }
}
```

### 4.6 配置定时任务

```kotlin
@Component
class ArticleTask {

    @Autowired
    private lateinit var articleDbPipeline: ArticleDbPipeline
    @Autowired
    private lateinit var redisScheduler: RedisScheduler
    @Autowired
    private lateinit var articleProcessor: ArticleProcessor

    /**
     * 爬取ai数据
     */
    @Scheduled(cron = "59 23 23 * * *")//秒，分，时，月，日,星期几
    fun aiTask() {
        println("爬取AI文章")
        val spider = Spider.create(articleProcessor)
        spider.addUrl("https://blog.csdn.net/nav/ai")
        articleDbPipeline.channelId = "ai"
        spider.addPipeline(articleDbPipeline)
        spider.setScheduler(redisScheduler)
        spider.start()
    }
		//爬取的时间段一定要分开
    fun dbTask() {
        println("爬取db文章")
        val spider = Spider.create(articleProcessor)
        spider.addUrl("https://blog.csdn.net/nav/db")
        articleDbPipeline.channelId = "db"
        spider.addPipeline(articleDbPipeline)
        spider.setScheduler(redisScheduler)
        spider.start()
    }

    fun webTask() {
        println("爬取web文章")
        val spider = Spider.create(articleProcessor)
        spider.addUrl("https://blog.csdn.net/nav/web")
        articleDbPipeline.channelId = "web"
        spider.addPipeline(articleDbPipeline)
        spider.setScheduler(redisScheduler)
        spider.start()
    }
}
```

