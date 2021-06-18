# Quarkus - 上下文和依赖注入

## 一、简介

### 1.1 什么是一个Bean

&emsp;&emsp;bean是一个容器托管对象，支持一组基本服务，例如注入依赖项，生命周期回调和拦截器。**Quarkus是根据JAVA2.0规范的上下文和依赖注入规范实现的。**



### 1.2 什么是容器管理

&emsp;&emsp;简单地说，程序不能直接控制对象实例的生命周期。而是，使用声明式方法（如注释、配置等）影响生命周期。容器是应用程序运行的环境。其创建并销毁Bean实例，将实例与制定上下文关联，并将它们注入其他豆类中。



### 1.3 依赖注入的好处

&emsp;&emsp;应用程序开发人员可以专注于业务逻辑，而不是找出"在哪里以及如何"获得其所有依赖关系的完全初始化组件。



> &emsp;&emsp;您可能听说过控制反转（IoC）变成原则的反转。依赖注入是IoC的实现技术之一。



### 1.4 简单Bean示例

```java
import javax.inject.Inject;
import javax.enterprise.context.ApplicationScoped;
import org.eclipse.microprofile.metrics.annotation.Counted;
//翻译
@ApplicationScoped 
public class Translator {

    @Inject
    Dictionary dictionary; //字典
		//翻译
    @Counted  
    String translate(String sentence) {
      // ...
    }
}
```



### 1.5 依赖注入的流程

#### 1.5.1 默认类型注入



#### 1.5.2 多个同类Bean注入

```java
public class Translator {

    @Inject//Instance 实现了 Iterable
    Instance<Dictionary> dictionaries; 

    String translate(String sentence) {
      for (Dictionary dict : dictionaries) { 
         // ...
      }
    }
}
```



#### 1.5.3 构造器注入和方法注入

```java
@ApplicationScoped
public class Translator {

    private final TranslatorHelper helper

    Translator(TranslatorHelper helper) { 
       this.helper = helper;
    }

    @Inject 
    void setDeps(Dictionary dic, LocalizationService locService) { 
      // ...
    }
}
```



#### 1.5.4 优先注入

```java
@Superior//此处会优先注入
@ApplicationScoped
public class SuperiorTranslator extends Translator {

    String translate(String sentence) {
      // ...
    }
}
```



### 1.6 Quarkus中的注解

| Annotation                                    | 详情                                                         |
| --------------------------------------------- | ------------------------------------------------------------ |
| `@javax.enterprise.context.ApplicationScoped` | 单例，懒加载                                                 |
| `@javax.inject.Singleton`                     | *暂时理解是需求处打上该注解会生成新的实例*                   |
| `@javax.enterprise.context.RequestScoped`     | 与请求相关联，一个请求一个实例                               |
| `@javax.enterprise.context.Dependent`         | 伪范围，一个需要一个实例                                     |
| `@javax.enterprise.context.SessionScoped`     | 此注解由对象支持。仅在使用扩展时可用。`javax.servlet.http.HttpSession``quarkus-undertow` |

> `quarkus`扩展可以提供其他自定义。例如：`quarkus-narayana-jtajavax.transaction.TransactionScoped`





### 1.7 Bean实例代理

&emsp;&emsp;客户端代理可以理解为一个方法委派类。真正的实例不会直接注入到使用处。

```java
@ApplicationScoped
class Translator {

    String translate(String sentence) {
      // ...
    }
}

// 类似于客户端代理实例
class Translator_ClientProxy extends Translator { 

    String translate(String sentence) {
      // 找到正确的转换器实例。。
      Translator translator = getTranslatorInstanceFromTheApplicationContext();
      // 委派方法调用
      return translator.translate(sentence);
    }
}
```

**客户代理带来的好处：**

- 懒加载-实例是在代理上调用方法后创建的。
- 能够将**“较窄”**的`Bean`注入到**“较宽”**的`Bean`当中。`@RequestScoped`到`@ApplicationScoped`
- 循环依赖
- 手动销毁Bean



### 1.8 生产Bean的几种方式

```java
@ApplicationScoped
public class Producers {

    @Produces//1、生产元数据
    double pi = Math.PI; 

    @Produces //2、根据方法生产
    List<String> names() {
       List<String> names = new ArrayList<>();
       names.add("Andy");
       names.add("Adalbert");
       names.add("Joachim");
       return names; 
    }
}

@ApplicationScoped//Bean类
public class Consumer {

   @Inject
   double pi;

   @Inject
   List<String> names;

   // ...
}
```



### 1.9 Bean的生命周期

```java
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

@ApplicationScoped
public class Translator {

    @PostConstruct//在Bean投入使用之前调用，执行一些初始化是安全的。
    void init() {
       // ...
    }

    @PreDestroy //在实例被销毁之前被调用，执行一些清理任务是安全的。
    void destroy() {
      // ...
    }
}
```



### 2.0 拦截器

&emsp;&emsp;拦截器用于将交叉问题与业务逻辑分开

```java
import javax.interceptor.Interceptor;
import javax.annotation.Priority;

@Logged //这是一个拦截器绑定注释，用于将我们的拦截器与Bean绑定。
@Priority(2020) //执行启动拦截器，并影响拦截器订购，数字越小优先级越高
@Interceptor //声明这是一个拦截器组建
public class LoggingInterceptor {

   @Inject
   Logger logger;

   @AroundInvoke//一种介于业务方法的方式
   Objec logInvocation(InvocationContext context) {
      // ...前日志
      Objec ret = context.proceed(); //转到拦截链中的下一个拦截器或调用截获的业务方法
      // ...后日志
      return ret;
   }

}
```

