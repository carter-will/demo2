# Spring Cloud

## 基本概念

### 微服务 

微服务英文名称Microservice，Microservice架构模式就是将整个Web应用组织为一系列小的Web服务。这些小的Web服务可以独立地编译及部署，并通过各自暴露的API接口相互通讯。它们彼此相互协作，作为一个整体为用户提供功能，却可以独立地进行扩展。

> 简而言之，微服务架构的风格，就是将单一程序开发成一个微服务，每个微服务运行在自己的进程中，并使用轻
> 量级机制通信，通常是 HTTP RESTFUL API 。这些服务围绕业务能力来划分构建的，并通过完全自动化部署机制来
> 独立部署 这些服务可以使用不同的编程语言，以及不同数据存储技术，以保证最低限度的集中式管理。

**微服务优势**

- 将 个复杂的业务分解成若干小的业务，每个业务拆分成 个服务，服务的边界明确，将复杂的问题简单 。服务按照业务拆分 编码也是按照业务来拆分，代码的可读性和可扩展性增加。
- 微服务系统的微服务单元有很强的横向扩展能力。
- 服务与服务之问通过 HTTP 网络通信协议来通信，单个微服务内部高度祸合，服务与服务之间完全独立，无调合。
- 由于微服务系统是按照业务进行拆分的，并且有坚实的服务边界，所以重写某个服务就相当于重写某 个业务的代码，非常简单。
- 微服务的每个服务单元都是独立部署的，即独立运行在某个进程里，大大减少了测试和部署的时间。
- 微服务在 CAP 理论中采用的是 AP 架构，即具有高可用和分区容错的特点 。高可用主要体现在系统 x 24 小时不间断的服务，它要求系统有大量的服务器集群，从而提高了系统的负载能力。另外，分区容错也使得系统更加健壮。

> **CAP**
>
> CAP原则又称CAP定理，指的是在一个分布式系统中， Consistency（一致性）、 Availability（可用性）、Partition tolerance（分区容错性），三者不可得兼。
>
> CAP原则是NOSQL数据库的基石.
>
> - C（一致性）：所有的节点上的数据时刻保持同步
> - A（可用性）：每个请求都能接受到一个响应，无论响应成功或失败
> - P（分区容错）：系统应该能持续提供服务，即使系统内部有消息丢失（分区）
>
> 常见应用：
>
> - CA without P：如果不要求P（不允许分区），则C（强一致性）和A（可用性）是可以保证的。但其实分区不是你想不想的问题，而是始终会存在，因此CA的系统更多的是允许分区后各子系统依然保持CA。
> - CP without A：如果不要求A（可用），相当于每个请求都需要在Server之间强一致，而P（分区）会导致同步时间无限延长，如此CP也是可以保证的。很多传统的数据库分布式事务都属于这种模式。
> - AP wihtout C：要高可用并允许分区，则需放弃一致性。一旦分区发生，节点之间可能会失去联系，为了高可用，每个节点只能用本地数据提供服务，而这样会导致全局数据的不一致性。现在众多的NoSQL都属于此类。

**微服务不足**

- 微服务的复杂度
- 分布式事务（两阶段提交）
- 服务的划分（领域驱动设计）
- 服务的部署

*对于微服务来说，它是 SOA 种实现，但是它比 ESB实现的 SOA 更加轻便、敏捷和简单。*

*在微服务架构中，有 大难题，那就是服务故障的传播性、服务的划分和分布式事务。*

---

微服务的九大特性：

- 服务组件化
- 按业务组织团队
- 做“产品”的态度
- 智能端点与哑管道
- 去中心化治理
- 去中心化管理数据
- 基础设施自动化（测试和部署）
- 容错设计
- 演进式设计

----

### Spring Cloud

springCloud是基于SpringBoot的一整套实现微服务的框架。他提供了微服务开发所需的配置管理、服务发现、断路器、智能路由、微代理、控制总线、全局锁、决策竞选、分布式会话和集群状态管理等组件。

相关组件架构图

