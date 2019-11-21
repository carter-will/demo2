# Dubbo

## RPC



## 简介

#### 常见应用架构

- **单一应用架构**

- - 当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。
  - 此时，用于简化增删改查工作量的 **数据访问框架****(ORM)** 是关键。

- **垂直应用架构**

- - 当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，将应用拆成互不相干的几个应用，以提升效率。
  - 此时，用于加速前端页面开发的 **Web****框架****(MVC)** 是关键。

- **分布式服务架构**

- - 当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。
  - 此时，用于提高业务复用及整合的 **分布式服务框架****(RPC)** 是关键。

- **流动计算架构**

- - 当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率。
  - 此时，用于提高机器利用率的 **资源调度和治理中心****(SOA)** 是关键。

![](.\Dubbo\pic\Dubbo服务治理.jpg)

[F5硬件负载均衡器](https://blog.csdn.net/zjlovety/article/details/82695978)

**Dubbo架构**

![](.\Dubbo\pic\Dubbo架构.jpg)

- **Provider:** 暴露服务的服务提供方。
- **Consumer:** 调用远程服务的服务消费方。
- **Registry:** 服务注册与发现的注册中心。
- **Monitor:** 统计服务的调用次调和调用时间的监控中心。
- **Container:** 服务运行容器。

**调用关系说明：**

0. 服务容器负责启动，加载，运行服务提供者。

1. 服务提供者在启动时，向注册中心注册自己提供的服务。

2. 服务消费者在启动时，向注册中心订阅自己所需的服务。

3. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
4. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
5. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

> 软负载均衡
>
> [软负载均衡的几种策略](https://www.iteye.com/blog/cccai-1234-2392507)
>
> [负载均衡算法与实现](https://www.cnblogs.com/CodeBear/archive/2019/03/11/10508880.html)

**Dubbo特性**

1. 联通性
   - 注册中心负责服务地址的注册与查找，相当于目录服务，服务提供者和消费者只在启动时与注册中心交互，注册中心不转发请求，压力较小
   - 监控中心负责统计各服务调用次数，调用时间等，统计先在内存汇总后每分钟一次发送到监控中心服务器，并以报表展示
   - 服务提供者向注册中心注册其提供的服务，并汇报调用时间到监控中心，此时间不包含网络开销
   - 服务消费者向注册中心获取服务提供者地址列表，并根据负载算法直接调用提供者，同时汇报调用时间到监控中心，此时间包含网络开销
   - 注册中心，服务提供者，服务消费者三者之间均为长连接，监控中心除外
   - 注册中心通过长连接感知服务提供者的存在，服务提供者宕机，注册中心将立即推送事件通知消费者
   - 注册中心和监控中心全部宕机，不影响已运行的提供者和消费者，消费者在本地缓存了提供者列表
   - 注册中心和监控中心都是可选的，服务消费者可以直连服务提供者

2. 健壮性
   - 监控中心宕掉不影响使用，只是丢失部分采样数据
   - 数据库宕掉后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务
   - 注册中心对等集群，任意一台宕掉后，将自动切换到另一台
   - 注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯
   - 服务提供者无状态，任意一台宕掉后，不影响使用
   - 服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复

3. 伸缩性
   - 注册中心为对等集群，可动态增加机器部署实例，所有客户端将自动发现新的注册中心
   - 服务提供者无状态，可动态增加机器部署实例，注册中心将推送新的服务提供者信息给消费者

4. 升级性
   - 当服务集群规模进一步扩大，带动IT治理结构进一步升级，需要实现动态部署，进行流动计算，现有分布式服务架构不会带来阻力：

###### Dubbo三种协议比较

| Feature     | 策略                                                         | 问题                                  |
| ----------- | ------------------------------------------------------------ | ------------------------------------- |
| Dubbo协议   | 采用NIO复用单一长连接，并使用线程池并发处理请求，减少握手和加大并发效率，性能较好（推荐使用） | 在大文件传输时，单一连接会成为瓶颈    |
| Rmi协议     | 可与原生RMI互操作，基于TCP协议                               | 偶尔会连接失败，需重建Stub            |
| Hessian协议 | 可与原生Hessian互操作，基于HTTP协议                          | 需hessian.jar支持，http短连接的开销大 |

###### 引用框架

| Feature             | 策略                                  | 问题                                             |
| ------------------- | ------------------------------------- | ------------------------------------------------ |
| Netty Transporter   | JBoss的NIO框架，性能较好（推荐使用）  | 一次请求派发两种事件，需屏蔽无用事件             |
| Mina Transporter    | 老牌NIO框架，稳定                     | 待发送消息队列派发不及时，大压力下，会出现FullGC |
| Grizzly Transporter | Sun的NIO框架，应用于GlassFish服务器中 | 线程池不可扩展，Filter不能拦截下一Filter         |

###### Dubbo序列化

| Feature               | 策略                                                 | 问题                                                         |
| --------------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| Hessian Serialization | 性能较好，多语言支持（推荐使用）                     | Hessian的各版本兼容性不好，可能和应用使用的Hessian冲突，Dubbo内嵌了hessian3.2.1的源码 |
| Dubbo Serialization   | 通过不传送POJO的类元信息，在大量POJO传输时，性能较好 | 当参数对象增加字段时，需外部文件声明                         |
| Json Serialization    | 纯文本，可跨语言解析，缺省采用FastJson解析           | 性能较差                                                     |
| Java Serialization    | Java原生支持                                         | 性能较差                                                     |

###### Dubbo代理方式

| Feature                | 策略                                           | 问题                                                         |
| ---------------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| Javassist ProxyFactory | 通过字节码生成代替反射，性能比较好（推荐使用） | 依赖于javassist.jar包，占用JVM的Perm内存，Perm可能要设大一些：java -XX:PermSize=128m |
| Jdk ProxyFactory       | JDK原生支持                                    | 性能较差                                                     |

###### 集群容错模式

| Feature           | 策略                                                         | 问题                                   |
| ----------------- | ------------------------------------------------------------ | -------------------------------------- |
| Failover Cluster  | 失败自动切换，当出现失败，重试其它服务器，通常用于读操作（推荐使用） | 重试会带来更长延迟                     |
| Failfast Cluster  | 快速失败，只发起一次调用，失败立即报错,通常用于非幂等性的写操作 | 如果有机器正在重启，可能会出现调用失败 |
| Failsafe Cluster  | 失败安全，出现异常时，直接忽略，通常用于写入审计日志等操作   | 调用信息丢失                           |
| Failback Cluster  | 失败自动恢复，后台记录失败请求，定时重发，通常用于消息通知操作 | 不可靠，重启丢失                       |
| Forking Cluster   | 并行调用多个服务器，只要一个成功即返回，通常用于实时性要求较高的读操作 | 需要浪费更多服务资源                   |
| Broadcast Cluster | 广播调用所有提供者，逐个调用，任意一台报错则报错，通常用于更新提供方本地状态 | 速度慢，任意一台报错则报错             |

###### 负载均衡算法

| Feature                    | 策略                                                         | 问题                                                         |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Random LoadBalance         | 随机，按权重设置随机概率（推荐使用）                         | 在一个截面上碰撞的概率高，重试时，可能出现瞬间压力不均       |
| RoundRobin LoadBalance     | 轮循，按公约后的权重设置轮循比率                             | 存在慢的机器累积请求问题，极端情况可能产生雪崩               |
| LeastActive LoadBalance    | 最少活跃调用数，相同活跃数的随机，活跃数指调用前后计数差，使慢的机器收到更少请求 | 不支持权重，在容量规划时，不能通过权重把压力导向一台机器压测容量 |
| ConsistentHash LoadBalance | 一致性Hash，相同参数的请求总是发到同一提供者，当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动 | 压力分摊不均                                                 |

###### Dubbo路由规则

| Feature      | 策略                                   | 问题                                         |
| ------------ | -------------------------------------- | -------------------------------------------- |
| 条件路由规则 | 基于条件表达式的路由规则，功能简单易用 | 有些复杂多分支条件情况，规则很难描述         |
| 脚本路由规则 | 基于脚本引擎的路由规则，功能强大       | 没有运行沙箱，脚本能力过于强大，可能成为后门 |

###### 运行容器

| Feature          | 策略                                                         | 问题                                     |
| ---------------- | ------------------------------------------------------------ | ---------------------------------------- |
| Spring Container | 自动加载META-INF/spring目录下的所有Spring配置                |                                          |
| Jetty Container  | 启动一个内嵌Jetty，用于汇报状态                              | 大量访问页面时，会影响服务器的线程和内存 |
| Log4j Container  | 自动配置log4j的配置，在多进程启动时，自动给日志文件按进程分目录 | 用户不能控制log4j的配置，不灵活          |

## 配置

#### XML配置

~~~
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://code.alibabatech.com/schema/dubbo
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd
        ">
 
    <dubbo:application name="hello-world-app"  />
 
    <dubbo:registry address="multicast://224.5.6.7:1234" />
 
    <dubbo:protocol name="dubbo" port="20880" />
 
    <dubbo:service interface="com.alibaba.dubbo.demo.DemoService" ref="demoServiceLocal" />
 
    <dubbo:reference id="demoServiceRemote" interface="com.alibaba.dubbo.demo.DemoService" />
 
</beans>
~~~

> 所有标签者支持自定义参数，用于不同扩展点实现的特殊配置,如：
>
> <dubbo:protocol name="jms">
>
> ​    <dubbo:parameter key="queue" value="10.20.31.22" />
>
> </dubbo:protocol>
>
> 或
>
> <dubbo:protocol name="jms" p:queue="10.20.31.22" />

**Configuration Relation:**

![](.\Dubbo\pic\configuration-ref.jpg)

-  <<dubbo:service/>>服务配置，用于暴露一个服务，定义服务的元信息，一个服务可以用多个协议暴露，一个服务也可以注册到多个注册中心。
- <<dubbo:reference/>>引用配置，用于创建一个远程服务代理，一个引用可以指向多个注册中心。

- <<dubbo:protocol/>协议配置，用于配置提供服务的协议信息，协议由提供方指定，消费方被动接受。
- <<dubo:application/>> 应用配置，用于配置当前应用信息，不管该应用是提供者还是消费者。
- <<dubbo:module/>>模块配置，用于配置当前模块信息，可选。
- <<dubbo:registry/>> 注册中心配置，用于配置连接注册中心相关信息。
- <<dubbo:monitor/>>监控中心配置，用于配置连接监控中心相关信息，可选。
- <<dubbo:provider/>> 提供方的缺省值，当ProtocolConfig和ServiceConfig某属性没有配置时，采用此缺省值，可选。
- <<dubbo:consumer/>>消费方缺省配置，当ReferenceConfig某属性没有配置时，采用此缺省值，可选。
- <<dubbo:method/>> 方法配置，用于ServiceConfig和ReferenceConfig指定方法级的配置信息。
- <<dubbo:argument/>> 用于指定方法参数配置。

![](.\Dubbo\pic\ConfigurationOverride.jpg)

以timeout为例，显示了配置的查找顺序，其它retries, loadbalance, actives等类似:

- - 方法级优先，接口级次之，全局配置再次之。
  - 如果级别一样，则消费方优先，提供方次之。
  - 其中，服务提供方配置，通过URL经由注册中心传递给消费方。
  - 建议由服务提供方设置超时，因为一个方法需要执行多长时间，服务提供方更清楚，如果一个消费方同时引用多个服务，就不需要关心每个服务的超时设置。
  - 理论上ReferenceConfig的非服务标识配置，在ConsumerConfig，ServiceConfig,      ProviderConfig均可以缺省配置。

#### 属性配置

> 如果公共配置很简单，没有多注册中心，多协议等情况，或者想多个Spring容器想共享配置，可以使用dubbo.properties作为缺省配置。
>
> Dubbo将自动加载classpath根目录下的dubbo.properties，可以通过JVM启动参数：-Ddubbo.properties.file=xxx.properties 改变缺省配置位置。
>
> 如果classpath根目录下存在多个dubbo.properties，比如多个jar包中有dubbo.properties，Dubbo会任意加载，并打印Error日志，后续可能改为抛异常。

映射规则：

- 将XML配置的标签名，加属性名，用点分隔，多个属性拆成多行：

- - 比如：dubbo.application.name=foo等价于<dubbo:application       name="foo" />
  - 比如：dubbo.registry.address=10.20.153.10:9090等价于<dubbo:registry       address="10.20.153.10:9090" />

- 如果XML有多行同名标签配置，可用id号区分，如果没有id号将对所有同名标签生效：

  - 比如：dubbo.protocol.rmi.port=1234等价于<dubbo:protocol       id="rmi" name="rmi" port="1099" /> (协议的id没配时，缺省使用协议名作为id)
  - 比如：dubbo.registry.china.address=10.20.153.10:9090等价于<dubbo:registry       id="china" address="10.20.153.10:9090" />

覆盖策略：

- JVM启动-D参数优先，这样可以使用户在部署和启动时进行参数重写，比如在启动时需改变协议的端口。

- XML次之，如果在XML中有配置，则dubbo.properties中的相应配置项无效。

- Properties最后，相当于缺省值，只有XML没有配置时，dubbo.properties的相应配置项才会生效，通常用于共享公共配置，比如应用名。

  ` -D  -->   XML  -->  Properties`

#### 注解配置

服务提供方注解

```
import com.alibaba.dubbo.config.annotation.Service;

@Service(version="1.0.0")
public class FooServiceImpl implements FooService {
     // ...... 
}
```

服务提供方配置

```
<!-- 公共信息，也可以用dubbo.properties配置 -->
<dubbo:application name="annotation-provider" />
<dubbo:registry address="127.0.0.1:4548" /> 
<!-- 扫描注解包路径，多个包用逗号分隔，不填pacakge表示扫描当前ApplicationContext中所有的类 -->
<dubbo:annotation package="com.foo.bar.service" />
```

服务消费方注解

```
import com.alibaba.dubbo.config.annotation.Reference;
import org.springframework.stereotype.Component;

@Component
public class BarAction { 
    @Reference(version="1.0.0")
    private FooService fooService; 
}
```

服务消费方配置

```
<!-- 公共信息，也可以用dubbo.properties配置 -->
<dubbo:application name="annotation-consumer" />
<dubbo:registry address="127.0.0.1:4548" /> 
<!-- 扫描注解包路径，多个包用逗号分隔，不填pacakge表示扫描当前ApplicationContext中所有的类 -->
<dubbo:annotation package="com.foo.bar.action" />
```

> 也可以使用：(等价于前面的：<dubbo:annotation package="com.foo.bar.service" />)
>
> <dubbo:annotation /><context:component-scan base-package="com.foo.bar.service">    <context:include-filter type="annotation" expression="com.alibaba.dubbo.config.annotation.Service" /> <</context:component-scan>>

#### API配置

> API仅用于OpenAPI, ESB, Test, Mock等系统集成，普通服务提供方或消费方，请采用配置方式使用Dubbo，

服务提供者：

```
import com.alibaba.dubbo.rpc.config.ApplicationConfig;
import com.alibaba.dubbo.rpc.config.RegistryConfig;
import com.alibaba.dubbo.rpc.config.ProviderConfig;
import com.alibaba.dubbo.rpc.config.ServiceConfig;
import com.xxx.XxxService;
import com.xxx.XxxServiceImpl;
 
// 服务实现
XxxService xxxService = new XxxServiceImpl();
 
// 当前应用配置
ApplicationConfig application = new ApplicationConfig();
application.setName("xxx");
 
// 连接注册中心配置
RegistryConfig registry = new RegistryConfig();
registry.setAddress("10.20.130.230:9090");
registry.setUsername("aaa");
registry.setPassword("bbb");
 
// 服务提供者协议配置
ProtocolConfig protocol = new ProtocolConfig();
protocol.setName("dubbo");
protocol.setPort(12345);
protocol.setThreads(200);
 
// 注意：ServiceConfig为重对象，内部封装了与注册中心的连接，以及开启服务端口
 
// 服务提供者暴露服务配置
ServiceConfig<XxxService> service = new ServiceConfig<XxxService>(); 
// 此实例很重，封装了与注册中心的连接，请自行缓存，否则可能造成内存和连接泄漏
service.setApplication(application);
service.setRegistry(registry); // 多个注册中心可以用setRegistries()
service.setProtocol(protocol); // 多个协议可以用setProtocols()
service.setInterface(XxxService.class);
service.setRef(xxxService);
service.setVersion("1.0.0");
 
// 暴露及注册服务
service.export();
```

服务消费者：

```
import com.alibaba.dubbo.rpc.config.ApplicationConfig;
import com.alibaba.dubbo.rpc.config.RegistryConfig;
import com.alibaba.dubbo.rpc.config.ConsumerConfig;
import com.alibaba.dubbo.rpc.config.ReferenceConfig;
import com.xxx.XxxService;
 
// 当前应用配置
ApplicationConfig application = new ApplicationConfig();
application.setName("yyy");
 
// 连接注册中心配置
RegistryConfig registry = new RegistryConfig();
registry.setAddress("10.20.130.230:9090");
registry.setUsername("aaa");
registry.setPassword("bbb");
 
// 注意：ReferenceConfig为重对象，内部封装了与注册中心的连接，以及与服务提供方的连接
 
// 引用远程服务
ReferenceConfig<XxxService> reference = new ReferenceConfig<XxxService>(); // 此实例很重，封装了与注册中心的连接以及与提供者的连接，请自行缓存，否则可能造成内存和连接泄漏
reference.setApplication(application);
reference.setRegistry(registry); // 多个注册中心可以用setRegistries()
reference.setInterface(XxxService.class);
reference.setVersion("1.0.0");
 
// 和本地bean一样使用xxxService
XxxService xxxService = reference.get(); // 注意：此代理对象内部封装了所有通讯细节，对象较重，请缓存复用
```

特殊：方法级设置

```
// 方法级配置
List<MethodConfig> methods = new ArrayList<MethodConfig>();
MethodConfig method = new MethodConfig();
method.setName("createXxx");
method.setTimeout(10000);
method.setRetries(0);
methods.add(method);
 
// 引用远程服务
ReferenceConfig<XxxService> reference = new ReferenceConfig<XxxService>(); 
// 此实例很重，封装了与注册中心的连接以及与提供者的连接，请自行缓存，否则可能造成内存和连接泄漏
...
reference.setMethods(methods); // 设置方法级配置
```

特殊：点对点直连

```
ReferenceConfig<XxxService> reference = new ReferenceConfig<XxxService>(); // 此实例很重，封装了与注册中心的连接以及与提供者的连接，请自行缓存，否则可能造成内存和连接泄漏
// 如果点对点直连，可以用reference.setUrl()指定目标地址，设置url后将绕过注册中心，
// 其中，协议对应provider.setProtocol()的值，端口对应provider.setPort()的值，
// 路径对应service.setPath()的值，如果未设置path，缺省path为接口名
reference.setUrl("dubbo://10.20.130.230:20880/com.xxx.XxxService"); 
```

## 应用

#### 启动时检查

> Dubbo缺省会在启动时检查依赖的服务是否可用，不可用时会抛出异常，阻止Spring初始化完成，以便上线时，能及早发现问题，默认check=true。
>
> 如果你的Spring容器是懒加载的，或者通过API编程延迟引用服务，请关闭check，否则服务临时不可用时，会抛出异常，拿到null引用，如果check=false，总是会返回引用，当服务恢复时，能自动连上。

可以通过check="false"关闭检查，比如，测试时，有些服务不关心，或者出现了循环依赖，必须有一方先启动。

关闭某个服务的启动时检查：(没有提供者时报错)

   <dubbo:reference   interface="com.foo.BarService" check="false" />   

关闭所有服务的启动时检查：(没有提供者时报错)

   <dubbo:consumer check="false"   />   

关闭注册中心启动时检查：(注册订阅失败时报错)

   <dubbo:registry check="false"   />   

也可以用dubbo.properties配置：

```
dubbo.reference.com.foo.BarService.check=false
dubbo.reference.check=false
dubbo.consumer.check=false
dubbo.registry.check=false
```

也可以用-D参数：

```
java -Ddubbo.reference.com.foo.BarService.check=false
java -Ddubbo.reference.check=false
java -Ddubbo.consumer.check=false 
java -Ddubbo.registry.check=false
```

> - dubbo.reference.check=false，强制改变所有reference的check值，就算配置中有声明，也会被覆盖。
> - dubbo.consumer.check=false，是设置check的缺省值，如果配置中有显式的声明，如：<dubbo:reference      check="true"/>，不会受影响。
>
> - dubbo.registry.check=false，前面两个都是指订阅成功，但提供者列表是否为空是否报错，如果注册订阅失败时，也允许启动，需使用此选项，将在后台定时重试。

引用缺省是延迟初始化的，只有引用被注入到其它Bean，或被getBean()获取，才会初始化。
 如果需要饥饿加载，即没有人引用也立即生成动态代理，可以配置：

```
 <dubbo:reference   interface="com.foo.BarService" init="true" />   
```

#### 集群容错

在集群调用失败时，Dubbo提供了多种容错方案，缺省为failover重试

![](.\Dubbo\pic\集群.jpg)

- 这里的Invoker是Provider的一个可调用Service的抽象，Invoker封装了Provider地址及Service接口信息。
- Directory代表多个Invoker，可以把它看成List<Invoker>，但与List不同的是，它的值可能是动态变化的，比如注册中心推送变更。
- Cluster将Directory中的多个Invoker伪装成一个Invoker，对上层透明，伪装过程包含了容错逻辑，调用失败后，重试另一个。
- Router负责从多个Invoker中按路由规则选出子集，比如读写分离，应用隔离等。
- LoadBalance负责从多个Invoker中选出具体的一个用于本次调用，选的过程包含了负载均衡算法，调用失败后，需要重选。

**集群容错模式：**

在集群调用失败时，Dubbo提供了多种容错方案，缺省为failover重试

**Failover Cluster**

- 失败自动切换，当出现失败，重试其它服务器。(缺省)
- 通常用于读操作，但重试会带来更长延迟。
- 可通过retries="2"来设置重试次数(不含第一次)。

**Failfast Cluster**

- 快速失败，只发起一次调用，失败立即报错
- 通常用于非幂等性的写操作，比如新增记录

**Failsafe Cluster**

- 失败安全，出现异常时，直接忽略。
- 通常用于写入审计日志等操作。

**Failback Cluster**

- 失败自动恢复，后台记录失败请求，定时重发。
- 通常用于消息通知操作。

**Forking Cluster**

- 并行调用多个服务器，只要一个成功即返回。
- 通常用于实时性要求较高的读操作，但需要浪费更多服务资源。
- 可通过forks="2"来设置最大并行数。

**Broadcast Cluster**

- 广播调用所有提供者，逐个调用，任意一台报错则报错。(2.1.0开始支持)
- 通常用于通知所有提供者更新缓存或日志等本地资源信息。

> 重试次数配置如：(failover集群模式生效) 
>
> <dubbo:service retries="2" />
>
> 或
>
> <dubbo:reference retries="2" />
>
> 或
>
> <<dubbo:reference>>
>
> ​    <dubbo:method name="findFoo" retries="2" />
>
> <</dubbo:reference>>

集群模式配置：

```
<dubbo:service cluster="failsafe" />
或
<dubbo:reference cluster="failsafe" />
```

#### 负载均衡

在集群负载均衡时，Dubbo提供了多种均衡策略，缺省为random随机调用

**Random LoadBalance**

- 随机，按权重设置随机概率。
- 在一个截面上碰撞的概率高，但调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重。

**RoundRobin LoadBalance**

- 轮循，按公约后的权重设置轮循比率。
- 存在慢的提供者累积请求问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上。

**LeastActive LoadBalance**

- 最少活跃调用数，相同活跃数的随机，活跃数指调用前后计数差。
- 使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差会越大。

**ConsistentHash LoadBalance**

- 一致性Hash，相同参数的请求总是发到同一提供者。
- 当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动。
- 缺省只对第一个参数Hash，如果要修改，请配置<dubbo:parameter  key="hash.arguments" value="0,1" />
- 缺省用160份虚拟节点，如果要修改，请配置<dubbo:parameter key="hash.nodes" value="320"      />

配置如：

```
<dubbo:service interface="..." loadbalance="roundrobin" />
或：
<dubbo:reference interface="..." loadbalance="roundrobin" />
或：
<dubbo:service interface="...">
    <dubbo:method name="..." loadbalance="roundrobin"/>
</dubbo:service>
或：
<dubbo:reference interface="...">
    <dubbo:method name="..." loadbalance="roundrobin"/>
</dubbo:reference>
```

#### 线程模型

![](.\Dubbo\pic\线程模型.jpg)

> **事件处理线程说明**
>
> - 如果事件处理的逻辑能迅速完成，并且不会发起新的IO请求，比如只是在内存中记个标识，则直接在IO线程上处理更快，因为减少了线程池调度。
> - 但如果事件处理逻辑较慢，或者需要发起新的IO请求，比如需要查询数据库，则必须派发到线程池，否则IO线程阻塞，将导致不能接收其它请求。
>
> - 如果用IO线程处理事件，又在事件处理过程中发起新的IO请求，比如在连接事件中发起登录请求，会报“可能引发死锁”异常，但不会真死锁。

- Dispatcher

- - all 所有消息都派发到线程池，包括请求，响应，连接事件，断开事件，心跳等。
  - direct       所有消息都不派发到线程池，全部在IO线程上直接执行。
  - message       只有请求响应消息派发到线程池，其它连接断开事件，心跳等消息，直接在IO线程上执行。
  - execution       只请求消息派发到线程池，不含响应，响应和其它连接断开事件，心跳等消息，直接在IO线程上执行。
  - connection       在IO线程上，将连接断开事件放入队列，有序逐个执行，其它消息派发到线程池。

- ThreadPool

- - fixed       固定大小线程池，启动时建立线程，不关闭，一直持有。(缺省)
  - cached       缓存线程池，空闲一分钟自动删除，需要时重建。
  - limited       可伸缩线程池，但池中的线程数只会增长不会收缩。(为避免收缩时突然来了大流量引起的性能问题)。

配置如：

- ```
  <dubbo:protocol name="dubbo" dispatcher="all" threadpool="fixed" threads="100" />
  ```

#### 直连提供者

在开发及测试环境下，经常需要绕过注册中心，只测试指定服务提供者，这时候可能需要点对点直连，
点对点直联方式，将以服务接口为单位，忽略注册中心的提供者列表，
A接口配置点对点，不影响B接口从注册中心获取列表。

![](.\Dubbo\pic\点对点直连.jpg)

1. 如果是线上需求需要点对点，可在<dubbo:reference>中配置url指向提供者，将绕过注册中心，**多个地址用分号隔开**，配置如下：

   ```
   <dubbo:reference id="xxxService" interface="com.alibaba.xxx.XxxService" url="dubbo://localhost:20890" />
   ```

2. 在JVM启动参数中加入-D参数映射服务地址，如：(key为服务名，value为服务提供者url，此配置优先级最高，1.0.15及以上版本支持)

   ```
   java -Dcom.alibaba.xxx.XxxService=dubbo://localhost:20890
   ```

3. 如果服务比较多，也可以用文件映射，如：

   (用-Ddubbo.resolve.file指定映射文件路径，此配置优先级高于<<dubbo:reference>>中的配置，1.0.15及以上版本支持)
   (2.0以上版本自动加载${user.home}/dubbo-resolve.properties文件，不需要配置)

   `java -Ddubbo.resolve.file=xxx.properties`

   然后在映射文件xxx.properties中加入：(key为服务名，value为服务提供者url)

   ` com.alibaba.xxx.XxxService=dubbo://localhost:20890`

#### 只订阅

? 为方便开发测试，经常会在线下共用一个所有服务可用的注册中心，这时，如果一个正在开发中的服务提供者注册，可能会影响消费者不能正常运行。

! 可以让服务提供者开发方，只订阅服务(开发的服务可能依赖其它服务)，而不注册正在开发的服务，通过直连测试正在开发的服务。

![](.\Dubbo\pic\只订阅.jpg)

禁用注册配置：

```
<dubbo:registry address="10.20.153.10:9090" register="false" />
或
<dubbo:registry address="10.20.153.10:9090?register=false" />
```

#### 只注册

？ 如果有两个镜像环境，两个注册中心，有一个服务只在其中一个注册中心有部署，另一个注册中心还没来得及部署，而两个注册中心的其它应用都需要依赖此服务，所以需要将服务同时注册到两个注册中心，但却不能让此服务同时依赖两个注册中心的其它服务。

！ 可以让服务提供者方，只注册服务到另一注册中心，而不从另一注册中心订阅服务。

禁止订阅服务：

```
<dubbo:registry id="hzRegistry" address="10.20.153.10:9090" />
<dubbo:registry id="qdRegistry" address="10.20.141.150:9090" subscribe="false" />
或者：
<dubbo:registry id="hzRegistry" address="10.20.153.10:9090" />
<dubbo:registry id="qdRegistry" address="10.20.141.150:9090?subscribe=false" />
```

#### 静态服务

有时候希望人工管理服务提供者的上线和下线，此时需将注册中心标识为非动态管理模式。

```
<dubbo:registry address="10.20.141.150:9090" dynamic="false" />
或者：
<dubbo:registry address="10.20.141.150:9090?dynamic=false" />
```

服务提供者初次注册时为禁用状态，需人工启用，断线时，将不会被自动删除，需人工禁用。

#### 多协议

##### **不同服务不同协议**

比如：不同服务在性能上适用不同协议进行传输，比如大数据用短连接协议，小数据大并发用长连接协议。consumer.xml:

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://code.alibabatech.com/schema/dubbo
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd
        ">
 
    <dubbo:application name="world"  />
    <dubbo:registry id="registry" address="10.20.141.150:9090" username="admin" password="hello1234" />
 
    <!-- 多协议配置 -->
    <dubbo:protocol name="dubbo" port="20880" />
    <dubbo:protocol name="rmi" port="1099" />
 
    <!-- 使用dubbo协议暴露服务 -->
    <dubbo:service interface="com.alibaba.hello.api.HelloService" version="1.0.0" ref="helloService" protocol="dubbo" />
    <!-- 使用rmi协议暴露服务 -->
    <dubbo:service interface="com.alibaba.hello.api.DemoService" version="1.0.0" ref="demoService" protocol="rmi" />
 
</beans>
```

##### **多协议暴露服务**

比如：需要与http客户端互操作,consumer.xml:

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://code.alibabatech.com/schema/dubbo
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd
        ">
 
    <dubbo:application name="world"  />
    <dubbo:registry id="registry" address="10.20.141.150:9090" username="admin" password="hello1234" />
 
    <!-- 多协议配置 -->
    <dubbo:protocol name="dubbo" port="20880" />
    <dubbo:protocol name="hessian" port="8080" />
 
    <!-- 使用多个协议暴露服务 -->
    <dubbo:service id="helloService" interface="com.alibaba.hello.api.HelloService" version="1.0.0" protocol="dubbo,hessian" />
 
</beans>

```

#### 多注册中心

##### **多注册中心注册**

consumer.xml:

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://code.alibabatech.com/schema/dubbo
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd
        ">
 
    <dubbo:application name="world"  />
 
    <!-- 多注册中心配置 -->
    <dubbo:registry id="hangzhouRegistry" address="10.20.141.150:9090" />
    <dubbo:registry id="qingdaoRegistry" address="10.20.141.151:9010" default="false" />
 
    <!-- 向多个注册中心注册 -->
    <dubbo:service interface="com.alibaba.hello.api.HelloService" version="1.0.0" ref="helloService" registry="hangzhouRegistry,qingdaoRegistry" />
 
</beans>
```

##### **不同服务使用不同注册中心**

**consumer.xml**:

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://code.alibabatech.com/schema/dubbo
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd
        ">
 
    <dubbo:application name="world"  />
 
    <!-- 多注册中心配置 -->
    <dubbo:registry id="chinaRegistry" address="10.20.141.150:9090" />
    <dubbo:registry id="intlRegistry" address="10.20.154.177:9010" default="false" />
 
    <!-- 向中文站注册中心注册 -->
    <dubbo:service interface="com.alibaba.hello.api.HelloService" version="1.0.0" ref="helloService" registry="chinaRegistry" />
 
    <!-- 向国际站注册中心注册 -->
    <dubbo:service interface="com.alibaba.hello.api.DemoService" version="1.0.0" ref="demoService" registry="intlRegistry" />
 
</beans>
```

##### **多注册中心引用**

比如：CRM需同时调用中文站和国际站的PC2服务，PC2在中文站和国际站均有部署，接口及版本号都一样，但连的数据库不一样。consumer.xml:

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://code.alibabatech.com/schema/dubbo
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd
        ">
 
    <dubbo:application name="world"  />
 
    <!-- 多注册中心配置 -->
    <dubbo:registry id="chinaRegistry" address="10.20.141.150:9090" />
    <dubbo:registry id="intlRegistry" address="10.20.154.177:9010" default="false" />
 
    <!-- 引用中文站服务 -->
    <dubbo:reference id="chinaHelloService" interface="com.alibaba.hello.api.HelloService" version="1.0.0" registry="chinaRegistry" />
 
    <!-- 引用国际站站服务 -->
    <dubbo:reference id="intlHelloService" interface="com.alibaba.hello.api.HelloService" version="1.0.0" registry="intlRegistry" />
 
</beans>
```

如果只是测试环境临时需要连接两个不同注册中心，使用竖号分隔多个不同注册中心地址：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://code.alibabatech.com/schema/dubbo
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd
        ">
 
    <dubbo:application name="world"  />
 
    <!-- 多注册中心配置，竖号分隔表示同时连接多个不同注册中心，同一注册中心的多个集群地址用逗号分隔 -->
    <dubbo:registry address="10.20.141.150:9090|10.20.154.177:9010" />
 
    <!-- 引用服务 -->
    <dubbo:reference id="helloService" interface="com.alibaba.hello.api.HelloService" version="1.0.0" />
 
</beans>
```

#### 服务分组

当一个接口有多种实现时，可以用group区分

```
<dubbo:service group="feedback" interface="com.xxx.IndexService" />
<dubbo:service group="member" interface="com.xxx.IndexService" />
<dubbo:reference id="feedbackIndexService" group="feedback" interface="com.xxx.IndexService" />
<dubbo:reference id="memberIndexService" group="member" interface="com.xxx.IndexService" />
任意组：(2.2.0以上版本支持，总是只调一个可用组的实现)
<dubbo:reference id="barService" interface="com.foo.BarService" group="*" />
```

#### 多版本

当一个接口实现，出现不兼容升级时，可以用版本号过渡，版本号不同的服务相互间不引用。

- 在低压力时间段，先升级一半提供者为新版本
- 再将所有消费者升级为新版本
- 然后将剩下的一半提供者升级为新版本

```
<dubbo:service interface="com.foo.BarService" version="1.0.0" />
<dubbo:service interface="com.foo.BarService" version="2.0.0" />
<dubbo:reference id="barService" interface="com.foo.BarService" version="1.0.0" />
<dubbo:reference id="barService" interface="com.foo.BarService" version="2.0.0" />
不区分版本：(2.2.0以上版本支持)
<dubbo:reference id="barService" interface="com.foo.BarService" version="*" />
```

#### 分组聚合

按组合并返回结果，比如菜单服务，接口一样，但有多种实现，用group区分，现在消费方需从每种group中调用一次返回结果，合并结果返回，这样就可以实现聚合菜单项。   

![](.\Dubbo\pic\分组聚合.jpg)

配置如：

```
(搜索所有分组)
<dubbo:reference interface="com.xxx.MenuService" group="*" merger="true" />
(合并指定分组)
<dubbo:reference interface="com.xxx.MenuService" group="aaa,bbb" merger="true" />
(指定方法合并结果，其它未指定的方法，将只调用一个Group)
<dubbo:reference interface="com.xxx.MenuService" group="*">
    <dubbo:method name="getMenuItems" merger="true" />
</dubbo:service>
(某个方法不合并结果，其它都合并结果)
<dubbo:reference interface="com.xxx.MenuService" group="*" merger="true">
    <dubbo:method name="getMenuItems" merger="false" />
</dubbo:service>
(指定合并策略，缺省根据返回值类型自动匹配，如果同一类型有两个合并器时，需指定合并器的名称)
<dubbo:reference interface="com.xxx.MenuService" group="*">
    <dubbo:method name="getMenuItems" merger="mymerge" />
</dubbo:service>
(指定合并方法，将调用返回结果的指定方法进行合并，合并方法的参数类型必须是返回结果类型本身)
<dubbo:reference interface="com.xxx.MenuService" group="*">
    <dubbo:method name="getMenuItems" merger=".addAll" />
</dubbo:service>
```

#### 参数验证



#### 结果缓存

结果缓存，用于加速热门数据的访问速度，Dubbo提供声明式缓存，以减少用户加缓存的工作量。

- lru 基于最近最少使用原则删除多余缓存，保持最热的数据被缓存。
- threadlocal      当前线程缓存，比如一个页面渲染，用到很多portal，每个portal都要去查用户信息，通过线程缓存，可以减少这种多余访问。
- jcache      与[JSR107](http://jcp.org/en/jsr/detail?id=107)集成，可以桥接各种缓存实现。

配置如下：

```
<dubbo:reference interface="com.foo.BarService" cache="lru" />
或：
<dubbo:reference interface="com.foo.BarService">
    <dubbo:method name="findBar" cache="lru" />
</dubbo:reference>
```

#### 泛化引用

泛接口调用方式主要用于客户端没有API接口及模型类元的情况，参数及返回值中的所有POJO均用Map表示，通常用于框架集成，比如：实现一个通用的服务测试框架，可通过GenericService调用所有服务实现。

```
<dubbo:reference id="barService" interface="com.foo.BarService" generic="true" />
GenericService barService = (GenericService) applicationContext.getBean("barService");
Object result = barService.$invoke("sayHello", new String[] { "java.lang.String" }, new Object[] { "World" });
import com.alibaba.dubbo.rpc.service.GenericService; 
... 
 
// 引用远程服务 
ReferenceConfig<GenericService> reference = new ReferenceConfig<GenericService>(); // 该实例很重量，里面封装了所有与注册中心及服务提供方连接，请缓存
reference.setInterface("com.xxx.XxxService"); // 弱类型接口名 
reference.setVersion("1.0.0"); 
reference.setGeneric(true); // 声明为泛化接口 
 
GenericService genericService = reference.get(); // 用com.alibaba.dubbo.rpc.service.GenericService可以替代所有接口引用 
 
// 基本类型以及Date,List,Map等不需要转换，直接调用 
Object result = genericService.$invoke("sayHello", new String[] {"java.lang.String"}, new Object[] {"world"}); 
 
// 用Map表示POJO参数，如果返回值为POJO也将自动转成Map 
Map<String, Object> person = new HashMap<String, Object>(); 
person.put("name", "xxx"); 
person.put("password", "yyy"); 
Object result = genericService.$invoke("findPerson", new String[]{"com.xxx.Person"}, new Object[]{person}); // 如果返回POJO将自动转成Map 
 
...
```

#### 泛化实现

泛接口实现方式主要用于服务器端没有API接口及模型类元的情况，参数及返回值中的所有POJO均用Map表示，通常用于框架集成，比如：实现一个通用的远程服务Mock框架，可通过实现GenericService接口处理所有服务请求。

```
<bean id="genericService" class="com.foo.MyGenericService" />
<dubbo:service interface="com.foo.BarService" ref="genericService" />
package com.foo;
public class MyGenericService implements GenericService {
 
    public Object $invoke(String methodName, String[] parameterTypes, Object[] args) throws GenericException {
        if ("sayHello".equals(methodName)) {
            return "Welcome " + args[0];
        }
    }
 
}
... 
GenericService xxxService = new XxxGenericService();  //com.alibaba.dubbo.rpc.service.GenericService可以替代所有接口实现 
 
ServiceConfig<GenericService> service = new ServiceConfig<GenericService>(); // 该实例很重量，里面封装了所有与注册中心及服务提供方连接，请缓存
service.setInterface("com.xxx.XxxService"); // 弱类型接口名 
service.setVersion("1.0.0"); 
service.setRef(xxxService); // 指向一个通用服务实现 
 
// 暴露及注册服务 
service.export();
```

#### 回声测试

回声测试用于检测服务是否可用，回声测试按照正常请求流程执行，能够测试整个调用是否通畅，可用于监控。

所有服务自动实现EchoService接口，只需将任意服务引用强制转型为EchoService，即可使用。     

```
<dubbo:reference id="memberService" interface="com.xxx.MemberService" />
MemberService memberService = ctx.getBean("memberService"); // 远程服务引用 
EchoService echoService = (EchoService) memberService; // 强制转型为EchoService 
String status = echoService.$echo("OK"); // 回声测试可用性 
assert(status.equals("OK"))
```

#### 上下文信息

上下文中存放的是当前调用过程中所需的环境信息。所有配置信息都将转换为URL的参数。

> 注意:  RpcContext是一个ThreadLocal的临时状态记录器，当接收到RPC请求，或发起RPC请求时，RpcContext的状态都会变化。    比如：A调B，B再调C，则B机器上，在B调C之前，RpcContext记录的是A调B的信息，在B调C之后，RpcContext记录的是B调C的信息。   

**服务消费方**

```
xxxService.xxx(); // 远程调用
boolean isConsumerSide = RpcContext.getContext().isConsumerSide(); // 本端是否为消费端，这里会返回true
String serverIP = RpcContext.getContext().getRemoteHost(); // 获取最后一次调用的提供方IP地址
String application = RpcContext.getContext().getUrl().getParameter("application"); // 获取当前服务配置信息，所有配置信息都将转换为URL的参数
// ...
yyyService.yyy(); // 注意：每发起RPC调用，上下文状态会变化
// ...
```

**服务提供方**

```
public class XxxServiceImpl implements XxxService {
 
    public void xxx() { // 服务方法实现
        boolean isProviderSide = RpcContext.getContext().isProviderSide(); // 本端是否为提供端，这里会返回true
        String clientIP = RpcContext.getContext().getRemoteHost(); // 获取调用方IP地址
        String application = RpcContext.getContext().getUrl().getParameter("application"); // 获取当前服务配置信息，所有配置信息都将转换为URL的参数
        // ...
        yyyService.yyy(); // 注意：每发起RPC调用，上下文状态会变化
        boolean isProviderSide = RpcContext.getContext().isProviderSide(); // 此时本端变成消费端，这里会返回false
        // ...
    } 
}
```

#### 隐式传参

> 注：path,group,version,dubbo,token,timeout几个key有特殊处理，请使用其它key值。

![](.\Dubbo\pic\隐式传参.jpg)

**服务消费方**

```
RpcContext.getContext().setAttachment("index", "1"); // 隐式传参，后面的远程调用都会隐式将这些参数发送到服务器端，类似cookie，用于框架集成，不建议常规业务使用
xxxService.xxx(); // 远程调用
// ...
```

【注】 setAttachment设置的KV，在完成下面一次远程调用会被清空。即多次远程调用要多次设置。

**服务提供方**

```
public class XxxServiceImpl implements XxxService { 
    public void xxx() { // 服务方法实现
        String index = RpcContext.getContext().getAttachment("index"); // 获取客户端隐式传入的参数，用于框架集成，不建议常规业务使用
        // ...
    } 
}
```

#### 异步调用

基于NIO的非阻塞实现并行调用，客户端不需要启动多线程即可完成并行调用多个远程服务，相对多线程开销较小。

配置声明：consumer.xml

```
<dubbo:reference id="fooService" interface="com.alibaba.foo.FooService">
      <dubbo:method name="findFoo" async="true" />
</dubbo:reference>
<dubbo:reference id="barService" interface="com.alibaba.bar.BarService">
      <dubbo:method name="findBar" async="true" />
</dubbo:reference>
```

```
fooService.findFoo(fooId); // 此调用会立即返回null
Future<Foo> fooFuture = RpcContext.getContext().getFuture(); // 拿到调用的Future引用，当结果返回后，会被通知和设置到此Future。
 
barService.findBar(barId); // 此调用会立即返回null
Future<Bar> barFuture = RpcContext.getContext().getFuture(); // 拿到调用的Future引用，当结果返回后，会被通知和设置到此Future。
 
// 此时findFoo和findBar的请求同时在执行，客户端不需要启动多线程来支持并行，而是借助NIO的非阻塞完成。
 
Foo foo = fooFuture.get(); // 如果foo已返回，直接拿到返回值，否则线程wait住，等待foo返回后，线程会被notify唤醒。
Bar bar = barFuture.get(); // 同理等待bar返回。
// 如果foo需要5秒返回，bar需要6秒返回，实际只需等6秒，即可获取到foo和bar，进行接下来的处理。
```

也可以设置是否等待消息发出：(异步总是不等待返回):

- sent="true"      等待消息发出，消息发送失败将抛出异常。
- sent="false"      不等待消息发出，将消息放入IO队列，即刻返回。

` <dubbo:method name="findFoo" async="true" sent="true" />`

如果你只是想异步，完全忽略返回值，可以配置return="false"，以减少Future对象的创建和管理成本：

` <dubbo:method name="findFoo" async="true" return="false" />` 

#### 本地调用

本地调用，使用了Injvm协议，是一个伪协议，它不开启端口，不发起远程调用，只在JVM内直接关联，但执行Dubbo的Filter。

```
Define injvm protocol:
<dubbo:protocol name="injvm" />
Set default protocol:
<dubbo:provider protocol="injvm" />
Set service protocol:
<dubbo:service protocol="injvm" />
Use injvm first:
<dubbo:consumer injvm="true" .../>
<dubbo:provider injvm="true" .../>
或
<dubbo:reference injvm="true" .../>
<dubbo:service injvm="true" .../>
```

注意：服务暴露与服务引用都需要声明injvm="true"

**自动暴露、引用本地服务**

从 dubbo 2.2.0 开始，每个服务默认都会在本地暴露；在引用服务的时候，默认优先引用本地服务；如果希望引用远程服务可以使用一下配置强制引用远程服务:

` <dubbo:reference ... scope="remote" />`

#### 参数回调

参数回调方式与调用本地callback或listener相同，只需要在Spring的配置文件中声明哪个参数是callback类型即可，Dubbo将基于长连接生成反向代理，这样就可以从服务器端调用客户端逻辑。

1. 共享服务接口

   ```
   CallbackService.java
   
   package com.callback;
    
   public interface CallbackService {
       void addListener(String key, CallbackListener listener);
   }
   ```

   ```
   CallbackListener.java
   
   package com.callback;
    
   public interface CallbackListener {
       void changed(String msg);
   }
   ```

2. 服务提供者

   ```
   CallbackServiceImpl.java
   
   package com.callback.impl;
    
   import java.text.SimpleDateFormat;
   import java.util.Date;
   import java.util.Map;
   import java.util.concurrent.ConcurrentHashMap;
    
   import com.callback.CallbackListener;
   import com.callback.CallbackService;
    
   public class CallbackServiceImpl implements CallbackService {
        
       private final Map<String, CallbackListener> listeners = new ConcurrentHashMap<String, CallbackListener>();
     
       public CallbackServiceImpl() {
           Thread t = new Thread(new Runnable() {
               public void run() {
                   while(true) {
                       try {
                           for(Map.Entry<String, CallbackListener> entry : listeners.entrySet()){
                              try {
                                  entry.getValue().changed(getChanged(entry.getKey()));
                              } catch (Throwable t) {
                                  listeners.remove(entry.getKey());
                              }
                           }
                           Thread.sleep(5000); // 定时触发变更通知
                       } catch (Throwable t) { // 防御容错
                           t.printStackTrace();
                       }
                   }
               }
           });
           t.setDaemon(true);
           t.start();
       }
     
       public void addListener(String key, CallbackListener listener) {
           listeners.put(key, listener);
           listener.changed(getChanged(key)); // 发送变更通知
       }
        
       private String getChanged(String key) {
           return "Changed: " + new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date());
       }
    
   }
   ```

   ```
   配置
   <bean id="callbackService" class="com.callback.impl.CallbackServiceImpl" />
   <dubbo:service interface="com.callback.CallbackService" ref="callbackService" connections="1" callbacks="1000">
       <dubbo:method name="addListener">
           <dubbo:argument index="1" callback="true" />
           <!--也可以通过指定类型的方式-->
           <!--<dubbo:argument type="com.demo.CallbackListener" callback="true" />-->
       </dubbo:method>
   </dubbo:service>
   ```

3. 服务消费者

   ```
   配置 consumer.xml
   <dubbo:reference id="callbackService" interface="com.callback.CallbackService" />
   ```

   ```
   调用示例 CallbackServiceTest.java
   ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:consumer.xml");
   context.start();
    
   CallbackService callbackService = (CallbackService) context.getBean("callbackService");
    
   callbackService.addListener("foo.bar", new CallbackListener(){
       public void changed(String msg) {
           System.out.println("callback1:" + msg);
       }
   });
   ```

   #### 事件通知

   在调用之前，调用之后，出现异常时，会触发oninvoke, onreturn, onthrow三个事件，可以配置当事件发生时，通知哪个类的哪个方法。

   1.**服务提供者与消费者共享服务接口**

   ```
   IDemoService.java
   interface IDemoService {
       public Person get(int id);
   }
   ```

   2.**服务提供者实现**

   ```
   DemoServiceImpl.java
   class NormalDemoService implements IDemoService {
       public Person get(int id) {
           return new Person(id, "charles`son", 4);
       }
   }
   ```

   3.**服务提供者配置**

   ```
   provider.xml
   <dubbo:application name="rpc-callback-demo" />
   <dubbo:registry address="10.20.153.186" />
   <bean id="demoService" class="com.alibaba.dubbo.callback.implicit.NormalDemoService" />
   <dubbo:service interface="com.alibaba.dubbo.callback.implicit.IDemoService" ref="demoService" version="1.0.0" group="cn"/>
   ```

   4.**服务消费者Callback接口及实现：**

   ```
   Nofify.java
   interface Nofify {
       public void onreturn(Person msg, Integer id);
       public void onthrow(Throwable ex, Integer id);
   }
   ```

   ```
   NofifyImpl.java
   class NofifyImpl implements Nofify {
       public Map<Integer, Person>    ret    = new HashMap<Integer, Person>();
       public Map<Integer, Throwable> errors = new HashMap<Integer, Throwable>();
       public void onreturn(Person msg, Integer id) {
           System.out.println("onreturn:" + msg);
           ret.put(id, msg);
       }
       public void onthrow(Throwable ex, Integer id) {
           errors.put(id, ex);
       }
   }
   ```

   5.**服务消费者配置：**

   ```
   consumer.xml
   <bean id ="demoCallback" class = "com.alibaba.dubbo.callback.implicit.NofifyImpl" />
   <dubbo:reference id="demoService" interface="com.alibaba.dubbo.callback.implicit.IDemoService" version="1.0.0" group="cn" >
         <dubbo:method name="get" async="true" onreturn = "demoCallback.onreturn" onthrow="demoCallback.onthrow" />
   </dubbo:reference>
   ```

   > callback与async功能正交分解：
   >  async=true，表示结果是否马上返回.
   >  onreturn 表示是否需要回调.
   >
   > 组合情况：(async=false 默认)
   >  异步回调模式：async=true onreturn="xxx"
   >  同步回调模式：async=false onreturn="xxx"
   >  异步无回调 ：async=true
   >  同步无回调 ：async=false

6.**TEST CASE**

```
IDemoService demoService = (IDemoService) context.getBean("demoService");
NofifyImpl notify = (NofifyImpl) context.getBean("demoCallback");
int requestId = 2;
Person ret = demoService.get(requestId);
Assert.assertEquals(null, ret);
//for Test：只是用来说明callback正常被调用，业务具体实现自行决定.
for (int i = 0; i < 10; i++) {
    if (!notify.ret.containsKey(requestId)) {
        Thread.sleep(200);
    } else {
        break;
    }
}
Assert.assertEquals(requestId, notify.ret.get(requestId).getId());
```

#### 本地存根

#### 本地伪装

#### 延迟暴露

如果你的服务需要Warmup时间，比如初始化缓存，等待相关资源就位等，可以使用delay进行延迟暴露。

延迟5秒暴露服务：

` <dubbo:service delay="5000" />`

延迟到Spring初始化完成后，再暴露服务：(基于Spring的ContextRefreshedEvent事件触发暴露)

` <dubbo:service delay="-1" />`

> 防止线程死锁、不能提供服务，规避办法
>
> 1. 强烈建议不要在服务的实现类中有applicationContext.getBean()的调用，全部采用IoC注入的方式使用Spring的Bean。
>
> 2. 如果实在要调getBean()，可以将Dubbo的配置放在Spring的最后加载。
>
> 3. 如果不想依赖配置顺序，可以使用<dubbo:provider deplay=”-1” />，使Dubbo在Spring容器初始化完后，再暴露服务。
>
> 4. 如果大量使用getBean()，相当于已经把Spring退化为工厂模式在用，可以将Dubbo的服务隔离单独的Spring容器。

#### 并发控制

限制com.foo.BarService的每个方法，**服务器端**并发执行（或占用线程池线程数）不能超过10个：

` <dubbo:service interface="com.foo.BarService" executes="10" />`

限制com.foo.BarService的sayHello方法，**服务器端**并发执行（或占用线程池线程数）不能超过10个：

```
<dubbo:service interface="com.foo.BarService">
    <dubbo:method name="sayHello" executes="10" />
</dubbo:service>
```

限制com.foo.BarService的每个方法，每**客户端**并发执行（或占用连接的请求数）不能超过10个：

```
<dubbo:service interface="com.foo.BarService" actives="10" />
或
<dubbo:reference interface="com.foo.BarService" actives="10" />
```

限制com.foo.BarService的sayHello方法，每**客户端**并发执行（或占用连接的请求数）不能超过10个：

```
<dubbo:service interface="com.foo.BarService">
    <dubbo:method name="sayHello" actives="10" />
</dubbo:service>
或
<dubbo:reference interface="com.foo.BarService">
    <dubbo:method name="sayHello" actives="10" />
</dubbo:service>
```

如果<<dubbo:service>>和<<dubbo:reference>>都配了actives，<<dubbo:reference>>优先，

Load Balance均衡：配置服务的**客户端**的loadbalance属性为leastactive，此Loadbalance会调用并发数最小的Provider（Consumer端并发数）.

```
<dubbo:reference interface="com.foo.BarService" loadbalance="leastactive" />
或
<dubbo:service interface="com.foo.BarService" loadbalance="leastactive" />
```

#### 连接控制

限制**服务器端**接受的连接不能超过10个：（以连接在Server上，所以配置在Provider上）

```
<dubbo:provider protocol="dubbo" accepts="10" />
<dubbo:protocol name="dubbo" accepts="10" />
```

限制**客户端服务**使用连接连接数：(如果是长连接，比如Dubbo协议，connections表示该服务对每个提供者建立的长连接数)

```
<dubbo:reference interface="com.foo.BarService" connections="10" />
或
<dubbo:service interface="com.foo.BarService" connections="10" />
```

如果<<dubbo:service>>和<<dubbo:reference>>都配了actives，<<dubbo:reference>>优先，

#### 延迟连接

延迟连接，用于减少长连接数，当有调用发起时，再创建长连接。

只对使用长连接的dubbo协议生效。

```
<dubbo:protocol name="dubbo" lazy="true" />
```

#### 粘滞连接

粘滞连接用于有状态服务，尽可能让客户端总是向同一提供者发起调用，除非该提供者挂了，再连另一台。

粘滞连接将自动开启延迟连接，以减少长连接数

```
<dubbo:protocol name="dubbo" sticky="true" />
```

#### 令牌验证

![](.\Dubbo\pic\令牌验证.jpg)

- 防止消费者绕过注册中心访问提供者
- 在注册中心控制权限，以决定要不要下发令牌给消费者
- 注册中心可灵活改变授权方式，而不需修改或升级提供者

可以全局设置开启令牌验证：

```
<!--随机token令牌，使用UUID生成-->
<dubbo:provider interface="com.foo.BarService" token="true" />
<!--固定token令牌，相当于密码-->
<dubbo:provider interface="com.foo.BarService" token="123456" />
```

也可在服务级别设置：

```
<!--随机token令牌，使用UUID生成-->
<dubbo:service interface="com.foo.BarService" token="true" />
<!--固定token令牌，相当于密码-->
<dubbo:service interface="com.foo.BarService" token="123456" />
```

还可在协议级别设置：

```
<!--随机token令牌，使用UUID生成-->
<dubbo:protocol name="dubbo" token="true" />
<!--固定token令牌，相当于密码-->
<dubbo:protocol name="dubbo" token="123456" />
```

#### 路由规则

向注册中心写入路由规则：(通常由监控中心或治理中心的页面完成)

```
RegistryFactory registryFactory = ExtensionLoader.getExtensionLoader(RegistryFactory.class).getAdaptiveExtension();
Registry registry = registryFactory.getRegistry(URL.valueOf("zookeeper://10.20.153.10:2181"));
registry.register(URL.valueOf("condition://0.0.0.0/com.foo.BarService?category=routers&dynamic=false&rule=" + URL.encode("host = 10.20.153.10 => host = 10.20.153.11") + "));
```

> 其中：
>
> - condition://
>
> - - 表示路由规则的类型，支持[条件路由规则](http://code.alibabatech.com/wiki/display/dubbo/User+Guide-zh#UserGuide-zh-%E6%9D%A1%E4%BB%B6%E8%B7%AF%E7%94%B1%E8%A7%84%E5%88%99)和[脚本路由规则](http://code.alibabatech.com/wiki/display/dubbo/User+Guide-zh#UserGuide-zh-%E8%84%9A%E6%9C%AC%E8%B7%AF%E7%94%B1%E8%A7%84%E5%88%99)，可扩展，必填。
>
> - 0.0.0.0
>
> - - 表示对所有IP地址生效，如果只想对某个IP的生效，请填入具体IP，必填。
>
> - com.foo.BarService
>
> - - 表示只对指定服务生效，必填。
>
> - category=routers
>
> - - 表示该数据为动态配置类型，必填。
>
> - dynamic=false
>
> - - 表示该数据为持久数据，当注册方退出时，数据依然保存在注册中心，必填。
>
> - enabled=true
>
> - - 覆盖规则是否生效，可不填，缺省生效。
>
> - force=false
>
> - - 当路由结果为空时，是否强制执行，如果不强制执行，路由结果为空的路由规则将自动失效，可不填，缺省为flase。
>
> - runtime=false
>
> - - 是否在每次调用时执行路由规则，否则只在提供者地址列表变更时预先执行并缓存结果，调用时直接从缓存中获取路由结果。
>   - 如果用了参数路由，必须设为true，需要注意设置会影响调用的性能，可不填，缺省为flase。
>
> - priority=1
>
> - - 路由规则的优先级，用于排序，优先级越大越靠前执行，可不填，缺省为0。
>
> - rule=URL.encode("host      = 10.20.153.10 => host = 10.20.153.11")
>
> - - 表示路由规则的内容，必填。

#### 条件路由规则

#### 脚本路由规则

#### 配置规则

#### 服务降级

向注册中心写入动态配置覆盖规则：(通过由监控中心或治理中心的页面完成)

```
RegistryFactory registryFactory = ExtensionLoader.getExtensionLoader(RegistryFactory.class).getAdaptiveExtension();
Registry registry = registryFactory.getRegistry(URL.valueOf("zookeeper://10.20.153.10:2181"));
registry.register(URL.valueOf("override://0.0.0.0/com.foo.BarService?category=configurators&dynamic=false&application=foo&mock=force:return+null"));
```

其中：

   mock=force:return+null   

- 表示消费方对该服务的方法调用都直接返回null值，不发起远程调用。
- 屏蔽不重要服务不可用时对调用方的影响。

还可以改为：

   mock=fail:return+null   

- 表示消费方对该服务的方法调用在失败后，再返回null值，不抛异常。
- 容忍不重要服务不稳定时对调用方的影响。

#### 优雅停机

Dubbo是通过JDK的ShutdownHook来完成优雅停机的，所以如果用户使用"kill -9 PID"等强制关闭指令，是不会执行优雅停机的，只有通过"kill PID"时，才会执行。   

> 原理：
>
> - 服务提供方
>
> - - 停止时，先标记为不接收新请求，新请求过来时直接报错，让客户端重试其它机器。
>   - 然后，检测线程池中的线程是否正在运行，如果有，等待所有线程执行完成，除非超时，则强制关闭。
>
> - 服务消费方
>
> - - 停止时，不再发起新的调用请求，所有新的调用在客户端即报错。
>   - 然后，检测有没有请求的响应还没有返回，等待响应返回，除非超时，则强制关闭。

设置优雅停机超时时间，缺省超时时间是10秒：(超时则强制关闭)

```
<dubbo:application ...>
    <dubbo:parameter key="shutdown.timeout" value="60000" /> <!-- 单位毫秒 -->
</dubbo:application>
```

如果ShutdownHook不能生效，可以自行调用：

   ProtocolConfig.destroyAll();   

#### 主机绑定

缺省主机IP查找顺序：

- 通过LocalHost.getLocalHost()获取本机地址。
- 如果是127.*等loopback地址，则扫描各网卡，获取网卡IP。

注册的地址如果获取不正确，比如需要注册公网地址，可以：

1. 可以在/etc/hosts中加入：机器名 公网IP，比如：

   test1 205.182.23.201   

2. 在dubbo.xml中加入主机地址的配置：

   <dubbo:protocol   host="205.182.23.201">   

3. 或在dubbo.properties中加入主机地址的配置：

   dubbo.protocol.host=205.182.23.201   

缺省主机端口与协议相关：

- dubbo:      20880
- rmi:      1099
- http:      80
- hessian:      80
- webservice:      80
- memcached:      11211
- redis:      6379

主机端口配置：

```
在dubbo.xml中加入主机地址的配置：
<dubbo:protocol name="dubbo" port="20880">
或在dubbo.properties中加入主机地址的配置：
dubbo.protocol.dubbo.port=20880
```

#### 日志适配

缺省自动查找：

- log4j
- slf4j
- jcl
- jdk

可以通过以下方式配置日志输出策略：

java -Ddubbo.application.logger=log4j

```
dubbo.properties
dubbo.application.logger=log4j
dubbo.xml
<dubbo:application logger="log4j" />
```

#### 访问日志

如果你想记录每一次请求信息，可开启访问日志，类似于apache的访问日志。

此日志量比较大，请注意磁盘容量。

将访问日志输出到当前应用的log4j日志：

```
<dubbo:protocol accesslog="true" />
```

将访问日志输出到指定文件：

```
<dubbo:protocol accesslog="foo/bar.log" />
```

#### 服务容器

#### Reference Config缓存

#### 分布式事务

[TCC分布式事务实现](https://blog.csdn.net/s13554341560b/article/details/79180327)

## 配置参考

所有配置项分为三大类，参见下表中的"作用"一列。

- 服务发现：表示该配置项用于服务的注册与发现，目的是让消费方找到提供方。
- 服务治理：表示该配置项用于治理服务间的关系，或为开发测试提供便利条件。
- 性能调优：表示该配置项用于调优性能，不同的选项对性能会产生影响。

所有配置最终都将转换为URL表示，并由服务提供方生成，经注册中心传递给消费方，各属性对应URL的参数，参见配置项一览表中的"对应URL参数"列。

**<<dubbo:service/>> **

服务提供者暴露服务配置：

配置类：com.alibaba.dubbo.config.ServiceConfig

**<<dubbo:reference/>>**

服务消费者引用服务配置：

配置类：com.alibaba.dubbo.config.ReferenceConfig

**<<dubbo:protocol/>>**

服务提供者协议配置：

配置类：com.alibaba.dubbo.config.ProtocolConfig

说明：如果需要支持多协议，可以声明多个<dubbo:protocol>标签，并在<dubbo:service>中通过protocol属性指定使用的协议。

**<<dubbo:registry/>>**

注册中心配置：

配置类：com.alibaba.dubbo.config.RegistryConfig

说明：如果有多个不同的注册中心，可以声明多个<dubbo:registry>标签，并在<dubbo:service>或<dubbo:reference>的registry属性指定使用的注册中心。

**<<dubbo:monitor/>>**

监控中心配置：

配置类：com.alibaba.dubbo.config.MonitorConfig

**<<dubbo:application/>>**

应用信息配置：

配置类：com.alibaba.dubbo.config.ApplicationConfig

**<<dubbo:module/>>**

模块信息配置：
配置类：com.alibaba.dubbo.config.ModuleConfig

**<<dubbo:provider/>>**

服务提供者缺省值配置：
配置类：com.alibaba.dubbo.config.ProviderConfig
说明：该标签为<dubbo:service>和<dubbo:protocol>标签的缺省值设置。

**<<dubbo:consumer/>>**

服务消费者缺省值配置：
配置类：com.alibaba.dubbo.config.ConsumerConfig
说明：该标签为<dubbo:reference>标签的缺省值设置。

**<<dubbo:method/>>**

方法级配置：
配置类：com.alibaba.dubbo.config.MethodConfig
说明：该标签为<dubbo:service>或<dubbo:reference>的子标签，用于控制到方法级，

**<<dubbo:argument/>>**

方法参数配置：
配置类：com.alibaba.dubbo.config.ArgumentConfig
说明：该标签为<dubbo:method>的子标签，用于方法参数的特征描述，比如：

```
<dubbo:method name="findXxx" timeout="3000" retries="2">
    <dubbo:argument index="0" callback="true" />
<dubbo:method>
```

**<<dubbo:parameter/>>**

选项参数配置：
配置类：java.util.Map
说明：该标签为<dubbo:protocol>或<dubbo:service>或<dubbo:provider>或<dubbo:reference>或<dubbo:consumer>的子标签，用于配置自定义参数，该配置项将作为扩展点设置自定义参数使用。

```
例如：
<dubbo:protocol name="napoli">
    <dubbo:parameter key="napoli.queue.name" value="xxx" />
</dubbo:protocol>
或
<dubbo:protocol name="jms" p:queue="xxx" />
```

## 协议参考

#### dubbo://

推荐使用Dubbo协议

Dubbo缺省协议采用单一长连接和NIO异步通讯，适合于小数据量大并发的服务调用，以及服务消费者机器数远大于服务提供者机器数的情况。

Dubbo缺省协议不适合传送大数据量的服务，比如传文件，传视频等，除非请求量很低。   

Dubbo protocol options:

```
  <dubbo:protocol name=“dubbo” port=“9090”   server=“netty” client=“netty” codec=“dubbo” serialization=“hessian2”   charset=“UTF-8” threadpool=“fixed” threads=“100” queues=“0” iothreads=“9”   buffer=“8192” accepts=“1000” payload=“8388608” />   
```

- Transporter

- - mina,netty, grizzy

- Serialization

- - dubbo,hessian2, java, json

- Dispatcher

- - all,direct, message, execution, connection

- ThreadPool

- - fixed, cached

Dubbo协议缺省每服务每提供者每消费者使用单一长连接，如果数据量较大，可以使用多个连接。

<dubbo:protocol name="dubbo" connections="2" />

- <dubbo:service      connections=”0”>或<dubbo:reference connections=”0”>表示该服务使用JVM共享长连接。(缺省)
- <dubbo:service      connections=”1”>或<dubbo:reference connections=”1”>表示该服务使用独立长连接。
- <dubbo:service      connections=”2”>或<dubbo:reference connections=”2”>表示该服务使用独立两条长连接。

为防止被大量连接撑挂，可在服务提供方限制大接收连接数，以实现服务提供方自我保护。

```
<dubbo:protocol name="dubbo" accepts="1000" />
```

> 缺省协议，使用基于mina1.1.7+hessian3.2.1的tbremoting交互。
>
> - 连接个数：单连接
> - 连接方式：长连接
> - 传输协议：TCP
> - 传输方式：NIO异步传输
> - 序列化：Hessian二进制序列化
> - 适用范围：传入传出参数数据包较小（建议小于100K），消费者比提供者个数多，单一消费者无法压满提供者，尽量不要用dubbo协议传输大文件或超大字符串。
> - 适用场景：常规远程服务方法调用

> **为什么要消费者比提供者个数多?**
> 因dubbo协议采用单一长连接，
> 假设网络为千兆网卡(1024Mbit=128MByte)，
> 根据测试经验数据每条连接最多只能压满7MByte(不同的环境可能不一样，供参考)，
> 理论上1个服务提供者需要20个服务消费者才能压满网卡。
>
> **为什么不能传大包?**
>
> 因dubbo协议采用单一长连接，
> 如果每次请求的数据包大小为500KByte，假设网络为千兆网卡(1024Mbit=128MByte)，每条连接最大7MByte(不同的环境可能不一样，供参考)，
> 单个服务提供者的TPS(每秒处理事务数)最大为：128MByte / 500KByte = 262。
> 单个消费者调用单个服务提供者的TPS(每秒处理事务数)最大为：7MByte / 500KByte = 14。
> 如果能接受，可以考虑使用，否则网络将成为瓶颈。
>
> **为什么采用异步单一长连接：**
>
> 因为服务的现状大都是服务提供者少，通常只有几台机器，
> 而服务的消费者多，可能整个网站都在访问该服务，
> 比如Morgan的提供者只有6台提供者，却有上百台消费者，每天有1.5亿次调用，
> 如果采用常规的hessian服务，服务提供者很容易就被压跨，
> 通过单一连接，保证单一消费者不会压死提供者，
> 长连接，减少连接握手验证等，
> 并使用异步IO，复用线程池，防止C10K问题。

> 约束：
>
> - 参数及返回值需实现Serializable接口
>
> - 参数及返回值不能自定义实现List, Map, Number, Date, Calendar等接口，只能用JDK自带的实现，因为hessian会做特殊处理，自定义实现类中的属性值都会丢失。()
>
> - Hessian序列化，只传成员属性值和值的类型，不传方法或静态变量，兼容情况：(由吴亚军提供)
>
> - 总结：会抛异常的情况：枚 举值一边多一种，一边少一种，正好使用了差别的那种，或者属性名相同，类型不同
>
>   服务器端和客户端对领域对象并不需要完全一致，而是按照最大匹配原则。

#### rmi://

RMI协议采用JDK标准的java.rmi.*实现，采用阻塞式短连接和JDK标准序列化方式。

- 如果服务接口继承了java.rmi.Remote接口，可以和原生RMI互操作，即：

- - 提供者用Dubbo的RMI协议暴露服务，消费者直接用标准RMI接口调用，
  - 或者提供方用标准RMI暴露服务，消费方用Dubbo的RMI协议调用。

- 如果服务接口没有继承java.rmi.Remote接口，

- - 缺省Dubbo将自动生成一个com.xxx.XxxService$Remote的接口，并继承java.rmi.Remote接口，并以此接口暴露服务，
  - 但如果设置了<dubbo:protocol name="rmi"       codec="spring" />，将不生成$Remote接口，而使用Spring的RmiInvocationHandler接口暴露服务，和Spring兼容。

> Java标准的远程调用协议。
>
> - 连接个数：多连接
> - 连接方式：短连接
> - 传输协议：TCP
> - 传输方式：同步传输
> - 序列化：Java标准二进制序列化
> - 适用范围：传入传出参数数据包大小混合，消费者与提供者个数差不多，可传文件。
> - 适用场景：常规远程服务方法调用，与原生RMI服务互操作

> 约束：
>
> - 参数及返回值需实现Serializable接口
>
> - dubbo配置中的超时时间对rmi无效，需使用java启动参数设置：-Dsun.rmi.transport.tcp.responseTimeout=3000，参见下面的RMI配置。
>
>   java -Dsun.rmi.transport.tcp.responseTimeout=3000

#### hession://

 Hessian协议用于集成Hessian的服务，Hessian底层采用Http通讯，采用Servlet暴露服务，Dubbo缺省内嵌Jetty作为服务器实现

Hessian是Caucho开源的一个RPC框架：[http://hessian.caucho.com](http://hessian.caucho.com/)，其通讯效率高于WebService和Java自带的序列化。

依赖：com.caucho.hessian

可以和原生Hessian服务互操作，即：

- 提供者用Dubbo的Hessian协议暴露服务，消费者直接用标准Hessian接口调用，
- 或者提供方用标准Hessian暴露服务，消费方用Dubbo的Hessian协议调用。

> 基于Hessian的远程调用协议。
>
> - 连接个数：多连接
> - 连接方式：短连接
> - 传输协议：HTTP
> - 传输方式：同步传输
> - 序列化：Hessian二进制序列化
> - 适用范围：传入传出参数数据包较大，提供者比消费者个数多，提供者压力较大，可传文件。
> - 适用场景：页面传输，文件传输，或与原生hessian服务互操作

> 约束：
>
> - 参数及返回值需实现Serializable接口
> - 参数及返回值不能自定义实现List, Map, Number, Date, Calendar等接口，只能用JDK自带的实现，因为hessian会做特殊处理，自定义实现类中的属性值都会丢失。

#### http://

采用Spring的HttpInvoker实现

> 基于http表单的远程调用协议。参见：[HTTP协议使用说明]
>
> - 连接个数：多连接
> - 连接方式：短连接
> - 传输协议：HTTP
> - 传输方式：同步传输
> - 序列化：表单序列化
> - 适用范围：传入传出参数数据包大小混合，提供者比消费者个数多，可用浏览器查看，可用表单或URL传入参数，暂不支持传文件。
> - 适用场景：需同时给应用程序和浏览器JS使用的服务。

> 约束：
>
> - 参数及返回值需符合Bean规范

#### webservice://

基于CXF的[frontend-simple](http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22cxf-rt-frontend-simple%22)和[transports-http](http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22cxf-rt-transports-http%22)实现。

CXF是Apache开源的一个RPC框架：[http://cxf.apache.org](http://cxf.apache.org/)，由Xfire和Celtix合并而来
依赖：org.apache.cxf.cxf-rt-frontend-simple, org.apache.cxf.cxf-rt-transports-http

可以和原生WebService服务互操作，即：

- 提供者用Dubbo的WebService协议暴露服务，消费者直接用标准WebService接口调用，
- 或者提供方用标准WebService暴露服务，消费方用Dubbo的WebService协议调用。

> 基于WebService的远程调用协议。
>
> - 连接个数：多连接
> - 连接方式：短连接
> - 传输协议：HTTP
> - 传输方式：同步传输
> - 序列化：SOAP文本序列化
> - 适用场景：系统集成，跨语言调用。

> 约束：
>
> - 参数及返回值需实现Serializable接口
> - 参数尽量使用基本类型和POJO。

#### thrift://

Thrift是Facebook捐给Apache的一个RPC框架，参见：[http://thrift.apache.org](http://thrift.apache.org/)

当前dubbo 支持的 thrift 协议是对 thrift 原生协议的扩展，在原生协议的基础上添加了一些额外的头信息，比如service name，magic number等。使用dubbo thrift协议同样需要使用thrift的idl compiler编译生成相应的java代码，后续版本中会在这方面做一些增强

依赖：org.apache.thrift.libthrift

所有服务共用一个端口：(与原生Thrift不兼容)

```
<dubbo:protocol name="thrift" port="3030" />
```

> Thrift不支持数据类型：
>
> - null值 (不能在协议中传递null值)

#### memcached://

#### redis://

## 注册中心参考

#### Multicast注册中心

不需要启动任何中心节点，只要广播地址一样，就可以互相发现

组播受网络结构限制，只适合小规模应用或开发阶段使用。

组播地址段: 224.0.0.0 - 239.255.255.255

![](.\Dubbo\pic\Multicast.jpg)

1. 提供方启动时广播自己的地址。
2. 消费方启动时广播订阅请求。
3. 提供方收到订阅请求时，单播自己的地址给订阅者，如果设置了unicast=false，则广播给订阅者。
4. 消费方收到提供方地址时，连接该地址进行RPC调用。

> <dubbo:registry address="multicast://224.5.6.7:1234" />
>
> 或
>
> <dubbo:registry protocol="multicast" address="224.5.6.7:1234" />

如果一个机器上同时启了多个消费者进程，消费者需声明unicast=false，否则只会有一个消费者能收到消息：

```
<dubbo:registry address="multicast://224.5.6.7:1234?unicast=false" />
或
<dubbo:registry protocol="multicast" address="224.5.6.7:1234">
    <dubbo:parameter key="unicast" value="false" />
</dubbo:registry>
```

#### Zookeeper注册中心

建议使用dubbo-2.3.3以上版本的zookeeper注册中心客户端

 Zookeeper是Apacahe Hadoop的子项目，是一个树型的目录服务，支持变更推送，适合作为Dubbo服务的注册中心，工业强度较高，可用于生产环境，并推荐使用，参见：[http://zookeeper.apache.org](http://zookeeper.apache.org/)   

![](.\Dubbo\pic\zookeeper.jpg)

流程说明：

- 服务提供者启动时

- - 向/dubbo/com.foo.BarService/providers目录下写入自己的URL地址。

- 服务消费者启动时

- - 订阅/dubbo/com.foo.BarService/providers目录下的提供者URL地址。
  - 并向/dubbo/com.foo.BarService/consumers目录下写入自己的URL地址。

- 监控中心启动时

- - 订阅/dubbo/com.foo.BarService目录下的所有提供者和消费者URL地址。

> 支持以下功能：
>
> - 当提供者出现断电等异常停机时，注册中心能自动删除提供者信息。
> - 当注册中心重启时，能自动恢复注册数据，以及订阅请求。
> - 当会话过期时，能自动恢复注册数据，以及订阅请求。
> - 当设置<dubbo:registry      check="false" />时，记录失败注册和订阅请求，后台定时重试。
> - 可通过<dubbo:registry      username="admin" password="1234" />设置zookeeper登录信息。
> - 可通过<dubbo:registry      group="dubbo" />设置zookeeper的根节点，不设置将使用无根树。
> - 支持*号通配符<dubbo:reference      group="*" version="*" />，可订阅服务的所有分组和所有版本的提供者。

在provider和consumer中增加zookeeper客户端jar包依赖: org.apache.zookeeper.zookeeper

支持zkclient和curator两种Zookeeper客户端实现

**ZKClient Zookeeper Registry**(缺省)

ZKClient是Datameer开源的一个Zookeeper客户端实现，开源比较早，参见：<https://github.com/sgroschupf/zkclient>

```
缺省配置：
<dubbo:registry ... client="zkclient" />
或
dubbo.registry.client=zkclient
或
zookeeper://10.20.153.10:2181?client=zkclient
```

需依赖 com.github.sgroschupf.zkclient

**Curator Zookeeper Registry**

Curator是Netflix开源的一个Zookeeper客户端实现，比较活跃

```
如果需要改为curator实现，请配置
<dubbo:registry ... client="curator" />
或
dubbo.registry.client=curator
或
zookeeper://10.20.153.10:2181?client=curator
```

依赖 com.netflix.curator.curator-framework

Zookeeper单机配置:

```
<dubbo:registry address="zookeeper://10.20.153.10:2181" />
或
<dubbo:registry protocol="zookeeper" address="10.20.153.10:2181" />
```

Zookeeper集群配置

```
<dubbo:registry address="zookeeper://10.20.153.10:2181?backup=10.20.153.11:2181,10.20.153.12:2181" />
或
<dubbo:registry protocol="zookeeper" address="10.20.153.10:2181,10.20.153.11:2181,10.20.153.12:2181" /
```

同一Zookeeper，分成多组注册中心:

```
<dubbo:registry id="chinaRegistry" protocol="zookeeper" address="10.20.153.10:2181" group="china" />
<dubbo:registry id="intlRegistry" protocol="zookeeper" address="10.20.153.10:2181" group="intl" />
```

#### Redis注册中心

Redis是一个高效的KV存储服务器

**Redis过期数据**
通过心跳的方式检测脏数据，服务器时间必须相同，并且对服务器有一定压力。

![](.\Dubbo\pic\redis.jpg)

数据结构：

- 使用Redis的Key/Map结构存储数据。

- - 主Key为服务名和类型。
  - Map中的Key为URL地址。
  - Map中的Value为过期时间，用于判断脏数据，脏数据由监控中心删除。(注意：服务器时间必需同步，否则过期检测会不准确)

- 使用Redis的Publish/Subscribe事件通知数据变更。

- - 通过事件的值区分事件类型：register, unregister, subscribe, unsubscribe。
  - 普通消费者直接订阅指定服务提供者的Key，只会收到指定服务的register, unregister事件。
  - 监控中心通过psubscribe功能订阅/dubbo/*，会收到所有服务的所有变更事件。

> 调用过程：
>
> 1. 服务提供方启动时，向Key:/dubbo/com.foo.BarService/providers下，添加当前提供者的地址。
> 2. 并向Channel:/dubbo/com.foo.BarService/providers发送register事件。
> 3. 服务消费方启动时，从Channel:/dubbo/com.foo.BarService/providers订阅register和unregister事件。
> 4. 并向Key:/dubbo/com.foo.BarService/providers下，添加当前消费者的地址。
> 5. 服务消费方收到register和unregister事件后，从Key:/dubbo/com.foo.BarService/providers下获取提供者地址列表。
> 6. 服务监控中心启动时，从Channel:/dubbo/*订阅register和unregister，以及subscribe和unsubsribe事件。
> 7. 服务监控中心收到register和unregister事件后，从Key:/dubbo/com.foo.BarService/providers下获取提供者地址列表。
> 8. 服务监控中心收到subscribe和unsubsribe事件后，从Key:/dubbo/com.foo.BarService/consumers下获取消费者地址列表。
>
> 选项：
>
> - 可通过<dubbo:registry      group="dubbo" />设置redis中key的前缀，缺省为dubbo。
>
> - 可通过<dubbo:registry      cluster="replicate" />设置redis集群策略，缺省为failover。
>
> - - failover:       只写入和读取任意一台，失败时重试另一台，需要服务器端自行配置数据同步。
>   - replicate:       在客户端同时写入所有服务器，只读取单台，服务器端不需要同步，注册中心集群增大，性能压力也会更大。

配置：

```
<dubbo:registry address="redis://10.20.153.10:6379" />
或
<dubbo:registry address="redis://10.20.153.10:6379?backup=10.20.153.11:6379,10.20.153.12:6379" />
或
<dubbo:registry protocol="redis" address="10.20.153.10:6379" />
或
<dubbo:registry protocol="redis" address="10.20.153.10:6379,10.20.153.11:6379,10.20.153.12:6379" />
```

#### Simple注册中心

注册中心本身就是一个普通的Dubbo服务，可以减少第三方依赖，使整体通讯方式一致。

> 此SimpleRegistryService只是简单实现，不支持集群，可作为自定义注册中心的参考，但不适合直接用于生产环境。

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
    http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd
    ">
 
    <!-- 当前应用信息配置 -->
    <dubbo:application name="simple-registry" />
 
    <!-- 暴露服务协议配置 -->
    <dubbo:protocol port="9090" />
 
    <!-- 暴露服务配置 -->
    <dubbo:service interface="com.alibaba.dubbo.registry.RegistryService" ref="registryService" registry="N/A" ondisconnect="disconnect" callbacks="1000">
        <dubbo:method name="subscribe"><dubbo:argument index="1" callback="true" /></dubbo:method>
        <dubbo:method name="unsubscribe"><dubbo:argument index="1" callback="false" /></dubbo:method>
    </dubbo:service>
 
    <!-- 简单注册中心实现，可自行扩展实现集群和状态同步 -->
    <bean id="registryService" class="com.alibaba.dubbo.registry.simple.SimpleRegistryService" />
 
</beans>
```

配置：

```
<dubbo:registry address="127.0.0.1:9090" />
或
<dubbo:service interface="com.alibaba.dubbo.registry.RegistryService" group="simple" version="1.0.0" ... >
<dubbo:registry address="127.0.0.1:9090" group="simple" version="1.0.0" />
```

#### Simple监控中心

监控中心也是一个标准的Dubbo服务，可以通过注册中心发现，也可以直连。

1. 暴露一个简单监控中心服务到注册中心: (如果是用安装包，不需要自己写这个配置，如果是自己实现监控中心，则需要)

   ```
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
       http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd
       ">
        
       <!-- 当前应用信息配置 -->
       <dubbo:application name="simple-monitor" />
        
       <!-- 连接注册中心配置 -->
       <dubbo:registry address="127.0.0.1:9090" />
        
       <!-- 暴露服务协议配置 -->
       <dubbo:protocol port="7070" />
        
       <!-- 暴露服务配置 -->
       <dubbo:service interface="com.alibaba.dubbo.monitor.MonitorService" ref="monitorService" />
        
       <bean id="monitorService" class="com.alibaba.dubbo.monitor.simple.SimpleMonitorService" />
        
   </beans>
   ```

2. 通过注册中心发现监控中心服务

   ```
   <dubbo:monitor protocol="registry" />
   或
   dubbo.monitor.protocol=registry
   ```

3. 暴露一个简单监控中心服务，但不注册到注册中心: (如果是用安装包，不需要自己写这个配置，如果是自己实现监控中心，则需要)

   ```
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
       http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd
       ">
        
       <!-- 当前应用信息配置 -->
       <dubbo:application name="simple-monitor" />
        
       <!-- 暴露服务协议配置 -->
       <dubbo:protocol port="7070" />
        
       <!-- 暴露服务配置 -->
       <dubbo:service interface="com.alibaba.dubbo.monitor.MonitorService" ref="monitorService" registry="N/A" />
        
       <bean id="monitorService" class="com.alibaba.dubbo.monitor.simple.SimpleMonitorService" />
        
   </beans>
   ```

4. 直连监控中心服务:

   ```
   <dubbo:monitor address="dubbo://127.0.0.1:7070/com.alibaba.dubbo.monitor.MonitorService" />
   或
   <dubbo:monitor address="127.0.0.1:7070" />
   或
   dubbo.monitor.address=127.0.0.1:7070
   ```

## Dubbo/RPC 常见问题

序列化，就是把数据结构或者是一些对象，转换为二进制串的过程，而反序列化是将在序列化过程中所生成的二进制串转换成数据结构或者对象的过程。

##### dubbo 支持不同的通信协议

- dubbo协议    默认就是走 dubbo 协议，单一长连接，进行的是 NIO 异步通信，基于 hessian 作为序列化协议。使用的场景是：传输数据量小（每次请求在 100kb 以内），但是并发量很高。
- rmi 协议   走 Java 二进制序列化，多个短连接，适合消费者和提供者数量差不多的情况，适用于文件的传输，一般较少用。
- hessian 协议   走 hessian 序列化协议，多个短连接，适用于提供者数量比消费者数量还多的情况，适用于文件的传输，一般较少用。
- http 协议   走 json 序列化
- webservice   走 SOAP 文本

##### 默认使用的是什么通信框架

 默认也推荐使用netty框架，还有mina。 

##### Dubbo原理



##### Dubbo调用流程

0. 服务容器负责启动，加载，运行服务提供者。
1. 务提供者在启动时，向注册中心注册自己提供的服务。 
2. 服务消费者在启动时，向注册中心订阅自己所需的服务。 
3. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。 
4. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。 
5. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心 

##### dubbo 支持的序列化协议？

dubbo 支持 hession、Java 二进制序列化、json、SOAP 文本序列化多种序列化协议。但是 hessian 是其默认的序列化协议。

##### 服务提供者能实现失效踢出是什么原理？

服务失效踢出基于 zookeeper 的临时节点原理。

##### Dubbo负载均衡策略

-   **Random LoadBalance 随机**，按权重设置随机概率。 **Dubbo的默认负载均衡策略** 

   在一个截面上碰撞的概率高，但调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重。 

-  **RoundRobin LoadBalance 轮循**，按公约后的权重设置轮循比率。 

    存在慢的提供者累积请求问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上。 

-  **LeastActive LoadBalance 最少活跃调用数**，相同活跃数的随机，活跃数指调用前后计数差。 

   使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差会越大。 

-  **ConsistentHash LoadBalance 一致性Hash**，相同参数的请求总是发到同一提供者。 

   当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动。 

##### 服务调用超时问题怎么解决

dubbo在调用服务不成功时，默认是会重试两次的。这样在服务端的处理时间超过了设定的超时时间时，就会有重复请求。

对于核心的服务中心，去除dubbo超时重试机制，并重新评估设置超时时间。 业务处理代码必须放在服务端，客户端只做参数验证和服务调用，不涉及业务流程处理 全局配置实例

```
<dubbo:provider delay="-1" timeout="6000" retries="0"/>
```

##### 服务调用是阻塞的吗？

默认是阻塞的，可以异步调用，没有返回值的可以这么做。

Dubbo 是基于 NIO 的非阻塞实现并行调用，客户端不需要启动多线程即可完成并行调用多个远程服务，相对多线程开销较小，异步调用会返回一个 Future 对象。

##### Dubbo 集群容错有几种方案？

| 集群容错方案      | 说明                                       |
| :---------------- | :----------------------------------------- |
| Failover Cluster  | 失败自动切换，自动重试其它服务器（默认）   |
| Failfast Cluster  | 快速失败，立即报错，只发起一次调用         |
| Failsafe Cluster  | 失败安全，出现异常时，直接忽略             |
| Failback Cluster  | 失败自动恢复，记录失败请求，定时重发       |
| Forking Cluster   | 并行调用多个服务器，只要一个成功即返回     |
| Broadcast Cluster | 广播逐个调用所有提供者，任意一个报错则报错 |

##### Dubbo 服务降级，失败重试怎么做？

 可以通过dubbo:reference 中设置 mock="return null"。mock 的值也可以修改为 true，然后再跟接口同一个路径下实现一个 Mock 类，命名规则是 “接口名称+Mock” 后缀。然后在 Mock 类里实现自己的降级逻辑 

##### Dubbo 配置文件是如何加载到Spring中的？

 Spring容器在启动的时候，会读取到Spring默认的一些schema以及Dubbo自定义的schema，每个schema都会对应一个自己的NamespaceHandler，NamespaceHandler里面通过BeanDefinitionParser来解析配置信息并转化为需要加载的bean对象！ 

##### Dubbo Monitor 实现原理？

Consumer端在发起调用之前会先走filter链；provider端在接收到请求时也是先走filter链，然后才进行真正的业务逻辑处理。

默认情况下，在consumer和provider的filter链中都会有Monitorfilter。

1. MonitorFilter向DubboMonitor发送数据 

2. DubboMonitor将数据进行聚合后（默认聚合1min中的统计数据）暂存到ConcurrentMapstatisticsMap，然后使用一个含有3个线程（线程名字：DubboMonitorSendTimer）的线程池每隔1min钟，调用SimpleMonitorService遍历发送statisticsMap中的统计数据，每发送完毕一个，就重置当前的Statistics的AtomicReference 
3. SimpleMonitorService将这些聚合数据塞入BlockingQueue queue中（队列大写为100000） 
4. SimpleMonitorService使用一个后台线程（线程名为：DubboMonitorAsyncWriteLogThread）将queue中的数据写入文件（该线程以死循环的形式来写）
5. SimpleMonitorService还会使用一个含有1个线程（线程名字：DubboMonitorTimer）的线程池每隔5min钟，将文件中的统计数据画成图表 

##### 什么是SPI

spi全称Service Provider Interface， 服务提供接口， 是Java提供的一套用来被第三方实现或者扩展的API。

SPI的核心思想是解耦，基于接口、策略模式、配置实现实现类的动态扩展。

##### Dubbo SPI

> Dubbo为什么不用java的SPI而选择自己再重写一套呢:
>
> - JDK 标准的 SPI 会一次性实例化扩展点所有实现，如果有扩展实现初始化很耗时，但如果没用上也加载，会很浪费资源。
> - 如果扩展点加载失败，连扩展点的名称都拿不到了。
> - 不支持AOP和IOC。

Dubbo 的SPI 扩展从 JDK 标准的 SPI (Service Provider Interface) 扩展点发现机制加强而来。改进了 JDK 标准的 SPI 的以上问题。

##### 什么是RPC

RPC（Remote Procedure Call Protocol）远程过程调用协议，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。简言之，RPC使得程序能够像访问本地系统资源一样，去访问远端系统资源。比较关键的一些方面包括：通讯协议、序列化、资源（接口）描述、服务框架、性能、语言支持等。

简单的说，RPC就是从一台机器(客户端)上通过参数传递的方式调用另一台机器(服务器)上的一个函数或方法(可以统称为服务)并得到返回的结果。

##### RPC架构组件

一个基本的RPC架构里面应该至少包含以下4个组件：

1. 客户端（Client）:服务调用方（服务消费者）
2. 客户端存根（Client Stub）:存放服务端地址信息，将客户端的请求参数数据信息打包成网络消息，再通过网络传输发送给服务端
3. 服务端存根（Server Stub）:接收客户端发送过来的请求消息并进行解包，然后再调用本地服务进行处理
4. 服务端（Server）:服务的真正提供者

![](\Dubbo\pic\RPC架构.png)

具体调用过程：

1、服务消费者（client客户端）通过调用本地服务的方式调用需要消费的服务；

2、客户端存根（client stub）接收到调用请求后负责将方法、入参等信息序列化（组装）成能够进行网络传输的消息体；

3、客户端存根（client stub）找到远程的服务地址，并且将消息通过网络发送给服务端；

4、服务端存根（server stub）收到消息后进行解码（反序列化操作）；

5、服务端存根（server stub）根据解码结果调用本地的服务进行相关处理；

6、本地服务执行具体业务逻辑并将处理结果返回给服务端存根（server stub）；

7、服务端存根（server stub）将返回结果重新打包成消息（序列化）并通过网络发送至消费方；

8、客户端存根（client stub）接收到消息，并进行解码（反序列化）；

9、服务消费方得到最终结果；

而RPC框架的实现目标则是将上面的第2-10步完好地封装起来，也就是把调用、编码/解码的过程给封装起来，让用户感觉上像调用本地服务一样的调用远程服务。

##### RPC和SOA、SOAP、REST的区别

1. REST

   可以看着是HTTP协议的一种直接应用，默认基于JSON作为传输格式,使用简单,学习成本低效率高,但是安全性较低。

2. SOAP

   SOAP是一种数据交换协议规范,是一种轻量的、简单的、基于XML的协议的规范。而SOAP可以看着是一个重量级的协议，基于XML、SOAP在安全方面是通过使用XML-Security和XML-Signature两个规范组成了WS-Security来实现安全控制的,当前已经得到了各个厂商的支持 。

   它有什么优点？简单总结为：易用、灵活、跨语言、跨平台。

3. SOA

   面向服务架构，它可以根据需求通过网络对松散耦合的粗粒度应用组件进行分布式部署、组合和使用。服务层是SOA的基础，可以直接被应用调用，从而有效控制系统中与软件代理交互的人为依赖性。

   SOA是一种粗粒度、松耦合服务架构，服务之间通过简单、精确定义接口进行通讯，不涉及底层编程接口和通讯模型。SOA可以看作是B/S模型、XML（标准通用标记语言的子集）/Web Service技术之后的自然延伸。

4. 以上三者有何区别

   没什么太大区别，他们的本质都是提供可支持分布式的基础服务，最大的区别在于他们各自的的特点所带来的不同应用场景 。

##### RPC的实现基础？

- 需要有非常高效的网络通信，比如一般选择Netty作为网络通信框架；
- 需要有比较高效的序列化框架，比如谷歌的Protobuf序列化框架；
- 可靠的寻址方式（主要是提供服务的发现），比如可以使用Zookeeper来注册服务等等；
- 如果是带会话（状态）的RPC调用，还需要有会话和状态保持的功能；

##### RPC使用了哪些关键技术？

1. 动态代理

   生成Client Stub（客户端存根）和Server Stub（服务端存根）的时候需要用到Java动态代理技术，可以使用JDK提供的原生的动态代理机制，也可以使用开源的：CGLib代理，Javassist字节码生成技术。

2. 序列化和反序列化

   在网络中，所有的数据都将会被转化为字节进行传送，所以为了能够使参数对象在网络中进行传输，需要对这些参数进行序列化和反序列化操作。

   序列化：把对象转换为字节序列的过程称为对象的序列化，也就是编码的过程。

   反序列化：把字节序列恢复为对象的过程称为对象的反序列化，也就是解码的过程。

   目前比较高效的开源序列化框架：如Kryo、FastJson和Protobuf等。

3. NIO通信

   出于并发性能的考虑，传统的阻塞式 IO 显然不太合适，因此我们需要异步的 IO，即 NIO。Java 提供了 NIO 的解决方案，Java 7 也提供了更优秀的 NIO.2 支持。可以选择Netty或者MINA来解决NIO数据传输的问题。

4. 服务注册中心

   可选：Redis、Zookeeper、Consul 、Etcd。一般使用ZooKeeper提供服务注册与发现功能，解决单点故障以及分布式部署的问题(注册中心)。

##### RPC实现原理架构

![](.\Dubbo\pic\RPC实现原理架构.png)

##### Dubbo 用到哪些设计模式？

 **工厂模式**Provider在export服务时，会调用ServiceConfig的export方法。ServiceConfig中有个字段： 

```java
private static final Protocol protocol = ExtensionLoader.getExtensionLoader(Protocol.
class).getAdaptiveExtension();
```

 Dubbo里有很多这种代码。这也是一种工厂模式，只是实现类的获取采用了JDK SPI的机制。这么实现的优点是可扩展性强，想要扩展实现，只需要在classpath下增加个文件就可以了，代码零侵入。另外，像上面的Adaptive实现，可以做到调用时动态决定调用哪个实现，但是由于这种实现采用了动态代理，会造成代码调试比较麻烦，需要分析出实际调用的实现类。 

 **装饰器模式**Dubbo在启动和调用阶段都大量使用了装饰器模式。以Provider提供的调用链为例，具体的调用链代码是在ProtocolFilterWrapper的buildInvokerChain完成的，具体是将注解中含有group=provider的Filter实现，按照order排序，最后的调用顺序是： 

> EchoFilter -> ClassLoaderFilter -> GenericFilter -> ContextFilter -> ExecuteLimitFilter -> TraceFilter -> TimeoutFilter -> MonitorFilter -> ExceptionFilter 

 更确切地说，这里是装饰器和责任链模式的混合使用。例如，EchoFilter的作用是判断是否是回声测试请求，是的话直接返回内容，这是一种责任链的体现。而像ClassLoaderFilter则只是在主功能上添加了功能，更改当前线程的ClassLoader，这是典型的装饰器模式。

**观察者模式**Dubbo的Provider启动时，需要与注册中心交互，先注册自己的服务，再订阅自己的服务，订阅时，采用了观察者模式，开启一个listener。注册中心会每5秒定时检查是否有服务更新，如果有更新，向该服务的提供者发送一个notify消息，provider接受到notify消息后，即运行NotifyListener的notify方法，执行监听器方法。

**动态代理模式**Dubbo扩展JDK SPI的类ExtensionLoader的Adaptive实现是典型的动态代理实现。Dubbo需要灵活地控制实现类，即在调用阶段动态地根据参数决定调用哪个实现类，所以采用先生成代理类的方法，能够做到灵活的调用。生成代理类的代码是ExtensionLoader的createAdaptiveExtensionClassCode方法。代理类的主要逻辑是，获取URL参数中指定参数的值作为获取实现类的key。

























