> 拦截器实例是它们拦截的Bean实例的依赖对象，即为每个截获的Bean创建一个新的拦截器实例。



### 2.1 事件和观察者

```java
class TaskCompleted {
  // ...
}

@ApplicationScoped
class ComplicatedService {

   @Inject
   Event<Task> event; //用于激发事件

   void doSomething() {
      // ...
      event.fire(new TaskCompleted()); //同步启动事件
   }

}

@ApplicationScoped
class Logger {
   void onTaskCompleted(@Observes TaskCompleted task) { //此方法在事件被激发时会被通知。TaskCompleted
      // ...log the task
   }

}
```







## 二、Quarkus依赖注入详解

&emsp;&emsp;`Quarkus DI`解决方案基于**Java2.0上下文和依赖注入规范**。但是，它不是`TCK`验证的完整`CDI`实现。只执行`CDI`功能的子集。



### 2.1 Bean发现

&emsp;&emsp;CDI中的Bean Discovery是一个复杂的过程，涉及底层模块架构的遗留部署结构和可访问性要求。但是，Quarkus正在使用简化的Bean Discovery。只有单个bean存档，Bean Discovery模式注释且没有可见性边界。



**Bean档案是合成的：**

- 应用`Classes`
- 包含`beans.xml`描述符的依赖项（已经不推荐）
- 包含`Jandex`索引的依赖项
- `Quarkus.Index`依赖于`application.properties`引用的依赖项
- 和`Quarkus`集成代码



&emsp;&emsp;没有打上注解的Bean将不会被发现。此行为由CDI定义。但是，即使声明类未注释使用bean定义注释，也会发现生产者方法和字段和观察方法（此行为与CDI中定义的内容不同）。实际上，声明bean类被认为是用@dependent注释的。

> &emsp;&emsp;`Quarkus` 定时任务可能会声明其他发现规则。例如，即使声明类未与bean定义注释，也会注册@scheduled业务方法。（即：类上不打注解，方法上的注解也会被扫描。）



### 2.2 如何生成Jandex索引

&emsp;&emsp;使用`Jandex`索引的依赖性自动扫描Bean。要生成索引只需将以下内容添加到pom.xml：

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.jboss.jandex</groupId>
      <artifactId>jandex-maven-plugin</artifactId>
      <version>1.0.7</version>
      <executions>
        <execution>
          <id>make-index</id>
          <goals>
            <goal>jandex</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

&emsp;&emsp;如果您无法修改依赖项，则仍然可以通过将`quarkus.index-dependency`项条目添加到`Application.properties`：

```properties
quarkus.index-dependency.<name>.group-id=
quarkus.index-dependency.<name>.artifact-id=
quarkus.index-dependency.<name>.classifier=(可选的)
```

&emsp;&emsp;例如，以下条目确保了org.acme：acme-api依赖项是索引的：



### 2.3 从发现中排除类型和依赖关系

**匹配方式：**

| 方式      | 例子           | 描述                     |
| --------- | -------------- | ------------------------ |
| 精确匹配  | `org.acme.Foo` | 匹配完全合格的类         |
| `*`号匹配 | `org.acme.*`   | 匹配此包下的所有类       |
| `**`匹配  | `org.acme.**`  | 匹配此包以及子包的所有类 |
| 类名匹配  | `Bar`          | 匹配为该名称的所有类     |



#### 2.3.1 示例一

```properties
quarkus.arc.exclude-types=org.acme.Foo,org.acme.*,Bar 
```

- 排除类型`org.acme.Foo`
- 将该包下的类排除`org.acme`
- 排除名为`Bar`的类



#### 2.3.2 示例二

&emsp;&emsp;用于排除对Bean扫描的依赖性工件。例如，因为它包含beans.xml描述符。

```properties
quarkus.arc.exclude-dependency.acme.group-id=org.acme 
quarkus.arc.exclude-dependency.acme.artifact-id=acme-services 
```

- 此组排除名为`ACME`
- 排除了`group-id:org.acme``artifact-id:acme-services `的依赖包



### 2.4 本机可执行文件和私有成员

&emsp;&emsp;`Quarkus`使用`GraalVM`构建一个可执行的本地版本，`Graalvm`的一个局限性是反射的使用。支持反射操作，但必须显式注册所有相关成员以进行反射。这些注册会导致更大的本机可执行文件。

&emsp;&emsp;如果`Quarkus DI`需要访问私有成员，则**必须使用反射**。这涉及注入字段，构造函数和初始化器，观察方法，生产者方法和字段，处理器和拦截器方法。

**如何避免使用私有成：**

```java
@ApplicationScoped
public class CounterBean {

    @Inject
    CounterService counterService; //包访问权限

    void onMessage(@Observes Event msg) {//包访问权限，构造器同理
    }
}
```



### 2.5 支持的功能

- 编程模型
  - 由Java类实现的托管Bean
    - `@PostConstruct`和`@PreDestroy`生命周期回调。
  - 生产者方法和领域，处理器
  - 限定符`Qualifiers`
  - 选择`Alternatives`
  - 定型`Stereotypes`
- 依赖注入和查找
  - 域、构造方法和 初始化器/setter方法 注入
  - 类型安全分辨率`Type-safe resolution`
  - 通过`javax.enterprise.Inject.Instance`进行编程查找
  - 类代理
  - 注入元数据
- 范围和上下文
  - `@Dependent`, `@ApplicationScoped`, `@Singleton`, `@RequestScoped` and `@SessionScoped`
  - 自定义范围和上下文
