# 学习大纲

![img](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20201026231909.png)

# 一、Netty简介及前置知识

### 1.1 为什么选择Netty？

&emsp;&emsp;为什么`Java`的网络编程框架大家都会推荐`Netty`,而不是`Java NIO`、`Mina`、`Grizzy`呢？

&emsp;&emsp;因为`Netty`围绕着网络应用框架的核心关注点可以做到**尽善尽美，其健壮性、性能、可扩展性在同领域的框架中都首屈一指。**

**网络应用框架核心关注点：**

- **I/O模型、线程模型和事件处理机制；**
- **易用性API接口；**
- **对数据协议、序列化的支持。**



### 1.2 回顾I/O模型

#### 1.2.1 IO的两个阶段

**IO请求可以分为两个阶段：**

- IO调用阶段：用户进程向内核发起系统调用。
- IO执行阶段：内核等待IO请求处理完成返回。又分为两个阶段
  - 等待数据就绪，并写入内核缓冲区；
  - 将内核缓冲区数据拷贝至用户态缓冲区；

![Drawing 0.png](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20201027211348.png)



#### 1.2.2 LINUX的5种IO模式

1. **同步阻塞I/O（BIO）**

![1.png](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20201027211539.png)

&emsp;&emsp;进程向内核发起 I/O 请求，发起调用的线程一直等待内核返回结果。一次完整的 I/O 请求称为BIO（Blocking IO，阻塞 I/O），所以 BIO 在实现异步操作时，**只能使用多线程模型，一个请求对应一个线程。**

2. **同步非阻塞I/O（NIO）**

![2.png](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20201027211648.png)

&emsp;&emsp;应用进程向内核发起 I/O 请求后不再会同步等待结果，而是会立即返回，通过轮询的方式获取请求结果。NIO 相比 BIO 虽然大幅提升了性能，但是轮询过程中大量的系统调用导致上下文切换开销很大。所以，单独使用非阻塞 I/O 时效率并不高，**而且随着并发量的提升，非阻塞 I/O 会存在严重的性能浪费。**

3. **I/O多路复用**

![3.png](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20201027211854.png)

&emsp;&emsp;多路复用实现了**一个线程处理多个 I/O 句柄的操作**。**多路指的是多个数据通道**，复用指的是使用一个或多个固定线程来处理每一个 Socket。select、poll、epoll 都是 I/O 多路复用的具体实现，线程一次 select 调用可以获取内核态中多个数据通道的数据状态。多路复用解决了同步阻塞 I/O 和同步非阻塞 I/O 的问题，是一种非常高效的 I/O 模型。

4. **信号驱动I/O**

![4.png](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20201027212110.png)

&emsp;&emsp;信号驱动 I/O 并不常用，它是一种半异步的 I/O 模型。在使用信号驱动 I/O 时，当数据准备就绪后，内核通过发送一个 SIGIO 信号通知应用进程，应用进程就可以开始读取数据了。

5. **异步I/O**

![5.png](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20201027212221.png)

&emsp;&emsp;异步 I/O 最重要的一点是从内核缓冲区拷贝数据到用户态缓冲区的过程也是由系统异步完成，应用进程只需要在指定的数组中引用数据即可。

&emsp;&emsp;**异步 I/O 与信号驱动 I/O 这种半异步模式的主要区别：**信号驱动 I/O 由内核通知何时可以开始一个 I/O 操作，而异步 I/O 由内核通知 I/O 操作何时已经完成。



### 1.3 Netty的IO模型

&emsp;&emsp;`Netty`的IO模型是基于**非阻塞I/O**实现的，底层依赖的是`JDK NIO`框架的多路复用器`Selector`。一个多路复用器 Selector 可以同时轮询多个 Channel，采用 epoll 模式后，只需要一个线程负责 Selector 的轮询，就可以接入成千上万的客户端。