![](http://img.ccblog.cn/img/20170118/181.jpg)

Spring Cloud 子项目包含：

- Spring Cloud Config ： 配置管理开发工具包，可以让你把配置放到远程服务器，目前支持本地存储、Git以及Subversion。
- Spring Cloud Bus ： 事件、消息总线，用于在集群（例如，配置变化事件）中传播状态变化，可与Spring Cloud Config联合实现热部署。
- Spring Cloud Netflix：针对多种Netflix组件提供的开发工具包，其中包括Eureka、Hystrix、Zuul、Archaius等。
- Netflix Eureka：云端负载均衡，一个基于 REST 的服务，用于定位服务，以实现云端的负载均衡和中间层服务器的故障转移。
- Netflix Hystrix：容错管理工具，旨在通过控制服务和第三方库的节点,从而对延迟和故障提供更强大的容错能力。
- Netflix Zuul：边缘服务工具，是提供动态路由，监控，弹性，安全等的边缘服务。
- Netflix Archaius：配置管理API，包含一系列配置管理API，提供动态类型化属性、线程安全配置操作、轮询框架、回调机制等功能。
- Spring Cloud for Cloud Foundry：通过Oauth2协议绑定服务到CloudFoundry，CloudFoundry是VMware推出的开源PaaS云平台。
- Spring Cloud Sleuth：日志收集工具包，封装了Dapper,Zipkin和HTrace操作。
- Spring Cloud Data Flow：大数据操作工具，通过命令行方式操作数据流。
- Spring Cloud Security：安全工具包，为你的应用程序添加安全控制，主要是指OAuth2。
- Spring Cloud Consul：封装了Consul操作，consul是一个服务发现与配置工具，与Docker容器可以无缝集成。
- Spring Cloud Zookeeper：操作Zookeeper的工具包，用于使用zookeeper方式的服务注册和发现。
- Spring Cloud Stream：数据流操作开发包，封装了与Redis,Rabbit、Kafka等发送接收消息。
- Spring Cloud CLI：基于 Spring Boot CLI，可以让你以命令行方式快速建立云组件。
- Netflix Ribbon ：提供云端负载均衡，有动作负载均衡策略可供选择，可配合服务发现和断路器使用
- Netflix Turbine ：Turbine是聚合服务器发送事件流数据的一个工具，用来监控集群下hystrix的metrics情况。
- OpenFeign： Feign是一种申明式、模板化的HTTP客户端。
- Spring Cloud Task： 提供云端计划任务管理、任务调度。
- Spring Cloud Connectors：便于云端应用程序在各种PaaS平台连接到后端，如：数据库和消息代理服务。
- Spring Cloud Cluster： 提供Leadership选举，如：Zookeeper,Redis,Hazelcast,Consul等常见状态模式的抽象和实现
- Spring Cloud Starters ：Spring Boot式的启动项目，为Spring Cloud提供开箱即用的依赖管理。

#### SpringCloud特点

1. 约定优于配置
2. 开箱即用、快速启动
3. 适用于各种环境
4. 轻量级的组件
5. 组件支持丰富，功能齐全

> 各核心组件功能：
>
> - **Eureka**  ： 服务注册与发现  Eureka Client和Eureka Server
> - **Feign**  ： 构造信息发起请求
> - **Ribbon** ：负载均衡选择应用服务
> - **Hystrix**  ：隔离、熔断以及降级
> - **Zuul** ：微服务网关，负责网络路由，做统一的降级、限流、认证授权、安全


“服务”是 个独立运行的单元组件，每个单元组件运行在独立的进程中，组件与组件之间通常使用 HTTP 这种轻量级的通信机制进行通信。

微服务具有以下的特点：

- [x] 按照业务来划分服务，单个服务代码量小，业务单 ，易于维护。
- [x] 每个微服务都有自己独立的基础组件，例如数据库、 缓存等，且运行在独立的进程中。
- [x] 微服务之间的通信是通过 HTTP 协议或者消息组件，且具有容错能力。
- [x] 微服务有 套服务治理的解决方案，服务之间不相合，可以随时加入和剔除服务。
- [x] 单个微服务能够集群化部署，并且有负载均衡的能力。
- [x] 整个微服务系统应该有 个完整的安全机制，包括用户验证、权限验证、资源保护等。
- [x] 整个微服务系统有链路追踪的能力。
- [x] 有一套完整的实时日志



## 服务的注册与发现

> 服务注册是在指向服务注册中心注册一个服务实例，服务提供者将自己的服务信息（如服务名、 IP 地址等〉告知服务注册中心。服务发现是指当服务消费者需要消费另外一个服务时，服务注册中心能够告知服务消费者它所要消费服务的实例信息（如服务名、IP地址等〉。通常情况下，一个服务既是服务提供者，也是服务消费者。服务消费者 般使用 HTTP协议或者消息组件这种轻盘级 的通 机制来进行服务消费。
>
> 服务注册中心不但需要定时接收每个服务的心跳（用来检查服务是否可用），而且每个服务会定期获取服务注册列表的信息，当服务实例数量很多时，服务注册中心承担了非常大的负载。由于服务注册中心在微服务系统中起到了至关重要的作用，所以必须实现高可用。 一般的做法是将服务注册中心集群化，每个服务注册中心的数据实时同步，

### Eureka

主要包括三种角色：

- Register Service ：服务注册中心，它是一个 Eureka Server ，提供服务注册和发现的功能。
- Provider Service ：服务提供者，它是一个Eureka Client ，提供服务。
- Consumer Service ：服务消费者，它是一个Eureka Cient ，消费服务。

Eureka概念：

- Register：服务注册
  当Eureka客户端向Eureka Server注册时，它提供自身的元数据，比如IP地址、端口，运行状况指示符URL，主页等。

  如果Eureka Client在配置文件中没有配置ServiceId,则默认为配置文件中配置的服务名，即${spring.application.name}的值

- Renew：服务续约
  Eureka客户会每隔30秒发送一次心跳来续约。 通过续约来告知Eureka Server该Eureka客户仍然存在，没有出现问题。 正常情况下，如果Eureka Server在90秒没有收到Eureka客户的续约，它会将实例从其注册表中删除。 建议不要更改续约间隔。

- Fetch Registries：获取注册列表信息
  Eureka客户端从服务器获取注册表信息，并将其缓存在本地。客户端会使用该信息查找其他服务，从而进行远程调用。该注册列表信息定期（每30秒钟）更新一次。每次返回注册列表信息可能与Eureka客户端的缓存信息不同， Eureka客户端自动处理。如果由于某种原因导致注册列表信息不能及时匹配，Eureka客户端则会重新获取整个注册表信息。 Eureka服务器缓存注册列表信息，整个注册表以及每个应用程序的信息进行了压缩，压缩内容和没有压缩的内容完全相同。Eureka客户端和Eureka 服务器可以使用JSON / XML格式进行通讯。在默认的情况下Eureka客户端使用压缩JSON格式来获取注册列表的信息。

- Cancel：服务下线
  Eureka客户端在程序关闭时向Eureka服务器发送取消请求。 发送请求后，该客户端实例信息将从服务器的实例注册表中删除。该下线请求不会自动完成，它需要调用以下内容：DiscoveryManager.getInstance().shutdownComponent()；

- Eviction 服务剔除
  在默认的情况下，当Eureka客户端连续90秒没有向Eureka服务器发送服务续约，即心跳，Eureka服务器会将该服务实例从服务注册列表删除，即服务剔除。


**构建高可用的Eureka Server集群**

[参考](https://blog.csdn.net/tuposky/article/details/78343401?fps=1&locationNum=6)

---

​:question:   为什么 Eureka Client 获取服务实例这么慢

​:a:     主要是存在以下几种延迟：

- Eureka Client 注册延迟
- Eureka Server 的响应缓存
- Eureka Client 缓存
- LoadBalancer的缓存

？ Eureka的自我保护

​:a:   默认情况下，如果Eureka Server在一定时间内没有接收到某个微服务实例的心跳，Eureka Server将会注销该实例（默认90秒）。但是当网络分区故障发生时，微服务与Eureka Server之间无法正常通信，这就可能变得非常危险了----因为微服务本身是健康的，此时本不应该注销这个微服务。

Eureka Server通过“自我保护模式”来解决这个问题----当Eureka Server节点在短时间内丢失过多客户端时（可能发生了网络分区故障），那么这个节点就会进入自我保护模式。一旦进入该模式，Eureka Server就会保护服务注册表中的信息，不再删除服务注册表中的数据（也就是不会注销任何微服务）。当网络故障恢复后，该Eureka Server节点会自动退出自我保护模式。

自我保护模式是一种对网络异常的安全保护措施。使用自我保护模式，而已让Eureka集群更加的健壮、稳定。

Spring Cloud 自我保护模式默认是开启的，如需关闭，可以配置：

```
eureka.server.enable-self-preservation=false
```



## 服务消费者

**【 服务的负载均衡 】** 

所有的服务都向服务注册中心注册，服务注册中心持有每个服务的应用名和 IP 地址等信息，同时每个服务也会获取所有服务注册列表信息。服务消费者集成负载均衡组件，该组件会为服务消费者获取服务注册列表信息，并每隔一段时间重新刷新获取该列表。当服务消费者消费服务时，负载均衡组件获取服务提供者所有实例的注册信息，并通过一定的负载均衡策略（开发者可以配置〉，选择一个服务提供者的实例 ，向该实例进行服务消费，这样就实现了负载均衡。

### Ribbon+RestTemplate

RestTemplate是Spring Resources中一个访问第三方RESTful API接口的网络请求框架。RestTemplate的设计原则和其他Spring Tempalte(例如JdbcTemplate、JmsTemplate)类似，都是为执行复杂任务提供了一个具有默认行为的简单方法。

RestTemplate是用来消费REST服务的，所以RestTemplate的主要方法都与REST的Http协议的一些方法紧密相连，例如HEAD、GET、POST、PUT、DELETE和OPTIONS等方法，这些方法在RestTemplate类对应的方法为headForHeaders()、getForObject()、postForObject()、put()和delete()等。

负载均衡是指将负载分摊到多个执行单元上，常见的负载均衡有两种方式。一种是独立进程单元，通过负载均衡策略，将请求转发到不同的执行单元上，例如 Ngnix 。另一种是将负载均衡逻辑以代码的形式封装到服务消费者的客户端上，服务消费者客户端维护了一份服务提供的信息列表，有了信息列表，通过负载均衡策略将请求分摊给多个服务提供者，从而达到负载均衡的目的。

在Spring Cloud 构建的微服务系统中， Ribbon 作为服务消费者的负载均衡器，有两种使用方式，一种是和 RestTemplate 相结合，另一种是和 Feign 相结合。Feign 已经默认集成了 Ribbon。

在Spring Cloud项目中，负载均衡器 Ribbon 会默认从Eureka Client 的服务注册列表中获取服务的信息，并缓存一份。根据缓存的服务注册列表信息，可以通过 LoadBalancerClient 来选择不同的服务实例，从而实现负载均衡。如果禁止 Ribbon Eureka 获取注册列表信息，则需要自己去维护一份服务注册列表信息。 根据自己维护服务注册列表的信息， Ribbon也可以实现负载均衡。

**LoadBalancerClient **  

综上所述 Ribbon的负载均衡主要是通过 LoadBalancerClient 来实现的，而LoadBalancer-Client 具体交给了 ILoadBalancer 来处理，ILoadBalancer 通过配置IRule、IPing ，向 EurekaClient获取注册列表的信息，默认每10 秒向 EurekaClient 发送一次“ping ”， 进而检查是否需要更新服务的注册列表信息。最后 ，在得到服务注册列表信息后， ILoadBalancer 根据IRule的 策略进行负载均衡。

而RestTemplate 加上@LoadBalance 注解后，在远程调度时能够负载均衡， 主要是维护了一个被@LoadBalance 注解的 RestTemplate 列表，并给该列表中 RestTemplate 对象添加了拦截器。在拦截器的方法中 ，将远程调度方法交给了 Ribbon的负载均衡器 LoadBalancerClient去处理
，从而达到了负载均衡的目的。

### Feign

Feign采用了声明式AP接口的风格，将Java Http客户端绑定到它的内部。

**FeignClient**

@Feign注解用于创建声明式API接口，该接口是RESTful风格的。Feign被设计成插拔式的，可以注入其他组件和Feign一起使用。

```
@AliasFor("name")
String name() default "";  //被调用的服务的ServiceId
@AliasFor("value")
String value() default "";  //被调用的服务的ServiceId
String url() default ””;    //硬编码的Url地址
boolean decode404() default false;    //即404时的处理，是被解码，还是抛异常
Class<?>[] configuration() default { }; //指明FeignClient的配置类
Class<?> fallback () default void.class; //配置熔断器的处理类
Class<?> fallbackFactory() default vo.class; 
String path() default "" ; 
boolean primary() default true ; 
```

Feign Client 默认的配置类为 FeignClientsConfiguration.

设置FeignClient 请求失败重试的策略，，通过覆盖了默认的 Retryer Bean

> Feign 是一个伪 Java Http 客户端， Feign 不做任何的请求 理。 Feign 通过处理注解生成Request 模板，从而简化了 Http API的开发。开发人员可以使用注解的方式定制 Request API 模板。在发送 HtφRequest 请求之前，Feign 通过处理注解的方式替换掉 Request 模板中的参数，生成真正的 Request ，并交给 Java Http 客户端去处理。利用这种方式，开发者只需要关注 Feign注解模板的开发，而不用关注 Http 请求本身，简化了 Http 请求的过程，使得 Http 请求变得简单和容易理解。

---

​:question: 在Feign中使用其他http工具包

在Feign中，最终发送Request请求以及接收Response响应都是由Client组件完成的。Client在Feign 源码中是一个接口，在默认的情况下，Client的实现类是 Client.Default, Client.Default 是由 HttpURLConnnection 来实现网络请求的。另外，Client还支持 HttpClient和OkhHttp 来进行网络请求。

需要在配置文件application.yml 中配置 feign.httpclient.enable为true ，从@ConditionalOnProperty 注解可知 ，这个配置可以不写，因为在默认的情况下就为true。

并且需要在pom文件引入依赖，feign-httpclient和feign-okhttp

:question:Feign如何实现负载均衡

负载均衡最终由loadBalancerContext执行,即Ribbon中的内容。

## 断路器

### Hystrix

为了解决分布式系统的雪崩效应，分布式系统引进了熔断器机制。

当一个服务的处理用户请求的失败次数在一定时间内小于设定的阀值时，熔断器处于关闭状态，服务正常;当服务处理用户请求的失败次数大于设定的阀值时，说明服务出现了故障，打开熔断器，这时所有的请求会执行快速失败，不执行业务逻辑。当处于打开状态的熔断器时一段时间后会处于半打开状态，并执行一定数量的请求,剩余的请求会执行快速失败，若执行的请求失败了，则继续打开熔断器;若成功了，则将熔断关闭。

这种机制有着非常重要的意义，它不仅能够防止系统的“雪崩”效应，还具有以下作用:

- 将资源进行隔离,如果某个服务里的某个 API 接口出现了故障，只会隔离该 API口。不会影响到其他 API 接口。被隔离的 API 接口会执行快速失败的逻辑，不会等待，请求不会阻塞。如果不进行这种隔离,请求会一直处于阻塞状态, 直到超时 。若有大量的请求同时涌入，都处于阻塞的状态，服务器的线程资源，迅速被消耗完。
- 服务降级的功能。当服务处于正常的状态时，大量的请求在短时间内同时涌入，超过了服务的处理能力，这时熔断器会被打开，将服务降级，以免服务器因负载过高而出现故障。
- 自我修复能力。当因某个微小的故障，例如网络服务商的问题，导致网络在短时间内不可用，熔断器被打开。如果不能自我监控、自我检测和自我修复，那么需要开发人员手动地去关闭熔断器，无疑会增加开发人员的工作量。

Hystrix的设计原则如下：

- 防止单个服务的故障耗尽整个服务的Servlet容器（例如Tomcat）的线程资源
- 快速失败机制，如果某个服务出现了故障，则调用该服务的请求快速失败，而不是线程等待
- 提供回退方案（fallback），在请求发生故障时，提供设定好的回退方案。
- 使用熔断机制，防止故障扩散到其他服务
- 提供熔断器的监控组件Hystrix Dashboard,可以实时监控熔断器的状态。

---

:question:在Feign中开启熔断器

需要在application.yml中配置：

```
feign:
	hystrix:
		enabled: true
```

### Hystrix Dashboard组件

在微服务架构中为了保证程序的可用性，防止程序出错导致网络阻塞，出现了断路器模型。断路器的状况反应了一个程序的可用性和健壮性，它是一个重要指标。Hystrix Dashboard是作为断路器状态的一个组件，提供了数据监控和友好的图形化界面。

### Hystrix Turbine 

Hystrix Turbine将每个服务Hystrix Dashboard数据进行了整合。

## 路由网关

微服务系统通过将资源以 API 接口的形式暴露给外界来提供服务。在微服务系统中， API接口资源通常是由服务网关（也称 API网关）统一暴露，内部服务不直接对外提供 API 资源的暴露。

网关的作用：

- 协议转换，路由转发
- Zuul、Ribbon 以及 Eureka 相结合，可以实现智能路由和负载均衡的功能， Zuul 能够将请求流量按某种策略分发到集群状态的多个服务实例。网关将所有服务的 API 接口统一聚合，并统一对外暴露。外界系统调用API 接口时，都是由网关对外暴露的API 接口，外界系统不需要知道微服务系统中各服务相互调用的复杂性。微服务系统也保护了其内部微服务单元的 API 接口，防止其被外界直接调用，导致服务的敏感信息对外暴露。
- 流量聚合，对流量进行监控，日志输出，对请求进行记录。
- 作为整个系统的前端工程，对流量进行控制，有限流的作用。可以用来实现流量监控，在高流量的情况下，对服务进行降级。
- 作为系统的前端边界，外部流量只能通过网关才能访问系统。API 接口从内部服务分离出来，方便做测试。
- 可以在网关层做权限的判断。网关服务可以做用户身份认证和权限认证，防止非法请求操作API 接口，对服务器起到保护作用。
- 可以在网关层做缓存

### Zuul

#### zuul工作原理

Zuul 是通过 Servlet 来实现的， Zuul 通过自定义的 ZuulServlet （类似于 Spring MVC的DispatcServlet 〕来对请求进行控制。 Zuul 的核心是一系列过滤器，可以在 Http 请求的发起和响应返回期间执行一系列的过滤器。 Zuul 包括以下4种过滤器：

- PRE 过滤器：它是在请求路由到具体的服务之前执行的，这种类型的过滤器可以做安全验证，例如身份验证、 参数验证等。
- ROUTING 过滤器：它用于将请求路由到具体的微服务 。在默认情况下，它使用Http Client 进行网络请求。
- POST 过滤器：它是在请求己被路由到微服务后执行的，一般情况下，用作收集统计信息、指标，以及将响应传输到客户端。
- ERROR 过滤器：它是在其他过滤器发生错误时执行的。

Zuul 采取了动态读取、编译和运行这些过滤器。过滤器之间不能直接通信，而是通过RequestContext 对象来共享数据，每个请求都会创建一个RequestContext 对象。Zuul过滤器具有以下关键特性：

- Type（类型）: Zuul 过滤器的类型，这个类型决定了过滤器在请求的哪个阶段起作用，例如 Pre、Post 阶段等。
- Execution Order （执行顺序）: 规定了过滤器的执行顺序， Order 的值越小，越先执行。
- Criteria （标准）: Filter 执行所需的条件。
- Action （行动）:如果符合执行条件，则执行Action （即逻辑代码）。

Zuul请求的生命周期图

![](https://img-blog.csdn.net/20170111153538629?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGFoYTcyODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 

#### zuul案例

1.在springboot启动类中加入注解@EnableZuulProxy ，开启Zuul功能。

2.路由的配置: 指定zuul.routes.xxxapi（需要负载的应用，自定义名称）的path和serviceld ，两者配合使用，就可以将指定类型的请求 Uri 路由到指定的Serviceld。如果某服务存在多个实例， Zuul 结合 Ribbon 会做负载均衡，将请求均分的部分路由到不同的服务实例。

3.如果想指定url做负载均衡，需要自己维护负载均衡的服务注册列表。首先将ribbon.eureka.enabled改为false,即Ribbon负载均衡客户端不想Eureka Client获取服务注册列表信息。然后自己维护一份注册列表，通过配置xxxapi-v1.ribbon.listOfServers来配置多个负载均衡的Url.

4.在Zuul上配置API接口的版本号：如果想要给所有的API接口上加一个v1作为版本号，需要配置 zuul.prefix: v1

5.在zuul上配置熔断器：在Zuul 实现熔断功能需要实现 ZuulFallbackProvider 接口。实现该接口有两个方法，一个是getRoute()方法，用于指定熔断功能应用于哪些路由的服务,另一个方法 fallbackResponse()为进入熔断功能时执行的逻辑 。

​	如果需要所有的路由服务都加熔断功能，只需要在 getRoute（）方法上返回"*"的匹配符，

6.在Zuul中使用过滤器：实现一个自定义的zuul过滤器，需要继承 ZuulFilter ，井实现 ZuulFilter 中的抽象方法，包括 filterType() 和filterOrder()，以及IZuulFilter的shouldFilter ()和Object run ()的两个方法。

---

:question:Zuul的常见使用方式

Zuul 是采用了类似于 SpringMVC的DispatchServlet 来实现的，采用的是异步阻塞模型，所以性能比 Ngnix 差。由于Zuul的横向扩展能力非常好，所以当负载过高时，可以通过添加实例来解决性能瓶颈。

一种常见的使用方式是对不同的渠道使用不同的 Zuul 来进行路由；

另外一种常见的集群是通过 Ngnix和Zuul 相互结合来做负载均衡。暴露在最外面的是Ngnix 从双热备进行 Keepalive, Ngnix 经过某种路由策略，将请求路由转发到 Zuul 集群上，Zuul 最终将请求分发到具体的服务上。

### Spring Cloud Gateway

Spring Cloud官方推出的第二代网关框架，取代Zuul网关。

使用一个RouteLocatorBuilder的bean去创建路由,可以添加各种**predicates**和**filters**, predicates是断言的意思，就是根据具体的请求的规则，由具体的route去处理，filters是各种过滤器，用来对请求做各种判断和修改。



## 分布式配置中心

### Spring Cloud Config

> 在Spring Cloud中，有分布式配置中心组件spring cloud config ，它支持配置服务放在配置服务的内存中（即本地），也支持放在远程Git仓库中。在spring cloud config 组件中，分两个角色，一是config server，二是config client。

服务配置的统一管理过程大致如下：

1. 首先，Config Server （配置服务）读取配置文件仓库的配置信息，其中配置文件仓库可以存放在配置服务的本地仓库，也可以放在远程的 Git 仓库（例如 GitHub Coding 等）。
2. 配置服务启动后，读取配置文件信息，读取完成的配置信息存放在配置服务的内存中。
3. 当启动服务A、B时，由于服务A、B指定了向配置服务读取配置信息，服务A、B向配置服务读取配置信息。
4. 当服务的配置信息需要修改且修改完成后，向配置服务发送 Po 请求进行刷新，这时服务A、B会向配置服务重写读取配置文件。

对于集群化的服务，可以通过使用消息总线来刷新多个服务实例。如果服务数量较多，对配置中心需要考虑集群化部署，从而使配置中心高可用，做分布式集群。

#### Spring Cloud Config 从本地读取配置文件

本地仓库是指将所有的配置文件统一写在 Config Server 工程目录下。 Config Server 暴露 HttpAPI 接口， Config Client 通过调用 Config Server的http API 接口来读取配置文件.

**构建Config Server**

1. 在springboot启动类上加@EnableConfigServer注解，开启Config Server的功能。
2. 在工程的配置文件 application.yml 中做相关的配置，通过 spring. profiles.active=native 来配置 onfig Server 本地读取配置，读取配置
   的路径为 classpath 下的 shared 目录。 
3. 在工程的Resources 目录下建一个 shared 文件夹，用于存放本地配置文件。

**构建Config Client**

在其配置文件bootstrap.yml 中做程序的配置，bootstrap 相对 application 有优先的执行顺序。spring.cloud.config.uri配置读取的Config Server地址；spring.cloud.config.fail-fast配置如果没有读取成功，是否执行快速失败；spring.profiles.active配置的是读取的环境。

bootstrap.yml配置文件中的变量 spring. application.name和变 spring.profiles.active ，两者以“-”相连，构成了向Config Server 读取的配置文件名。

#### Spring Cloud Config从远程git仓库读取配置文件

> **好处：** 将配置统一管理，并且可以通过Spring Cloud Bus在不人工启动程序的情况下对Config Client的配置进行刷新。

修改Config Server的application.yml文件为：

```
server: 
	port ：8769
spring : 
	cloud : 
		config : 
			server: 
				git: 
                  uri: https://github.com/forezp/SpringcloudConfig 
                  searchPaths: respo 
                  username: miles02 163.com
                  password : 
				label : master 
application: 
	name: config-server
```

uri 为远程 Git 仓库的地址， serachPaths 为搜索远程仓库的文件夹地址， usemame和passwor是Git 仓库的登录名和密码。如果是私人 Git 库，登录名和密码是必须的；如果是公开的Git仓库，可以不需要。label为git仓库的分支名。

#### 构建Config Server集群

Config Server和Config Client向Eureka Server 注册，且将 Config Server 多实例集群部署。

![](http://upload-images.jianshu.io/upload_images/2279594-babe706075d72c58.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

## 消息总线

### Spring Cloud Bus

Spring Cloud Bus 是用轻量的消息代理将分布式的节点连接起来，可以用于广播配置文件的更改或者服务的监控管理。一个关键的思想就是，消息总线可以为微服务做监控，也可以实现应用程序之间相互通信。Spring Cloud Bus 可选的消息代理组建包括 RabbitMQ、AMQP、Kafka等

消息总线更新微服务的配置流程如下：

![](http://www.ityouknow.com/assets/images/2017/springcloud/configbus1.jpg) 

改造Config Client的配置和代码：

1. 需要使其支持mq中间件，能够接受消息获取配置
2. 需要在更新的配置类上加@RefreshScope注解，只有这样，才会在不重启服务的情况下更新配置。
3. 发送post请求  /bus/refresh,请求刷新配置。

## 服务链路追踪

### Spring Cloud Sleuth

微服务架构是一个分布式架构，微服务系统按业务划分服务单元，一个微服务系统往往有很多个服务单元。由于服务单元数量众多 ，业务的复杂性较高，如果出现了错误和异常，很难去定位。主要体现在一个请求可能需要调用很多个服务，而内部服务的调用复杂性决定了问题难以定位。所以在微服务架构中，必须实现分布式链路追踪，去跟进一个请求到底有哪些服务参与，参与的顺序又是怎样的 ，从而达到每个请求的步骤清晰可见，出了问题能够快速定位的目的。

常见的链路追踪组件有 Google的Dapper、Twitter的Zipkin 以及阿里的 Eagleeye（鹰眼）等，它们都是非常优秀的链路追踪开源组件。

**基本术语**

- Span: 基本工作单元，发送一个远程调度任务就会产生一个 Span, Span 是用一个64位ID唯一标识的，Trace 是用另一个64位ID唯一标识的。 Span 还包含了其他的信息，例如摘要、时间戳事件、Span的ID及进程 ID。
- Trace: 由一系列 Span 组成的，呈树状结构。请求一个微服务系统的 API接口，这个API 接口需要调用多个微服务单元，调用每个微服务单元都会产生一个新的Span ，所有由这个请求产生的Span 组成了这个 Trace。
-  Annotation ：用于记录一个事件，一些核心注解用于定义一个请求的开始和结束，这些注解如下：
  - cs-Client Sent：客户端发送一个请求，这个注解描述了 Span 的开始
  - sr-Server Received ：服务端获得请求并准备开始处理它，如果将其sr减去 cs 时间戳，便可得到网络传输的时间。
  - ss-Server Sent ：服务端发送响应，该注解表明请求处理的完成（当请求返回客户端），用ss的时间戳减去sr的时间戳，便可以得到服务器请求的时间。
  - cr-Client Received 客户端接收响应， 此时 Span 结束，如果cr的时间戳减去 cs 时间戳，便可以得到整个请求所消耗的时间

收集的链路数据上传到zipkin-sever可以通过http形式，也支持消息组件。

Zipkin支持将链路数据存储在Mysql、Elasticsearch和Cassandra数据库中。

## 微服务监控

### Spring Boot Admin

## 微服务安全

### Spring Cloud Security

Spring Security

> Spring Security 采用“安全层”的概念 使每一层都尽可能安全，连续的安全层可以达到全面的防护。Spring Security 可以在 Controller 层、 Service 层、 DAO 层等以加注解的方式来保护应用程序的安全。 Spring Security 提供了细粒度的权限控制，可以精细到每一个API 接口、每 一个业务的方法，或者每一个操作数据库的 DAO 层的方法 。
>
> 使用 Spring Security 有很多原因，其中一个重要原因是它对环境的无依赖性、低代码耦合性。将工程重现部署到一个新的服务器上，不需要为 Spring curity 做什么工作。Spring Security 提供了数十个安全模块，模块与模块间的耦合性低，模块之间可以自由组合来实现特定需求的安全功能，具有较高的可定制性。总而言之，Spring Security 具有很好的可复用性和可定制性。 
>
> 在安全方面，有两个主要的领域，一时“ 认证 ”，即你是谁，二是”授权 ”，即你拥有什么权限，Spring Security 的主要目标就是在这两个领域。“认证“是认证主体的过程，通常是指可以在应用程序中执行操作的用户、设备或其他系统。“授权”是指决定是否允许己认证的主体执行某 一项操作

在 Spring Security中，主要包含了两个依赖，即spring-security-web和spring-security-config 

Spring Boot对Spring Security 框架做了封装，仅仅是封装，并没有改动Spring Security两个包的内容，并加上了 Spring Boot 的起步依赖的特性。

### Spring Cloud OAuth2

### JWT



## 其他微服务架构

### Dubbo

核心内容：

- RPC 远程调用：封装了长连接 NIO 框架，如 Netty Mina ，采用的是多线程模式。
- 集群容错：提供了基于接口方法的远程调用的功能 并实现了负载均衡策略、失败容错等功能。
- 服务发现：集成了 Apache的Zookeeper 组件，用于服务的注册和发现。

架构流程如下：

1. 服务提供者向服务中心注册服务。
2. 服务消费者订阅服务。
3. 服务消费者发现服务。
4. 服务消费者远程调度服务提供者进行服务消费，在调度过程中，使用了负载均衡策略、失败容错的功能。
5. 服务消费者和提供者，在内存中记录服务的调用次数和调用时间，并定时每分钟发送一次统计数据到监控中心。

Dubbo特性：

- 连通性：注册中心负责服务的注册， 监控中心负责收集调用次数、调用时间，注册中心、服务提供者、服务消费者为长连接。
- 健壮性：监控中心宕机不影响其他服务的使用；注册中心集群，任意一个实例宕机自动切换到另一个注册中心实例；服务实例集群，任意 一个实例宕机，自动切换到另外一个可用的实例。
- 伸缩性：可以动态增减注册中心和服务的实例数量。
- 升级性：服务集群升级，不会对现有架构造成压力。

#### Spring Cloud 与Dubbo对比

| 微服务关注点 | Spring Cloud            | Dubbo     |
| ------ | ----------------------- | --------- |
| 配置管理   | Config                  | --        |
| 服务发现   | Eureka、Consul、Zookeeper | Zookeeper |
| 负载均衡   | Ribbon                  | 自带        |
| 网关     | Zuul                    | --        |
| 分布式追踪  | Spring Cloud Sleuth     | --        |
| 容错     | Hystrix                 | 不完善       |
| 通信方式   | HTTP、Message            | RPC       |
| 安全模块   | Spring Cloud Security   | --        |



