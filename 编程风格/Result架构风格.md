> 此资源来源百度，加上了一点自己的理解，如果只想迅速上手开发，直接看完统一接口也就行了

#### RESTful 架构风格概述

&emsp;&emsp;在移动互联网的大潮下，随着docker等技术的兴起，『微服务』的概念也越来越被大家接受并应用于实践，日益增多的web service逐渐统一于RESTful 架构风格，如果开发者对RESTful 架构风格不甚了解，则开发出的所谓RESTful API总会貌合神离，不够规范。 



#### 1.Restful架构风格

&emsp;&emsp;RESTful架构风格最初由Roy T. Fielding（HTTP/1.1协议专家组负责人）在其2000年的博士学位论文中提出。HTTP就是该架构风格的一个典型应用。从其诞生之日开始，它就因其可扩展性和简单性受到越来越多的架构师和开发者们的青睐。一方面，随着云计算和移动计算的兴起，许多企业愿意在互联网上共享自己的数据、功能；另一方面，在企业中，RESTful API（也称RESTful Web服务）也逐渐超越SOAP成为实现SOA的重要手段之一。时至今日，RESTful架构风格已成为企业级服务的标配。 

> Restful风格推出之后,就因其扩展性和简单性收到了青睐.
>
> REST即Representational State Transfer的缩写，可译为"表现层状态转化”。



##### 1.1RESTful架构风格的特点****

1. **资源**

&emsp;&emsp;所谓"资源"，就是网络上的一个实体，或者说是网络上的一个具体信息。它可以是一段文本、一张图片、一首歌曲、一种服务，总之就是一个具体的实在。资源总要通过某种载体反应其内容，文本可以用txt格式表现，也可以用HTML格式、XML格式表现，甚至可以采用二进制格式；图片可以用JPG格式表现，也可以用PNG格式表现；JSON是现在最常用的资源表示格式。 

&emsp;&emsp;结合我的开发实践，我对资源和数据理解如下： 

&emsp;&emsp;资源是以json(或其他Representation)为载体的、面向用户的一组数据集，资源对信息的表达倾向于概念模型中的数据：

-  资源总是以某种Representation为载体显示的，即序列化的信息
- **常用的Representation是json(推荐)或者xml（不推荐）等**
- Represntation 是REST架构的表现层

> 资源就是一些文本啊,图片啊.当然我们使用最多的json。【此处应考虑更改说明】
>
> Represntation ：这个单词文章中拼错了。表现的意思



2. **统一接口**

&emsp;&emsp;RESTful架构风格规定，数据的元操作，即CRUD(create, read, update和delete,即数据的增删查改)操作，分别对应于HTTP方法：GET用来获取资源，POST用来新建资源（也可以用于更新资源），PUT用来更新资源，DELETE用来删除资源，这样就统一了数据操作的接口，仅通过HTTP方法，就可以完成对数据的所有增删查改工作。 

- GET（SELECT）：从服务器取出资源（一项或多项）。
- POST（CREATE）：在服务器新建一个资源。
- PUT（UPDATE）：在服务器更新资源（客户端提供完整资源数据）。
- PATCH（UPDATE）：在服务器更新资源（客户端提供需要修改的资源数据）。
- DELETE（DELETE）：从服务器删除资源。

> 进行增删改查时按照规定使用请求方式



3. **URI**

&emsp;&emsp;可以用一个URI（统一资源定位符）指向资源，即每个URI都对应一个特定的资源。要获取这个资源，访问它的URI就可以，因此URI就成了每一个资源的地址或识别符。 

&emsp;&emsp;一般的，每个资源至少有一个URI与之对应，最典型的URI即URL。 

> 就是使用链接访问。【不清楚理解是否正确】



4. **无状态**

&emsp;&emsp;所谓的无状态，即所有的资源可以用过URI定位，而这个定位与其他资源无关，也不会因为其他资源的改变而改变。

&emsp;&emsp;例如：查询员工工工资是，如果查询是需要登陆系统的，这就是有状态。

&emsp;&emsp;&emsp;&emsp;&emsp;而不需要登录，只需要一个链接（URI）即可获取信息，就是无状态。

> 由一个url与之对应，可以通过HTTP中的`GET`方法得到资源，这是典型的RESTful风格。 



##### 1.2 ROA、SOA、REST与RPC

&emsp;&emsp;`ROA`即Resource Oriented Architecture（面向资源架构），RESTful 架构风格的服务是围绕`资源`展开的，是典型的ROA架构（虽然“A”和“架构”存在重复，但说无妨），虽然ROA与SOA并不冲突，甚至把ROA看做SOA的一种也未尝不可，但由于RPC也是SOA，比较久远一点点论文、博客或图书也常把SOA与RPC混在一起讨论，因此，RESTful 架构风格的服务通常被称之为ROA架构，很少提及SOA架构，以便更加显式的与RPC区分。 

