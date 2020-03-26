# 前言：

&emsp;&emsp;这是Netty实战的精简版，更加的简练。方便快速入门并”不失全面“。

# 第一章 Netty——异步和事件驱动

## 引言：

&emsp;&emsp;假设开发的系统必须能够扩展到15万名并发用户，并且不能有任何的性能损失。那么该怎么办。此时第一个该想到的就是`Reactor + WebFlux`而其底层就是使用`Netty`进行网络通信。

&emsp;&emsp;Netty实战这本书就很好的介绍了`Netty`，其是一个极为丰富的网络编程工具集，本文主要探究它的能力。但其终究是一个框架，它的设计原则：每个小点都和它的技术性内容一样重要，穷其精妙。本文也将探讨：

- 关注点分离——业务和网络逻辑解耦；
- 模块化和可复用性；
- 可测试性作为首要的要求。

> 总结：本文介绍`Netty`的核心概念以及构建块。



## 1.1、Java网络编程

&emsp;&emsp;早期网络编程开发人员智能使用C语言的套接字库，Java（1995-2002）引入了面向对象的套接字【使用门面隐藏了棘手细节问题】。但还是需要大量的样板代码。

### 1.1.1 回顾网络编程

**1、代码演示**

```java
public void serverSocket(String[] args) throws Exception {
    //创建一个新的serverSocket，用于监听指定端口上的连接请求
    ServerSocket serverSocket = new ServerSocket(portNumber);
    //1.accept（）方法是阻塞的，直到一个连接建立
    Socket clientSocket = serverSocket.accept();
    //2.拿到连接的输入输出流
    BufferedReader in = new BufferedReader(
            new InputStreamReader(clientSocket.getInputStream())
    );
    PrintWriter out = new PrintWriter(clientSocket.getOutputStream(),true/*自动冲流*/);
    String request, response;
    while ((request = in.readLine()) != null) {//循环处理开始
        if ("Done".equals(request)) {
            break;//收到结束请求，退出循环处理
        }
        //3.传递给服务器的处理方法
        response = processRequest(request);
        //4.服务器的响应被发送给了客户端
        out.println(response);
    }
}
```

**2、回顾方法**

- `ServerSocket的accept()`：会阻塞，直到连接建立，随后返回一个新的Socket用于客户端和服务器之间的通信。**可使用多线程处理，使其可以继续监听传入的连接；**
- `readLine()`:会阻塞，直到有一个换行符或者回车结尾的字符串被读取；



**3、网络编程处理多请求的示例图：**

![image-20200318205142739](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200318205148.png)

> 这种方案的缺点：
>
> - 只能支撑小数量的客户端；
> - 任何时候都可能有大量的线程处理休眠状态，等待输入或输出数据就绪；
> - 每个线程的调用栈都分配内存，其默认大小区间为`64KB~1MB`;
> - Java虚拟机在物理上虽然可以支持非常大数量的线程，但上下文切换带来的开销就会带来麻烦。



### 1.1.2 Java NIO

&emsp;&emsp;NIO的本意是`New Input/Output`新IO，是在2002年引入的，位于JDK1.4的`java.nio`包中。对网络资源的利用率提供了相当多的控制：

- 可以使用`setsockopt()`配置套接字，以便读/写调用在没有数据的时候立即返回。【非阻塞】
- 可以使用操作系统的事件通知`API`，注册一组非阻塞套接字，以确定它们中是否有任何的套接字已经有数据可供读写。



**1、选择器`java.nio.channels.Selector`**

![image-20200318211454698](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200318211454.png)

&emsp;&emsp;其使用事件通知`API`已确定在一组非阻塞套接字中有哪些已经就绪能够进行`I/O`相关的操作。**这样就可以在任何时间检查`I/O`的完成状态。【此时一个线程就可以处理多个并发的连接】**

> NIO的优点：
>
> - 使用较少的线程便可以处理许多连接，因此也减少了内存管理和上下文切换所带来的开销；
> - 当没有`I/O`操作需要处理的时候，线程也可以被用于其他任务；



## 1.2 Netty简介

&emsp;&emsp;Netty使用较简单的抽象隐藏底层实现的复杂性，为常见的编程任务封装了解决方案。在网络编程领域，`Netty`是`Java`的卓越框架。

**Netty特性总结：**

|   分类   | Netty的特性                                                  |
| :------: | ------------------------------------------------------------ |
|   设计   | 统一的API，支持多种传输类型，阻塞和非阻塞的简单而强大的线程模型<br/>简单而强大的线程模型<br/>真正的无连接数据报套接字支持<br/>链接逻辑组件以支持复用 |
| 易于使用 | 详实的`javadoc`和大量的示例集<br/>不需要超过JDK1.6+的依赖。（一些可选的特性可能需要Java1.7+ `和/或` 额外的依赖 ） |
|   性能   | **拥有比Java的核心API更高的吞吐量以及更低的延迟**<br/>**得益于池化和复用，拥有更低的资源消耗**<br/>**最少的内存复制** |
|  健壮性  | **不会因为慢速、快速或者超载的连接而导致`OutOfMemoryError`**<br/>**消除在高速网络中NIO应用程序常见的不公平读/写比率** |
|  安全性  | 完整的`SSL/TLS`以及`StartTLS`支持<br/>可用于受限环境下，如`Applet`和`OSGI` |
| 社区驱动 | 发布快速而且频繁                                             |



### 1.2.1 谁在用Netty

- `Firebase`：用来做HTTP长连接；
- `Urban Airship`：用来支持各种各样的推送通知；
- `Twitter的Finagle`：基于Netty的系统间通信框架；

> &emsp;&emsp;还有Apple、Twitter、Facebook、Google、Square和Instagram，目前流行的开源项目，如Infinispan、HornetQ、Vert.x、Apache Cassandra和Elasticsearch [8] ，它们所有的核心代码都利用了Netty强大的网络抽象 。



### 1.2.2 异步和事件驱动

&emsp;&emsp;一个即是**异步**又是**事件驱动**的系统表现出一种特殊的、极具有价值的行为：**`可以以任意的顺序响应在任意时间点产生的事件`**。

&emsp;&emsp;使用这种能力可实现更高级别的可伸缩性，**系统、网络、进程在需要处理的工作不断增长时，可以通过某种可行的方式或者扩大他的处理能力来适应这种增长的能力。**

&emsp;&emsp;非阻塞网络调用，**不必等待一个操作的完成**。完全异步的`I/O`正是基于这个特性构建的，并且更进一步：异步方法会立即返回，并且在它完成时，会直接或者在稍后的某个时间点通知用户。

&emsp;&emsp;选择器使得系统能够通过较少的线程便可监视许多连接上的事件。**使用非阻塞`I/O`来处理更快速、更经济，Netty设计底蕴的关键。**

