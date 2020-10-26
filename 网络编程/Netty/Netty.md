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



## 1.3 Netty核心组件

**Netty的主要构件块：**代表了不同类型的构造：资源、逻辑以及通知

- `Channel`
- `回调`
- `Future`
- `事件和ChannelHandler`



### 1.3.1 Channel

&emsp;&emsp;`Channel`是`Java NIO`的基本构造，其代表一个实体（文件、套接字等）的开放连接【读操作和写操作】。

&emsp;&emsp;`Channel`可以看作是传入（入站）或者传出（出战）数据的载体。其可以打开或者关闭，连接或者断开连接。



### 1.3.2 回调

&emsp;&emsp;回调其实就是一个`方法`，在操作完成后通知相关方最常见的方式之一。`Netty`内部就是使用了回调来处理事件。

```java
/**
 * 通道入站处理程序适配器 案例
 * @author King
 * @version 1.0
 * @date 2020/5/9 23:00
 */
public class ConnectHandler extends ChannelInboundHandlerAdapter {
    @Override//<-- 当一个新的连接已经被建立时，channelActive(ChannelHandlerContext)将会被调用
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("客户端 " + ctx.channel().remoteAddress() + " 已连接");
    }
}
```



### 1.3.3 Future

&emsp;&emsp;`Future`提供了另一种在操作完成时通知应用程序的方式。`JDK`内置了`java.util.concurrent.Future`接口，但其需要手动检查操作的完成情况。`Netty`提供了自己的实现`ChannelFuture`。

&emsp;&emsp;`ChannelFuture`提供了集中额外的方法，这些方法使得我们能够注册一个或者多个`ChannelFutureListener`实例。**监听器的回调方法`opreationComplete()`**,将会在对应的操作完成时被调用。**Netty消除了手动检查的麻烦。**

&emsp;&emsp;每个`Netty`的出站`I/O`操作都将返回一个`ChannelFuture`,其不会阻塞。**Netty完全是异步和事件驱动的。**

#### 1.3.3.1 异步的获取连接

```java
Channel channel = ...;
//非阻塞
ChannelFuture future = channel.connect(     //← 异步地连接到远程节点
    new InetSocketAddress("192.168.0.1", 25)
);
```

&emsp;&emsp;`connect()`方法将会直接返回，而不会阻塞。

#### 1.3.3.2 回调案例

```java
Channel channel = ...;
//非阻塞
ChannelFuture future = channel.connect(  //← -- 异步地连接到远程节点
    new InetSocketAddress("192.168.0.1", 25));

future.addListener(new ChannelFutureListener() {   //← --  注册一个ChannelFutureListener，以便在操作完成时获得通知
    @Override
    public void operationComplete(ChannelFuture future) { //← --  ❶ 检查操作
的状态
       if (future.isSuccess()){ 
            ByteBuf buffer = Unpooled.copiedBuffer(  //← -- 如果操作是成功的，则创建一个ByteBuf以持有数据
               "Hello",Charset.defaultCharset());
           ChannelFuture wf = future.channel()
                .writeAndFlush(buffer);   //← -- 将数据异步地发送到远程节点。返回一个ChannelFuture
            ....
        } else {
            Throwable cause = future.cause();　　//← --　如果发生错误，则访问描述原因的Throwable
            cause.printStackTrace();
        }
    }
});
```

&emsp;&emsp;以上例子演示了如何利用`ChannelFutureListener`，案例中在`connect()`后返回的`ChannelFuture`上添加了一个监听器。如果成功写入`Channel`，如果不成功检索`Throwable`。

&emsp;&emsp;可以把`ChannelFutureListener`看作是回调的一个更加精细的版本。回调和`Future`是互补的机制。



### 1.3.4 事件和ChannelHandler

&emsp;&emsp;`Netty`使用不用的事件来通知我们状态的改变或者是操作的状态。这使得我们**能够基于已经发生的事件来触发适当的动作。**

&emsp;&emsp;`Netty`中的事件是按照它们与入站或出战数据流的相关性进行分类的。

- **入站事件：**
  - 连接已被激活或者连接失活；
  - 数据读取；
  - 用户事件；
  - 错误事件；
- **出站事件：**
  - 打开或者关闭到远程节点的连接；
  - 将数据写到或者冲刷到套接字；

&emsp;&emsp;每个事件都可以被分发给`ChannelHandler`类中的我们实现的处理方法。

![image-20200522135609983](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200522135618.png)

&emsp;&emsp;此图展示了一个事件是如何被一个这样的`ChannelHandler`链处理的。`Netty`的`ChannelHandler`为处理器提供了基本的抽象。可以将其理解为一种为了响应特定事件而被执行的回调。

&emsp;&emsp;`Netty`提供了大量预定义的可以开箱即用的`ChannelHandler`实现，包括用于各种协议(如 `HTTP` 和 `SSL/TLS`) 的`ChannelHandler`。



### 1.3.5 总结

#### 1.3.5.1 Future、回调和ChannelHandler

&emsp;&emsp;异步编程模型是建立在`Future`和回调的概念之上的，事件派发到`ChannelHandler`进行处理。

&emsp;&emsp;如果要进行拦截操作以及高速地转换**入站数据**和**出站数据**。都只需要提供回调或者利用操作所返回的`Future`。这使得链接操作变得既简单又高效。



#### 1.3.5.2 选择器、事件和EventLoop

&emsp;&emsp;`Netty`通过触发事件将`Selector`从应用程序中抽象出来，`Netty`的内部，会为每个`Channel`分配一个`EventLoop`，用以处理所有事件：

- 注册感兴趣的事件；
- 将事件派发给`ChannelHandler`;
- 安排进一步的动作；

&emsp;&emsp;`EventLoop`本身只由一个线程驱动，处理了一个`Channel`的所有`I/O`事件，这个简单而强大的设计消除了你可能有的在你的`ChannelHandler`中需要进行同步的任何顾虑。因此，我们可以专注于我们的逻辑。





# 第二章 第一个Netty应用程序

![image-20200523093648422](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200523093648.png)



&emsp;&emsp;一个客户端将消息发送给服务器，而服务器再将消息会送给客户端的案例，途中展示了整个应用程序的功能。



## 2.1 编写Echo服务器

&emsp;&emsp;所有的`Netty`服务器都需要以下两部分：

- **`ChannelHandler`：**用于实现服务端对从客户端接收的数据的处理，即业务逻辑。
- **引导：**配置服务器的启动代码，将服务器绑定到它要监听连接请求的端口上。



### 2.1.1 ChannelHandler和业务逻辑

&emsp;&emsp;`ChannelHandler`，它是一个接口族的父接口，负责接收并响应时间通知。每个`Netty`服务端要响应传入的消息，就需要实现`ChannelInboundHandler`接口，用来定义响应入站。【比较简单的程序继承`ChannelInboundHandlerAdapter`类就行了】。

**案例中用到的方法：**

- `channelRead()`：对于每个传入的消息都要掉用；
- `channelReadComplete()`：通知ChannelInboundHandler最后一次对channelRead()的调用是当前批量读取中的最后一条消息。
- `exceptionCaught()`：在读取操作期间，有异常排除时会调用。

**实现服务端：**

```java
@Sharable//标示一个channel-Handler可以被多个Channel安全地共享
public class EchoServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf in = (ByteBuf) msg;
        //将消息记录到控制台
        System.out.println(
                "服务器收到：" + in.toString(CharsetUtil.UTF_8)
        );
        ctx.write(in);//将收到的消息在写给发送者，不冲流
    }
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        //最后一条消息：冲流并关闭该channel
        ctx.writeAndFlush(Unpooled.EMPTY_BUFFER)
                .addListener(ChannelFutureListener.CLOSE);
    }
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();//打印异常栈跟踪
        ctx.close();//关闭该Channel
    }
}
```

&emsp;&emsp;`ChannelInboundHandlerAdapter`有直观的API，并且它的每个方法都可以被重写以挂钩事件生命周期的恰当点上。

> 如果不处理异常会发生什么：
>
> &emsp;&emsp;`Channel`都拥有一个与之相关联的`ChannelPipeline`，其持有一个`ChannelHandler`的实例链。
>
> &emsp;&emsp;默认情况下，`ChannelHandler`会把对它的方法的调用转发给链中的下一个`ChannelHandler`。如果`exceptionCaught()`方法没有被该链中的某处实现，那么所接收的异常将会被传递到`ChannelPipeline`的尾端被记录。
>
> &emsp;&emsp;也就是说应用中应该至少提供一个实现了`exceptionCaught()`方法的`ChannelHandler`

### 2.1.2 实际业务中注意的点

- 针对不同类型的事件来调用`ChannelHandler`;
- 应用程序通过实现或者扩展`ChannelHandler`来挂钩到事件的声明周期；
- 在架构上，`ChannelHandler`有助于保持业务逻辑与网络处理代码的分离；



### 2.1.3 引导服务器

#### 2.1.3.1 引导作用

- 绑定到服务器将在其上监听并接受传入连接请求的端口；
- 配置`Channel`，以将有关的入站消息通知给`ServerHandler`实例；【EchoServerHandler】



> 说明：
>
> &emsp;&emsp;接下来可能遇到术语传输，网络协议的标准多层视图中，传输层提供了端到端的或者主机到主机的通信服务。
>
> &emsp;&emsp;因特网通信是建立在`TCP`传输之上的。

#### 2.1.3.2 创建引导程序的步骤

- 创建一个`ServerBootstrap`的实例以引导和绑定服务器；
- 创建并分配一个`NioEventLoopGroup`实例以进行时间的处理，如接受新连接以及读/写数据；
- 指定服务器绑定的本地的`InetSocketAddress`；
- 使用`Handerler`的实例初始化每个新的`Channel`；
- 调用`ServerBootstrap.bind()`方法以绑定服务器；

#### 2.1.3.3 引导程序代码

```java
public class EchoServer {
    private final int port;

    public EchoServer(int port) {
        this.port = port;
    }

    public void start() throws Exception {
        final EchoServerHandler serverHandler = new EchoServerHandler();

        //1、创建event-loopGroup
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            //2、创建server bootstrap
            ServerBootstrap b = new ServerBootstrap();

            b.group(group)
                    .channel(NioServerSocketChannel.class)//3、指定所使用的NIO传输Channel
                    .localAddress(new InetSocketAddress(port))//4、使用指定的端口设置套接字地址
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        //5、添加一个EchoServer-Handler到子Channel的ChannelPipeline
                        @Override
                        protected void initChannel(SocketChannel channel) throws Exception {
                            //EchoServerHandler被标注为@Shareable，所以我们可以总是使用同样的实例
                            channel.pipeline().addLast(serverHandler);
                        }
                    });
            ChannelFuture f = b.bind().sync();//6、异步的绑定服务器；调用sync()方法阻塞等待直到绑定完成
            f.channel().closeFuture().sync();//7、获取Channel的closeFuture并且阻塞当前线程直到它完成

        } finally {
            group.shutdownGracefully().sync();//8、关闭EventLoopGroup，释放所有的资源
        }
    }

    public static void main(String[] args) throws Exception {
        if (args.length != 1) {
            System.err.println(
                    "Usage：" + EchoServer.class.getSimpleName() + " "
            );
        }
        int port = Integer.parseInt(args[0]);//设置端口号，会有异常
        new EchoServer(port).start();//启动服务器
    }
}
```

&emsp;&emsp;在`1`处使用了`NioEventLoopGroup`来接受和处理新的链接，而`3`处应该使用`NioServerSocketChannel`作为对应的通道类型；