&emsp;&emsp;RPC风格曾是Web Service的主流，最初是基于XML-RPC协议（一个远程过程调用（remote procedure call，RPC)的分布式计算协议），后来渐渐被SOAP协议（简单对象访问协议（Simple Object Access Protocol））取代；RPC风格的服务，不仅可以用`HTTP`，还可以用`TCP`或其他通信协议。但RPC风格的服务，受开发服务采用语言的束缚比较大，如.NET框架中，开发web service的传统方式是使用WCF，基于WCF开发的服务即RPC风格的服务，使用该服务的客户端通常要用C#来实现，如果使用python或其他语言，很难实现可以直接与服务通信客户端；进入移动互联网时代后，RPC风格的服务很难在移动终端使用，而RESTful风格的服务，由于可以直接以`json`或`xml`为载体承载数据，以HTTP方法为统一接口完成数据操作，客户端的开发不依赖于服务实现的技术，移动终端也可以轻松使用服务，这也加剧了REST取代RPC成为web service的主导。 

> - ROA：现象资源
> - SOA：面向服务
> - REST：表述性状态传递
> - RPC： 远程过程调用 
> - WCF:Windows Communication Foundation微软开发的一系列数据通信的应用程序框架
>
> 基本上无用的内容跟，不看也罢。

![RPC风格](.\images\RPC-service.png)

![RESTful](.\images\RESTful-service.png)



##### 1.3 本真REST与hybrid风格

- 本真REST ：即我上文阐述的RESTful架构风格，具有上述的4个特点，是真正意义上的RESTful风格 。
- hybrid风格 ：只是借鉴了RESTful的一些优点，具有一部分RESTful的特点，但对外依然宣称是RESTful风格的服务。 
  - 主流用法：使用Get方法获取资源，用Post实现资源的创建、修改和删除。
  - 主要用于：在原有的RPC风格的服务上，包装一层Restful的外壳。

>开发RESTful 服务，如果没有历史包袱，不建议使用hybrid风格。 



#### 2.认证机制

&emsp;&emsp;通过介绍了解到，Restful风格是无状态的，这时认证机制就尤为重要。因为被访问资源很可能就是隐私资源。这时如果没有认证随意访问是很危险的。

&emsp;&emsp;因此需要认证机制和权限机制（不能让所有登录用户查看所有人的信息）。常用的认证机制包括 `session auth`(即通过用户名密码登录)，`basic auth`，`token auth`和`OAuth`，服务开发中常用的认证机制为后三者。 



##### 2.1 Basic Auth

&emsp;&emsp;Basic Auth是配合RESTful API 使用的最简单的认证方式，只需提供用户名密码即可，但由于有把用户名密码暴露给第三方客户端的风险，在生产环境下被使用的越来越少。因此，在开发对外开放的RESTful API时，尽量避免采用Basic Auth .



##### 2.2 Token Auth

&emsp;&emsp;Token Auth并不常用，它与Basic Auth的区别是，不将用户名和密码发送给服务器做用户认证，而是向服务器发送一个事先在服务器端生成的token来做认证。因此Token Auth要求服务器端要具备一套完整的Token创建和管理机制，该机制的实现会增加大量且非必须的服务器端开发工作，也不见得这套机制足够安全和通用，因此Token Auth用的并不多。



##### 2.3 OAuth

&emsp;&emsp;OAuth（开放授权）是一个开放的授权标准，允许用户让第三方应用访问该用户在某一web服务上存储的私密的资源（如照片，视频，联系人列表），而无需将用户名和密码提供给第三方应用。

&emsp;&emsp;OAuth允许用户提供一个令牌，而不是用户名和密码来访问他们存放在特定服务提供者的数据。每一个令牌授权一个特定的第三方系统（例如，视频编辑网站)在特定的时段（例如，接下来的2小时内）内访问特定的资源（例如仅仅是某一相册中的视频）。这样，OAuth让用户可以授权第三方网站访问他们存储在另外服务提供者的某些特定信息，而非所有内容。

&emsp;&emsp;正是由于OAUTH的严谨性和安全性，现在OAUTH已成为RESTful架构风格中最常用的认证机制，和RESTful架构风格一起，成为企业级服务的标配。

&emsp;&emsp;目前OAuth已经从OAuth1.0发展到OAuth2.0，但这二者并非平滑过渡升级，OAuth2.0在保证安全性的前提下大大减少了客户端开发的复杂性，因此，Gevin建议在实战应用中采用OAuth2.0认证机制。