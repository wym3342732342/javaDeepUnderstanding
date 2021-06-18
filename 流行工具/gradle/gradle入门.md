# Gradle入门教程





## 三、Gradle构建脚本

`Gradle`构建脚本文件用来处理两件事情：

- 项目
- 任务

&emsp;&emsp;每个`Gradle`生成表示一个或多个项目。一个项目表示一个JAR库或Web应用程序，也可能表示由其他项目产生的`JAR`文件组装的`ZIP`。

&emsp;&emsp;简单地说，一个项目是由不同的任务组成。一个任务是指构建执行的一块工作。任务可能是编译一些类，创建一个 JAR ，产生的 `Javadoc` 或发布一些归档文件库。



### 3.1 编写构建脚本

&emsp;&emsp;`Gradle`提供了一个域特定语言(`DSL`)，用于描述构建。它使用`Groovy`语言，使其更容易来形容和构建。`Gradle`中的每一个构建脚本使用`UTF-8`进行编码保存，并命名为`build.gradle`。

**创建build.gradle文件：**

```groovy
task firstgvy{
	doLast{
		println 'hello,gradle! this is the first gradle'
	}
}

task upper{
	doLast{
		String expString = 'yb first gradle'
		println "Original: " + expString
		println "Upper case: " + expString.toUpperCase()
	}
}

task count {
	doLast {
		4.times {
			print "$it "
		}
		println ""
	}
}
```

**执行：**

```shell
# 1、gradle -q firstgvy
hello,gradle! this is the first gradle
# 2、gradle -q upper
Original: yb first gradle
Upper case: YB FIRST GRADLE
# 3、gradle -q count
0 1 2 3 
```



### 3.2 构建的三个阶段

&emsp;&emsp;`gradle`构建的生命周期主要分为三个阶段：`Initialization`，`Configuration`，`Execution`

- `Initialization`：`Gradle`支持单个或多个工程的构建。在`Initialization`阶段，`Gradle`决定哪些工程将参与到当前构建过程，并为每一个这样的工程创建一个`Project`实例。一般情况下，**参与构建的工程信息将在`settings.gradle`中定义**。 
- `Configuration`：在这一阶段，配置`project`的实例。所有工程的构建脚本都将被执行。`Task`、`configuration`和许多其他的对象将被创建和配置。 
- `Execution`：在之前的`configuration`阶段，`task`的一个子集被创建并配置。这些子集来自于作为参数传入`gradle`命令的`task`名字，在`execution`阶段，这一子集将**被依次执行**。



### 3.3 Groovy的JDK方法

&emsp;&emsp;`Groovy`增加了很多有用的方法到标准的`Java`类。例如，从`Java API`可迭代实现它遍历`Iterable`接口的元素

的`each() `方法。

```groovy
task groovyJDKMethod {
	String myName = "maxuan"
	myName.each(){
		println "${it}"
	}
}
```

**执行：**

```shell
gradle -q groovyJDKMethod     
m
a
x
u
a
n
```





## 四、Gradle任务

&emsp;&emsp;`Gradle`构建脚本描述一个或多个项目，每个项目都由不同的任务组成。

&emsp;&emsp;任务是构建执行的一项工作，可以是编译一些类，将类文件存储到单独的目标文件夹中，创建JAR，生成 Javadoc或将一些归档发布到存储库。



### 4.1 定义任务

**创建一个任务：**

```groovy
task yuanbao {
  doLast {
    println "yuan bao shi dai"
  }
}
```

**执行：**

```shell
gradle -q yuanbao
yuan bao shi dai
```



#### 4.1.1 创建一个依赖其他任务的任务

```groovy
task yuanbao {
    println "yuan bao shi dai"
}

task yuanbaoWW(dependsOn: yuanbao) {
	println "NB"
}
```

**执行：**

```shell
gradle -q yuanbaoWW
yuan bao shi dai
NB
```

> **注意：**任务之间不能互相依赖。



### 4.2 定位任务

&emsp;&emsp;如果要查找在构建文件中定义的任务，则必须使用相应的标准项目属性。这意味着**每个任务都可以作为项目的属性**，使用任务名称作为属性名称。

```groovy
task yuanbao

println yuanbao.name
println project.yuanbao.name
```

**执行命令：**

```shell
gradle -q yuanbao  
yuanbao
yuanbao
```



**任务集合的使用：**

```groovy
task hello
task hello1

println tasks.hello.name

println tasks['hello'].name
println tasks['hello1'].name
```

**执行命令：**

```shell
gradle -q hello  
hello
hello
hello1
```



### 4.3 向任务添加依赖关系

&emsp;&emsp;要将一个任务依赖于另一个任务，这意味着**当一个任务完成时，另一个任务将开始**。 每个任务都使用任务名称进行区分。 任务名称集合由其任务集合引用。 要引用另一个项目中的任务，**应该使用项目路径作为相应任务名称的前缀。**