&emsp;&emsp;在`5`处使用了`ChannelInitializer`，当一个新的连接被接受时，一个新的子`Channel`将被创建，而`ChannelInitializer`将会把`Handler `的实例添加到该`Channel`的`ChannelPopeline`中。此时这个`Handler`将会收到有关入站消息的通知；

&emsp;&emsp;对于`6`处的绑定服务器操作，调用`sync()`方法将导致当前的`Thread`阻塞，一直到绑定操作完成为止；

&emsp;&emsp;`7`处的意思是阻塞到`channel`关闭。`8`处为关闭`EventLoopGroup`释放资源，包括所有创建的线程；

&emsp;&emsp;`NIO`是使用最为管饭的传输方式，也可以使用`OIO`传输，需要指定为`OioServerSocketChannel`,`OioEventLoopGroup`。



## 2.2 编写Echo客户端

**Echo客户端将会：**

1. 连接到服务器；
2. 发送一个或者多个消息；
3. 对于每个消息，等待并接收从服务端发回的相同的消息；
4. 关闭连接。



### 2.2.1 通过ChannelHandler实现客户端逻辑

```java
@Sharable//标记该类的实例可以被多个Channel共享
public class EchoClientHandler
        extends SimpleChannelInboundHandler<ByteBuf> {

    /**
     * 服务器连接已经创建之后
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush(
                Unpooled.copiedBuffer("Netty rocks!",//当被通知Channel是活跃的时候，发送一条消息
                        CharsetUtil.UTF_8)
        );
    }

    /**
     * 从服务器接收到一条消息
     * @param channelHandlerContext
     * @param in
     * @throws Exception
     */
    @Override
    protected void channelRead0(ChannelHandlerContext channelHandlerContext, ByteBuf in) throws Exception {
        System.out.println(//记录已接受消息的转储
                "Client recerved：" +
                        in.toString(CharsetUtil.UTF_8)
        );
    }

    /**
     * 处理过程中发生异常
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //在发生异常时，记录并关闭Channel
        cause.printStackTrace();
        ctx.close();//关闭Channel
    }
}
```

&emsp;&emsp;客户端使用`SimpleChannelInboundHandler`类处理任务：

- `channelActive`：与服务器的连接已经建立之后将被调用；
- `channelRead0`：当从服务器接受到一条消息时被调用；**【服务器发送的消息可能被分块接收。发送5个字节时，可能会发送一个3字节，一个2字节。`channelRead0()`方法可能被调用两次。作为面向流的协议，TCP保证了字节数组会被有序接收(发送时的顺序)】**
- `exceptionCaught`：在处理过程中引发异常时被调用。

> **SimpleChannelInboundHandler与ChannelInboundHandler**
>
> &emsp;&emsp;为什么在客户端使用的是`SimpleChannelInboundHandler`而不是`ChannelInboundHandler`，这和两个因素有关**业务逻辑如何处理消息以及Netty如何管理资源**。
>
> &emsp;&emsp;在客户端，`channelRead0()`方法完成时，已经有了传入消息，并且已经处理完它了。`SimpleChannelInboundHandler`负责释放指向保存该消息的`ByteBuf`的内存引用。
>
> &emsp;&emsp;在服务端中，仍要将传入消息回送给发送者，而`write()`操作是异步的，直到`channelRead()`方法返回后可能仍然没有完成。消息会在`channelReadComplete()`方法中，当`writeAndFlush()`方法被调用时被释放。



### 2.2.2 引导客户端

&emsp;&emsp;引导客户端类似于引导服务器，不同的是客户端需要使用主机和端口参数来连接远程地址。

```java
public class EchoClient {
    private final String host;
    private final int port;

    public EchoClient(String host, int port) {
        this.host = host;
        this.port = port;
    }

    public void start() throws Exception {
        EchoClientHandler clientHandler = new EchoClientHandler();
        NioEventLoopGroup group = new NioEventLoopGroup();
        try {
            //指定EventLoopGroup以处理客户端事件；需要适用于NIO的实现
            Bootstrap b = new Bootstrap();
            b.group(group)
                    .channel(NioSocketChannel.class)//适用于NIO传输的Channel类型
                    .remoteAddress(new InetSocketAddress(host, port))//设置服务器的InetSocketAddress
                    .handler(
                            new ChannelInitializer<SocketChannel>() {
                                //在创建channel时，想ChannelPipeline中添加一个EchoClientHandler实例
                                @Override
                                protected void initChannel(SocketChannel ch) throws Exception {
                                    ch.pipeline().addLast(clientHandler);
                                }
                            }
                    )
            ;
            ChannelFuture f = b.connect().sync();//连接到远程节点，阻塞等待知道连接完成
            f.channel().closeFuture().sync();//阻塞直到Channel关闭
        } finally {
            group.shutdownGracefully().sync();//关闭线程池并且释放所有的资源
        }
    }

    public static void main(String[] args) throws Exception {
        if (args.length != 2) {
            System.err.println(
                    "Usage:" + EchoClient.class.getSimpleName() + "<host><port>"
            );
            return;
        }
        String host = args[0];
        int port = Integer.parseInt(args[1]);
        new EchoClient(host, port).start();
    }
}
```

**整个客户端的编写过程：**

- 为初始化客户端，创建了一个`Bootstrap`实例；
- 为进行事件处理分配了一个`NioEventLoopGroup`实例，其中事件处理包括创建新的连接以及处理入站和出站数据；
- 为服务器连接创建了一个`InetSocketAddress`实例；
- 当连接被创建时，一个`EchoClientHandler`实例会被安装到（该Channel的）ChannelPipeline中；
- 在一切都设置完成后，调用`Bootstrap.connect()`方法连接到远程节点；



## 2.3 小节

&emsp;&emsp;本节完成的小案例可以伸缩到支持数千个并发连接——每秒可以比普通的基于套接字的`Java`应用程序处理多得多的消息。





# 第三章 Netty的组件和设计

&emsp;&emsp;从高层次的角度来看，`Netty`解决了两个相应的关注领域，可以理解为**技术和体系结构**。其是基于`Java NIO`的异步的和事件驱动的实现。保证了高负载下应用程序性能的最大化和可伸缩性。`Netty`也包含了一组设计模式，将应用程序逻辑从网络层解耦。简化开发过程，同时也最大限度地提高了可测试性、模块化以及代码的可重用性。



## 3.1 Channel、EventLoop和ChannelFuture

&emsp;&emsp;这三个类可以说是Netty网络抽象的代表：

- `Channel`：Socket
- `EventLoop`：控制流、多线程处理、并发
- `ChannelFuture`：异步通知



### 3.1.1 Channel接口

&emsp;&emsp;抽象了基本的`I/O`操作（`bind()、connect()、read()和write()`）依赖于底层网络传输所提供的**原语**。其基本的构造是`class Socket`，其所提供的`Api`大大降低了直接使用`Socket`类的复杂度。

**部分实现类清单：**

- `EmbeddedChannel`
- `LocalServerChannel`
- `NioDatagramChannel`
- `NioSctpChannel`
- `NioSocketChannel`



### 3.1.2 EventLoop接口

&emsp;&emsp;`EventLoop`定义了`Netty`的核心抽象，用于处理连接的生命周期中所发生的事件。

**其大致的工作流程：**

![image-20200722213836990](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200722213844.png)

这些关系是：

- 一个`EventLoopGroup`包含一个或者多个`EventLoop`；
- 一个`EventLoop`在它的生命周期内只有一个`Thread`绑定；
- 所有由`EventLoop`处理的`I/O`事件都将在它专有的`Thread`上被处理；
- 一个`Channel`在它的生命周期内之注册于一个`EventLoop`;
- 一个`EventLoop`可能会被分配给一个或者多个`Channel`。

> &emsp;&emsp;注意，在这种设计中，一个给定`Channel`的`I/O`操作都是由相同的`Thread`执行的，实际上消除了对于同步的需求。



### 3.1.3 ChannelFuture接口

&emsp;&emsp;`Netty`中所有的`I/O`操作都是异步的。因为一个操作可能不会立即返回，所以需要一个**某个时间点确定某结果的方法**。`Netty`提供了`ChannelFuture`接口，其`addListener()`方法注册了一个`ChannelFutureListener`，以便在某个操作完成时，无论是否成功都会得到通知。

&emsp;&emsp;可以将`ChannelFuture`看作是将来要执行操作的占位符。其肯定会被执行，并且所有属于同一个`Channel`的操作都被保证其将以它们被调用的顺序被执行。



## 3.2 ChannelHandler 和 ChannelPipeline

### 3.2.1 ChannelHandler接口

&emsp;&emsp;`Netty`的主要组件，所有处理入站和出站数据的应用程序逻辑的容器。因为`ChannelHandler`的方法是由网络事件触发的。



### 3.2.2 ChannelPipeline接口

&emsp;&emsp;`ChannelPipeline`提供了`ChannelHandler`链，并定义了用于在该链上传播入站和出站事件流的`API`。【当Channel被创建时会被自动分配到其所属的ChannelPipeline】

`ChannelHandler`安装到`ChannelPipeline`的过程：

1. 一个`ChannelInitializer`的实现被注册到了`ServerBootstrap`中；
2. 当`ChannelInitializer.initChannel()`方法被调用时，`ChannelInitializer`将在`ChannelPipeline`中安装一组自定义的`ChannelHandler`;
3. `ChannelInitializer`将它自己从`ChannelPipeline`中移除。



![image-20200724111617525](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200724111617.png)

&emsp;&emsp;`ChannelHandler`是专为支持广泛的用途而设计的，可以将其看作是处理往来`Channel-Pipleline`事件（包括数据）的任何代码的通用容器【如上图，出/入站】。

![image-20200724150914797](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200724150915.png)

&emsp;&emsp;这两种`ChannelHandler`就像是两条流水线，他们接收事件、执行他们所实现的处理逻辑、并将数据传递给链中的下一个`ChannelHandler`【由添加次数决定】。也可以说，`ChannelPipeline`是这些`ChannelHandler`的编排顺序。

&emsp;&emsp;当一个消息或者任何其他的**入站事件**被读取，那么它会从`ChannelPipeline`的头部开始流动，并且传递给第一个`ChannelInboundHandler`，处理完成后会传递给下一个`Handler`。最终到达`ChannelPipeline`的尾端。

&emsp;&emsp;**数据的出站**，数据将从`ChannelOutboundHandler`链的尾端开始流动，直到它到达链的头部为止。

&emsp;&emsp;`Netty`能区分`Inbound`和`Outbound`的处理器，并且确保数据只会在具有相同定向数据类型的两个`ChannelHandler`之间传递。

&emsp;&emsp;当`ChannelHandler`被添加到`ChannelPipeline`时，它将会被分配一个`ChannelHandler-Context`，其代表了`Handler`和`Pipleline`之间的绑定。

&emsp;&emsp;在`Netty`中有**两种发送消息的方式**。一是直接写到`Channel`中。二是写到`ChannelHandler`相关联的`Context`中。**前一种方式消息会从`Pipeline`的尾端开始流动，后一种方式消息从`ChannelPipeline`中的下一个`Handler`开始流动。**

> &emsp;&emsp;通过使用`ChanneHandlerContext`，事件可以被传递给`ChannelHandler`链中的下一个`ChannelHandler`。【有时会忽略一些不感兴趣的事件】。`Netty`提供的两个`Handler`[出、入站]抽象基类。通过`ChannelHandlerContext`上的对应方法，其提供了将事件传递给下一个`ChannelHandler`。



