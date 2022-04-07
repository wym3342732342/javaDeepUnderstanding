# Java Modularity

&emsp;&emsp;从`JDK9`开始，`module`成为与`class`，`interface`，`package`等同等重要的一等公民，是(需要成为)`Javaers`日常高频处理的词汇。`Java`的`modularity`起源于2008年的`Jigsaw`项目，并从2014年开始在`JDK9`的开发过程中设计并实现。

&emsp;&emsp;引入`module`不像引入像`lambada`表达式**仅是语法的改变**，它涉及到了`JLS（java language specification）`，`JVM`，`JDK`，`JAR`,` JNI`,` JVM TI（tool interface）`和`JDWP(javadebug wire protocol，java调试通信协议)`等多个模块的改动。

&emsp;&emsp;本文结合一个样例从`JDK`引入`module`机制的原因，引入`module`涉及的改动点，`module`目标，`module`与`jar`的关系，`module`与`reflection`的关系，`automatic module`和`unnamed module`等方面介绍`modularity`相关机制，希望给想在实际项目中使用`java module`的程序员提供参考。

> &emsp;&emsp;JDK9带来的模块化是一次重大的改变，JDK8以前的特性基本上是随意使用，但JDK9的平台模块化系统是一个全新的改变。贯穿了-> 思考、设计、编写全过程。
>
>  
>
> &emsp;&emsp;模块可以是任何东西，从一组**代码实体、组件或UI类型到框架元素，再到完整的可重用的库。**





## 一、为什么引入modularity机制

1. **classpath问题：**`Java`的`classloader`采用`parent`委托模式，即`application`的`classloader`在加载一个`class`时首先委托给`parent`加载器，`parent`加载器再向上委托其`parent`加载器，依此类推，直到查找到已加载的类或者未查找到，此时再由`classloader`加载。但对一些通用框架，这种机制并不适合，如`JNDI`，`JDBC`，`JAXB`等其接口定义在`bootstrap classloader`中，而其`SPI`是由其他厂商实现，定义在`system classloader`（`bootstrap classloader`的子`loader`，即`application classloader`）中，`parent`无法加载`child`的类，这就需要打破`parent delegation model`；另外`web`容器为了隔离不同应用的类加载器，也定制`classloader`。上述各种场景使得类加载，类依赖关系比较复杂，例如：一个`Web`应用依赖的`hibernate`和`spring`都依赖了不同的`log4j`，很容易导致如下所列的一些运行时异常;
   - NoClassDefFoundError: 编译成功，运行时异常。
   - ClassNotFoundException
   - NoSuchMethodError
   - ClassCastException：不同的类加载器之间的对象进行类型强转。
2. **安全性问题：**`Jdk`中包括很多`internal`类，它们虽然设置为`public`，但`oracle`官方一直**警告开发人员不要使用**；不过，警告信息对开发人员来讲等同于说可以使用，而且这些`api`又能很方便的满足开发需求，从而导致被广泛地“滥用”。例如：`GC`，`Unsafe`，`BASE64Encoder`,`sun包`等;
3. **性能问题**：`jdk`从`1.0`到`8.0`，为了保证兼容性，基本都是增加`api`，从而导致整个`JDK`有几百M，非常的臃肿，很多**`api`实际上很少被用到**，导致程序启动时较慢。在`cloud native`成为主流的今天，非常需要一个能够弹性伸缩的`jdk image`，同时也能够满足`IoT`场景下嵌入式设备的使用需求;

> &emsp;&emsp;Jar文件仅仅是将一组类方便的放在一起而已。一旦加入到classpath中，JVM就对Jar中的所有classes一视同仁放到同一个根root目录下，而不管这些class文件位置在哪。想象一个应用的成千上万的类放置在同一个目录下而没有结构的样子，这对于管理和维护将是一场噩梦。代码库越大，问题越大。例如有20悠久历史的Java平台本身！！！
>
> &emsp;&emsp;**`Java9`模块化可以按需自定义`runtime`!这也就是`jdk9`文件夹下没有了`jre`目录的原因**





