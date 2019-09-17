# 一、环境准备

## 1.1 【Ⅰ】SpringBoot下配置搭建

### 1.1.1 maven依赖

```xml
<!--添加springboot父工程坐标-->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.4.RELEASE</version>
</parent>

<properties>
    <java.version>1.8</java.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <!-- lombok -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <!-- springboot test -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <!-- mysql驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.6</version>
    </dependency>
    <!-- mybatis plus -->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.1.2</version>
    </dependency>
    <!-- log不用引入 -->
</dependencies>
<build>
    <plugins>
        <!-- springboot 插件 -->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

### 1.1.2 配置类

```java
@Configuration
@MapperScan("club.maddm.mapper")//配置类上写也会生效
public class MybatisPlusConfig {
    /*
    * 分页插件，自动识别数据库类型
    * 多租户，请参考官网【插件扩展】
    */
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }

    /*
        * oracle数据库配置JdbcTypeForNull
        * 参考：https://gitee.com/baomidou/mybatisplus-boot-starter/issues/IHS8X
        不需要这样配置了，参考 yml:
        mybatis-plus:
          confuguration
            dbc-type-for-null: 'null' 
       @Bean
       public ConfigurationCustomizer configurationCustomizer(){
           return new MybatisPlusCustomizers();
       }

       class MybatisPlusCustomizers implements ConfigurationCustomizer {

           @Override
           public void customize(org.apache.ibatis.session.Configuration configuration) {
               configuration.setJdbcTypeForNull(JdbcType.NULL);
           }
       }
   */
}
```

### 1.1.3 配置文件（Application.yml,现已经全部注解完成【不推荐】）

```yaml
mybatis-plus:
  # 如果是放在src/main/java目录下 classpath:/com/yourpackage/*/mapper/*Mapper.xml
  # 如果是放在resource目录 classpath:/mapper/*Mapper.xml
#  mapper-locations: classpath:/mapper/*Mapper.xml # 此处不妨注释掉
  #实体扫描，多个package用逗号或者分号分隔
  typeAliasesPackage: club.maddm.pojo
  global-config:
    #主键类型  0:"数据库ID自增", 1:"用户输入ID",2:"全局唯一ID (数字类型唯一ID)", 3:"全局唯一ID UUID";
    id-type: 3
    #字段策略 0:"忽略判断",1:"非 NULL 判断"),2:"非空判断"
    field-strategy: 2
    #驼峰下划线转换
    db-column-underline: true
    #mp2.3+ 全局表前缀 mp_
    #table-prefix: mp_
    #刷新mapper 调试神器
    #refresh-mapper: true
    #数据库大写下划线转换
    #capital-mode: true
    # Sequence序列接口实现类配置
#    key-generator: com.baomidou.mybatisplus.incrementer.OracleKeyGenerator
    #逻辑删除配置（下面3个配置）
#    logic-delete-value: 1
#    logic-not-delete-value: 0
#    sql-injector: com.baomidou.mybatisplus.mapper.LogicSqlInjector
    #自定义填充策略接口实现
#    meta-object-handler: com.baomidou.springboot.MyMetaObjectHandler
#  configuration:
    #配置返回数据库(column下划线命名&&返回java实体是驼峰命名)，自动匹配无需as（没开启这个，SQL需要写as： select user_id as userId）
#      map-underscore-to-camel-case: true
#    cache-enabled: false
    #配置JdbcTypeForNull, oracle数据库必须配置
#    jdbc-type-for-null: 'null'

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mybatis_plus
    username: root
    password: 123456
    
logging: #不添加logging启动器也可以
  level:
    club.maddm.mptest: debug #设置dao层的日志级别