### 3.2.3 更加深入地了解ChannelHandler

&emsp;&emsp;Netty提供了许多不同类型的`ChannelHaandler`，实现它们即可完成不同的功能。`Netty`以适配器类的形式提供了大量默认的`ChannelHandler`实现。开发时只需要重写那些想要特殊处理的方法和事件。

> &emsp;&emsp;有一些适配器类可以将编写自定义的`ChannelHanddler`所需要的努力降到最低限度。
>
> **经常会用到的适配器类：**
>
> - `ChannelHandlerAdapter`
> - `ChannelInboundHandlerAdapter`
> - `ChannelOutboundHandlerAdapter`
> - `ChannelDuplexHandler`

&emsp;&emsp;接下来研究三个`ChannelHandler`的子类型：**编码器、解码器 和`SimpleChannelInboundHandler<T>`**——`ChannelInboundHandlerAdapter`的一个子类。



#### 3.2.3.1 编码器和解码器

&emsp;&emsp;当使用Netty发送或者接收一个消息的时候，就会发生数字转换。入站消息会被解码`[字节->另一种格式，通常是一个Java对象]`，出站消息，就会发生相反的转换`[当前格式->字节]`

&emsp;&emsp;对应于特定的需要，Netty为编码器和解码器提供了不同类型的抽象类。通常来说这些基类的名称将类似于`ByteToMessageDecoder`或`MessageToByteEncoder`，对于特殊类型可能有`ProtobufEncoder`和`ProtobufDecoder`——用来预置用来支持`Google`的`Protocol Buffers`。

&emsp;&emsp;当然所有的编码器和解码器都实现了`ChannelOutboundHandler`或者`ChannelInboundHandler`。**对于入站数据来说，ChannelRead方法/事件已经被重写了，对于每个入站`Channel`读取的信息，这个方法都会被调用。随后它将调用预置解码器所提供的`decode()`方法，并且发给下一个`ChannelInboundHandler`。[出站消息的模式是相反方向的]**



#### 3.2.3.2 抽象类SimpleChannelInboundHandler

&emsp;&emsp;该`Handler`有一个范形`<T>`，其代表了待处理消息的`Java`类型。其最重要的方法是`CHannelRead0(ChannelHandlerContext,T)`。除了要求不要阻塞当前`I/O`线程之外，怎么实现完全取决于业务。



## 3.3 引导

&emsp;&emsp;`Netty`的引导类为应用程序的网络层配置提供了容器，这**涉及将一个进程绑定到某个指定的端口，或者将一个进程连接到另一个开放了端口的进程。**【前者即引导服务器[监听传入的连接]，后者为引导一个客户端[建立一个或多个进行的连接]】。

> **面向连接的协议：**
>
> &emsp;&emsp;请记住，严格来说，“连接”这个术语仅适用于面向连接的协议，如`TCP`，其保证了两个连接端点之间消息的有序传递。



&emsp;&emsp;`Netty`中提供了两种引导：一种用于客户端（简单地称为`Bootstrap`），而另一种（`ServerBootstrap`）用于服务器。无论要用于什么场景，只需要考虑是服务端还是客户端。

|       类别       |     `Bootstrap`      | `ServerBootstrap`  |
| :--------------: | :------------------: | :----------------: |
| 网络编程中的作用 | 连接到远程主机和端口 | 绑定到一个本地端口 |
| `EventLoopGroup` |          1           |         2          |

&emsp;&emsp;为什么`SerberBootstrap`需要两个（也可以是同一个实例）？因为服务器需要两组不同的`Channel`。第一组将只包含一个`ServerChannel`，代表服务器自身的已绑定到某个本地端口的正在监听的套接字。而第二组将包含所有已创建的用来处理传入客户端连接的Channel【对于每个服务器已经接受的连接都有一个】。

![image-20200729193132219](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200729193132.png)

&emsp;&emsp;与`ServerChannel`相关联的`EventLoopGroup`将分配一个负责为传入连接请求创建`Channel`的`EventLoop`。一旦连接被接受，第二个`EventLoopGroup`就会给它的`Channel`分配一个`EventLoop`。



# 第四章 传输

主要内容：

- **OIO**：阻塞传输
- **NIO**：异步传输
- **Local**：JVM内部的异步通信
- **Embedded**：用于测试`ChannelHandler`



&emsp;&emsp;流经网络的数据都是**字节**形式，这些字节如何流动是取决于网络传输的【一个帮助我们抽象底层数据传输机制的概念】。

&emsp;&emsp;`Netty`提供了传输实现的一套通用API。这套`API`不仅使用起来比`JDK`简单，而且当切换传输方式时不会收到影响【`OIO -> NIO`】。



## 4.1 案例研究：传输迁移



### 4.1.1 不通过Netty使用OIO和NIO

**1、未使用Netty的阻塞网络编程**