## 二、modularity涉及到的改动

&emsp;&emsp;`JDK`的`modularity`机制是一次由底到上的全局性的大升级，它涉及到`JVM`，`JDK`，`tools`等多层次的改动，尤其是`JDK`，全部的`package`按照`module`重新划分，移除了[http://java.xml.ws](https://link.zhihu.com/?target=http%3A//java.xml.ws)，java.corba，java.transaction 等包，如下给出JVM，JTS，JVM TI的主要改动点。

### 2.1、JVM改动点

- 3.16 Modules：介绍引入module后，module的描述文件module-info.java对包和类的访问控制（即module的strong encapsulation特性）产生的影响。
- 4.2.3 Module and Package Names：module命名规范。
- 4.4.11 The CONSTANT_Module_infoStructure：module的常量池结构。
- 4.7.25 The Module Attribute：类的module属性，表明一个module所依赖的其他modules，export和open的packages，使用和提供的services。
- 4.7.26 The ModulePackages Attribute：表明一个module export和open的packages。
- 4.7.27 The ModuleMainClass Attribute：module主类。
- 5.3.6 Modules and Layers：介绍module与layer，classloader之间的关系。



### 2.2、JLS改动点

- 6.5.3 Meaning of Module Names and Package Names：module命名规范。
- 7.2 Host Support for Modules and Packages：应用系统决定如果创建和存储module和package。
- 7.7 Module Declarations：module创建语法规范。
- 13.3 Evolution of Packages and Modules：提供package和module访问控制机制。



### 2.3、JVM TI改动点

- 增加“Bytecode Instrumentation of code inmodules”，agent通过AddModuleReads,AddModuleExports, AddModuleOpens, AddModuleUses 和AddModuleProvides等api修改module的运行时行为。







## 三、modularity目标

1. **明确的依赖配置（reliable configuration）：**通过配置文件明确module之间的依赖关系，不允许有循环依赖，以此代替classpath记载类机制。
2. **封装增强（strong encapsulation）：**增加module层的访问控制机制。引入module后，包和类之间的访问控制由module层的访问控制和原有的包和类的访问控制机制两级机制决定。
3. **可变的JDK文件大小：**jdk按照模块重新划分为70个（jdk11.0.1）modules，开发者在发布应用时，根据所需定制自己的image，进而带来性能和安全的提升。



> **简单来说：**
>
> - 分而治之（Divide and conquer approach）：对于非常大的问题通常需要将大问题分解成一个个的小问题，然后单独解决他们。
> - 实现具有封装性和明确定义的接口：模块化后就可以隐藏模块的内部实现（称为封装encapsulation），同时暴露给用户的东西称为接口(interface)。





## 四、Jar与module的关系

&emsp;&emsp;`Module`的`strong encapsulation`由已有类型（`package`，`class`，`method`）的访问控制机制和`moduel`的可读性（`readability`）、可访问性（`accessibility`）共同决定的。见下表：

> 例如：一个`default`的`class`所在的`package`即使`exports`，也不能被`requires`的`module`访问到。而如果没有`export`的`package`，即使是`public`的类型，也不能被其他`module`访问。
>
> &emsp;&emsp;**模块化可以在更大的粒度上进行封装，对类型的保护变成`private`。** 



| 访问修饰符                     | 类自身访问 | 包内访问 | 子类访问 | `其他类(exported)` | `其他类(unexported)` |
| ------------------------------ | ---------- | -------- | -------- | ------------------ | -------------------- |
| public                         | Y          | Y        | Y        | `Y`                | `N`                  |
| private(不能限制top-level类)   | Y          | N        | N        | `N`                | `N`                  |
| 无控制符(default)              | Y          | Y        | N        | `N`                | `N`                  |
| protected(不能限制top-level类) | Y          | Y        | Y        | `N`                | `N`                  |





## 五、Java Platform Module System简介

&emsp;&emsp;JPMS(JAVA平台模块化系统)引入了`modules`这个语言结构来重用组件。这使得我们可以将**类型和packages**组合到一个模块`module`中。

**模块包括下面三个信息：**

- **名称：**模块的唯一名称，例如：`club.maddm.common`，类似于包名；
- **输入：**什么是模块需要和使用到的？什么是模块编译和运行所必需的？
- **输出：**什么是模块要输出或暴露给其他模块的？



**两个重要的目标：**

- **强封装Strong encapsulation：**由于每一个模块都声明了哪些包是公开的哪些包是内部的，java编译和运行时就可以实施这些规则来确保外部模块**无法确保使用内部类型。**
- **可靠配置Reliable configuration：**由于每一模块**都声明了哪些是它所需的**，那么在运行时就可以检查它所需的所有模块在应用启动运行前是否都有。

> &emsp;&emsp;默认地，一个模块中的每一个java类型只能被该模块中的其他类型所访问。要想暴露类型给外部的模块使用，需要明确指定哪些包packages要暴露export。任何模块只能在包的层级上暴露，一旦暴露了某个包，那这个包中的所有的类型就都可以被外部模块访问。如果一个Java类型所在的包没有暴露，那么外部其他模块是无法import它的，即使这个类型是public的。



### 5.1 Project Jigsaw

&emsp;&emsp;需要编写模块化的代码，就需要先将Java平台模块化。JAVA9模块化**JDK如下图**：

![image-20220313094722120](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20220313094722.png)



**其有如下几个目标：**

- **可伸缩平台（`Scalable platform`）：**逐渐从一个庞大的运行时平台到有能力缩小到更小的计算机设备；
- **安全性和可维护性（`Security and maintainability`）：**更好的组织了平台代码使得更好维护。隐藏内部APIs和更明确的接口定义了提升了平台的安全性。
- **提升应用程序性能（`Improved application performance`）：**只有必须的运行时`runtimes`的跟更小的平台可以带来更快的性能。
- **更简单的开发体验（`Easier developer experience`）：**模块系统与模块平台的结合使得开发者更容易构建应用和库。
- **版本控制（`versioning`）：**目前JPMS不支持版本控制。



> `java --list-modules`：查看所有模块；
>
> `java --describe-module java.sql`：查看某个模块的详情；



### 5.2 模块详情简介



```java
java --describe-module java.sql

java.sql@17
exports java.sql
exports javax.sql
requires java.base mandated
requires java.xml transitive
requires java.logging transitive
requires java.transaction.xa transitive
uses java.sql.Driver
```

- **requires**：输入
- **exports**：输出
- **uses**：使用服务：消费者
- **providers**：提供服务：服务实现者
- **transtive**：传递性



&emsp;&emsp;`java.base`是最基础的模块，也是java开发所需要的最小模块，没有它写不了java代码。该模块会**隐含的加入所有模块**。

- `java`：指核心的java平台模块。即官方的标准模块。
  - **核心Java模块：**指核心的Java SE APIs，例如：java.bas、java.xml
  - **企业级模块**：包含一些如java.corba（包含遗留CORBA技术）和java.transaction（提供数据库事务APIs）。*注意它与Java EE不同，Java EE是一个完全不同的规范。Jave SE和Java EE有一些重叠的地方，为了避免这些重叠，在Java9中已经将这些企业级模块标记为废弃，在将来的版本中可能会被移除掉*
  - **聚合模块（Aggregator）**：这些模块本身没有包含任何API，而是作为一种非常简便的方式将多个模块绑在一起。目前平台有两个聚合模块java.se以及java.se.ee（JavaSe加上与JavaEE重叠的部分）。聚合模块一般是将核心的模块组合在一起使用，要小心使用。
- `javafx`：指java FX模块，即用于构建桌面应用的平台模块。
- `jdk`：指核心的JDK模块。这些不是Java语言规范的一部分，但包含了一些有价值的工具和`APIS`。
- `oracle`：如果是oracle open jdk 就可以看到一些已oracle为前缀的模块。**不建议使用**



**增加模块后的结构：**

![image-20220313104238875](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20220313104239.png)