- 拦截器
  - 业务方法拦截器：`@AroundInvoke`
  - 生命周期事件回调的拦截器：`@PostConstruct`, `@PreDestroy`, `@AroundConstruct`
  - 事件和观察者方法，包括异步事件和事务观察者方法





### 2.6 限制

- `@ConversationScoped`不支持
- 不支持装饰器
- 不支持便携式扩展
- `BeanManager` - 仅实施以下方法: `getBeans()`, `createCreationalContext()`, `getReference()`, `getInjectableReference()` , `resolve()`, `getContext()`, `fireEvent()`, `getEvent()` 和 `createInstance()`.
- 专业化不受支持
- `beans.xml`描述符内容被忽略
- 不支持钝化和钝化的范围
- `interctiveor`在超类上的方法尚未实现
- 不支持`@Interceptors`



### 2.7 非标准功能

#### 2.7.1 Bean类实例化

##### 2.7.1.1 默认懒加载

&emsp;&emsp;默认情况下，`CDI Bean`是在需要时延迟创建的。“needed”的确切含义取决于`bean`的范围。

- 在注入的实例上调用方法时，需要**正常的范围Bean**`（@ApplicationScoped，@RequestScoped等）`（规范的上下文参考）。

  换句话说，注入正常的范围bean，因为注入客户端代理而不是bean的上下文实例。

- 注入时创建具有**伪范围**`（@dependent和@singleton）`的bean

```java
@Singleton // => 伪范围
class AmazingService {
  String ping() {
    return "amazing";
  }
}

@ApplicationScoped // => 范围
class CoolService {
  String ping() {
    return "cool";
  }
}

@Path("/ping")
public class PingResource {

  @Inject
  AmazingService s1;//在使用的瞬间出发

  @Inject
  CoolService s2; //注射本身不会导致CoolService的实例化。注入客户端代理。

  @GET
  public String ping() {
    return s1.ping() + s2.ping(); //s2在第一次调用时触发
  }
}
```



##### 2.7.1.2 启动事件

&emsp;&emsp;但是，如果您确实需要急切地实例化bean，您可以：

- 声明`startupevent`的观察者 - 在这种情况下，bean的范围无关紧要：

  ```java
  @ApplicationScoped
  class CoolService {
    void startup(@Observes StartupEvent event) { 
    }
  }
  ```

  - 在启动过程中创建一个CoolService来为observer方法调用提供服务。

- 在startupEvent的观察者中使用bean - 必须使用正常范围的bean，如懒惰默认情况下使用：

  ```java
  @Dependent
  class MyBeanStarter {
  
    void startup(@Observes StartupEvent event, AmazingService amazing, CoolService cool) { 
      cool.toString(); 
    }
  }
  ```

  - `AmazingService`是在注入期间创建的。
  - `CoolService`是一个**正常的范围Bean**，因此它会在第一次调用方法时实例化。

- 使用`@io.quarkus.runtime.startup`注释bean，如`startup`注解中所述：

  ```java
  @Startup 
  @ApplicationScoped
  public class EagerAppBean {
  
     private final String name;
  
     EagerAppBean(NameGenerator generator) { 
       this.name = generator.createName();
     }
  }
  ```

  - 对于使用`@startup`注释的每个bean，生成了`startupevent`的合成观察者。使用默认优先级。
  - 当应用程序启动时调用Bean构造函数，并且生成的上下文实例存储在应用程序上下文中。

  > &emsp;&emsp;如应用程序初始化和终止指南中所述，鼓励`Quarkus`用户始终更喜欢`@observes startupevent`来`@initialized（applicationscoped.class）`。





#### 2.7.2 请求上下文声明周期

&emsp;&emsp;请求上下文也处于活动状态：

- 在通知同步观察者方法期间。

&emsp;&emsp;请求上下文已销毁：

- 事件的观察者通知完成后，如果在通知启动时该事件尚未处于活动状态。

> &emsp;&emsp;当为观察者通知初始化请求上下文时，将激发限定符为`@Initialized（RequestScoped.class）`的事件。此外，当请求上下文被销毁时，将触发带有限定符`@beforedommused（RequestScoped.class）`和`@destromed（RequestScoped.class）`的事件。



#### 2.7.3 配置文件注入域

&emsp;&emsp;在`CDI`中，如果声明现场注入点，则需要使用`@inject`和可选的一组限定符。

```java
@Inject
@ConfigProperty(name = "cool")
String coolProperty;
```

&emsp;&emsp;&emsp;在`Quarkus`中，如果注入的字段声明至少一个限定符，则可以完全跳过`@Inject`注释。

```java
@ConfigProperty(name = "cool")
String coolProperty;
```

> &emsp;&emsp;对于下面讨论的一个特殊情况的显着例外，仍然需要`@Inject`仍然需要构造函数和方法注入。



#### 2.7.4 简化构造器注入

&emsp;&emsp;在`CDI`中，一个**正常的范围bean**必须始终声明`No-Args`构造函数（除非您声明任何其他构造函数，否则该构造函数通常由编译器生成）。但是，这一要求使构造函数注射复杂化 - 您需要提供一个虚拟的No-Args构造函数，以使事物在CDI中工作。

```java
@ApplicationScoped
public class MyCoolService {

  private SimpleProcessor processor;

  MyCoolService() { // 需要虚拟构造函数
  }

  @Inject // 构造器注入
  MyCoolService(SimpleProcessor processor) {
    this.processor = processor;
  }
}
```