```



## 1.2【Ⅰ】传统SSM配置搭建

```java
//暂时省略。。。。。。。。。。。
```



## 1.3 【Ⅱ】mapper接口继承BaseMapper

```java
public interface PropeoMapper extends BaseMapper<Propeo> {
}
```

# 二、常用注解

> 详细的说明可查看源码包： [mybatis-plus-annotation](https://gitee.com/baomidou/mybatis-plus/tree/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation)

### [@TableName](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/TableName.java)

- 描述：表名注解

|       属性       |  类型   | 必须指定 | 默认值 | 描述                                                         |
| :--------------: | :-----: | :------: | :----: | ------------------------------------------------------------ |
|      value       | String  |    否    |   ""   | 表名                                                         |
|    resultMap     | String  |    否    |   ""   | xml 中 resultMap 的 id                                       |
|      schema      | String  |    否    |   ""   | schema(@since 3.1.1)                                         |
| keepGlobalPrefix | boolean |    否    | false  | 是否保持使用全局的 tablePrefix 的值(如果设置了全局 tablePrefix 且自行设置了 value 的值)(@since 3.1.1) |

### [@TableId](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/TableId.java)

- 描述：主键注解

| 属性  |  类型  | 必须指定 |   默认值    |    描述    |
| :---: | :----: | :------: | :---------: | :--------: |
| value | String |    否    |     ""      | 主键字段名 |
| type  |  Enum  |    否    | IdType.NONE |  主键类型  |

#### [IdType](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/IdType.java)用于上方type

|      值       |            描述             |
| :-----------: | :-------------------------: |
|     AUTO      |         数据库自增          |
|     INPUT     |          自行输入           |
|   ID_WORKER   | 分布式全局唯一ID 长整型类型 |
|     UUID      |       32位UUID字符串        |
|     NONE      |           无状态            |
| ID_WORKER_STR | 分布式全局唯一ID 字符串类型 |

### [@TableField](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/TableField.java)

- 描述：字段注解(非主键)

|       属性       |  类型   | 必须指定 |        默认值         |                             描述                             |
| :--------------: | :-----: | :------: | :-------------------: | :----------------------------------------------------------: |
|      value       | String  |    否    |          ""           |                            字段名                            |
|        el        | String  |    否    |          ""           | 映射为原生 `#{ ... }` 逻辑,相当于写在 xml 里的 `#{ ... }` 部分 |
|      exist       | boolean |    否    |         true          |                      是否为数据库表字段                      |
|    condition     | String  |    否    |          ""           | 字段 `where` 实体查询比较条件,有值设置则按设置的值为准,没有则为默认全局的 `%s=#{%s}`,[参考](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/SqlCondition.java) |
|      update      | String  |    否    |          ""           | 字段 `update set` 部分注入, 例如：update="%s+1"：表示更新时会set version=version+1(该属性优先级高于 `el` 属性) |
|     strategy     |  Enum   |    否    | FieldStrategy.DEFAULT |              字段验证策略 3.1.2+使用下面3个替代              |
|  insertStrategy  |  Enum   |    N     |        DEFAULT        | 举例：NOT_NULL: `insert into table_a(<if test="columnProperty != null">column</if>) values (<if test="columnProperty != null">#{columnProperty}</if>)` (since v_3.1.2) |
|  updateStrategy  |  Enum   |    N     |        DEFAULT        | 举例：IGNORED: `update table_a set column=#{columnProperty}` (since v_3.1.2) |
|  whereStrategy   |  Enum   |    N     |        DEFAULT        | 举例：NOT_EMPTY: `where <if test="columnProperty != null and columnProperty!=''">column=#{columnProperty}</if>` (since v_3.1.2) |
|       fill       |  Enum   |    否    |   FieldFill.DEFAULT   |                       字段自动填充策略                       |
|      select      | boolean |    否    |         true          |                     是否进行 select 查询                     |
| keepGlobalFormat | boolean |    否    |         false         |       是否保持使用全局的 format 进行处理(@since 3.1.1)       |

#### [FieldStrategy](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/FieldStrategy.java)

|    值     |                           描述                            |
| :-------: | :-------------------------------------------------------: |
|  IGNORED  |                         忽略判断                          |
| NOT_NULL  |                        非NULL判断                         |
| NOT_EMPTY | 非空判断(只对字符串类型字段,其他类型字段依然为非NULL判断) |
|  DEFAULT  |                       追随全局配置                        |

#### [FieldFill](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/FieldFill.java)

|      值       |         描述         |
| :-----------: | :------------------: |
|    DEFAULT    |      默认不处理      |
|    INSERT     |    插入时填充字段    |
|    UPDATE     |    更新时填充字段    |
| INSERT_UPDATE | 插入和更新时填充字段 |

### [@Version](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/Version.java)

- 描述：乐观锁注解、标记 `@Verison` 在字段上

### [@EnumValue](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/EnumValue.java)

- 描述：通枚举类注解(注解在枚举字段上)

### [@TableLogic](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/TableLogic.java)

- 描述：表字段逻辑处理注解（逻辑删除）

|  属性  |  类型  | 必须指定 | 默认值 |     描述     |
| :----: | :----: | :------: | :----: | :----------: |
| value  | String |    否    |   ""   | 逻辑未删除值 |
| delval | String |    否    |   ""   |  逻辑删除值  |

### [@SqlParser](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/SqlParser.java)

- 描述：租户注解 目前只支持注解在 mapper 的方法上(3.1.1开始支持注解在mapper上)

|  属性  |  类型   | 必须指定 | 默认值 |                             描述                             |
| :----: | :-----: | :------: | :----: | :----------------------------------------------------------: |
| filter | boolean |    否    | false  | true: 表示过滤SQL解析，即不会进入ISqlParser解析链，否则会进解析链并追加例如tenant_id等条件 |