&emsp;&emsp;在 I/O 多路复用的场景下，当有数据处于就绪状态后，需要一个事件分发器（Event Dispather），它负责将读写事件分发给对应的读写事件处理器（Event Handler）。事件分发器有两种设计模式：Reactor 和 Proactor，Reactor 采用同步 I/O， Proactor 采用异步 I/O。

&emsp;&emsp;Reactor 实现相对简单，适合处理耗时短的场景，对于耗时长的 I/O 操作容易造成阻塞。Proactor 性能更高，但是实现逻辑非常复杂，目前主流的事件驱动模型还是依赖 select 或 epoll 来实现。

![6.png](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20201027215756.png)

&emsp;&emsp;上图所描述的便是 Netty 所采用的主从 Reactor 多线程模型，所有的 I/O 事件都注册到一个 I/O 多路复用器上，当有 I/O 事件准备就绪后，I/O 多路复用器会将该 I/O 事件通过事件分发器分发到对应的事件处理器中。该线程模型避免了同步问题以及多线程切换带来的资源开销，真正做到高性能、低延迟。



### 1.4 完美弥补Java NIO的缺陷，易用的API

&emsp;&emsp;在 JDK 1.4 投入使用之前，只有 BIO 一种模式。开发过程相对简单。新来一个连接就会创建一个新的线程处理。随着请求并发度的提升，BIO 很快遇到了性能瓶颈。JDK 1.4 以后开始引入了 NIO 技术，支持 select 和 poll；JDK 1.5 支持了 epoll；JDK 1.7 发布了 NIO2，支持 AIO 模型。Java 在网络领域取得了长足的进步。

&emsp;&emsp;既然 JDK NIO 性能已经非常优秀，为什么还要选择 Netty？这是因为 Netty 做了 JDK 该做的事，但是做得更加完备。我们一起看下 Netty 相比 JDK NIO 有哪些突出的优势。

- **易用性。** 我们使用 JDK NIO 编程需要了解很多复杂的概念，比如 Channels、Selectors、Sockets、Buffers 等，编码复杂程度令人发指。相反，Netty 在 NIO 基础上进行了更高层次的封装，屏蔽了 NIO 的复杂性；Netty 封装了更加人性化的 API，统一的 API（阻塞/非阻塞） 大大降低了开发者的上手难度；与此同时，Netty 提供了很多开箱即用的工具，例如常用的行解码器、长度域解码器等，而这些在 JDK NIO 中都需要你自己实现。

- **稳定性。** Netty 更加可靠稳定，修复和完善了 JDK NIO 较多已知问题，例如臭名昭著的 select 空转导致 CPU 消耗 100%，TCP 断线重连，keep-alive 检测等问题。

- **可扩展性。** Netty 的可扩展性在很多地方都有体现，这里我主要列举其中的两点：一个是可定制化的线程模型，用户可以通过启动的配置参数选择 Reactor 线程模型；另一个是可扩展的事件驱动模型，将框架层和业务层的关注点分离。大部分情况下，开发者只需要关注 ChannelHandler 的业务逻辑实现。



### 1.5 耕地的资源消耗

&emsp;&emsp;作为网络通信框架，需要处理海量的网络数据，那么必然面临有大量的网络对象需要创建和销毁的问题，对于 JVM GC 并不友好。为了降低 JVM 垃圾回收的压力，Netty 主要采用了两种优化手段：

- **对象池复用技术。** Netty 通过复用对象，避免频繁创建和销毁带来的开销。

- **零拷贝技术。** 除了操作系统级别的零拷贝技术外，Netty 提供了更多面向用户态的零拷贝技术，例如 Netty 在 I/O 读写时直接使用 DirectBuffer，从而避免了数据在堆内存和堆外内存之间的拷贝。

&emsp;&emsp;因为 Netty 不仅做到了高性能、低延迟以及更低的资源消耗，还完美弥补了 Java NIO 的缺陷，所以在网络编程时越来越受到开发者们的青睐。



### 1.6 网络框架的选型