&emsp;&emsp;无需在`Quarkus`中声明正常范围Bean的**伪构造函数** - 它们是自动生成的。此外，如果只有一个构造函数，则无需`@Inject`。

```java
@ApplicationScoped
public class MyCoolService {

  private SimpleProcessor processor;

  MyCoolService(SimpleProcessor processor) {
    this.processor = processor;
  }
}
```

> &emsp;&emsp;如果Bean类扩展了未声明No-Args构造函数的类，则我们不会自动生成无args构造函数。



#### 2.7.5 删除未使用的bean

&emsp;&emsp;容器在构建过程中尝试删除所有未使用的bean。可以通过设置禁用此优化：`quarkus.arc.remove-unused-beans`:`none` or `false`。

**未使用的bean：**

- 不是内置bean或拦截器;
- 没有资格注射到任何注射点;
- 不排除任何扩展;
- 没有名字;
- 不声明观察员;
- 不申报任何有资格向任何注射点注射的生产商;
- 不直接符合注射到任何javax.enterprise.inject.instance或javax.inject.provider注入点



&emsp;&emsp;**此优化适用于所有形式的Bean声明：**`Bean`类，`Producer`方法，`Producer`字段。

&emsp;&emsp;用户可以通过使用`io.quarkus.arc.Unremovable`对容器进行注释来指示容器不要删除任何特定的bean（即使它们满足上面指定的所有规则）。此注释可以放置在类型、生产者方法和生产者字段上。



**也可以通过`application.properties`：**

| 方式      | 例子           | 描述                     |
| --------- | -------------- | ------------------------ |
| 精确匹配  | `org.acme.Foo` | 匹配完全合格的类         |
| `*`号匹配 | `org.acme.*`   | 匹配此包下的所有类       |
| `**`匹配  | `org.acme.**`  | 匹配此包以及子包的所有类 |
| 类名匹配  | `Bar`          | 匹配为该名称的所有类     |

**示例：**

```properties
quarkus.arc.unremovable-types=org.acme.Foo,org.acme.*,Bar
```

&emsp;&emsp;使用开发模式（运行` ./mvnw clean compile quarkus:dev`）时，您可以通过 `application.properties` 中的以下行启用附加日志记录来查看有关正在删除哪些 `bean` 的更多信息。

```properties
quarkus.log.category."io.quarkus.arc.processor".level=DEBUG
```



#### 2.7.6 默认Bean

&emsp;&emsp;`Quarkus`添加了`CDI`目前不支持的一个功能，即如果没有其他具有相同类型和限定符的`bean`通过任何可用的方式（bean类、生产者、合成bean…) 注入那么便使用该默认类，这是使用`@io.quarkus.arc.DefaultBean`注释完成的。

**示例：**

```java
@Dependent
public class TracerConfiguration {

    @Produces
    public Tracer tracer(Reporter reporter, Configuration configuration) {
        return new Tracer(reporter, configuration);
    }

    @Produces
    @DefaultBean
    public Configuration configuration() {
        // 创建配置
    }

    @Produces
    @DefaultBean
    public Reporter reporter(){
        // 创建报告者
    }
}
```



&emsp;&emsp;其思想是扩展为用户自动配置东西，此时如果不打算使用默认配置即可以采用下面的方式：

```java
@Dependent
public class CustomTracerConfiguration {

    @Produces
    public Reporter reporter(){
        // 创建自定Reporter
    }
}
```



#### 2.7.7 根据环境文件注入

&emsp;&emsp;`Quarkus`添加了`CDI`当前不支持的功能，即通过`@io.Quarkus.arc.profile.IfBuildProfile`和`@io.Quarkus.arc.profile.UnlessBuildProfile`注释，在启用`Quarkus`构建时配置文件时**有条件地启用bean**。当与`@io.quarkus.arc.DefaultBean`一起使用时，这些注释允许为不同的构建概要文件创建不同的bean配置。

&emsp;&emsp;例如，假设一个应用程序包含一个名为`Tracer`的`bean`，它**在测试或开发模式下不需要执行任何操作**，但在生产环境的正常容量下工作。创建此类`bean`的优雅方法如下：

```java
@Dependent
public class TracerConfiguration {

    @Produces
    @IfBuildProfile("prod")
    public Tracer realTracer(Reporter reporter, Configuration configuration) {
        return new RealTracer(reporter, configuration);
    }

    @Produces
    @DefaultBean
    public Tracer noopTracer() {
        return new NoopTracer();
    }
}
```



&emsp;&emsp;相反，如果要求`Tracer Bean`也在`dev`模式下工作，并且只默认不做任何测试，那么`@UnlessBuildProfile`将是理想的选择。代码如下所示：

```java
@Dependent
public class TracerConfiguration {

    @Produces
    @UnlessBuildProfile("test") // this will be enabled for both prod and dev build time profiles
    public Tracer realTracer(Reporter reporter, Configuration configuration) {
        return new RealTracer(reporter, configuration);
    }

    @Produces
    @DefaultBean
    public Tracer noopTracer() {
        return new NoopTracer();
    }
}
```

> &emsp;&emsp;运行时概要文件对使用`@IfBuildProfile`和`@UnlessBuildProfile`的`bean`解析绝对没有影响。



#### 2.7.8 根据配置文件注入

&emsp;&emsp;`Quarkus`添加了`CDI`当前不支持的功能，即通过`@io.Quarkus.arc.properties.IfBuildProperty`注释，在`Quarkus`构建时属性具有特定值时**有条件地启用Bean**。当与`@io.quarkus.arc.DefaultBean`一起使用时，此注释允许为不同的构建属性**创建不同的Bean配置**。