```java
public class PlainOioServer {
    public void serve(int port) throws IOException {
        final ServerSocket socket = new ServerSocket(port);//将服务器绑定指定端口
        try {
            for (;;) {
                final Socket clientSocket = socket.accept();//接受连接
                System.out.println(
                        "Accepted connection from " + clientSocket
                );
                new Thread(() -> {
                    OutputStream out;
                    try {
                        out = clientSocket.getOutputStream();
                        out.write("Hi!\r\n".getBytes(
                                StandardCharsets.UTF_8
                        ));//将消息写给已连接的客户端
                        out.flush();
                        clientSocket.close();//关闭连接
                    } catch (IOException e) {
                        e.printStackTrace();
                    }finally {
                        try {
                            clientSocket.close();
                        } catch (IOException ex) {
                            // ignore on close
                        }
                    }
                }).start();//启动线程
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

&emsp;&emsp;以上代码完全可以处理中等数量的并发客户端。但并不能很好地伸缩到支撑到成千上万的并发接入连接。**想要改写成异步网络编程，就需要重写。**



**未使用Netty的异步网络编程：**

```java
public class PlainNioServer {
    public void serve(int port) throws IOException {
        ServerSocketChannel serverChannel = ServerSocketChannel.open();
        serverChannel.configureBlocking(false);
        ServerSocket ssocket = serverChannel.socket();
        InetSocketAddress address = new InetSocketAddress(port);
        ssocket.bind(address);//将服务器绑定到选定端口
        Selector selector = Selector.open();//打开Selector来处理Channel
        //将ServerSocket注册到selector以接受连接
        serverChannel.register(selector, SelectionKey.OP_ACCEPT);
        final ByteBuffer msg = ByteBuffer.wrap("Hi\r\n".getBytes());
        for (; ; ) {
            try {
                selector.select();//等待需要处理的新事件；阻塞将一直持续到下一个传入事件
            } catch (IOException ex) {
                ex.printStackTrace();
                break;
            }
            //获取所有接收事件的Selection-Key实例
            Set<SelectionKey> readyKeys = selector.selectedKeys();
            Iterator<SelectionKey> iterator = readyKeys.iterator();
            while (iterator.hasNext()) {
                SelectionKey key = iterator.next();
                iterator.remove();
                try {
                    if (key.isAcceptable()) {//检查事件是否是一个新的已经就绪可以被接受的连接
                        ServerSocketChannel server = (ServerSocketChannel) key.channel();
                        SocketChannel client = server.accept();
                        client.configureBlocking(false);
                        client.register(selector,
                                SelectionKey.OP_WRITE | SelectionKey.OP_READ,
                                msg.duplicate());//接受客户端，并将它注册到选择器
                        System.out.println(
                                "Accepted connection from " + client
                        );
                    }
                    if (key.isWritable()) {//检查套接字是否已经准备好写数据
                        SocketChannel client = (SocketChannel) key.channel();
                        ByteBuffer buffer = (ByteBuffer) key.attachment();
                        while (buffer.hasRemaining()) {
                            if (client.write(buffer) == 0) {//将数据写到已经连接的客户端
                                break;
                            }
                        }
                        client.close();//关闭连接
                    }
                } catch (IOException ex) {
                    key.cancel();
                    try {
                        key.channel().close();
                    } catch (IOException cex) {

                    }
                }
            }
        }
    }
}
```



### 4.1.2 通过Netty使用OIO和NIO

**使用Netty的阻塞网络处理：**

```java
public class NettyOioServer {
    public void server(int port) throws Exception {
        final ByteBuf buf = Unpooled.unreleasableBuffer(
                Unpooled.copiedBuffer("Hi!\r\n", StandardCharsets.UTF_8)
        );
        EventLoopGroup group = new OioEventLoopGroup();

        try {
            ServerBootstrap b = new ServerBootstrap();//创建Server-Bootstrap
            b.group(group)
                .channel(OioServerSocketChannel.class)//使用OioEventLoopGroup以允许阻塞模式
                .localAddress(new InetSocketAddress(port))
                .childHandler(
                        //指定ChannelInitializer，对于每个已接受的连接都调用它
                        new ChannelInitializer<SocketChannel>() {
                            @Override
                            protected void initChannel(SocketChannel ch)
                                    throws Exception {
                                ch.pipeline().addLast(
                                        new ChannelInboundHandlerAdapter() {
                                            //添加一个ChannelInboundHandlerAdapter以拦截和处理事件
                                            @Override
                                            public void channelActive(ChannelHandlerContext ctx) throws Exception {
                                                ctx.writeAndFlush(buf.duplicate())
                                                        //将消息写到客户端，并添加ChannelFutureListener，以便消息一被写完就关闭连接
                                                        .addListener(ChannelFutureListener.CLOSE);
                                            }
                                        }
                                );
                            }
                        }
                );
            ChannelFuture f = b.bind().sync();//绑定服务器以接受连接
            f.channel().closeFuture().sync();
        }finally {
            group.shutdownGracefully().sync();//释放所有资源
        }
    }
}
```



**非阻塞的Netty版本：**

```java
public class NettyNioServer {
    public void server(int port) throws Exception {
        final ByteBuf buf = Unpooled.unreleasableBuffer(
                Unpooled.copiedBuffer("Hi!\r\n", StandardCharsets.UTF_8)
        );
        EventLoopGroup group = new NioEventLoopGroup();

        try {
            ServerBootstrap b = new ServerBootstrap();//创建Server-Bootstrap
            b.group(group)
                .channel(NioServerSocketChannel.class)//使用NioEventLoopGroup以允许阻塞模式
                .localAddress(new InetSocketAddress(port))
                .childHandler(
                        //指定ChannelInitializer，对于每个已接受的连接都调用它
                        new ChannelInitializer<SocketChannel>() {
                            @Override
                            protected void initChannel(SocketChannel ch)
                                    throws Exception {
                                ch.pipeline().addLast(
                                        new ChannelInboundHandlerAdapter() {
                                            //添加一个ChannelInboundHandlerAdapter以拦截和处理事件
                                            @Override
                                            public void channelActive(ChannelHandlerContext ctx) throws Exception {
                                                ctx.writeAndFlush(buf.duplicate())
                                                        //将消息写到客户端，并添加ChannelFutureListener，以便消息一被写完就关闭连接
                                                        .addListener(ChannelFutureListener.CLOSE);
                                            }
                                        }
                                );
                            }
                        }
                );
            ChannelFuture f = b.bind().sync();//绑定服务器以接受连接
            f.channel().closeFuture().sync();
        }finally {
            group.shutdownGracefully().sync();//释放所有资源
        }
    }
}
```

&emsp;&emsp;对比发现只有两个位置不用。

![image-20200730160350146](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200730160350.png)

&emsp;&emsp;不管使用那种传输方式，`Netty`暴露的都是相同的`API`，不管选用哪种方式代码几乎都可以不受影响。传输的实现都依赖于`interface Channel、ChannelPipeline、ChannelHandler`。



## 4.2 传输API

![image-20200730160923355](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200730160923.png)

&emsp;&emsp;传输`API`的核心是`Interface Channel`，其被用于所有的`I/O`操作。

**关系说明：**

- 每个`Channel`都会被分配一个`ChannelPipeline`和`ChannelConfig`。
- `ChannelConfig`包含了该`Channel`的所有配置设置，并且支持热更新。
- `Channel`是独一无二的，为了保证顺序将`Channel`声明为`Comparable`的一个子接口。如果两个不同的`Channel`返回了相同的散列码。那么`AbstractChannel`中的`compareTo()`方法的实现会抛出一个`Errot`。
- `ChannelPipeline`持有所有将应用于入站和出站数据以及事件的`ChannelHandler`实例。

**ChannelHandler的典型用途包括：**

- 将数据从一种格式转换为另一种格式；
- 提供异常的通知；
- 提供`Channel`变为活动的活着非活动的通知；
- 提供当`Channel`注册到`EventLoop`或者从`EventLoop`注销时的通知；
- 提供有关用户自定义事件的通知。



> **拦截过滤器：**`ChannelPipeline`实现了一种常见的设计模式——`拦截过滤器(Intercepting Filter)`。Unix管道是另外一个熟悉的例子：多个命令被链接在一起，其中一个命令的输出端将连接到命令行中下一个命令的输入端。



&emsp;&emsp;我们可以根据需要添加或者移除`ChannelHandler`实例来修改`ChannelPipeline`。例如：每当`STARTTLS`协议被请求时，可以向`ChannelPipeline`添加一个适当的`ChannelHandler（SslHandler）`来按需地支持`STARTTLS`协议。



**Channel的方法：**

| 方法名          | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| `eventLoop`     | 返回分配给`Channel`的`EventLoop`                             |
| `pipeLine`      | 返回分配给`Channel`的`ChannelPipeline`                       |
| `isActive`      | 如果`Channel`是活动的，则返回`true`。活动的意义可能依赖于底层的传输。例如，一个`Socket`传输一旦连接到了远程节点便是活动的，而一个`Datagram`传输一旦被打开便是活动的。 |
| `localAddress`  | 返回本地的`SokcetAddress`                                    |
| `remoteAddress` | 返回远程的`SokcetAddress`                                    |
| `write`         | 将数据写到远程节点。这个数据将被传递给`ChannelPipeline`，并且排对知道他被冲刷。 |
| `flush`         | 将之前已写的数据冲刷到底层传输，如一个`Socket`.              |
| `writeAndFlush` | 一个渐便的方法，等同于调用`write()`并接着调用`flush()`.      |



**写到Channel：**

```java
Channel channel = ...;
//创建持有要写数据的ByteBuf
ByteBuf buf = Unpooled.copiedBuffer("your data", StandardCharsets.UTF_8);
ChannelFuture cf = channel.writeAndFlush(buf);//写数据并冲刷它
cf.addListener(new ChannelFutureListener() {//添加ChannelFutureListener 以便在写操作完成后接收通知
    @Override
    public void operationComplete(ChannelFuture future) throws Exception {
        if (future.isSuccess()) {//写操作完成，并且没有错误发生
            System.out.println("write successful");
        }else {
            System.out.println("write error");//记录错误
            future.cause().printStackTrace();
        }
    }
});
```

&emsp;&emsp;`Netty`的`Channel`实现是线程安全的，因此可以存储引用，并且每当需要向远程节点写数据时都可以使用它。即便是有多个线程都在使用。

**多线程使用Channel：**

```java
final Channel channel = ...;
//创建持有要写数据的ByteBuf【可重复使用】
ByteBuf buf = Unpooled.copiedBuffer("your data", StandardCharsets.UTF_8).retain();
Runnable writer = new Runnable() {//创建数据写入的Runnable
    @Override
    public void run() {
        channel.writeAndFlush(buf.duplicate());
    }
};
//获取到线程池Executor的引用
ExecutorService executor = Executors.newCachedThreadPool();
//将任务交由线程池管理
executor.execute(writer);
//再次递交任务
executor.execute(writer);
```



## 4.3 内置的传输

**Netty所提供的传输：**

| 名称       | 包                            | 描述                                                         |
| ---------- | ----------------------------- | ------------------------------------------------------------ |
| `NIO`      | `io.netty.channel.socket.nio` | 使用`java.nio.channels`包作为基础——基于选择器的方式。        |
| `Epoll`    | `io.netty.channel.epoll`      | 由`JNI`驱动的`epoll()`和非阻塞`IO`。这个传输支持只有在`Linux`上可用的多种特性，如`SO_REUSEPORT`，比`NIO`传输更快，而且是完全非阻塞的。 |
| `OIO`      | `io.netty.channel.socket.oio` | 使用`java.net`包作为基础——使用阻塞流                         |
| `Local`    | `io.netty.channel.local`      | 可以在`VM`内部通过管道进行通信的本地传输                     |
| `Embedded` | `io.netty.channel.embedded`   | `Embedded`传输，允许使用`ChannelHandler`而又不需要一个真正的基于网络的传输。这在测试`ChannelHandler`实现时非常有用。 |



### 4.3.1 NIO——非阻塞I/O

&emsp;&emsp;`Netty`的`NIO`传输基于`Java`提供的[异步/非阻塞]网络编程的通用抽象。

&emsp;&emsp;**选择器背后的基本概念是充当一个注册表**，有选择器可以获取`Channel`状态变化后的通知。

**可能存在的状态：**

- 新的`Channel`已被接受并且就绪；
- `Channel`连接已经完成；
- `Channel`有已经就绪的可供读取的数据；
- `Channel`可用于写数据；

&emsp;&emsp;选择器会持续检查状态变化，当状态变化后作出响应。然后选择器会被重置，并重复这个过程。



**选择操作的位模式：**

| 名称         | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| `OP_ACCEPT`  | 请求在接受新连接并创建`Channel`时获得通知                    |
| `OP_CONNECT` | 请求在建立一个连接时获得通知                                 |
| `OP_READC`   | 请求当数据已经就绪，可以从`Channel`中读取时获得通知          |
| `OP_WRITE`   | 请求当可以向`Channel`中写更多的数据时获得通知。这处理了套接字缓冲区被完全填满时的情况，这种情况通常发生在数据的发送速度比远程节点可处理的速度更快的时候。 |

![image-20200730204646126](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200730204653.png)

> **零拷贝：**
>
> &emsp;&emsp;零拷贝(zero-copy)目前只有在使用`NIO`和`Epoll`传输时才可使用的特性。它可以快速高效地将数据从文件系统移动到网络接口，而不再需要从内核空间复制到用户控件。其在`FTP`和`HTTP`这样的协议中可以显著地提高性能。【并不是所有系统都支持，例如实现了数据加密或者压缩的文件系统是不可用的。只能传输文件的原始内容。但传输已加密的文件则不是问题】



### 4.3.2 Epoll——用于Linux的本地非阻塞传输

&emsp;&emsp;Linux作为高性能网络编程的平台，其发展飞快，催生了大量先进特性的开发。这其中就包括了`epoll`——一个高度可扩展的`I/O`事件通知特性。该API在Linux内核版本`2.5.44`(2002)引入，提供了比旧的`POSIX select`和`poll`系统调用更好的性能。同时现在也是`Linux`上非阻塞网络编程的标准。`LINUX NIO API`使用了这些`epoll`调用。

&emsp;&emsp;`Netty`为`Linux`提供了一组`NIO API`，其以一种和它本身的设计更加一致的方式使用`epoll`，并且以一种更加轻量的方式使用中断。当我们的服务运行于Linux系统时，应该优先使用这个版本的传输。其在高负载下它的性能要优于`JDK`的`NIO`实现。

**简单的使用Epoll：**

- `NioEventLoopGroup ——> EpollEventLoopGroup`
- `NioServerSocketChannel.class ——> EpollServerSocketChannel.class`



### 4.3.3 OIO——旧的阻塞I/O

&emsp;&emsp;`Netty`的`OIO`传输实现代表一种折中。其可以通过常规的传输`API`使用，其建立在`java.net`包的阻塞实现之上。其只适用于一些特殊的用途。

&emsp;&emsp;例如：移植使用了一些进行阻塞调用的库（如`JDBC`），而将其逻辑转换为非阻塞的可能也是不切实际的。

**Netty是如何能够使用和用于异步传输相同的API来支持OIO的？**

&emsp;&emsp;其利用了`SO_TIMEOUT`这个Socket标记指定了`操作完成最大毫秒数`.如果在指定的时间内没有完成则会抛出`SocketTimeOut Exception`。`Netty`将捕获这个异常并继续处理循环。在`EventLoop`下次运行时再次尝试。



### 4.3.4 用于JVM内部通信的Local传输

&emsp;&emsp;`Netty`提供了一个`Local`传输，用于在**同一个`JVM`**中运行的客户端和服务器程序之间的异步通信。【这个传输也支持对于所有`Netty`传输实现都共同的`API`】

&emsp;&emsp;在这种传输中，和服务器`Channel`相关联的`SocketAddress`并没有绑定物理网络地址。而是存储到注册表中，并在`Channel`关闭时注销。这个传输并不接受真正的网络流量，所以并不能够和其他传输实现进行互操作。除了必须在同一`JVM`中没有其他限制。

![image-20200731234446720](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200731234446.png)



### 4.3.5 Embedded传输

 &emsp;&emsp;`Netty`提供了一种额外的传输，使得我们可以将一组`ChannelHandler`作为帮助器**嵌入**到其他的`ChannelHandler`。通过这种方式可以扩展一个`ChannelHandler`的功能，而不需要修改其内部代码。

&emsp;&emsp;`Embedded`传输的关键是一个被称为`EmbeddedChannel`的具体的`Channel`实现。



## 4.4 传输的用例

&emsp;&emsp;并不是所有的传输都支持所有的核心协议，其可能会限制你的选择。

**支持的传输和网络协议：**

| 传输             | TCP  | UDP  | SCTP* | UDT  |
| ---------------- | :--: | :--: | :---: | :--: |
| `NIO`            |  X   |  X   |   X   |  X   |
| `Epoll(仅Linux)` |  X   |  X   |   —   |  —   |
| `OIO`            |  X   |  X   |   X   |  X   |



>**在Linux上启动SCTP**
>
>&emsp;&emsp;SCTP需要内核的支持，并且需要安装用户库。
>
>1、对于Ubuntu：
>
>```shell
>sudo apt-get install libsctpl
>```
>
>2、对于Fedora：
>
>```shell
>sudo yum install kernel-modules-extra.x86_64 lksctp-tools.x86_64
>```



**一些可能会用到的用例：**

- **非阻塞代码库**：当我们使用非阻塞调用时在`Linux`上使用`NIO`或者`epoll`始终是个好主意。虽然`NIO/epoll`皆在处理大量的并发连接，但是在处理较小数目的并发连接时也能很好地工作。尤其是考虑到它在链接之间共享线程的方式。
- **阻塞代码库**：当我们的代码库严重的依赖于阻塞`I/O`时，可以尝试将其直接转换为`Netty`的`NIO`传输。也不必直接重写可以分阶段迁移。
- **在同一JVM内部的通信**：在同一JVM内部是不需要暴露网络服务的。可以通过`Local`传输的完美用例。并且在需要暴露网络服务时只需要把传输改为`NIO/OIO`。
- **测试ChannelHandler实现**：当编写`ChannelHandler`实现编写单元测试时，可以考虑`Embedded`传输。这方便测试代码而不需要创建大量的模拟[mock]对象。



**应用程序最佳传输：**

| 应用程序的需求                 | 推荐的传输                    |
| ------------------------------ | ----------------------------- |
| 非阻塞代码库或者一个常规的起点 | `NIO(或者在Linux上使用epoll)` |
| 阻塞代码库                     | `OIO`                         |
| 在同一个JVM内部通信            | `Local`                       |
| 测试`ChannelHandler`的实现     | `Embedded`                    |





# 第五章、ByteBuf

&emsp;&emsp;在网络传输中基本单位总是**字节**。Java NIO提供了`ByteBuffer`作为字节容器[相对复杂]。而在`Netty`中其替代品`ByteBuf`就相对实用并且解决的`JDK API`的局限性。



## 5.1 ByteBuf的API

&emsp;&emsp;Netty的数据处理API通过两个组件暴露——`abstract class ByteBuf`和`interface ByteBufHolder`。



**ByteBuf API的优点：**

- 可以被用户自定义的缓冲区类型扩展；
- 通过内置的复合缓冲区类型实现了透明的零拷贝；
- 容量可以按需增长(类似于JDK的StringBuilder)；
- 在读和写这两种模式之间切换不需要调用`ByteBuffer`的`flip()`方法；
- 读和写使用了不同的索引；
- 支持方法的链式调用；
- 支持引用计数；
- 支持池化。



## 5.2 ByteBuf类——Netty的数据容器

&emsp;&emsp;所有的网络通信都是涉及字节序列的移动，所以高效易用的数据结构明显是必不可少的。`Netty ByteBuf`的实现满足并超越了这些需求。



### 5.2.1 它是如何工作的

&emsp;&emsp;`ByteBuf`维护了两个不同的索引：一个用于读取，一个用于写入。当你从`ByteBuf`读取时，它的`readerIndex`将会被递增已经被读取的字节数。同样地，当你写入`ByteBuf`时，它的`writerIndex`也会被递增。

**一个读索引和写索引都设置为0的16字节ByteBuf：**

![image-20200801223223255](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200801223223.png)

&emsp;&emsp;考虑一下两个索引之间的关系，如果打算读取字节直到`readerIndex`达到`writerIndex`同样的值时会发生什么？

&emsp;&emsp;将会达到“可以读取的”数据的末尾。试图读取超出该点的数据将会触发一个`IndexOutOfBoundsException`。



&emsp;&emsp;名称以`read`或者`write`开头的`ByteBuf`方法，将会推进其对应的索引，而名称以`set`或者`get`开头的操作则不会。后面的这些方法将作为一个参数传入的一个相应索引上执行操作。



&emsp;&emsp;我们可以指定`ByteBuf`的最大容量。试图移动写索引超过这个值将会触发一个异常。默认的限制是`Interger.MAX_VALUE`



### 5.2.2 ByteBuf的使用模式

#### 5.2.2.1 堆缓冲区

&emsp;&emsp;此种方式适合于**有遗留的数据需要处理的情况**。此种方式作为`ByteBuf`最常用的模式，也被称为支撑数组(`backing array`)，其能够在没有池化的情况下提供快速的分配和释放。

```java
ByteBuf heapBuf = ... ;
if (heapBuf.hasArray()) {//检查ByteBuf是否又一个支撑数组
    byte[] array = heapBuf.array();//如果有，则获取对数组的引用
    //计算第一个字节的偏移量
    int offset = heapBuf.arrayOffset() + heapBuf.readerIndex();
    int length = heapBuf.readableBytes();//获得可读字节数
    //使用数组、偏移量和长度作为参数调用你的方法
    heandleArray(array, offset, length);
}
```

> **注意：**当`hasArray()`方法返回`flase`时，尝试访问支撑数组将触发一个`UnsupportedOperationException`。这个模式类似于`JDK`的`ByteBuffer`



#### 5.2.2.2 直接缓冲区

&emsp;&emsp;`JDK1.4`中引入的`ByteBuffer`类允许`JVM`实现通过本地调用来分配内存。这样做避免了在每次调用本地`I/O`操作之 前/后 将缓冲区的内容复制到一个中间缓冲区（或者从中间缓冲区把内容复制到缓冲区）。

&emsp;&emsp;通过`ByteBuf`的Javadoc明确指出：“直接缓冲区的内容将驻留在常规的会被垃圾回收的堆之外”。这也让直接缓冲区成为了网络数据传输的理想选择。如果数据包含在堆分配的缓冲区中，那么在通过套接字发送它之前**`JVM`将会在内部把你的缓冲区复制到一个直接缓冲区中**。 

&emsp;&emsp;直接缓冲区的主要缺点是，相对于基于堆的缓冲区，他们的分配和释放都较为昂贵。当处理遗留代码时，因为数据不在堆上可能要不得不进行一次复制。

```java
ByteBuf directBuf = ...;
if (!directBuf.hasArray()) {//如果不是由数组支撑，那么就是直接缓冲区
    int length = directBuf.readableBytes();//获取可读字节数
    //分配一个新的数组来保存具有该长度的字节数据
    byte[] array = new byte[length];
    //将字节复制到该数组
    directBuf.getBytes(directBuf.readerIndex(), array);
    //使用数组、偏移量和长度作为参数调用比的方法
    handleArray(array, 0, length);
}
```



#### 5.2.2.3 复合缓冲区

&emsp;&emsp;复合缓冲区，它为多个`ByteBuf`提供了一个聚合视图。在这个视图中可以根据需求添加或者删除`ByteBuf`实例。`JDK ByteBuffer`实现完全缺失该特性。

&emsp;&emsp;`Netty`通过`ByteBuf`子类——`CompositeByteBuf`实现了这个模式。其提供了一个将多个缓冲区表示为单个合并缓冲区的虚拟。

> **注意：**当`CompositeByteBuf`中的实例只有一个`ByteBuf`实例时调用`hasArray()`方法是调用该实例的`hasArray()`的值；否则它将返回`false`。



**例子：**想象一下什么场景下会使用复合缓冲区？

&emsp;&emsp;例如说现在由两个部分——**头部和主体**组成的`HTTP`协议传输的信息。这两个部分由不同的模块产生，直到消息被发送的时候组装。当该应用程序可以选择为多个消息重用相同的消息主体时，对于每个消息都会创建一个新的头部。

&emsp;&emsp;我们肯定是不想为每个消息都重新分配这两个缓冲区的，所以使用`CompositeByteBuf`是一个完美的选择。其消除了没必要的复制，又暴露了通用的`ByteBuf API`。

![image-20200803141533786](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200803141539.png)



**使用`JDK ByteBuffer`实现上述需求：**

```java
ByteBuffer header = new ByteBuffer();
ByteBuffer body = new ByteBuffer();
//使用数组保存消息部分
ByteBuffer[] message = new ByteBuffer[]{header, body};
//创建一个新ByteBuffer并复制header和body的副本进行合并