```groovy
//1、第一种方式
task taskX{
	println 'taskX'
}
task taskY(dependsOn: taskX){
	println 'taskY'
}
//2、第二种方式
task taskX{
	doLast{
		println 'taskX'
	}
}

task taskY{
	doLast{
		println 'taskY'
	}
}

taskY.dependsOn taskX
```

**执行命令：**

```shell
gradle -q taskY
taskX
taskY
```



#### 4.3.1 通过名称添加依赖

```groovy
task taskX{
	doLast{
		println 'taskX'
	}
}


taskX.dependsOn{
	tasks.findAll{
		task -> task.name.startsWith('lib')
	}
}


task lib1{
	doLast{
		println 'lib1'
	}
}

task lib2{
	doLast{
		println 'lib2'
	}
}

task notALib{
	doLast{
		println 'notALib'
	}
}
```

**执行命令：**

```shell
gradle -q taskX
lib1
lib2
taskX
```



### 4.4 向任务添加描述

&emsp;&emsp;可以向任务添加描述。 执行 `Gradle` 任务时会显示此描述。 这可以通过使用 `description` 关键字。

```groovy
task copy(type: Copy) {
	description '将资源目录复制到目标目录。'
	from 'resources'
	into 'target'
	include('**/*.txt','**/*.xml','**/*.properties')
	println("description applied")
}
```

**执行命令：**

```shell
gradle -q copy
description applied
```



### 4.5 跳过任务

&emsp;&emsp;如果用于跳过任务的逻辑不能用谓词表示，则可以使用 `StopExecutionException` 。 **如果操作抛出此异常，则会跳过此操作的进一步执行以及此任务的任何后续操作的执行, 构建继续执行下一个任务。**

```gr
task compile {
	doLast{
		println 'We are doing the compile.'
	}
}

compile.doFirst{
	if(true) {
		throw new StopExecutionException()
	}
}

task myTask(dependsOn: 'compile'){
	doLast{
		println 'I am not affected'
	}
}
```

**执行命令：**

```shell
gradle -q myTask
I am not affected
```

> &emsp;&emsp;`Gradle`在处理任务时有不同的阶段, 首先，有一个配置阶段，其中直接在任务的闭包中指定的代码被执行, 针对每个可用任务执行配置块，而不仅针对稍后实际执行的那些任务。





## 五、Gradle依赖管理

&emsp;&emsp;`Gradle`构建脚本定义了构建项目的过程；每个项目包含一些依赖项和一些发表项。依赖性意味着支持构建项目的东西，例如来自其他项目的所需`JAR`文件以及类路径中的外部`JAR`。以及发布表示项目的结果，如测试类文件，如`war`文件。

&emsp;&emsp;`Gradle`负责构建和发布结果。 发布基于定义的任务。 可能希望将文件复制到本地目录，或将其上传到远程`Maven`或`lvy`存储库，或者可以在同一个多项目构建中使用另一个项目的文件。 发布的过程称为发布。

### 5.1 声明依赖关系

&emsp;&emsp;`Gradle`遵循一些特殊语法来定义依赖关系。 以下脚本定义了两个依赖项，一个是`Hibernate core 5.4.21`，第二个是`junit 5.0`和更高版本。如下面的代码所示，可在文件中使用此代码。

```groovy
plugins {
    id 'java'
}
repositories {
    mavenCentral()
}
dependencies {
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



### 5.2 依赖关系配置

&emsp;&emsp;依赖关系配置只是定义了一组依赖关系。 您可以使用此功能声明从Web下载外部依赖关系。这定义了以下不同的标准配置。

- **编译：**编译项目的生产源所需的依赖关系。
- **运行时：**运行时生产类所需的依赖关系。 默认情况下，**还包括编译时依赖项。**
-  **测试编译：**编译项目测试源所需的依赖项。 默认情况下，**包括编译的产生的类和编译时的依赖。** 
- **测试运行时 ：**运行测试所需的依赖关系。 默认情况下，**包括运行时和测试编译依赖项。**



### 5.3 存储库

&emsp;&emsp;在添加外部依赖关系时，`Gradle`在存储库中查找它们。存储库只是文件的集合，按分组，名称和版本来组织构造。 默认情况下，Gradle不定义任何存储库。 我们必须**至少明确地定义一个存储库**。 一个Java工程通常会依赖于外部的jar包，Gradle可以使用Maven的仓库来获取或者发布相应的jar包。 **Gradle配置Maven中央仓库**

 **下面的代码片段定义了如何定义 maven 仓库。 在 build.gradle 文件中使用此代码：**

```groovy
repositories {
    mavenCentral()
}
```

**下面的代码是定义远程 maven 。 在 build.gradle 文件中可使用下面代码：**

```groovy
repositories {
    maven {
        url "http://repo.mycompany.com/maven2"
    }
}
```