&emsp;&emsp;我们上面提到的`Tracer`场景也可以通过以下方式实现：

```java
@Dependent
public class TracerConfiguration {

    @Produces
    @IfBuildProperty(name = "some.tracer.enabled", stringValue = "true")
    public Tracer realTracer(Reporter reporter, Configuration configuration) {
        return new RealTracer(reporter, configuration);
    }

    @Produces
    @DefaultBean
    public Tracer noopTracer() {
        return new NoopTracer();
    }
}
```

> 在运行时设置的属性对使用`@IfBuildProperty`的`bean`解析绝对没有影响。



#### 2.7.9 声明选定的备选方案

&emsp;&emsp;在`CDI`中，可以通过`@Priority`全局地为应用程序选择**备用Bean**，或者使用`beans.xml`描述符为**`bean`存档**选择**备用`bean`**。`Quarkus`有一个简化的`bean`发现，`beans.xml`的内容被忽略(**推崇使用配置类**)。

&emsp;&emsp;`@Priority`的缺点是它有`@Target({TYPE,PARAMETER})`，因此不能用于`producer`方法和字段。为了解决这个问题并简化代码，`Quarkus`提供了`io.Quarkus.arc.AlternativePriority`注解。它基本上是`@Alternative plus@Priority`的捷径。此外，它还可用于`Bean`生产商。

&emsp;&emsp;但是，也可以使用统一配置为应用程序选择替代方案。`quarkus.arc.selected-alternatives`属性接受用于匹配备用`bean`的字符串值列表。如果有任何值匹配，那么`Integer#MAX_VALUE`的优先级**将用于相关`bean`**。通过`@priority`或`@AlternativePriority`声明的优先级将被重写。

**示例：**

| 方式      | 例子           | 描述                     |
| --------- | -------------- | ------------------------ |
| 精确匹配  | `org.acme.Foo` | 匹配完全合格的类         |
| `*`号匹配 | `org.acme.*`   | 匹配此包下的所有类       |
| `**`匹配  | `org.acme.**`  | 匹配此包以及子包的所有类 |
| 类名匹配  | `Bar`          | 匹配为该名称的所有类     |

**application.properties示例：**

```properties
quarkus.arc.selected-alternatives=org.acme.Foo,org.acme.*,Bar
```



#### 2.7.10 简化生产者方法声明

&emsp;&emsp;在CDI中，生产者方法必须始终用`@products`注释。

```java
class Producers {

  @Inject
  @ConfigProperty(name = "cool")
  String coolProperty;

  @Produces
  @ApplicationScoped
  MyService produceService() {
    return new MyService(coolProperty);
  }
}
```

&emsp;&emsp;在`Quarkus`中，如果生产者方法使用范围注释、原型或限定符进行注释，则可以完全跳过`@products`注释。

```java
class Producers {

  @ConfigProperty(name = "cool")
  String coolProperty;

  @ApplicationScoped
  MyService produceService() {
    return new MyService(coolProperty);
  }
}
```



#### 2.7.11 静态方法拦截

&emsp;&emsp;拦截器规范很清楚，`around invoke`方法不能声明为静态的。然而，这种限制主要是由技术限制造成的。由于Quarkus是一个**面向构建时的堆栈**，允许**额外的类转换**，这些限制不再适用。可以使用拦截器绑定对非私有静态方法进行注释：

```java
class Services {

  @Logged//一个拦截器的绑定
  static BigDecimal computePrice(long amout) { //如果存在与日志相关联的拦截器，则会拦截每个方法调用。
    BigDecimal price;
    // 执行操作。。。
    return price;
  }
}
```



##### 2.7.11.1 局限性

- 出于向后兼容性的原因，只考虑方法级绑定（否则，声明类级绑定的`bean`类的静态方法将突然被截获）。
- 私有静态方法永远不会被截获。
- 在拦截静态方法时，并非所有现有的拦截器都可以正常工作，例如：`InvocationContext#getTarget()` 返回`null`。

> 拦截器可以使用`InvocationContext.getMethod()`检测静态方法并相应地调整行为。



#### 2.7.12 处理final类和方法

&emsp;&emsp;在普通`CDI`中，标记为`final`和`/`或具有`final`方法的类不适合创建代理，这反过来意味着拦截器和普通作用域`bean`不能正常工作。当尝试将`CDI`与其他`JVM`语言（如`Kotlin`）结合使用时，这种情况非常常见，默认情况下，类和方法就是`final`的。

&emsp;&emsp;我们需要将`Quarkus.arc.transform-unproyable-classes`设置为`true`（这是默认值），让`Quarkus`克服这些限制。



#### 2.7.13 容器管理并发

&emsp;&emsp;`CDI Bean`没有标准的并发控制机制。然而，**`Bean`实例**可以从多个线程共享和并发访问。在这种情况下，它应该是线程安全的。您可以使用标准的Java构造（`volatile`、`synchronized`、`ReadWriteLock`等），或者让容器控制并发访问。`Quarkus`为此拦截器绑定提供`@io.Quarkus.arc.Lock`和内置拦截器。与**被拦截`bean`的上下文实例关联的每个拦截器实例都拥有一个单独的`ReadWriteLock`，其中包含非公平排序策略**。

> &emsp;&emsp;`io.quarkus.arc.Lock`是一个常规的拦截器绑定，因此可以用于任何范围的任何`Bean`。但是，它对于“共享”作用域特别有用，例如`@Singleton`和`@ApplicationScoped`。

