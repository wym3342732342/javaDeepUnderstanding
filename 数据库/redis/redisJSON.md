# RedisJSON

## 一、介绍

### 1.1 RedisJSON简介

&emsp;&emsp;`RedisJSON`是一个 `Redis` 模块，它实现了`JSON`数据交换标准`ECMA-404`，作为**原生数据类型**。它允许从`Redis`中存储、更新和获取`JSON`值。



### 1.2 特点

- 完全支持`JSON`标准；
- 使用类似`JSONPath`的语法用来**选择文档中的元素**；
- 文档以二进制数据的形式存储**树结构**中，可以**快速访问**子元素；
- 所有JSON值类型都是**原子操作**；





## 二、安装

### 2.1 加载模块安装

- 方式一：

  ```shell
  redis-server --loadmodule /usr/lib/redis/module/rejson.so
  ```

- 方式二：在redis.conf中,添加一下内容

  ```conf
  loadmodule /usr/lib/redis/module/rejson.so
  ```



### 2.2 docker安装

```shell
docker run -di -p 6379:6379 --name redis-redisjson redislabs/rejson:latest
```





## 三、操作

### 3.1 命令行操作

> 1. 命令和子命令的名称是大写的，例如`JSON.SET` 和`INDENT`
> 2. 强制参数用尖括号括起来，例如`<path>`
> 3. 可选参数用方括号括起来，例如`[index]`
> 4. 其他可选参数由三个句点字符表示，即`...`
> 5. 管道字符|表示异或



1. `SET`命令

   ```shell
   JSON.SET <key> <path> <json> [NX | XX]
   ```

   - 说明：

     - `NX`：如果不存在就添加
     - `XX`：如果存在就更新

   - 测试：

     ```shell
     JSON.SET doc $ '{"a":2,"b":3,"nested":{"a":4,"b":null}}'
     # 返回：ok
     # 查看set结果：JSON.GET doc
     ```

2. `GET`命令

   ```shell
   JSON.GET <key> [INDENT indentation-string] [NEWLINE line-break-string] [SPACE space-string] [path ...]
   ```

   - 说明：

     - `INDENT`：设置嵌套级别
     - `NEWLINE`：每行末尾打印的字符串
     - `SPACE`：k、v之间的字符串
     - `PATH`：可以是多个，默认是root

   - 测试：

     ```shell
     1. JSON.GET doc INDENT "\t" NEWLINE "\n" SPACE " " .
     # 输出： "{\n\t\"a\": 2,\n\t\"b\": 3,\n\t\"nested\": {\n\t\t\"a\": 4,\n\t\t\"b\": null\n\t}\n}"
     2. JSON.GET doc $..b
     # 输出： "[3,null]"
     3. JSON.GET doc ..a $..b
     # 输出： "{\"$..b\":[3,null],\"..a\":[2,4]}"
     ```

3. `MGET`命令：多key合并查询

   ```shell
   JSON.MGET <key> [key ...] <path>
   ```

   - 测试：

     ```shell
     # 数据准备
     JSON.SET doc1 $ '{"a":1, "b": 2, "nested": {"a": 3}, "c": null}'
     JSON.SET doc2 $ '{"a":4, "b": 5, "nested": {"a": 6}, "c": null}'
     # 测试
     JSON.MGET doc1 doc2 $..a
     # 输出结果
     1) "[1,3]"
     2) "[4,6]"
     ```

4. `DEL`命令

   ```shell
   JSON.DEL <key> [path]
   ```

   - 说明：不存在的`key`或`path`会被忽略，返回integer

   - 测试：

     ```shell
     # 删除key
     127.0.0.1:6379> keys *
     1) "doc2"
     2) "doc1"
     3) "doc"
     
     127.0.0.1:6379> JSON.DEL doc
     (integer) 1
     
     127.0.0.1:6379> keys *
     1) "doc2"
     2) "doc1"
     
     ### 删除path:间接演示path的使用
     127.0.0.1:6379> JSON.GET doc1
     "{\"a\":1,\"b\":2,\"nested\":{\"a\":3},\"c\":null}"
     
     127.0.0.1:6379> JSON.DEL doc1 $.a
     (integer) 1
     
     127.0.0.1:6379> JSON.GET doc1
     "{\"c\":null,\"b\":2,\"nested\":{\"a\":3}}"
     
     127.0.0.1:6379> JSON.GET doc2
     "{\"a\":4,\"b\":5,\"nested\":{\"a\":6},\"c\":null}"
     
     127.0.0.1:6379> JSON.DEL doc2 $..a
     (integer) 2
     
     127.0.0.1:6379> JSON.GET doc2
     "{\"c\":null,\"b\":5,\"nested\":{}}"
     ```

5. `NUMINCRBY`：增加数字的值

   ```shell
   JSON.NUMINCRBY <key> <path> <number>
   ```

6. `NUMMULTBY`：数字乘法

   ```shell
   JSON.NUMMULTBY <key> <path> <number>
   ```

7. `STRAPPEND`：追加字符串

   ```shell
   JSON.STRAPPEND <key> [path] <json-string>
   ```