ByteBuffer message2
        = ByteBuffer.allocate(header.remaining() + body.remaining());

message2.put(header);
message2.put(body);
message2.flip();
```



**使用`CompositeByteBuf`：**

&emsp;&emsp;上边的案例的实现效率低下且显得比较笨拙。那么使用`CompositeByteBuf`就要优雅几分了。

```java
CompositeByteBuf messageBuf = Unpooled.compositeBuffer();
ByteBuf headerBuf = Unpooled.buffer();
ByteBuf bodyBuf = Unpooled.buffer();
//将ByteBuf实例追加到CompositeByteeBuf
messageBuf.addComponents(headerBuf, bodyBuf);
messageBuf.removeComponent(0);//删除位置于索引位置为0（第一个组件）的ByteBuf
for (ByteBuf buf : messageBuf) {//循环遍历所有的ByteBuf实例
    System.out.println(buf.toString());
}
```



**访问`CompositeByteBuf`中的数据：**

```java
CompositeByteBuf compBuf = Unpooled.compositeBuffer();
//获取可读字节数
int length = compBuf.readableBytes();
//分配一个具有可读字节数长度的新数组
byte[] array = new byte[length];
//将字节读到该数组中
compBuf.getBytes(compBuf.readerIndex(), array);
//使用偏移量和长度作为参数使用该数组
handleArray(array, 0, array.length);
```



&emsp;&emsp;`Netty`使用了`CompositeByteBuf`来优化套接字的`I/O`操作，尽可能消除了由`JDK`的缓冲区所导致的性能以及内存使用率的尴尬。**这种优化在Netty的核心代码中，并没有暴露出来。**

> `CompositeByteBuf API`除了从`ByteBuf`继承的方法，`CompositeByteBuf`提供了大量的附加功能。



## 5.3 字节级操作

&emsp;&emsp;`ByteBuf`提供了许多超出基本读、写操作的方法用于修改它的数据。



### 5.3.1 随机访问索引

&emsp;&emsp;和普通的`Java`字节数组中一样，`ByteBuf`的索引是从零开始的，最后一个字节的索引总是`capacity() - 1`。

```java
ByteBuf buffer = Unpooled.buffer();
for (int i = 0; i < buffer.capacity(); i++) {
    byte b = buffer.getByte(i);
    System.out.println((char) b);
}
```

> &emsp;&emsp;使用索引值参数方法来访问数据既不会改变`readerIndex`也不会改变`writerIndex`。如果有需要，可以通过调用`readerIndex(index)`或者`writerIndex(index)`来手动移动这两者。



### 5.3.2 顺序访问索引

![image-20200803162615898](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200803162616.png)

&emsp;&emsp;`ByteBuf`同时具有读索引和写索引，`JDK ByteBuffer`却只有一个索引【调用`flip()`的原因】。

#### 5.3.2.1 可丢弃字节

&emsp;&emsp;可丢弃字节即已经被读过的字节，通过调用`discardReadBytes()`方法，可以丢弃它们并回收空间。该分段的初始大小为`0`，存储在`readerIndex`中，会随着`read`操作的执行而增加，`get`操作不会移动`readerIndex`。![image-20200803172832535](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200803172832.png)

&emsp;&emsp;上图就是调用`discardreadBytes()`方法后的结果。**只在需要的时候调用该方法，因为可读字节需要移动到开始位置，很可能导致内存复制。**



#### 5.3.2.2 可读字节

&emsp;&emsp;可读字节分段中存储了实际数据。**新分配的、包装的、复制的**缓冲区的默认`readerIndex`为0。任何名称为`read`或者`skip`开头的操作都将检索或者跳过位于当前`readerIndex`的数据。并且增加其已读字节数。

&emsp;&emsp;如果被调用的方法需要一个`ByteBuf`参数作为写入的目标，并且没有指定目标索引参数，那么该目标缓冲区的`writerIndex`也会被增加，例如：`readBytes(ByteBuf dest)`

&emsp;&emsp;如果尝试在缓冲区的可读字节数已经耗尽时从中读取数据，那么将会引发一个`IndexOutOfBoundsException`。

**读取所有可读的字节：**

```java
ByteBuf buffer = Unpooled.buffer();
while (buffer.isReadable()) {
    System.out.println(buffer.readByte());
}
```



#### 5.3.2.3 可写字节

&emsp;&emsp;可写字节分段是指一个拥有未定义内容的、写入就绪的内存区域。新分配的缓存区的`writerIndex`的默认值为0。任何`write`开头的操作都将从当前的`writerIndex`处开始写数据，并将它增加已经写入的字节数。

&emsp;&emsp;如果写操作的目标也是`ByteBuf`，并且没有指定源索引的值，则缓冲区的`readerIndex`也会被增加相同的大小：`writeBytes(ByteBuf dest)`，超过容量的写入将会引发`IndexOutBoundException`。

**使用随机数写入缓冲区的方法：**

&emsp;&emsp;`writeableBytes()`用来判断是否有足够的空间。

```java
ByteBuf buffer = Unpooled.buffer();
while (buffer.writableBytes() >= 4) {
    buffer.writeInt(random.nextInt());
}
```



### 5.3.3 索引管理

&emsp;&emsp;`JDK InputStream`定义了`mark(int readlimit)`和`reset()`，这些方法分别被用来将流中的当前位置标记为指定的值，以及将流重置到该位置。

&emsp;&emsp;`Netty ByteBuf`也可以通过`markReaderIndex()`、`markWriterIndex()`、`resetWriterIndex()`和`resetReaderIndex`来标记和重置`readerIndex`和`writerIndex`。这些方法只是没有`readlimit`参数。

&emsp;&emsp;如果想要将索引移动到指定位置，那么可以使用`readerIndex(int)`或者`writerIndex(int)`。设置无效位置会导致`IndexOutOfBoundsException`。

&emsp;&emsp;使用`clear()`方法可以将`readerIndex`和`writerIndex`都设置为0。并且不会清楚内存中的内容。

**clear调用之前：**

![image-20200804085207973](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200804085208.png)

**clear调用之后：**

![image-20200804085301843](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200804085302.png)

&emsp;&emsp;`clear()`比调用`discardReadBytes()`轻量得多，因为它将只是重置索引不会复制任何的内存。



### 5.3.4 查找操作

&emsp;&emsp;在`ByteBuf`中有很多用来确定值的索引方法。最简单的是`indexOf()`。较为复杂的查找可以通过哪些需要一个`ByteBufProcessor`作为参数的方法达成。

**该接口只有一个方法：**

```java
boolean process(byte value);//检测输入值是否是正在查找的值
```



&emsp;&emsp;`ByteBufProcessor`针对一些常见的值定义了许多便利的方法。



**针对包含有`NULL`结尾的内容的`Flash`套接字：**

&emsp;&emsp;此方法能简单高效的消费该`Flash`数据，只会在处理期间进行较少的边界检查。

```java
forEachByte(ByteBufProcessor.FIND_NUL)
```

**查找回车符`\r`的例子：**

```java
ByteBuf buffer = ... ;
int index = buffer.forEachByte(ByteBufProcessor.FIND_CR)
```



### 5.3.5 派生缓冲区

&emsp;&emsp;派生缓冲区为`ByteBuf`提供了以专门的方式来**呈现其内容视图**。

**这类视图的创建方法：**

- `duplicate()`
- `slice()`
- `slice(int,int)`
- `Unpooled.unmodifiableBuffer(...)`
- `order(ByteOrder)`
- `readSlice(int)`

&emsp;&emsp;这些方法都将返回一个新的`ByteBuf`实例，其具有自己的**读索引、写索引和标记索引**。其内部存储和`JDK`的`ByteBuffer`一样是**共享的**。

> **ByteBuf复制：**非共享，创建数据副本
>
> - `copy()`
> - `copy(int,int)`



**使用slice(int,int)操作分段【共享】：**

```java
Charset utf8 = StandardCharsets.UTF_8;
//创建一个用于保存给定字符串字节的ByteBuf
ByteBuf buf = Unpooled.copiedBuffer("Netty in action rocks", utf8);
//创建该ByteBuf从索引0 开始到索引15结束的新切片
ByteBuf sliced = buf.slice(0, 15);