```java
import io.quarkus.arc.Lock;

@Lock 
@ApplicationScoped
class SharedService {

  void addAmount(BigDecimal amout) {
    // ...改变bean的一些内部状态
  }

  @Lock(value = Lock.Type.READ, time = 1, unit = TimeUnit.SECONDS)  
  BigDecimal getAmount() {
    // ...同时读取值是安全的
  }
}
```

- 类上声明的@`Lock`（映射到`(@LockLock.Type.WRITE)`）指示容器**锁定`Bean`实例**，以进行任何业务方法的调用，即客户端具有“独占访问”，不允许并发调用。
- `@Lock(Lock.Type.READ)`重写在类级别指定的值。这意味着任何数量的客户机都可以同时调用该方法，除非**`Bean`实例**被`@Lock(Lock.Type.WRITE)`锁定。
- 您还可以指定“等待时间”。如果无法在给定时间内获取锁，则会引发`LockException`。



### 2.8 构建时扩展

#### 2.8.1 可移植扩展

&emsp;&emsp;`Quarkus`整合了构建时优化，以提供即时启动和低内存占用。这种方法的缺点是无法支持`CDI`可移植扩展。不过，大多数功能都可以使用`Quarkus`扩展实现。



#### 2.8.2 扩展默认Bean定义注解集

&emsp;&emsp;如**`Bean`发现(2.1)**中所述，不发现没有`Bean`定义注释的`Bean`类。但是，`BeanDefiningAnnotationBuildItem`可用于扩展默认的`bean`定义注释集（`@Dependent`、`@Singleton`、`@ApplicationScoped`、`@RequestScoped`和`@Stereotype`注释）：

```java
@BuildStep
BeanDefiningAnnotationBuildItem additionalBeanDefiningAnnotation() {
    return new BeanDefiningAnnotationBuildItem(DotName.createSimple("javax.ws.rs.Path")));
}
```

> &emsp;&emsp;默认情况下，作为`BeanDefiningAnnotationBuildItem`结果的`Bean`注册是不可移除的。另请参见移除未使用的`bean`。



#### 2.8.3 资源注解

&emsp;&emsp;`ResourceAnnotationBuildItem`用于指定可以解析非CDI注入点（如JavaEE资源）的资源注释。

> &emsp;&emsp;集成商还必须提供相应的 `io.quarkus.arc.ResourceReferenceProvider` 实现。

```java
@BuildStep
void setupResourceInjection(BuildProducer<ResourceAnnotationBuildItem> resourceAnnotations, BuildProducer<GeneratedResourceBuildItem> resources) {
    resources.produce(new GeneratedResourceBuildItem("META-INF/services/io.quarkus.arc.ResourceReferenceProvider",
        JPAResourceReferenceProvider.class.getName().getBytes()));
    resourceAnnotations.produce(new ResourceAnnotationBuildItem(DotName.createSimple(PersistenceContext.class.getName())));
}
```



#### 2.8.4 附加Beans

&emsp;&emsp;`AdditionalBeanBuildItem` 用于指定在发现期间要分析的其他 `bean` 类。额外的 `bean` 类被透明地添加到容器处理的应用程序索引中。

```java
@BuildStep
List<AdditionalBeanBuildItem> additionalBeans() {
     return Arrays.asList(
          new AdditionalBeanBuildItem(SmallRyeHealthReporter.class),
          new AdditionalBeanBuildItem(HealthServlet.class));
}
```

> &emsp;&emsp;默认情况下，作为 `AdditionalBeanBuildItem` 结果的 `bean` 注册是可移除的。另请参阅删除未使用的 `Bean`。



#### 2.8.5 合成Beans

&emsp;&emsp;有时，能够注册一个**合成 `bean`** 是非常有用的。合成 `bean` 的 `Bean` 属性不是从 `java` 类、方法或字段派生的。相反，属性**由扩展指定**。在 `CDI` 中，这可以使用 `AfterBeanDiscovery.addBean()` 方法来实现。在 `Quarkus` 中，可以通过三种方式注册合成 `bean`。



##### 2.8.5.1 BeanRegistrarBuildItem

&emsp;&emsp;构建步骤可以生成 `BeanRegistrarBuildItem` 并利用 `io.quarkus.arc.processor.BeanConfigurator` API来构建合成 `bean` 定义。

```java
@BuildStep
BeanRegistrarBuildItem syntheticBean() {
     return new BeanRegistrarBuildItem(new BeanRegistrar() {

            @Override
            public void register(RegistrationContext registrationContext) {
                 registrationContext.configure(String.class).types(String.class).qualifiers(new MyQualifierLiteral()).creator(mc -> mc.returnValue(mc.load("foo"))).done();
            }
        }));
}
```

> - `BeanConfigurator` 的输出被记录为字节码。因此，在如何创建合成 `bean` 实例方面存在一些限制。另请参阅 `BeanConfigurator.creator()` 方法。
> - 可以通过从 `RegistrationContext.beans()` 方法返回的方便的 `BeanStream` 轻松**过滤所有基于类的 `bean`**。



##### 2.8.5.2 BeanRegistrationPhaseBuildItem

&emsp;&emsp;如果构建步骤需要在注册期间生成其他构建项，则应使用 `BeanRegistrationPhaseBuildItem`。原因是注入的对象仅在 `@BuildStep` 方法调用期间有效。