### [@KeySequence](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-annotation/src/main/java/com/baomidou/mybatisplus/annotation/KeySequence.java)

- 描述：序列主键策略 `oracle`
- 属性：value、resultMap

| 属性  |  类型  | 必须指定 |   默认值   |                             描述                             |
| :---: | :----: | :------: | :--------: | :----------------------------------------------------------: |
| value | String |    否    |     ""     |                            序列名                            |
| clazz | Class  |    否    | Long.class | id的类型, 可以指定String.class，这样返回的Sequence值是字符串"1" |





# 三、CRUD 接口

## 3.1 Mapper CRUD 接口

> 说明:
>
> - 通用 CRUD 封装[BaseMapper](https://gitee.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-core/src/main/java/com/baomidou/mybatisplus/core/mapper/BaseMapper.java)接口，为 `Mybatis-Plus` 启动时自动解析实体表关系映射转换为 `Mybatis` 内部对象注入容器
> - 泛型 `T` 为任意实体对象
> - 参数 `Serializable` 为任意类型主键 `Mybatis-Plus` 不推荐使用复合主键约定每一张表都有自己的唯一 `id` 主键
> - 对象 `Wrapper` 为 [条件构造器](https://mp.baomidou.com/guide/wrapper.html)

### 3.1.1 、增



#### 1. 根据实体类新增：insert

> Id：会写回ID

```java
/**
 * <p>
 * 插入一条记录
 * </p>
 *
 * @param entity 实体对象
 * @return 插入成功记录数
 */
int insert(T entity);
```

### 3.1.2、删

#### 1. 根据ID删除：deleteById

```java
/**
 * <p>
 * 根据 ID 删除
 * </p>
 *
 * @param id 主键ID
 * @return 删除成功记录数
 */
int deleteById(Serializable id);
```

#### 2.根据KV删除：deleteByMap

> key：列名，value：条件值
>
> 用于相等的情况，所有的map异曲同工

```java
/**
 * <p>
 * 根据 columnMap 条件，删除记录
 * </p>
 *
 * @param columnMap 表字段 map 对象
 * @return 删除成功记录数
 */
int deleteByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
```

#### 3. 根据条件构造器删除：delete

> 在【四】处详细展开。为空时为没有条件

```java
/**
 * <p>
 * 根据 entity 条件，删除记录
 * </p>
 *
 * @param wrapper 实体对象封装操作类（可以为 null）
 * @return 删除成功记录数
 */
int delete(@Param(Constants.WRAPPER) Wrapper<T> wrapper);
```

#### 4. 根据Id的集合删除：deleteBatchIds

```java
/**
 * <p>
 * 删除（根据ID 批量删除）
 * </p>
 *
 * @param idList 主键ID列表(不能为 null 以及 empty)
 * @return 删除成功记录数
 */
int deleteBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
```

### 3.1.3、改

#### 1. 根据Id修改：updateById

```java
/**
 * <p>
 * 根据 ID 修改
 * </p>
 *
 * @param entity 实体对象
 * @return 修改成功记录数
 */
int updateById(@Param(Constants.ENTITY) T entity);
```

#### 2. 根据条件构造器修改：update

```java
/**
 * <p>
 * 根据 whereEntity 条件，更新记录
 * </p>
 *
 * @param entity        实体对象 (set 条件值,可为 null)
 * @param updateWrapper 实体对象封装操作类（可以为 null,里面的 entity 用于生成 where 语句）
 * @return 修改成功记录数
 */
int update(@Param(Constants.ENTITY) T entity, @Param(Constants.WRAPPER) Wrapper<T> updateWrapper);
```

### 3.1.4、查

#### 1. 根据Id查询：selectById

```java
/**
 * <p>
 * 根据 ID 查询
 * </p>
 *
 * @param id 主键ID
 * @return 实体
 */
T selectById(Serializable id);
```

#### 2. 根据Id集合查询：selectBatchIds

```java
/**
 * <p>
 * 查询（根据ID 批量查询）
 * </p>
 *
 * @param idList 主键ID列表(不能为 null 以及 empty)
 * @return 实体集合
 */
List<T> selectBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
```

#### 3. 根据KV条件查询：selectByMap

```java
/**
 * <p>
 * 查询（根据 columnMap 条件）
 * </p>
 *
 * @param columnMap 表字段 map 对象
 * @return 实体集合
 */
List<T> selectByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
```

#### 4. 根据条件构造器查询一个：selectOne

```java
/**
 * <p>
 * 根据 entity 条件，查询一条记录
 * </p>
 *
 * @param queryWrapper 实体对象
 * @return 实体
 */
T selectOne(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

#### 5. 根据构造器查询总记录数：selectCount

```java
/**
 * <p>
 * 根据 Wrapper 条件，查询总记录数
 * </p>
 *
 * @param queryWrapper 实体对象
 * @return 满足条件记录数
 */
Integer selectCount(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

#### 6. 根据构造器查询全部：selectList

```java
/**
 * <p>
 * 根据 entity 条件，查询全部记录
 * </p>
 *
 * @param queryWrapper 实体对象封装操作类（可以为 null）
 * @return 实体集合
 */
List<T> selectList(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

#### 7.根据构造器查询： selectMaps

> 会将数据库字段映射为KV形式

```java
/**
 * <p>
 * 根据 Wrapper 条件，查询全部记录
 * </p>
 *
 * @param queryWrapper 实体对象封装操作类（可以为 null）
 * @return 字段映射对象 Map 集合
 */
List<Map<String, Object>> selectMaps(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

#### 8. selectObjs

```java
/**
 * <p>
 * 根据 Wrapper 条件，查询全部记录
 * 注意： 只返回第一个字段的值
 * </p>
 *
 * @param queryWrapper 实体对象封装操作类（可以为 null）
 * @return 字段映射对象集合
 */
List<Object> selectObjs(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

#### 9. 分页查询：selectPage

```java
/**
 * <p>
 * 根据 entity 条件，查询全部记录（并翻页）
 * </p>
 *
 * @param page         分页查询条件（可以为 RowBounds.DEFAULT）
 * @param queryWrapper 实体对象封装操作类（可以为 null）
 * @return 实体分页对象
 */
IPage<T> selectPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

#### 10 .分页查询【Map结构】：selectMapsPage

```java
/**
 * <p>
 * 根据 Wrapper 条件，查询全部记录（并翻页）
 * </p>
 *
 * @param page         分页查询条件
 * @param queryWrapper 实体对象封装操作类
 * @return 字段映射对象 Map 分页对象
 */
IPage<Map<String, Object>> selectMapsPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```



## 3.2 Service CRUD 接口

没啥用暂时省略。。



# 四、条件构造器

> - 不支持以及不赞成在 RPC 调用中把 Wrapper 进行传输
>
> - 如果需要在RPC中传输，建议使用DTO,接收到DTO后再构建条件

## 4.1 AbstractWrapper

> 说明:
>
> QueryWrapper(LambdaQueryWrapper) 和 UpdateWrapper(LambdaUpdateWrapper) 的父类
> 用于生成 sql 的 where 条件, entity 属性也用于生成 sql 的 where 条件
> 注意: entity 生成的 where 条件与 使用各个 api 生成的 where 条件**没有任何关联行为**

### 1. 全部判断：allEq



**1.1 全部[eq](#)(或个别[isNull](#))**

```java
allEq(Map<R, V> params)
allEq(Map<R, V> params, boolean null2IsNull)
allEq(boolean condition, Map<R, V> params, boolean null2IsNull)
```

> `boolean condition`:表示是否加入生成sql，默认为`true`

个别参数说明:

`params` : `key`为数据库字段名,`value`为字段值
`null2IsNull` : 为`true`则在`map`的`value`为`null`时调用 [isNull](https://mp.baomidou.com/guide/wrapper.html#isnull) 方法,为`false`时则忽略`value`为`null`的

- 例1: `allEq({id:1,name:"老王",age:null})`--->`id = 1 and name = '老王' and age is null`
- 例2: `allEq({id:1,name:"老王",age:null}, false)`--->`id = 1 and name = '老王'`

**1.2 断言式函数过滤字段**

```java
allEq(BiPredicate<R, V> filter, Map<R, V> params)
allEq(BiPredicate<R, V> filter, Map<R, V> params, boolean null2IsNull)
allEq(boolean condition, BiPredicate<R, V> filter, Map<R, V> params, boolean null2IsNull) 
```

个别参数说明:

`filter` : 过滤函数,是否允许字段传入比对条件中
`params` 与 `null2IsNull` : 同上

- 例1: `allEq((k,v) -> k.indexOf("a") > 0, {id:1,name:"老王",age:null})`--->`name = '老王' and age is null`
- 例2: `allEq((k,v) -> k.indexOf("a") > 0, {id:1,name:"老王",age:null}, false)`--->`name = '老王'`



### 2. 相等 ：eq

```java
eq(R column, Object val);//column：列名，val：值
eq(boolean condition, R column, Object val);
```

- 等于 =
- 例: `eq("name", "老王")`--->`name = '老王'`



### 3. 不等于 ：ne

```java
ne(R column, Object val)
ne(boolean condition, R column, Object val)
```

- 不等于 <>
- 例: `ne("name", "老王")`--->`name <> '老王'`



### 4. 大于 ：gt

```java
gt(R column, Object val)
gt(boolean condition, R column, Object val)
```

- 大于 >
- 例: `gt("age", 18)`--->`age > 18`



### 5. 大于等于 ：ge

```java
ge(R column, Object val)
ge(boolean condition, R column, Object val)
```

- 大于等于 >=
- 例: `ge("age", 18)`--->`age >= 18`



### 6. 小于：lt

```java
lt(R column, Object val)
lt(boolean condition, R column, Object val)
```

- 小于 <
- 例: `lt("age", 18)`--->`age < 18`



### 7. 小于等于：le

```java
le(R column, Object val)
le(boolean condition, R column, Object val)
```

- 小于等于 <=
- 例: `le("age", 18)`--->`age <= 18`



### 8. between

```java
between(R column, Object val1, Object val2)
between(boolean condition, R column, Object val1, Object val2)
```

- BETWEEN 值1 AND 值2
- 例: `between("age", 18, 30)`--->`age between 18 and 30`



### 9. notBetween

```java
notBetween(R column, Object val1, Object val2)
notBetween(boolean condition, R column, Object val1, Object val2)
```

- NOT BETWEEN 值1 AND 值2
- 例: `notBetween("age", 18, 30)`--->`age not between 18 and 30`



### 10. like

```java
like(R column, Object val)
like(boolean condition, R column, Object val)
```

- LIKE '%值%'
- 例: `like("name", "王")`--->`name like '%王%'`



### 11. notLike

```java
notLike(R column, Object val)
notLike(boolean condition, R column, Object val)
```

- NOT LIKE '%值%'
- 例: `notLike("name", "王")`--->`name not like '%王%'`



### 12. likeLeft

```java
likeLeft(R column, Object val)
likeLeft(boolean condition, R column, Object val)
```

- LIKE '%值'
- 例: `likeLeft("name", "王")`--->`name like '%王'`



### 13. likeRight

```java
likeRight(R column, Object val)
likeRight(boolean condition, R column, Object val)
```

- LIKE '值%'
- 例: `likeRight("name", "王")`--->`name like '王%'`



### 14.为空： isNull

```java
isNull(R column)
isNull(boolean condition, R column)
```

- 字段 IS NULL
- 例: `isNull("name")`--->`name is null`



### 15. 非空：isNotNull

```java
isNotNull(R column)
isNotNull(boolean condition, R column)
```

- 字段 IS NULL
- 例: `isNotNull("name")`--->`name is not null`

### 16. in

```java
in(R column, Collection<?> value)
in(boolean condition, R column, Collection<?> value)
```

- 字段 IN (value.get(0), value.get(1), ...)
- 例: `in("age",{1,2,3})`--->`age in (1,2,3)`



```java
in(R column, Object... values)
in(boolean condition, R column, Object... values)
```

- 字段 IN (v0, v1, ...)
- 例: `in("age", 1, 2, 3)`--->`age in (1,2,3)`

### 17. notIn

```java
notIn(R column, Collection<?> value)
notIn(boolean condition, R column, Collection<?> value)
```

- 字段 IN (value.get(0), value.get(1), ...)
- 例: `notIn("age",{1,2,3})`--->`age not in (1,2,3)`



```java
notIn(R column, Object... values)
notIn(boolean condition, R column, Object... values)
```

- 字段 NOT IN (v0, v1, ...)
- 例: `notIn("age", 1, 2, 3)`--->`age not in (1,2,3)`



### 18. inSql

```java
inSql(R column, String inValue)
inSql(boolean condition, R column, String inValue)
```

- 字段 IN ( sql语句 )
- 例: `inSql("age", "1,2,3,4,5,6")`--->`age in (1,2,3,4,5,6)`
- 例: `inSql("id", "select id from table where id < 3")`--->`id in (select id from table where id < 3)`

### 19. notInSql

```java
notInSql(R column, String inValue)
notInSql(boolean condition, R column, String inValue)
```

- 字段 NOT IN ( sql语句 )
- 例: `notInSql("age", "1,2,3,4,5,6")`--->`age not in (1,2,3,4,5,6)`
- 例: `notInSql("id", "select id from table where id < 3")`--->`age not in (select id from table where id < 3)`



### 20. 分组：groupBy

```java
groupBy(R... columns)
groupBy(boolean condition, R... columns)
```

- 分组：GROUP BY 字段, ...
- 例: `groupBy("id", "name")`--->`group by id,name`



### 21. 升序排序：orderByAsc【低前】

```java
orderByAsc(R... columns)
orderByAsc(boolean condition, R... columns)
```

- 排序：ORDER BY 字段, ... ASC
- 例: `orderByAsc("id", "name")`--->`order by id ASC,name ASC`



### 22. 降序排序：orderByDesc【高前】

```java
orderByDesc(R... columns)
orderByDesc(boolean condition, R... columns)
```

- 排序：ORDER BY 字段, ... DESC
- 例: `orderByDesc("id", "name")`--->`order by id DESC,name DESC`

### 23. 排序：orderBy

```java
orderBy(boolean condition, boolean isAsc, R... columns)
```

- 排序：ORDER BY 字段, ...
- 例: `orderBy(true, true, "id", "name")`--->`order by id ASC,name ASC`

### 24. 分组后过滤：having

```java
having(String sqlHaving, Object... params)
having(boolean condition, String sqlHaving, Object... params)
```

- HAVING ( sql语句 )
- 例: `having("sum(age) > 10")`--->`having sum(age) > 10`
- 例: `having("sum(age) > {0}", 11)`--->`having sum(age) > 11`



### 25. 或拼接：or

```java
or()
or(boolean condition)
```

- 拼接 OR

注意事项:

主动调用`or`表示紧接着下一个**方法**不是用`and`连接!(不调用`or`则默认为使用`and`连接)

- 例: `eq("id",1).or().eq("name","老王")`--->`id = 1 or name = '老王'`



### 26. 或嵌套：or

```java
or(Function<Param, Param> func)
or(boolean condition, Function<Param, Param> func)
```

- OR 嵌套
- 例: `or(i -> i.eq("name", "李白").ne("status", "活着"))`--->`or (name = '李白' and status <> '活着')`



### 27. 与嵌套：and

```java
and(Function<Param, Param> func)
and(boolean condition, Function<Param, Param> func)
```

- AND 嵌套
- 例: `and(i -> i.eq("name", "李白").ne("status", "活着"))`--->`and (name = '李白' and status <> '活着')`



### 28. 嵌套：nested


```java
nested(Function<Param, Param> func)
nested(boolean condition, Function<Param, Param> func)
```

- 正常嵌套 不带 AND 或者 OR
- 例: `nested(i -> i.eq("name", "李白").ne("status", "活着"))`--->`(name = '李白' and status <> '活着')`

### 29.拼接sql：apply

```java
apply(String applySql, Object... params)
apply(boolean condition, String applySql, Object... params)
```

- 拼接 sql

注意事项:

该方法可用于数据库**函数** 动态入参的`params`对应前面`applySql`内部的`{index}`部分.这样是不会有sql注入风险的,反之会有!

- 例: `apply("id = 1")`--->`id = 1`
- 例: `apply("date_format(dateColumn,'%Y-%m-%d') = '2008-08-08'")`--->`date_format(dateColumn,'%Y-%m-%d') = '2008-08-08'")`
- 例: `apply("date_format(dateColumn,'%Y-%m-%d') = {0}", "2008-08-08")`--->`date_format(dateColumn,'%Y-%m-%d') = '2008-08-08'")`



### 30.拼接sql到最后：last

```java
last(String lastSql)
last(boolean condition, String lastSql)
```

- 无视优化规则直接拼接到 sql 的最后

注意事项:

只能调用一次,多次调用以最后一次为准 有sql注入的风险,请谨慎使用

- 例: `last("limit 1")`



### 31. 拼接exists语句：exists

```java
exists(String existsSql)
exists(boolean condition, String existsSql)
```

- 拼接 EXISTS ( sql语句 )
- 例: `exists("select id from table where age = 1")`--->`exists (select id from table where age = 1)`



### 32. 拼接notExists语句：notExists

```java
notExists(String notExistsSql)
notExists(boolean condition, String notExistsSql)
```

- 拼接 NOT EXISTS ( sql语句 )
- 例: `notExists("select id from table where age = 1")`--->`not exists (select id from table where age = 1)`



## 4.2 QueryWrapper

>说明:
>
>继承自 AbstractWrapper ,自身的内部属性 entity 也用于生成 where 条件
>及 LambdaQueryWrapper, 可以通过 new QueryWrapper().lambda() 方法获取

### 1. 设置查询字段：select

```java
select(String... sqlSelect)
select(Predicate<TableFieldInfo> predicate)
select(Class<T> entityClass, Predicate<TableFieldInfo> predicate)
```

- 设置查询字段

说明:

以上方分法为两类.
第二类方法为:过滤查询字段(主键除外),入参不包含 class 的调用前需要`wrapper`内的`entity`属性有值! 这两类方法重复调用以最后一次为准

- 例: `select("id", "name", "age")`
- 例: `select(i -> i.getProperty().startsWith("test"))`

### 2.排除字段：excludeColumns 【过期】

- 排除查询字段

已从`3.0.5`版本上移除此方法!

## 4.3 UpdateWrapper

> 说明:
>
> 继承自 `AbstractWrapper` ,自身的内部属性 `entity` 也用于生成 where 条件
> 及 `LambdaUpdateWrapper`, 可以通过 `new UpdateWrapper().lambda()` 方法获取!



### 1. 修改该字段值：set

```java
set(String column, Object val)
set(boolean condition, String column, Object val)
```

- SQL SET 字段
- 例: `set("name", "老李头")`
- 例: `set("name", "")`--->数据库字段值变为**空字符串**
- 例: `set("name", null)`--->数据库字段值变为`null`

### 2.通过sql修改： setSql

```java
setSql(String sql)
```

- 设置 SET 部分 SQL
- 例: `setSql("name = '老李头')`

## 4.4 获取lambda

> - 获取 `LambdaWrapper`
>   在`QueryWrapper`中是获取`LambdaQueryWrapper`
>   在`UpdateWrapper`中是获取`LambdaUpdateWrapper`



## 4.5 使用 Wrapper 自定义SQL

> 需求来源:
>
> 在使用了`mybatis-plus`之后, 自定义SQL的同时也想使用`Wrapper`的便利应该怎么办？ 在`mybatis-plus`版本`3.0.7`得到了完美解决 版本需要大于或等于`3.0.7`, 以下两种方案取其一即可

### Service.java

```java
mysqlMapper.getAll(Wrappers.<MysqlData>lambdaQuery().eq(MysqlData::getGroup, 1));
```

### 方案一 注解方式 Mapper.java

```java
@Select("select * from mysql_data ${ew.customSqlSegment}")
List<MysqlData> getAll(@Param(Constants.WRAPPER) Wrapper wrapper);
```

### 方案二 XML形式 Mapper.xml

```xml
<select id="getAll" resultType="MysqlData">
	SELECT * FROM mysql_data ${ew.customSqlSegment}
</select>
```



# 五、MP插件

### 5.1 分页插件



### 1. 分页插件配置

#### 1.1 SpringBoot配置

```java
//Spring boot方式
@EnableTransactionManagement
@Configuration
@MapperScan("com.baomidou.cloud.service.*.mapper*")
public class MybatisPlusConfig {

    /**
     * 分页插件
     */
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
}
```

#### 1.2 SSM配置

```xml
<!-- spring xml 方式 -->
<plugins>
    <plugin interceptor="com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor">
        <property name="sqlParser" ref="自定义解析类、可以没有" />
        <property name="dialectClazz" value="自定义方言类、可以没有" />
    </plugin>
</plugins>
```



### 2. 分页插件的案例

### 2.1 Mapper

```java
//可以不继承BaseMapper
public interface PropeoMapper extends BaseMapper<Propeo> {
    /**
     * <p>
     * 查询 : 根据年龄查询用户列表，分页显示
     * 注意!!: 如果入参是有多个,需要加注解指定参数名才能在xml中取值
     * </p>
     *
     * @param page 分页对象,xml中可以从里面进行取值,传递参数 Page 即自动分页,
     * 必须放在第一位(你可以继承Page实现自己的分页对象)
     * @return 分页对象
     */
    @Select(" SELECT * FROM PROPEO WHERE AGE = #{age} ")
    IPage<Propeo> selectPage(Page page, @Param("age") Integer age);
}
```



### 2.2 测试类

```java
/**
 * 测试分页插件
 */
@Test
public void testPagePlugins() {
    Page<Object> page = new Page<>(1, 2, 0);

    // 不进行 count sql 优化，解决 MP 无法自动优化 SQL 问题，这时候你需要自己查询 count 部分
    // page.setOptimizeCountSql(false);

    // 当 total 为非 0 时(默认为 0),分页插件不会进行 count 查询
    IPage<Propeo> propeoIPage = propeoMapper.selectPage(page, 18);

    // 要点!! 分页返回的对象与传入的对象是同一个
    //对比两个地址码
    if (page.toString().equals(propeoIPage.toString())) {
        List<Propeo> records = propeoIPage.getRecords();
        //循环输出
        records.forEach(System.out::println);
    }
}
```



# 六、代码生成器

## 6.1 SpringBoot下的配置使用

#### 1. 相关依赖

```xml
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>2.1</version>
</dependency>
<!-- MP代码生成器相关依赖 -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.1.2</version>
</dependency>
```



#### 2.使用演示

```java

/**
  * 演示例子，执行 main 方法控制台输入模块表名回车自动生成对应项目目录中
  */
@Test
public void testGenerator() {
    // 代码生成器
    AutoGenerator mpg = new AutoGenerator();

    // 全局配置
    GlobalConfig gc = new GlobalConfig();
    String projectPath = System.getProperty("user.dir");//读取当前工程目录

    gc.setOutputDir(projectPath + "/src/main/java");//生成的目录
    gc.setAuthor("wangyiming");//开发人员
    gc.setOpen(false);//是否打开输出目录
    // gc.setSwagger2(true); 实体属性 Swagger2 注解

    mpg.setGlobalConfig(gc);//设置全局相关配置

    // 数据源配置
    DataSourceConfig dsc = new DataSourceConfig();
    dsc.setUrl("jdbc:mysql://localhost:3306/mybatis_plus?useUnicode=true&useSSL=false&characterEncoding=utf8");
    // dsc.setSchemaName("public");
    dsc.setDriverName("com.mysql.jdbc.Driver");
    dsc.setUsername("root");
    dsc.setPassword("123456");


    mpg.setDataSource(dsc);//配置数据源相关配置

    // 包配置
    PackageConfig pc = new PackageConfig();

    //输入模块名
    pc.setModuleName(scanner("模块名"));//获取模块名，报名的最后


    pc.setParent("club.maddm");//配置报名前缀 最后会加上模块名

    mpg.setPackageInfo(pc);//配置包

    // 自定义配置
    InjectionConfig cfg = new InjectionConfig() {//抽象的对外接口
        @Override
        public void initMap() {
            // to do nothing
        }
    };

    // 如果模板引擎是 freemarker
    //        String templatePath = "/templates/mapper.xml.ftl";
    // 如果模板引擎是 velocity
    String templatePath = "/templates/mapper.xml.vm";

    // 自定义输出配置
    List<FileOutConfig> focList = new ArrayList<>();
    // 自定义配置会被优先输出
    focList.add(new FileOutConfig(templatePath) {
        @Override
        public String outputFile(TableInfo tableInfo) {
            // 自定义输出文件名 ， 如果你 Entity 设置了前后缀、此处注意 xml 的名称会跟着发生变化！！
            return projectPath + "/src/main/resources/mapper/" + pc.getModuleName()
                + "/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
        }
    });
    /*
        cfg.setFileCreate(new IFileCreate() {
            @Override
            public boolean isCreate(ConfigBuilder configBuilder, FileType fileType, String filePath) {
                // 判断自定义文件夹是否需要创建
                checkDir("调用默认方法创建的目录");
                return false;
            }
        });
        */
    cfg.setFileOutConfigList(focList);//自定义输出文件

    mpg.setCfg(cfg);//配置相关自定义设置

    // 配置模板
    TemplateConfig templateConfig = new TemplateConfig();

    // 配置自定义输出模板
    //指定自定义模板路径，注意不要带上.ftl/.vm, 会根据使用的模板引擎自动识别
    // templateConfig.setEntity("templates/entity2.java");
    // templateConfig.setService();
    // templateConfig.setController();

    //模板的相关配置
    templateConfig.setXml(null);
    mpg.setTemplate(templateConfig);

    // 策略配置
    StrategyConfig strategy = new StrategyConfig();
    strategy.setNaming(NamingStrategy.underline_to_camel);//数据库表映射到实体的命名策略：下划线转驼峰命名
    strategy.setColumnNaming(NamingStrategy.underline_to_camel);//数据库表字段映射到实体的命名策略：下划线转驼峰命名
    //        strategy.setSuperEntityClass("com.baomidou.ant.common.BaseEntity");//自定义继承的Entity类全称，带包名

    strategy.setEntityLombokModel(true);//【实体】是否为lombok模型（默认 false）
    strategy.setRestControllerStyle(true);//生成RestController<控制器
    // RestController公共父类
    //        strategy.setSuperControllerClass("com.baomidou.ant.common.BaseController");
    // 写于父类中的公共字段
    //        strategy.setSuperEntityColumns("id");//自定义基础的Entity类，公共字段

    //输入表名，针对那些表生储层
    strategy.setInclude(scanner("表名，多个英文逗号分割").split(","));


    strategy.setControllerMappingHyphenStyle(true);//从数据库表到文件的命名策略,是否采用连字符模式
    strategy.setTablePrefix(pc.getModuleName() + "_");//表前缀
    mpg.setStrategy(strategy);//数据库配置

    //        mpg.setTemplateEngine(new FreemarkerTemplateEngine());//模板引擎
    mpg.setTemplateEngine(new VelocityTemplateEngine());//使用velocity模板
    mpg.execute();
}

/**
  * <p>
  * 读取控制台内容
  * </p>
  */
public String scanner(String tip) {
    Scanner scanner = new Scanner(System.in);
    StringBuilder help = new StringBuilder();
    help.append("请输入" + tip + "：");
    System.out.println(help.toString());

    if (true) {
        return "propeo";
    }
    //        if (scanner.hasNext()) {
    //            String ipt = scanner.next();
    //            if (StringUtils.isNotEmpty(ipt)) {
    //                return ipt;
    //            }
    //        }
    throw new MybatisPlusException("请输入正确的" + tip + "！");
}
```