//将打印"Netty in Action"
System.out.println(sliced.toString(utf8));
//更新索引0处的字节
buf.setByte(0, (byte) 'J');
//将会成功：因为是共享的
assert buf.getByte(0) == sliced.getByte(0);
```



**使用copy(int,int)操作分段【复制】：**

```java
Charset utf8 = StandardCharsets.UTF_8;
//创建一个用于保存给定字符串字节的ByteBuf
ByteBuf buf = Unpooled.copiedBuffer("Netty in action rocks", utf8);
//创建该ByteBuf从索引0 开始到索引15结束的新切片
ByteBuf copy = buf.copy(0, 15);

//将打印"Netty in Action"
System.out.println(copy.toString(utf8));
//更新索引0处的字节
buf.setByte(0, (byte) 'J');
//将会成功：因为数据是不共享的
assert buf.getByte(0) != copy.getByte(0);
```



### 5.3.6 读/写操作

&emsp;&emsp;一共有两种类型的读/写操作：

- `get()`和`set()`操作，从给定的索引开始，并且保持索引不变；
- `read()`和`write()`操作，从给定的索引开始，并且会根据已经访问过的字节数对索引进行调整。

**`get()`操作：**

| 名称                     | 描述                                               |
| ------------------------ | -------------------------------------------------- |
| `getBoolean(int)`        | 返回给定索引处的`Boolean`值                        |
| `getByte(int)`           | 返回给定索引处的字节                               |
| `getUnsignedByte(int)`   | 将给定索引处的无符号字节值作为`short`返回          |
| `getMedium(int)`         | 返回给定索引处的24位的中等`int`值                  |
| `getUnsignedMedium(int)` | 返回给定索引处的无符号24位的中等`int`值            |
| `getInt(int)`            | 返回给定索引处的`int`值                            |
| `getUnsignedInt(int)`    | 将给定索引处的无符号`int`值作为`long`返回          |
| `getLong(int)`           | 返回给定索引处的`long`值                           |
| `getShort(int)`          | 返回给定索引处的`short`值                          |
| `getUnsignedShort(int)`  | 将给定索引处的无符号`short`值作为`int`返回         |
| `getBytes(int,...)`      | 将该缓冲区中从给定索引开始的数据传送到指定的目的地 |

**`set()`操作：**

| 名称                             | 描述                              |
| -------------------------------- | --------------------------------- |
| `setBoolean(int,boolean)`        | 设定给定索引处的`Boolean`值       |
| `setByte(int index,int value)`   | 设定给定索引处的字节值            |
| `setMedium(int index,int value)` | 设定给定索引处的24位的中等`int`值 |
| `setInt(int index,int value)`    | 设定给定索引处的`int`值           |
| `setLong(int index,long value)`  | 设定给定索引处的`long`值          |
| `setShort(int index,int value)`  | 设定给定索引处的`short`值         |



**get/set使用案例：**

```java
Charset utf8 = StandardCharsets.UTF_8;
//创建一个新的ByteBuf以保存给定字符串的字节
ByteBuf buf = Unpooled.copiedBuffer("Netty in Action rocks!", utf8);
//打印第一个字符'N'
System.out.println((char) buf.getByte(0));
//存储当前的readerIndex 和 writerIndex
int readerIndex = buf.readerIndex();
int writerIndex = buf.writerIndex();

//将索引0处的字节更新为字符'B'
buf.setByte(0, (byte) 'B');
//打印第一个字符，现在是'B'
System.out.println((char) buf.getByte(0));

//将会成功，因为这些操作并不会修改相应的索引
assert readerIndex == buf.readerIndex();
assert writerIndex == buf.writerIndex();
```



**`read()`操作：**

| 名称                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `readBoolean()`                                              | 返回当前`readerIndex`处的`Boolean`，并将`readerIndex`增加1   |
| `readByte()`                                                 | 返回当前`readerIndex`处的字节，并将`readerIndex`增加1        |
| `readUnsignedByte()`                                         | 将当前`readerIndex`处的无符号字节值作为`short`返回，并将`readerIndex`增加1 |
| `readMedium()`                                               | 返回当前`readerIndex`处的24位的中等`int`值，并将`readerIndex`增加3 |
| `readUnsignedMedium()`                                       | 返回当前`readerIndex`处的24位的无符号的中等`int`值，并将`readerIndex`增加3 |
| `readInt()`                                                  | 返回当前`readerIndex`的`int`值，并将`readerIndex`增加4       |
| `readUnsignedInt()`                                          | 返回当前`readerIndex`处的无符号`int`值作为`long`值返回，并将`readerIndex`增加4 |
| `readLong()`                                                 | 将当前`readerIndex`处的`long`值，并将`readerIndex`增加8      |
| `readShort()`                                                | 返回当前`readerIndex`处的`short`值，并将`readerIndex`增加2   |
| `readUnsignedShort()`                                        | 将当前`readerIndex`处的无符号`short`值作为`int`值返回，并将`readerIndex`增加2 |
| `readBytes(ByteBuf|byte[] destination,int dstIndex[,int length])` | 将当前`ByteBuf`中从当前`readerIndex`处开始的（如果设置了，length长度的字节）数据传送到一个目标`ByteBuf`或者`byte[]`，从目标的`desIndex`开始的位置。本地的`readerIndex`将被增加到已经传输的字节数。 |



**`write()`写操作：**

| 名称                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `writeBoolean(boolean)`                                      | 在当前的`writeIndex`处写入一个`Boolean`，并将`writerIndex`增加1 |
| `writeByte(int)`                                             | 在当前`writerIndex`处写入一个字节值，并将`writerIndex`增加1  |
| `writeMedium(int)`                                           | 在当前`writerIndex`处写入一个中等的`int`值，并将`weiterIndex`增加3 |
| `writeInt(int)`                                              | 在当前`writerIndex`处写入一个`int`值，并将`writerIndex`增加4 |
| `writeLong(long)`                                            | 在当前`writerIndex`处写入一个`long`值，并将`writerIndex`增加8 |
| `writeShort(int)`                                            | 在当前`writerIndex`处写入一个`short`值，并将`writerIndex`增加2 |
| `writeBytes(sourceByreBuf|byte[] [,int srcIndex,int length])` | 从当前`writerIndex`开始，传输来自于指定源(`ByteBuf`或者`byte[]`)的数据。如果提供了`srcIndex`和`length`，则从`srcIndex`开始读取，并且处理长度为`length`的字节。当前`writerIndex`将会被增加所写入的字节数。 |

**read和write操作：**

```java
Charset utf8 = StandardCharsets.UTF_8;
//创建一个新的ByteBuf以保存给定字符串的字节
ByteBuf buf = Unpooled.copiedBuffer("Netty in Action rocks!", utf8);
//打印第一个字符'N'
System.out.println((char) buf.readByte());
//存储当前的readerIndex 和 writerIndex
int readerIndex = buf.readerIndex();
int writerIndex = buf.writerIndex();
//将"?"追加到缓冲区
buf.writeByte((byte) '?');