```java
@BuildStep
void syntheticBean(BeanRegistrationPhaseBuildItem beanRegistrationPhase,
            BuildProducer<MyBuildItem> myBuildItem,
            BuildProducer<BeanConfiguratorBuildItem> beanConfigurators) {
   beanConfigurators.produce(new BeanConfiguratorBuildItem(beanRegistrationPhase.getContext().configure(String.class).types(String.class).qualifiers(new MyQualifierLiteral()).creator(mc -> mc.returnValue(mc.load("foo")))));
   myBuildItem.produce(new MyBuildItem());
}
```

> 有关更多信息，请参阅 `BeanRegistrationPhaseBuildItem` **javadoc**。



##### 2.8.5.3 

&emsp;&emsp;最后，构建步骤可以生成 `SyntheticBeanBuildItem` 来注册合成 `bean`，其实例可以通过记录器轻松生成。扩展的 `BeanConfigurator` 接受 `io.quarkus.runtime.RuntimeValue` 或 `java.util.function.Supplier`。

```java
@BuildStep
@Record(STATIC_INIT)
SyntheticBeanBuildItem syntheticBean(TestRecorder recorder) {
   return SyntheticBeanBuildItem.configure(Foo.class).scope(Singleton.class)
                .runtimeValue(recorder.createFoo())
                .done();
}
```



#### 2.8.6 注解转换

&emsp;&emsp;一个非常常见的任务是覆盖在 `bean` 类上找到的注释。例如，您可能希望将拦截器绑定添加到特定的 `bean` 类。这是如何做到的 - 使用 `AnnotationsTransformerBuildItem`：

```java
@BuildStep
AnnotationsTransformerBuildItem transform() {
    return new AnnotationsTransformerBuildItem(new AnnotationsTransformer() {

        public boolean appliesTo(org.jboss.jandex.AnnotationTarget.Kind kind) {
            return kind == org.jboss.jandex.AnnotationTarget.Kind.CLASS;
        }

        public void transform(TransformationContext context) {
            if (context.getTarget().asClass().name().toString().equals("com.foo.Bar")) {
                context.transform().add(MyInterceptorBinding.class).done();
            }
        }
    });
}
```



#### 2.8.7 额外的拦截器绑定

&emsp;&emsp;在极少数情况下，以编程方式将现有注释注册为拦截器绑定可能会很方便。这类似于纯 `CDI` 通过 `BeforeBeanDiscovery#addInterceptorBinding() `实现的。虽然在这里我们将使用` InterceptorBindingRegistrarBuildItem` 来完成它。请注意，您可以一次性注册多个注释：

```java
@BuildStep
InterceptorBindingRegistrarBuildItem addInterceptorBindings() {
    InterceptorBindingRegistrarBuildItem additionalBindingsRegistrar = new InterceptorBindingRegistrarBuildItem(new InterceptorBindingRegistrar() {
        @Override
        public Collection<DotName> registerAdditionalBindings() {
            Collection<DotName> result = new HashSet<>();
            result.add(DotName.createSimple(MyAnnotation.class.getName()));
            result.add(DotName.createSimple(MyOtherAnnotation.class.getName()));
            return result;
        }
    });
    return additionalBindingsRegistrar;
}
```



#### 2.8.8 注入点转换

&emsp;&emsp;时不时地以编程方式更改注入点的限定符是很方便的。您可以使用`InjectionPointTransformerBuildItem`实现这一点。以下示例演示如何将转换应用于包含限定符`MyQualifier`的`Foo`类型的注入点：

```java
@BuildStep
InjectionPointTransformerBuildItem transformer() {
    return new InjectionPointTransformerBuildItem(new InjectionPointsTransformer() {

        public boolean appliesTo(Type requiredType) {
            return requiredType.name().equals(DotName.createSimple(Foo.class.getName()));
        }

        public void transform(TransformationContext context) {
            if (context.getQualifiers().stream()
                    .anyMatch(a -> a.name().equals(DotName.createSimple(MyQualifier.class.getName())))) {
                context.transform()
                        .removeAll()
                        .add(DotName.createSimple(MyOtherQualifier.class.getName()))
                        .done();
            }
        }
    });
}
```



#### 2.8.9 观察者变化

&emsp;&emsp;任何观察者方法定义都可以被否决或使用`ObserverTransformerBuildItem`进行转换。可以转换的属性包括：

- `qualifiers`
-  `reception`
-  `priority`
-  `transaction phase`
- `asynchronous`

```java
@BuildStep
ObserverTransformerBuildItem transformer() {
    return new ObserverTransformerBuildItem(new ObserverTransformer() {

        public boolean appliesTo(Type observedType, Set<AnnotationInstance> qualifiers) {
            return observedType.name.equals(DotName.createSimple(MyEvent.class.getName()));
        }

        public void transform(TransformationContext context) {
            // 否决MyEvent的所有观察员
            context.veto();
        }
    });
}
```



#### 2.8.10 Bean部署验证

&emsp;&emsp;一旦`bean`部署就绪，扩展就可以执行额外的验证并检查找到的`bean`、观察者和注入点。注册`BeanDeploymentValidatorBuildItem`:

```java
@BuildStep
BeanDeploymentValidatorBuildItem beanDeploymentValidator() {
    return new BeanDeploymentValidatorBuildItem(new BeanDeploymentValidator() {
         public void validate(ValidationContext validationContext) {
             for (InjectionPointInfo injectionPoint : validationContext.get(Key.INJECTION_POINTS)) {
                 System.out.println("Injection point: " + injectionPoint);
             }
         }
    });
}
```

> &emsp;可以通过`ValidationContext.beans()`方法返回的`BeanStream`方便地过滤所有注册的`bean`。



