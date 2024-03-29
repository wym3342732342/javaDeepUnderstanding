# DUBBO3

![image-20210706131703678](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20210706131709.png)

## 一、简介

![image-20210706131744310](https://gitee.com/tizo_kingbb/picImg/raw/master/img/20210706131744.png)

### 1.1 官方简介

&emsp;&emsp;`Apache Dubbo`是一款微服务开发框架，它提供了**RPC通信**与**微服务治理**两大关键能力。这意味着，使用 `Dubbo` 开发的微服务，将具备相互之间的**远程发现**与**通信能力**， 同时利用 `Dubbo` 提供的丰富**服务治理**能力，可以实现诸如**服务发现**、**负载均衡**、**流量调度**等服务治理诉求。同时 `Dubbo` 是**高度可扩展**的，用户几乎可以*在任意功能点去定制自己的实现*，以*改变框架的默认行为来满足自己的业务需求*。

&emsp;&emsp;`Dubbo3` 基于 `Dubbo2` 演进而来，在保持原有核心功能特性的同时， `Dubbo3` 在**易用性、超大规模微服务实践、云原生基础设施适配**等几大方向上进行了全面升级。 以下文档都将基于 `Dubbo3`展开。



### 1.2 什么是Dobbo3

&emsp;&emsp;如开篇所述，`Dubbo` 提供了构建**云原生微服务业务的一站式解决方案**，可以使用 `Dubbo` **快速定义并发布微服务组件**，同时基于 `Dubbo` 开箱即用的丰富特性及超强的扩展能力，构建运维整个微服务体系所需的各项**服务治理能力**，如 `Tracing`、`Transaction` 等。

**`Dubbo` 提供的基础能力包括：**

- 服务发现
- 流式通信
- 负载均衡
- 流量治理
- 。。。



&emsp;&emsp;`Dubbo` 计划提供丰富的多语言客户端实现，其中 `Java`、`Golang` 版本是**当前稳定性、活跃度最好的版本**，其他多语言客户端[]正在持续建设中。



&emsp;&emsp;自开源以来，`Dubbo` 就被一众大规模互联网、IT公司选型，经过多年企业实践积累了大量经验。`Dubbo3` 是站在巨人肩膀上的下一代产品，它汲取了上一代的优点并针对已知问题做了大量优化，因此，`Dubbo` 在**解决业务落地与规模化实践方面有着无可比拟的优势**：

- **开箱即用：**
  - 易用性高，如 Java 版本的面向接口代理特性能实现本地透明调用；
  - 功能丰富，基于原生库或轻量扩展即可实现绝大多数的微服务治理能力；
- **超大规模微服务集群实践：**
  - 高性能的跨进程通信协议；
  - 地址发现、流量治理层面，轻松支持百万规模集群实例；
- **企业级微服务治理能力：**
  - 服务测试；
  - 服务Mock；

&emsp;&emsp;`Dubbo3` 是在云原生背景下诞生的，使用 `Dubbo` 构建的微服务**遵循云原生思想**，能更好的复用底层云原生基础设施、**贴合云原生微服务架构**。这体现在：

- 服务支持部署在容器、`Kubernetes`平台，服务生命周期可实现与平台调度周期对齐；
- 支持经典 `Service Mesh` 微服务架构，引入了 `Proxyless Mesh` 架构，进一步简化 `Mesh` 的落地与迁移成本，提供更灵活的选择；
- 作为桥接层，支持与 `SpringCloud`、`gRPC` 等异构微服务体系的互调互通



### 1.3 一站式为服务解决方案

&emsp;&emsp;`Dubbo` 提供了从**服务定义**、**服务发现**、**服务通信**到**流量管控**等几乎所有的服务治理能力，并且尝试从使用上对用户屏蔽底层细节，以提供更好的**易用性**。

&emsp;**&emsp;定义服务在 `Dubbo` 中非常简单与直观，可以选择使用与某种语言绑定的方式（如 `Java` 中可直接定义 `Interface`）**，也可以使用`Protobuf IDL` 语言中立的方式。无论选择哪种方式，站在服务消费方的视角，**都可以通过 `Dubbo` 提供的透明代理直接编码**。

> &emsp;&emsp;需要注意的是，在 `Dubbo` 中，我们提到服务时，通常是指 `RPC` 粒度的、提供某个具体业务增删改功能的接口或方法，与一些微服务概念书籍中泛指的服务并不是一个概念。

&emsp;&emsp;点对点的服务通信是 `Dubbo` 提供的另一项基本能力，`Dubbo` 以`RPC` 的方式将请求数据（`Request`）发送给后端服务，并接收服务端返回的计算结果（`Response`）。`RPC` 通信对用户来说是完全透明的，**使用者无需关心请求是如何发出去的、发到了哪里，每次调用只需要拿到正确的调用结果就行。**同步的 `Request-Response` 是默认的通信模型，它最简单但却不能覆盖所有的场景，因此，**`Dubbo` 提供更丰富的通信模型：**

- 消费端异步请求(`Client Side Asynchronous Request-Response`)
- 提供端异步执行（`Server Side Asynchronous Request-Response`）
- 消费端请求流（`Request Streaming`）
- 提供端响应流（`Response Streaming`）
- 双向流式通信（`Bidirectional Streaming`）



&emsp;&emsp;**`Dubbo` 的服务发现机制**，让微服务组件之间可以独立演进并任意部署，消费端可以在无需感知对端部署位置与 `IP` 地址的情况下完成通信。`Dubbo` 提供的是 `Client-Based` 的服务发现机制，使用者可以有多种方式启用服务发现：

- 使用独立的注册中心组件，如 `Nacos`、`Zookeeper`、`Consul`、`Etcd` 等；
- 将**服务的组织与注册交给底层容器平台**，如 `Kubernetes`，这被理解是一种**更云原生**的方式；



&emsp;&emsp;**透明地址发现让 `Dubbo` 请求可以被发送到任意 `IP` 实例上**，这个过程中**流量被随机分配**。当需要对流量进行更丰富、更细粒度的管控时，就可以用到 **`Dubbo` 的流量管控策略**，`Dubbo` 提供了包括**负载均衡、流量路由、请求超时、流量降级、重试**等策略，基于这些基础能力可以轻松的实现更多场景化的路由方案，包括金丝雀发布、A/B测试、权重路由、同区域优先等，更酷的是，Dubbo 支持流控策略在运行态动态生效，无需重新部署。