&emsp;&emsp;很多开发者都使用过 Tomcat，Tomcat 作为一款非常优秀的 Web 服务器看上去已经帮我们解决了类似问题，那么它与 Netty 到底有什么不同？

&emsp;&emsp;Netty 和 Tomcat 最大的区别在于对通信协议的支持，可以说 Tomcat 是一个 HTTP Server，它主要解决 HTTP 协议层的传输，而 Netty 不仅支持 HTTP 协议，还支持 SSH、TLS/SSL 等多种应用层的协议，而且能够自定义应用层协议。

&emsp;&emsp;Tomcat 需要遵循 Servlet 规范，在 Servlet 3.0 之前采用的是同步阻塞模型，Tomcat 6.x 版本之后已经支持 NIO，性能得到较大提升。然而 Netty 与 Tomcat 侧重点不同，所以不需要受到 Servlet 规范的约束，可以最大化发挥 NIO 特性。

&emsp;&emsp;如果你仅仅需要一个 HTTP 服务器，那么我推荐你使用 Tomcat。术业有专攻，Tomcat 在这方面的成熟度和稳定性更好。但如果你需要做面向 TCP 的网络应用开发，那么 Netty 才是你最佳的选择。

&emsp;&emsp;此外，比较出名的网络框架还有 Mina 和 Grizzly。Mina 是 Apache Directory 服务器底层的 NIO 框架，由于 Mina 和 Netty 都是 Trustin Lee 主导的作品，所以两者在设计理念上基本一致。Netty 出现的时间更晚，可以认为是 Mina 的升级版，解决了 Mina 一些设计上的问题。比如 Netty 提供了可扩展的编解码接口、优化了 ByteBuffer 的分配方式，让用户使用起来更为便捷、安全。Grizzly 出身 Sun 公司，从设计理念上看没有 Netty 优雅，几乎是对 Java NIO 比较初级的封装，目前业界使用的范围也很小。

&emsp;&emsp;综上所述，Netty 是我们一个较好的选择。



### 1.7 Netty的发展现状

#### 1.7.1 对比往期版本【Netty4】

&emsp;&emsp;目前主流推荐 Netty 4.x 的稳定版本，Netty 3.x 到 4.x 版本发生了较大变化，属于不兼容升级，下面我们初步了解下 4.x 版本有哪些值得你关注的变化和新特性。

- **项目结构：**模块化程度更高，包名从 org.jboss.netty 更新为 io.netty，不再属于 Jboss。

- **常用 API：**大多 API 都已经支持流式风格，更多新的 API 参考以下网址：https://netty.io/news/2013/06/18/4-0-0-CR5.html。

- **Buffer 相关优化：**Buffer 相关功能调整了现在 5 点。
  1. ChannelBuffer 变更为 ByteBuf，Buffer 相关的工具类可以独立使用。由于人性化的 Buffer API 设计，它已经成为 Java ByteBuffer 的完美替代品。
  2. Buffer 统一为动态变化，可以更安全地更改 Buffer 的容量。
  3. 增加新的数据类型 CompositeByteBuf，可以用于减少数据拷贝。
  4. GC 更加友好，增加池化缓存，4.1 版本开始 jemalloc 成为默认内存分配方式。
  5. 内存泄漏检测功能。

- **通用工具类：**io.netty.util.concurrent 包中提供了较多异步编程的数据结构。

- **更加严谨的线程模型控制**，降低用户编写 ChannelHandler 的心智，不必过于担心线程安全问题。

> 其中还有更多细节变化，感兴趣的同学可以参考以下网址：https://netty.io/wiki/new-and-noteworthy-in-4.0.html。

#### 1.7.2 谁在用Netty

- **服务治理：**Apache Dubbo、gRPC。

- **大数据：**Hbase、Spark、Flink、Storm。

- **搜索引擎：**Elasticsearch。

- **消息队列：**RocketMQ、ActiveMQ。





# 二、Netty架构认知