&emsp;&emsp;如果扩展需要在“验证”阶段生成其他生成项，则应该改用`ValidationPhaseBuildItem`。原因是注入的对象仅在`@BuildStep`方法调用期间有效。

```java
@BuildStep
void validate(ValidationPhaseBuildItem validationPhase,
            BuildProducer<MyBuildItem> myBuildItem,
            BuildProducer<ValidationErrorBuildItem> errors) {
   if (someCondition) {
     errors.produce(new ValidationErrorBuildItem(new IllegalStateException()));
     myBuildItem.produce(new MyBuildItem());
   }
}
```

>&emsp;&emsp;有关更多信息，请参阅`ValidationPhaseBuildItem` javadoc。



#### 2.8.11 自定义上下文

&emsp;&emsp;扩展可以通过`ContextRegistrarBuildItem`注册自定义`InjectableContext`实现：

```java
@BuildStep
ContextRegistrarBuildItem customContext() {
    return new ContextRegistrarBuildItem(new ContextRegistrar() {
         public void register(RegistrationContext registrationContext) {
            registrationContext.configure(CustomScoped.class).normal().contextClass(MyCustomContext.class).done();
         }
    });
}
```

&emsp;&emsp;如果扩展需要在“上下文注册”阶段生成其他生成项，则应该改用`ContextRegistrationPhaseBuildItem`。原因是注入的对象仅在`@BuildStep`方法调用期间有效。

```java
@BuildStep
void addContext(ContextRegistrationPhaseBuildItem contextRegistrationPhase,
            BuildProducer<MyBuildItem> myBuildItem,
            BuildProducer<ContextConfiguratorBuildItem> contexts) {
   contexts.produce(new ContextConfiguratorBuildItem(contextRegistrationPhase.getContext().configure(CustomScoped.class).normal().contextClass(MyCustomContext.class)));
   myBuildItem.produce(new MyBuildItem());
}
```

> 有关详细信息，请参阅`ContextRegistrationPhaseBuildItem` javadoc。



#### 2.8.12 可用的生成时元数据

&emsp;&emsp;使用`BuildExtension.BuildContext`操作的上述任何扩展都可以利用生成期间生成的特定生成时元数据。`io.quarkus.arc.processor.BuildExtension.Key`中的内置密钥是：

- `ANNOTATION_STORE`
  - 包含一个AnnotationStore，它在应用注释转换器后保留有关所有AnnotationTarget注释的信息
- `INJECTION_POINTS`
  - `Collection<InjectionPointInfo>` 包含所有注入点
- `BEANS`
  - `Collection<BeanInfo>` 包含所有Bean
- `REMOVED_BEANS`
  - `Collection<BeanInfo>` 包含所有移除的Beans；有关详细信息，请参阅删除未使用的bean
- `OBSERVERS`
  - `Collection<ObserverInfo>` 包含所有观察者
- `SCOPES`
  - `Collection<ScopeInfo>` 包含所有作用域，包括自定义作用域
- `QUALIFIERS`
  - `Map<DotName, ClassInfo>` 包含所有限定符
- `INTERCEPTOR_BINDINGS`
  - `Map<DotName, ClassInfo>` 包含所有拦截器绑定
- `STEREOTYPES`
  - `Map<DotName, ClassInfo>` 包含所有刻板印象

&emsp;&emsp;要获得这些，只需查询给定键的扩展上下文对象。请注意，这些元数据在生成过程中可用，这意味着扩展只能利用在调用之前生成的元数据。如果扩展尝试检索尚未生成的元数据，则将返回null。以下是哪些扩展可以访问哪些元数据的摘要：

- `AnnotationsTransformer`
  - 不应该依赖任何元数据，因为这是调用的第一个CDI扩展之一
- `ContextRegistrar`
  - 有权访问 `ANNOTATION_STORE`
- `InjectionPointsTransformer`
  - 有权访问 `ANNOTATION_STORE`, `QUALIFIERS`, `INTERCEPTOR_BINDINGS`, `STEREOTYPES`
- `ObserverTransformer`
  - 有权访问 `ANNOTATION_STORE`, `QUALIFIERS`, `INTERCEPTOR_BINDINGS`, `STEREOTYPES`
- `BeanRegistrar`
  - 有权访问 `ANNOTATION_STORE`, `QUALIFIERS`, `INTERCEPTOR_BINDINGS`, `STEREOTYPES`, `BEANS`
- `BeanDeploymentValidator`
  - 有权访问所有生成元数据



### 2.9 发展模式

&emsp;&emsp;在开发模式中，会自动注册两个特殊端点，以提供JSON格式的一些基本调试信息：

- HTTP GET `/quarkus/arc/beans` - 返回所有bean的列表

  - 可以使用查询参数筛选输出：

    - `scope` - 包含范围以给定值结尾的bean，即。

      `http://localhost:8080/quarkus/arc/beans?scope=ApplicationScoped`

    - `beanClass` -包含以给定值开头的bean类，即

       `http://localhost:8080/quarkus/arc/beans?beanClass=org.acme.Foo`

    - `kind` - 包括指定类型的bean (`CLASS`, `PRODUCER_FIELD`, `PRODUCER_METHOD`, `INTERCEPTOR` or `SYNTHETIC`), i.e. `http://localhost:8080/quarkus/arc/beans?kind=PRODUCER_METHOD`

- HTTP GET `/quarkus/arc/observers` - 返回所有观察者方法的列表

>这些端点仅在开发模式下可用，即通过mvn运行应用程序时`quarkus:dev` （或`./gradlew quarkusDev`）。





### 2.10 ArC配置参考