8. `STRLEN`：字符串的长度

   ```shell
   JSON.STRLEN <key> [path]
   ```

9. `ARRAPPEND`：追加数组元素

   ```shell
   JSON.ARRAPPEND <key> <path> <json> [json ...]
   ```

10. `ARRINDEX`：搜索指定元素在数组中第一次出现的位置,如果存在返回索引,不存在返回-1

    ```shell
    JSON.ARRINDEX <key> <path> <json-scalar> [start [stop]]
    ### [start [stop]] 从start开始(包含)到stop(不包含)的范围
    ```

11. `ARRINSERT`：在数组指定位置插入元素

    ```shell
    JSON.ARRINSERT <key> <path> <index> <json> [json ...]
    ### index: 0是数组第一个元素,负数表示从末端开始计算
    ```

12. `ARRLEN`：数组的长度

    ```shell
    JSON.ARRLEN <key> [path]
    ### 如果key或path不存在,返回null
    ```

13. `ARRPOP`：删除返回数组中指定位置的元素

    ```shell
    JSON.ARRPOP <key> [path [index]]
    ### index: 默认是-1,最后一个元素
    ```

14. `ARRTRIM`：去掉元素,使其仅包含指定的包含范围的元素

    ```shell
    JSON.ARRTRIM <key> <path> <start> <stop>
    ```

15. `OBJKEYS`：返回对象中的key

    ```shell
    JSON.OBJKEYS <key> [path]
    ```

16. `OBJLEN`：返回对象key的数量

    ```shell
    JSON.OBJLEN <key> [path]
    ```

17. `TYPE`：返回json value的数据类型

    ```shell
    JSON.TYPE <key> [path]
    ```

18. `DEBUG`：返回key的字节数

    ```shell
    JSON.DEBUG MEMORY <key> [path]
    ```



### 3.2 Java操作

1. Maven依赖

   ```xml
   <dependency>
     <groupId>com.redislabs</groupId>
     <artifactId>jrejson</artifactId>
     <version>1.4.0</version>
   </dependency>
   ```

2. github地址：`https://github.com/RedisJSON/JRedisJSON`

3. java代码：

   ```java
   import cn.hutool.core.lang.Console;
   import com.redislabs.modules.rejson.JReJSON;
   import com.redislabs.modules.rejson.Path;
   import org.junit.jupiter.api.Test;
   
   public class RedisJsonTest {
       @Test
       public void test(){
           // 获取连接
           JReJSON client = new JReJSON("127.0.0.1", 6379);
   
           // 添加字符串(路径为根路径)并返回
           client.set("name","lcj", Path.ROOT_PATH);
           String name = client.get("name");
           Console.log("字符串name:{}",name);
   
           name = client.get("name", String.class, Path.of("."));
           Console.log("字符串name:{}",name);
   
           name = client.get("name", String.class, Path.ROOT_PATH);
           Console.log("字符串name:{}",name);
       }
   }
   ```





## 四、实验内容

### 4.1 内存使用（实验和文档不一致）

```shell
使用json.debug memory key计算大小

字符串最少占用24字节
127.0.0.1:6379> json.set str . '""'
OK
127.0.0.1:6379> json.debug memory str
(integer) 24

数组最少占用24个字节
127.0.0.1:6379> json.set arr . []
OK
127.0.0.1:6379> json.debug memory arr
(integer) 24

json对象最少占用72个字节
127.0.0.1:6379> json.set obj . {}
OK
127.0.0.1:6379> json.debug memory obj
(integer) 72
```



### 4.2 路径与法

&emsp;&emsp;`RedisJSON`目前支持两种查询语法：`JSONPath`语法和`RedisJSON`第一个版本的路径语法。

&emsp;&emsp;`RedisJSON`根据路径查询的**第一个字符决定使用哪种语法**。如果查询以字符`$`开头，则使用`JSONPath`语法。否则，它默认为路径语法。

- JSONPath:
  - RedisJSON 2.0引入了JSONPath支持。
  - JSONPath查询可以解析JSON文档中的多个位置。在这种情况下，JSON命令将操作每个可能的位置。这是对遗留查询的重大改进，早期查询只在第一条路径上运行。
  - 注意，在使用JSONPath时，命令响应的结构通常不同。
  - 新语法支持括号表示法，允许在键名中使用特殊字符，如冒号“：”或空格。

- Legacy Path syntax (RedisJSON v1):
  - RedisJSON的第一个版本有以下实现。RedisJSON v2仍然支持它。
  - 路径总是从JSON值的根开始。根由字符（.）表示。对于引用根的子级的路径，可以选择在路径前面加上根前缀。
  - 要访问数组元素，请将其索引括在一对方括号内。索引是基于0的，0是数组的第一个元素。可以使用负偏移来访问从数组末端开始的元素。-1是数组中的最后一个元素

- **json key的规则**：必须以字符,`$`,`_`开头可以包含字符,数字,`$`,`_`大小写敏感



## 扩展

```shell
## 源码
github: https://github.com/RedisJSON/RedisJSON

## 演示
website: https://oss.redis.com/redisjson/

## 官网
https://redis.com/modules/redis-json/
```

