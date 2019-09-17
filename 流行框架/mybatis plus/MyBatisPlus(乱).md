# 知识梳理补充

## 1、mybatis pilus介绍

> 再对mybatis没有更改的情况下，增强其功能。

## 2、一些配置（需要梳理）

```
存放一些全局配置的类：GlobalConfig
mp配置类：MybatisSqlSessionFactoryBean
```

![image-20190525152550253](/Users/yiming/Library/Application Support/typora-user-images/image-20190525152550253.png)



```yaml
mybatis-plus:
  #外部化xml配置
  #config-location: classpath:mybatis-config.xml
  #指定外部化 MyBatis Properties 配置，通过该配置可以抽离配置，实现不同环境的配置部署
  #configuration-properties: classpath:mybatis/config.properties
  #xml扫描，多个目录用逗号或者分号分隔（告诉 Mapper 所对应的 XML 文件位置）
  mapper-locations: classpath*:/mapper/*.xml
  #MyBaits 别名包扫描路径，通过该属性可以给包中的类注册别名
  type-aliases-package: net.xinhuamm.noah.api.model.entity,net.xinhuamm.noah.api.model.dto
  #如果配置了该属性，则仅仅会扫描路径下以该类作为父类的域对象
  #type-aliases-super-type: java.lang.Object
  #枚举类 扫描路径，如果配置了该属性，会将路径下的枚举类进行注入，让实体类字段能够简单快捷的使用枚举属性
  #type-enums-package: com.baomidou.mybatisplus.samples.quickstart.enums
  #项目启动会检查xml配置存在(只在开发时候打开)
  check-config-location: true
  #SIMPLE：该执行器类型不做特殊的事情，为每个语句的执行创建一个新的预处理语句,REUSE：该执行器类型会复用预处理语句,BATCH：该执行器类型会批量执行所有的更新语句
  default-executor-type: REUSE
  configuration:
    # 是否开启自动驼峰命名规则（camel case）映射，即从经典数据库列名 A_COLUMN（下划线命名） 到经典 Java 属性名 aColumn（驼峰命名） 的类似映射
    map-underscore-to-camel-case: false
    # 全局地开启或关闭配置文件中的所有映射器已经配置的任何缓存，默认为 true
    cache-enabled: false
    #懒加载
    #aggressive-lazy-loading: true
    #NONE：不启用自动映射 PARTIAL：只对非嵌套的 resultMap 进行自动映射 FULL：对所有的 resultMap 都进行自动映射
    #auto-mapping-behavior: partial
    #NONE：不做任何处理 (默认值)WARNING：以日志的形式打印相关警告信息 FAILING：当作映射失败处理，并抛出异常和详细信息
    #auto-mapping-unknown-column-behavior: none
    #如果查询结果中包含空值的列，则 MyBatis 在映射的时候，不会映射这个字段
    call-setters-on-nulls: true
    # 这个配置会将执行的sql打印出来，在开发或测试的时候可以用
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    db-config:
      #表名下划线命名默认true
      table-underline: true
      #id类型
      id-type: auto
      #是否开启大写命名，默认不开启
      #capital-mode: false
      #逻辑已删除值,(逻辑删除下有效) 需要注入逻辑策略LogicSqlInjector 以@Bean方式注入
      logic-not-delete-value: 0
      #逻辑未删除值,(逻辑删除下有效)
      logic-delete-value: 1
      #数据库类型
      db-type: sql_server
```

## 3、一些与mybatis的对比说明

### 3.1 插入数据并返回主键值

![image-20190525170437913](/Users/yiming/Desktop/持久层相关知识/mybatis plus img/image-20190525170437913.png)

> mybatis需要进行一些设置，而mp呢会自动的将主键值写回对象

## 4、打印出sql语句，设置日志级别为debug£

```yaml
logging: #不添加logging启动器也可以
  level:
    club.maddm.mptest: debug #设置dao层的日志级别
```