System.out.println(buf.toString(utf8));//打印：etty in Action rocks!?
assert readerIndex == buf.readerIndex();
assert writerIndex != buf.writerIndex();//将会成功，因为read/write方法将会移动index
```



### 5.3.7 更多的操作

| 名称              | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| `isReadable()`    | 如果至少有一个字节可供读取，则返回`true`                     |
| `isWritable()`    | 如果至少有一个字节可被写入，则返回`true`                     |
| `readableBytes()` | 返回可被读取的字节数                                         |
| `writableBytes()` | 返回可被写入的字节数                                         |
| `capacity()`      | 返回`ByteBuf`可容纳的字节数。在此之后，他会尝试再次扩展知道达到`maxCapacity()` |
| `maxCapacity()`   | 返回`ByteBuf`可以容纳的最大字节数                            |
| `hasArray()`      | 如果`ByteBuf`由一个字节数组支撑，则返回`true`                |
| `array()`         | 如果`ByteBuf`有一个字节数组支撑则返回该数组；否则，它将抛出一个`UnsupportedOperationException`异常 |



## 5.4 ByteBufHolder接口

&emsp;&emsp;在我们进行数据传输时会发现，除了数据本身外，我们还得存储各种属性值。例如`HTTP`响应除了内容还有状态码、cookie等/

&emsp;&emsp;为了处理这种常见的用例，`Netty`提供了`ByteBufHolder`。其为`Netty`提供了很多高级特性。**如缓冲区池化，从池中借用`ByteBuf`，并且在需要时自动释放。**

&emsp;&emsp;`ByteBufHolder`只有几种用于访问底层数据和引用计数的方法。【不包含继承自`ReferenceCounted`】

| 名称          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| `content()`   | 返回由这个`ByteBufHolder`所持有的`ByteBuf`                   |
| `copy()`      | 返回这个`ByteBufHolder`的一个深拷贝，包括一个其所包含`ByteBuf`的非共享拷贝。 |
| `duplicate()` | 返回这个`ByteBufHolder`的一个浅拷贝，包扩一个其所包含的`ByteBuf`的共享拷贝。 |



## 5.5 ByteBuf分配

#### 5.5.1 按需分配：ByteBufAllocator接口

&emsp;&emsp;`ByteBufAllocator`接口可以实现`ByteBuf`的池化，它可以用来分配我们所描述过的任何类型的`ByteBuf`实例。因为池化的存在降低了分配和释放内存的开销。其不会以任何方式改变`ByteBuf API`。

**ByteBufAllocator提供的方法：**

| 名称                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `buffer()`<br/>`buffer(int initialCapacity)`<br/>`buffer(int initialCapacity,int maxCapacity)` | 返回一个基于堆或者直接内存存储的`ByteBuf`                    |
| `heapBuffer()`<br />`heapBuffer(int initialCapacity)`<br />`heapBuffer(int initialCapacity,int maxCapacity())` | 返回一个基于对内存储存的`ByteBuf`                            |
| `directBuffer()`<br />`directBuffer(int initialCapacity)`<br />`directBuffer(int initialCapacity,int maxCapacity())` | 返回一个基于直接内存存储的`ByteBuf`                          |
| `compositeBuffer()`<br />`compositeBuffer(int maxNumComponents)`<br />`compositeDirectBuffer()`<br />`compositeDirectBuffer(int maxNumComponents)`<br />`compositeHeapBuffer()`<br />`compositeHeapBuffer(int maxNumComponents)` | 返回一个可以通过添加最大到指定数目的基于堆或者直接内存存储的缓冲区来扩展的`CompositeByteBuf` |
| `ioBuffer()`                                                 | 返回一个用于套接字的`I/O`操作的`ByteBuf`                     |

&emsp;&emsp;**可以通过`Channel`(每个都有不同的`ByteBufAllocator`实例)或者绑定到`ChannelHanandler`的`Context`获取一个`ByteBufAllocator`的引用。**

```java
Channel channel = ...;
//从Channel获取一个到ByteBufAllocator的引用
ByteBufAllocator allocator = channel.alloc();

ChannelHandlerContext ctx = ...;
//从ChannelHandlerContext获取一个到ByteBufAllocator的引用
ByteBufAllocator alloc = ctx.alloc();
```



&emsp;&emsp;`Netty`提供了两种`ByteBufAllocator`的实现：

- `PooledByteBufAllocator`：池化ByteBuf的实例，提高了性能并最大限度地减少内存碎片。此实现使用了一种**jemalloc**高效分配内存的方法，
- `UnpooledByteByfAllocator`：不池化，每个调用返回一个新的实例。

> `Netty`默认使用了`PolledByteBufAllocator`，我们可以在`ChannelConfig API`或者引导程序中指定一个不同的分配器来更改。





####  5.5.2 Unpooled缓冲区

&emsp;&emsp;`Unpooled`用于在不能获取到`ByteBufAllocator`引用的情况下用来创建未池化的`ByteBuf`实例。

| 名称                                                         | 描述                                        |
| ------------------------------------------------------------ | ------------------------------------------- |
| `buffer()`<br/>`buffer(int initialCapacity)`<br/>`buffer(int initialCapacity,int maxCapacity)` | 返回一个未池化的基于堆内存存储的`ByteBuf`   |
| `directBuffer()`<br />`directBuffer(int initialCapacity)`<br />`directBuffer(int initialCapacity,int maxCapacity())` | 返回一个未池化的基于直接内存存储的`ByteBuf` |
| `wrappedBuffer()`                                            | 返回一个包装了给定数据的`ByteBuf`           |
| `copiedBuffer()`                                             | 返回一个复制了给定数据的`ByteBuf`           |

> &emsp;&emsp;`Unpooled`类还使得`ByteBuf`同样可用于那些并不需要`Netty`的其他组件的非网络项目，使得其能得益于高性能的可扩展的缓冲区`API`。





#### 5.5.3 ByteBufUtil类

&emsp;&emsp;`ByteBufUtil`提供了用于操作`ByteBuf`的静态辅助方法。这个`API`是通用的，并且和池化无关，所以这些方法已然在分配类的外部实现。

&emsp;&emsp;在这些静态方法中最有价值的应该是`hexdump()`方法，它以十六进制的表示形式打印`ByteBuf`的内容。【以后将会发现十六进制有非常多的好处】

&emsp;&emsp;另外一个有用的方法就是`boolean equals(ByteBuf,ByteBuf)`，其用来判断两个`ByteBuf`实例的相等性。

&emsp;&emsp;当我们有了自己的`ByteBuf`子类，就会发现`ByteBufUtil`的其他有用方法了。



## 5.6 引用计数

&emsp;&emsp;引用计数是一种通过在某个对象所持有的资源**不再被其他对象引用时释放该对象所持有的资源来优化内存使用和性能的技术**。

&emsp;&emsp;`Netty`在第4版中为`ByteBuf`和`ByteBufHolder`引入了引用计数技术，它们都实现了`interface ReferenceCounted`。

&emsp;&emsp;引用计数就是跟踪某个特定对象的活动引用的数量。一个`ReferenceCounted`实现的实例将通常以活动的引用计数为1作为开始。只要引用计数大于0，就能保证对象不会被释放。当活动引用的数量减少到0时，该实例就会被释放。【确切语义可能是特定于实现的】已经释放的对象大部分是不可再用的。



&emsp;&emsp;引用计数对于池化实现（`PooledByteBufAllocator`）来说至关重要。

```java
Channel channel = ...;
//从Channel获取一个到ByteBufAllocator的引用
ByteBufAllocator allocator = channel.alloc();