### 2.1 Netty整体架构

<img src="https://gitee.com/tizo_kingbb/picImg/raw/master/img/20201101155505.png" alt="Drawing 0.png" style="zoom:100%;" />

&emsp;&emsp;`Netty`结构一共分为三个模块【层】：`Core`核心层、`Protocol Support`协议支持层、`Transport Service`传输服务层。



#### 2.1.1 Core 核心层

&emsp;&emsp;`Netty`最精华的部分，它提供了底层网络通信的通用抽象和实现，包括可扩展的事件模型、通用的通信`API`、**支持零拷贝的`ByteBuf`**等。



#### 2.1.2 Protocol Support协议支持层

&emsp;&emsp;协议支持层基本上**覆盖了主流协议的编解码实现**，如 `HTTP`、`SSL`、`Protobuf`、`压缩`、`大文件传输`、`WebSocket`、`文本`、`二进制`等主流协议，此外 Netty 还支持自定义应用层协议。Netty **丰富的协议**支持降低了用户的开发成本，基于 Netty 我们**可以快速开发 HTTP、WebSocket 等服务。**



#### 2.1.3 Transport Service 传输服务层

&emsp;&emsp;提供了网络传输能力的定义和实现方法。其**支持`Socket`、`HTTP`隧道、虚拟机管道等传输方式。**`Netty`对`TCP`、`UDP`等数据传输做了抽象和封装，用户可以聚焦在业务逻辑实现上，而不必关心底层数据传输的细节。



> &emsp;&emsp;Netty的模块设计具备较高的**通用型和可扩展性**，其不仅是一个优秀的网络框架，还可以作为网络编程的工具箱。**并且Netty的设计理念非常优雅，值得我们学习借鉴。**



### 2.2 Netty逻辑架构

![Drawing 1.png](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20201101164417.png)

&emsp;&emsp;`Netty`的逻辑处理架构为典型的**网络分层架构设计**，共分为网络通信层、事件调度层、服务编排层，每一层各司其职。**上图中包含了Netty每一层所用到的核心组件。**



#### 2.2.1 网络通信层

&emsp;&emsp;网络通信层的指责是执行网络`I/O`操作。其支持多种网络协议和`I/O`模型的连接操作。**当网络数据读取到内核缓冲区后，会触发各种网络事件，这些网络事件会分发给事件调度层进行处理。**

&emsp;&emsp;网络通信层的**核心组件**包含**`BootStrap`、`ServerBootStrap`、`Channel`**三个组件。



##### 2.2.1.1 BootStrap & ServerBootStrap

&emsp;&emsp;Bootstap是“引导”的意思，其负责整个`Netty`程序的启动、初始化、服务器连接等过程，它相当于一条主线，串联了Netty的其他核心组件。

&emsp;&emsp;Netty 中的引导器共分为两种类型：一个为**用于客户端引导的 Bootstrap**，另一个为用于**服务端引导的 ServerBootStrap**，它们都继承自抽象类 AbstractBootstrap。

![Drawing 2.png](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20201101171505.png)

- `Bootstrap`：用于连接远端服务器，只绑定一个`EventLoopGroup`。
- `ServerBootStrap`：用于服务端启动绑定本地端口，会绑定两个`EventLoopGroup`，分别称为`Boss`和`Worker`。
  - `Boss`：相当于老板，负责不停的接受新的连接。
  - `Worker`：相当于员工，接受老板的命令。处理连接。

##### 2.2.1.2 Channel

&emsp;&emsp;其就是字面意思“通道”，是**网络通信的载体**。Channel提供了基本的`API`用于网络`IO`操作，如 `register`、`bind`、`connect`、`read`、`write`、`flush` 等。Netty自己实现的Channel是以 JDK NIO Channel 为基础的，相对于 JDK NIO，Netty 的 Channel 提供了更高层次的抽象，同时屏蔽了底层 Socket 的复杂性。



![Drawing 3.png](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20201101173614.png)



































