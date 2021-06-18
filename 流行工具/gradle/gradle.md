# Gradle入门教程



## 一、简介

&emsp;&emsp;`Gradle` 是一种开源自动化构建工具，支持多语言环境，受 `Ant`、`Maven` 思想的影响，集二者之大成，相比 `Ant` 的不规范，`Maven` 的配置复杂、生命周期限制严重，`Gradle` 既规范也更灵活，可以使用`DSL` (领域特定语言，如`Groovy` 或 `Kotlin`）编写构建脚本，脚本更干练。

**特性：**

- 高度定制化：模块化可扩展，更灵活。
- 构建迅速：支持并行构建，自动复用之前任务构建的结果以提高效率
- 功能强大：支持多语言环境，包含 `Java`、`Android`、`C++`、`Groovy`、`Javascript` 等项目的构建



## 二、安装

**`Gradle`安装流程：**

- 确保JDK安装正确
- 下载最新`Gradle`：[官网下载]([Gradle | Releases](https://gradle.org/releases/))
- 解压到指定目录
- 设置PATH：`:/usr/local/Cellar/gradle/7.0.2/bin`,注意win的是`;路径/bin`
- `gradle -v`测试

```shell
------------------------------------------------------------
Gradle 7.0.2
------------------------------------------------------------

Build time:   2021-05-14 12:02:31 UTC
Revision:     1ef1b260d39daacbf9357f9d8594a8a743e2152e

Kotlin:       1.4.31
Groovy:       3.0.7
Ant:          Apache Ant(TM) version 1.10.9 compiled on September 27 2020
JVM:          11.0.5 (Oracle Corporation 11.0.5+10-LTS)
OS:           Mac OS X 10.16 x86_64
```



## 三、任务简介

创建`build.gradle`:

```groovy
task hello {
    println 'Hello world!'
}
```

**运行：**需要在build.gradle的目录

```shell
gradle -q hello
```

> &emsp;&emsp;`-q`的作用是静默输出，使输出更加清晰。



&emsp;&emsp;这里发生了什么? 这个构建脚本定义了一个独立的`task`, 叫做`hello`, 并且加入了一个`action`，当你运行 `gradle hello`, `Gradle` 执行叫做 `hello` 的 `task`, 也就是执行了你所提供的 `action`. 这个 `action` 是一个包含了一些 `Groovy` 代码的闭包(`Closure`)。



#### 3.1 扩展阅读

&emsp;&emsp;`task`块内可以**定义前置、后置执行的方法（闭包）`doFirst`、`doLast`**，按字面意思来理解就可以，但要注意，定义多个`doLast`或`doFirst`无法保证执行顺序。

```groovy
task taskX{
	doLast{
		println 'taskX'
	}
}
```



## 四、构建基础

### 4.1 projects和tasks

&emsp;&emsp;`projects` 和 `tasks`是 Gradle 中最重要的两个概念。

&emsp;&emsp;**任何一个 Gradle 构建都是由一个project或多个 projects 组成**。每个 project 或许是一个 jar 包或者一个 web 应用，它也可以是一个由许多其他项目中产生的 jar 构成的 zip 压缩包。

&emsp;&emsp;**每个 project 都由多个 tasks 组成**。**每个 task 都代表了构建执行过程中的一个原子性操作**。如编译，打包，生成 javadoc，发布到某个仓库等操作。

&emsp;&emsp;**简单来说，project 相当于一个构建工程的一个模块，而 task 是其中一个模块的一个操作**



### 4.2 调用Groovy

**示例：**

```groovy
task upper {
  String str = 'this is a simple test'
  println "原始值：" + str
  println "转大写后：" + str.toUpperCase()
}
```

**执行命令：**

```shell
gradle -q upper
```

> `Groovy` 兼容 `Java `语法，我们可以通过在 `task` 中调用 `Groovy` 或 `Java` 的方法来完成想做的操作



### 4.3 定义项目

&emsp;&emsp;在 `Maven` 中，可以明确定义项目版本，构建时会将这个版本包含在 `war` 或` jar` 等制品的文件名称中，推送到`Maven`私服中也需要设置 `group` `artifactId` `version` 信息，那么 Gradle 中如何定义呢？

&emsp;&emsp;`Gradle` 中，对应`Maven` 的三个参数，将 `artifactId` 变成了 `rootProject.name`，那么只需额外定义 `group` 与 `version`

**在 `build.gradle` 中设置：**

```groovy
group = 'club.maddm'
version = '0.0.1-SNAPSHOT'
```

**`Gradle` 配置中还有一个特殊的配置文件，`gradle.properties`，可以在里边配置变量供 `build.gradle` 读取：**

```properties
version=0.0.1-SNAPSHOT
group=club.maddm
```



### 4.4 Java构建入门

**通过`gradle init`构建项目：**

```shell
gradle init
> Task :wrapper

Select type of project to generate:
  1: basic
  2: application
  3: library
  4: Gradle plugin
Enter selection (default: basic) [1..4] 2

Select implementation language:
  1: C++
  2: Groovy
  3: Java
  4: Kotlin
  5: Swift
Enter selection (default: Java) [1..5] 3

Select build script DSL:
  1: Groovy
  2: Kotlin
Enter selection (default: Groovy) [1..2] 1

Select test framework:
  1: JUnit 4
  2: TestNG
  3: Spock
  4: JUnit Jupiter
Enter selection (default: JUnit 4) [1..4]

Project name (default: demo):

Source package (default: demo):


> Task :init
Get more help with your project: https://docs.gradle.org/5.4.1/userguide/tutorial_java_projects.html

BUILD SUCCESSFUL
2 actionable tasks: 2 executed
```



**生成代码结构：**

.
├── HELP.md
├── build
│   ├── bootJarMainClassName
│   ├── classes
│   │   └── java
│   │       └── main
│   │           └── club
│   │               └── maddm
│   │                   └── gradledemo
│   │                       └── GradleDemoApplication.class
│   ├── generated
│   │   └── sources
│   │       ├── annotationProcessor
│   │       │   └── java
│   │       │       └── main
│   │       └── headers
│   │           └── java
│   │               └── main
│   ├── libs
│   │   └── gradle-demo-0.0.1-SNAPSHOT.jar
│   ├── resources
│   │   └── main
│   │       └── application.properties
│   └── tmp
│       ├── bootJar
│       │   └── MANIFEST.MF
│       ├── bootWar
│       │   └── MANIFEST.MF
│       ├── compileJava
│       │   └── source-classes-mapping.txt
│       └── war
│           └── MANIFEST.MF
├── build.gradle  依赖管理
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradle.properties
├── gradlew
├── gradlew.bat
├── settings.gradle
└── src  //写代码的地方
    ├── main
    │   ├── java
    │   │   └── club
    │   │       └── maddm
    │   │           └── gradledemo
    │   │               └── GradleDemoApplication.java
    │   └── resources
    │       └── application.properties
    └── test
        └── java
            └── club
                └── maddm
                    └── gradledemo
                        └── GradleDemoApplicationTests.java



&emsp;&emsp;当 `init` task 执行的时候，优先调用 `wrapper` task 来生成 `gradlew` 、`gradlew.bat` 以及新项目所需的代码结构

[官方文档](https://guides.gradle.org/building-java-applications/)



## 五、依赖管理

&emsp;&emsp;`Gradle`遵循一些特殊语法来定义依赖关系。 以下脚本定义了两个依赖项，一个是`Hibernate core 5.4.21`，第二个是`junit 5.0`和更高版本。如下面的代码所示，可在文件中使用此代码。

**例子：**

```groovy
plugins {
    id 'java'
}
repositories {
    mavenCentral()//使用maven中央仓库
}
dependencies {//依赖
    implementation 'org.hibernate:hibernate-core:5.4.21.Final'
    testImplementation 'junit:junit:5.+'
}
```

**对比maven：**

```xml
<dependencies>
   <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>5.0</version>
            <scope>test</scope>
  </dependency>
</dependencies>
```



### 5.1 导入依赖API

- `compile`：从仓库里下载并编译，支持依赖传递
- `api`：新语法，等同compile
- `implementation`：新语法，与api相比不支持传递依赖，减少循环编译优化效率

> `compile/api/implementation`导入的依赖都是编译期与运行期都会提供的（打进制品中）



### 5.2 屏蔽依赖API

- `providedCompile`：编译期参与编译，运行期不提供（但生成的war包中，会将这些依赖打入`WEB-INF/lib-provided`中）
- `providedRuntime`：不参与编译但运行期需要，比如 `mysql `驱动包，如果在代码里用到了此依赖则编译失败

### 5.3 测试期依赖API

- `testCompile`：测试期编译，非测试期不编译
- `testImplementation`：与`implementation`相同，仅是测试周期的



```groovy
dependencies {
//    compileClasspath 'org.springframework.boot:spring-boot-starter-data-jpa'
//    runtimeClasspath 'mysql:mysql-connector-java'
//    testCompileClasspath 'org.springframework.boot:spring-boot-starter-test'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'mysql:mysql-connector-java'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```



### 5.4 排除依赖API

#### 5.4.1 引入依赖时排除

```groovy
dependencies {
    implementation ('org.springframework.boot:spring-boot-starter-data-jpa'){
        exclude group: "jakarta.persistence", module: "jakarta.persistence-api"
    }
}
```



#### 5.4.2 全局排除

```groovy
configurations.all {
    exclude group: "jakarta.persistence", module: "jakarta.persistence-api"
}
//或
configurations{
    all*.exclude group: "jakarta.persistence", module: "jakarta.persistence-api"
}
```



#### 5.5 依赖管理

```groovy
dependencyManagement{
    dependencies{//定义 dependencies 块，指定依赖版本
        dependency "org.springframework.boot:spring-boot-starter-data-redis:2.5.1"
    }
    imports{//引入BOM，类似引入Maven的parent
        mavenBom "org.springframework.boot:spring-boot-dependencies:2.5.1"
    }
}
```



## 六、使用gradle创建boot项目

**`Gradle`项目创建流程：**

- 使用IDEA（或命令行）创建`Gradle`项目
- 更改`build.gradle`和`settings.gradle`
- 编写代码
- 使用`bootJar`、`bootWar`命令打包

**`build.gradle`文件详解：**

```groovy
plugins {//插件
    id 'org.springframework.boot' version '2.5.1'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
    id 'war'//如果要打war包需要加上这个命令
}

group = 'club.maddm'//项目group
version = '0.0.1-SNAPSHOT'//项目版本
sourceCompatibility = '11'//使用java11

repositories {//配置仓库
    maven { url 'https://maven.aliyun.com/repository/public/' }//使用阿里云仓库
  //mavenCentral()//使用maven中央仓库
}

dependencies {//管理依赖
    implementation 'org.springframework.boot:spring-boot-starter-web'
    providedCompile 'org.springframework.boot:spring-boot-starter-tomcat'//排除tomcat，打war包需要

    testImplementation 'org.springframework.boot:spring-boot-starter-test'

    components {
        withModule('org.springframework:spring-beans') {
            allVariants {
                withDependencyConstraints {
                    // Need to patch constraints because snakeyaml is an optional dependency
                    it.findAll { it.name == 'snakeyaml' }.each { it.version { strictly '1.19' } }
                }
            }
        }
    }
}


test {
    useJUnitPlatform()
}
```

**`settings.gradle`文件详解：**

```groovy
rootProject.name = 'gradle-demo'//项目名称

//子包的相关内容也在这个里面
```



## 附：

### 参考文献：

- [Gradle基础入门教程]([[Gradle教程]Gradle 基础入门 - 东北小狐狸 - 博客园 (cnblogs.com)](https://www.cnblogs.com/hellxz/p/helloworld-gradle.html))：整体流程
- [Gradle官网]([Gradle User Manual](https://docs.gradle.org/current/userguide/userguide.html))：解决不能实现的版本问题