//从ByteBufAllocator分配一个ByteBuf
ByteBuf byteBuf = allocator.directBuffer();
//检查引用计数是否为预期的1
assert byteBuf.refCnt() == 1;
```

**释放引用计数的对象：**

```java
ByteBuf byteBuf = 。。。;
//检查引用计数是否为预期的1
assert byteBuf.refCnt() == 1;
//减少该对象的活动引用，当减少到0时该对象被释放，并且该方法返回true
boolean release = byteBuf.release();
```

&emsp;&emsp;试图访问一个已经被释放的引用计数对象，将会导致一个`IllegalReferenceCountException`。一个特定的`ReferenceCounted`实现类，可以用它自己的独特方式来定义他的计数规则。

> 谁负责释放呢，一般来说由最后访问对象的那一方来负责将它释放。





# 第六章 ChannelHandler和ChannelPipeline

&emsp;&emsp;此章我们将讲解`ChannelHandler`和`ChannelPileline`如何链接在一起去处理逻辑，将会研究这些类的各种用例，以及一个重要的关系`ChannelHandlerContext`。



## 6.1 C hannelHandler家族

### 6.1.1 Channel的生命周期

&emsp;&emsp;`interface Channel `定义了一组和`ChannelInboundHandler API`密切相关的简单但功能强大的状态模型。

**如下：**

|         状态          |                             描述                             |
| :-------------------: | :----------------------------------------------------------: |
| `ChannelUnregistered` |         `Channel`已经被创建，但还未注册到`EventLoop`         |
|  `ChannelRegistered`  |              `Channel`已经被注册到了`EventLoop`              |
|    `ChannelActive`    | `Channel`处于活动状态(已经连接到他的远程节点)。其现在可以接受和发送数据了。 |
|   `ChannelInactive`   |                 `Channel`没有连接到远程节点                  |

![image-20200824183838144](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200824183845.png)

&emsp;&emsp;`Channel`的正常生命周期如上图，当这些状态发生改变时，将会生成对应的事件。这些事件将会被转发给`ChannelPipeline`中的`ChannelHandler`，其可以随后对它们作出响应。



### 6.1.2 ChannelHandler的生命周期

&emsp;&emsp;以下列出了`interface ChannelHandler`定义的生命周期操作，在`ChannelHandler`被添加到`ChannelPipeline`中或者从`ChannelPipeline`中移除时会调用这些操作。这些方法中的每一个都接受一个`ChannelHandlerContext`参数。

| 类型              | 描述                                                  |
| ----------------- | ----------------------------------------------------- |
| `handlerAdded`    | 当把`ChannelHandler`添加到`ChannelPipeline`中时被调用 |
| `handlerRemoved`  | 当从`ChannelPipeline`中移除`ChannelHandler`时被调用   |
| `exceptionCaught` | 当处理过程中在`ChannelPipeline`中有错误产生时被调用   |

**Netty定义的两个重要的`ChannelHandler`子接口：**

- `ChannelInboundHandler`：处理入站数据以及各种状态变化；
- `ChannelOutboundHandler`：处理出站数据并且允许拦截所有的操作;



### 6.1.3 ChannelInboundHandler接口

&emsp;&emsp;以下是`interface ChannelInboundHandler`的生命周期。这些将会在数据被接收时或者与其对应的`Channel`状态发生改变时被调用。【这些方法和`Channel`的生命周期密切相关】

|            类型             | 描述                                                         |
| :-------------------------: | :----------------------------------------------------------- |
|     `channelRegistered`     | 当`channel`已经注册到它的`EventLoop`并且能够处理`I/O`时被调用。 |
|    `channelUnregistered`    | 当`Channel`从它的`EventLoop`注销并且无法处理任何`I/O`时被调用。 |
|       `channelActive`       | 当`channel`处于活动状态时被调用;`channel`已经连接/绑定并且已经就绪。 |
|      `channelInactive`      | 当`Channel`离开活动状态并且不再连接它的远程节点时被调用      |
|    `channelReadComplete`    | 当`Channel`上的一个读操作完成时被调用                        |
|        `channelRead`        | 当从`Channel`读取数据时被调用                                |
| `ChannelWritabilityChanged` | 当`Channel`的可写状态发生改变时被调用，<br />用户可以确保写操作不会完成得太快（以避免发生`OutOfMemoryError`）<br />或者可以在`Channel`变为再次可写时恢复写入。<br />可以通过调用`Channel`得`isWritable()`方法<br />来检测`Channel`的可读性。与可写性相关的阀值可以通过<br />`Channel.config().setWriteHighWaterMark()`<br />和`Channel.config().setWriteLowWaterMark()`方法来设置。 |
|    `userEventTriggered`     | 当<br />`ChannelnboundHandler.fireUserEventTriggered()`方法被调用时被调用，因为一个`POJO`被传经了`ChannelPipeline` |

&emsp;&emsp;当某个`ChannelInboundHandler`的实现重写`channelRead()`方法时，**它将负责显式地释放与池化的`ByteBuf`实例相关的内存。**`Netty`为此提供了一个实用方法`ReferenceCountUtil.release()`。

```java
@Sharable
public class DiscardHandler 
        extends ChannelInboundHandlerAdapter {//扩展ChannelInboundHandlerAdapter
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg)
            throws Exception {//丢弃已接收的消息
        ReferenceCountUtil.release(msg);
    }
}
```

&emsp;&emsp;`Netty`将使用`WARN`级别的**日志消息记录未释放的资源**，使得可以非常简单地在代码中发现违规的实例。但是这种方式管理资源可能很繁琐。这时我们可以考虑使用`SimpleChannelInboundHandler`。

```java
@Sharable
public class SimpleDiscardHandler
        extends SimpleChannelInboundHandler {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Object msg)
            throws Exception {
        //不需要任何的显示资源释放
    }
}
```

&emsp;&emsp;由于`SimpleChannelInboundHandler`会自动释放资源，所以**不应该存储指向任何消息的引用供将来实用。因为这些引用都会失效。**



### 6.1.4 ChannelOutboundHandler接口

&emsp;&emsp;出站操作和数据将由`ChannelOutboundHandler`处理。其方法将被`Channel`、`ChannelPipeline`以及`ChannelHandlerContext`调用。

&emsp;&emsp;`ChannelOutboundHandler`有一个强大功能，可以按需推迟操作或者事件。这意味着可以通过一些复杂的方法来处理请求。【例如：远程节点的写入被暂停了，那么可以推迟冲刷操作并在稍后继续】



**该接口定义的方法【忽略ChannelHandler】：**

| 类型                                                         | 描述                                                  |
| ------------------------------------------------------------ | ----------------------------------------------------- |
| `bind(ChannelHandlerContext,SocketAddress,ChannelPromise)`   | 当请求将`Channel`绑定到本地地址时被调用。             |
| `connect(ChannelHandlerContext,SocketAddress, SocketAddress,ChannelPromise)` | 当请求将`Channel`连接到远程节点时被调用。             |
| `disconnet(ChannelHandlerContext,ChannelPromise)`            | 当请求将`Channel`从远程节点断开时被调用。             |
| `close(ChannelHandlerContext,ChannelPromise)`                | 当请求关闭`Channel`时被调用。                         |
| `deregister(ChannelHandlerContext,ChannelPromise)`           | 当请求将`Channel`从它的`EventLoop`注销时被调用。      |
| `read(ChannelHandlerContext)`                                | 当请求从`Channel`读取更多数据时被调用。               |
| `flush(ChannelHandlerContext)`                               | 当请求通过`Channel`将入队数据冲刷到远程节点时被调用。 |
| `write(ChannelHandlerContext,object,ChannelPromise)`         | 当请求通过`Channel`将数据写到远程节点时被调用。       |

> **ChannelPromise和ChannelFuture：**
>
> &emsp;&emsp;`ChannelOutboundHandler`中大部分方法都需要一个`ChannelPromise`参数，以便在操作完成时得到通知。`ChannelPromise`是`ChannelFuture`的一个子类，其定义了一些可写的方法，如`setSuccess()`和`setFailure()`。从而使`ChannelFuture`不可变。



### 6.1.5 ChannelHandler适配器

&emsp;&emsp;可以使用`ChannelInboundHandlerAdapter`和`ChannelOutboundHandlerAdapter`作为自己的`ChannelHandler`的起始点。**这两个适配器分别提供了`ChannelInboundHandler`和`ChannelOutboundHandler`的基本实现。**

![image-20200825223853839](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200825223901.png)

&emsp;&emsp;大致的结构如上图，`ChannelHandlerAdapter`还提供了`isSharable()`。如果其对应实现被标注为`Sharable`，那么这个方法将返回`true`即可以被添加到多个`ChannelPipeline`中。

&emsp;&emsp;在`ChannelInboundHandlerAdapter`和`ChannelOutboundHandlerAdapter`中所提供的方法体调用了其相关联的`ChannelHandlerContext`上的等效方法，从而将事件转发到了`ChannelPipeline`中的下一个`ChannelHandler`中。



### 6.1.6 资源管理

&emsp;&emsp;每当通过调用`ChannelInboundHandler.channelRead()`或者`ChannelOutboundHandler.write()`方法来处理数据时，**都需要确保没有任何的资源泄漏**。

&emsp;&emsp;还有就是`Netty`通过引用计数来处理池化的`ByteBuf`，所以完全使用完成某个`ByteBuf`后，调整其引用计数是很重要的。



&emsp;&emsp;`Netty`为了帮助用户诊断潜在的（资源泄漏）问题，其提供了`ResourceLeakDetector`，其将会对应用程序的缓冲区分配做**大约1%的采样来检测内存泄漏。**相关的开销是非常小的。

**日志打印样本：**

```log
LEAK: ByteBuf.release() was not called before it's garbage-collected. Enable
advanced leak reporting to find out where the leak occurred. To enable
advanced leak reporting, specify the JVM option
'-Dio.netty.leakDetectionLevel=ADVANCED' or call
ResourceLeakDetector.setLevel().
```

**Netty目前定义的4种泄漏检测级别：**

| 级别       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| `DISABLED` | 禁止泄漏检测。只有在详尽的测试之后才应该设置为这个值。       |
| `SIMPLE`   | 使用`1%`的默认采样率检测并报告任何发现的泄漏，默认级别，适合绝大部分的情况。 |
| `ADVANED`  | 使用默认的采样率，报告所发现的任何泄漏以及对应的消息被访问的位置。 |
| `PARANOID` | 类似于`ADVANCED`，但是其将会对每次（对消息）访问都进行采样。这对性能将会有很大的影响，应该只在调试阶段使用。 |

**设置级别的方式：**

```shell
java -Dio.netty.leakDetectionLevel=ADVANCED
```

&emsp;&emsp;携带JVM选项重启应用将会看到最近被泄漏的缓冲区被访问的位置。下面是一个典型的由单元测试产生的泄漏报告：

```shell
Running io.netty.handler.codec.xml.XmlFrameDecoderTest
15:03:36.886 [main] ERROR io.netty.util.ResourceLeakDetector - LEAK:
　　 ByteBuf.release() was not called before it's garbage-collected.
Recent access records: 1
#1: io.netty.buffer.AdvancedLeakAwareByteBuf.toString(
　　AdvancedLeakAwareByteBuf.java:697)
io.netty.handler.codec.xml.XmlFrameDecoderTest.testDecodeWithXml(
　　XmlFrameDecoderTest.java:157)
io.netty.handler.codec.xml.XmlFrameDecoderTest.testDecodeWithTwoMessages(
　　XmlFrameDecoderTest.java:133)
...
```

&emsp;&emsp;实现`ChannelInboundHandler.channelRead()`和`ChannelOutboundHandler.write()`方法时使用这个诊断工具来防治泄漏。

&emsp;&emsp;`channelRead()`操作直接消费入站信息的情况，也就是说不会通过`ChannelHandlerContext.fireChannelRead()`方法将入站信息转发给下一个`ChannelInboundHandler`。

**消费并释放入站消息：**

```java
@Sharable
public class DiscardHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg)
            throws Exception {
        ReferenceCountUtil.release(msg);//通过该方法释放资源
    }
}
```

> SimpleChannelInboundHandler会自动释放资源



**丢弃并释放出站消息：**

```java
@Sharable
public class DiscardOutboundHandler extends ChannelOutboundHandlerAdapter {
    @Override
    public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
        ReferenceCountUtil.release(msg);//通过该方法释放资源
        promise.setSuccess();//通知promise数据已经被处理了
    }
}
```

&emsp;&emsp;注意出站消息处理后需要通知`ChannelPromise`，否则可能会出现`ChannelFutureListener`收不到某个消息已经被处理了的通知。

&emsp;&emsp;总之，如果一个消息被消费或者丢弃了，并且没有传递给`ChannelPipeline`中的下一个`ChannelOutboundHandler`，那么用户就有责任调用`ReferenceCountUtil.release()`来释放资源。如果消息到达了实际的传输层，那么当他被写入时或者`Channel`关闭时，都会被自动释放。





## 6.2 ChannelPipeline接口

&emsp;&emsp;`ChannelPipeline`是一个**拦截流经**`Channel`的入站和出站事件的`ChannelHandler`**实例链**。这些`ChannelHandler`之间交互就组成了一个应用数据和事件处理逻辑的核心。

&emsp;&emsp;每新建一个`Channel`都会被分配一个新的`ChannelPipeline`。这项关联是永久性的；`Channel`既不能附加另一个`ChannelPipeline`，也不能与当前`Pipeline`进行分离。**在Netty组件的生命周期中，这是一项固定的操作。不需要开发人员的任何干预。**

&emsp;&emsp;**根据事件的起源，事件会将`ChannelInboundHandler`或`ChannelOutboundHandler`处理。随后，通过调用`ChannelHandlerCOntext`实现，它将被转发给同一超类型的下一个`ChannelHandler`。**

> &emsp;&emsp;`ChannelHandlerContext`使得`ChannelHandler`能够和它的`ChannelPipeline`以及其他的`ChannelHandler`交互。`ChannelHandler`可以通知其所属的`ChannelPipeline`中的下一个`ChannelHandler`，甚至可以动态修改他所属的`ChannelPipeline`。
>
> &emsp;&emsp;`ChannelHandlerContext`具有丰富的用于处理事件和执行`I/O`操作的`API`。

![image-20200827213805049](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20200827213805.png)

&emsp;&emsp;上图展示了一个典型的同时具有入站和出站`ChannelHandler`的`ChannelPipeline`的布局。`ChannelPipeline`还提供了通过本身传播事件的方法。如果一个入站事件被触发，它将被从`ChannelPipeline`的头部开始一直被传播到尾端。一个出站`I/O`事件将从`ChannelPipeline`的最右边开始，然后向左传播。

> **ChannelPipeline相对论：**
>
> &emsp;&emsp;从事件途径`ChannelPipeline`的角度来看，`ChannelPipeline`的头部和尾端取决于该事件是入站的还是出站的。在`Netty`中总是将`ChannelPipeline`的入站口作为头部【左侧】，而出站口作为尾端【右侧】
>
> &emsp;&emsp;当完成了通过调用`ChannelPipeline.add*()`方法将入站处理器(`ChannelInboundHandler`)和出站处理器(`ChannelOutboundHandler`)混合添加到`ChannelPipeline`之后，每一个`ChannelHandler`从头部到尾端的顺序位置就是添加时的顺序。**这些处理器从左到右进行编号，第一个入站事件看到的`ChannelHandler`将是1，而第一个出站事件看到的`ChannelHandler`将是5。**



&emsp;&emsp;`ChannelPipeline`传播事件规则：其会测试`Pipeline`中的**下一个`ChannelHandler`的类型是否和事件的运动方向相匹配。**如果不匹配将会跳过该`ChannelHandler`并前进到下一个，知道找到和该事件所期望的方向相匹配为止。**[部分Handler可能同时实现了出入站的接口]**



































