##tomcat目录结构

- bin    用于存放tomcat启动，停止等批处理和shell脚本

  startup.sh     startup.bat        shutdown.sh  shutdown.bat

- conf 用于存放tomcat的相关配置文件

  conf/Catalina  存储针对每个虚拟机的Content配置

  conf/context.xml   用于定义所有Web应用均需要加载的Context配置

  conf/catalinaproperties  tomcat环境变量

  conf/catalina.policy  当tomcat在安全模式下运行时，此文件为默认的安全策略配置

  conf/logging.properties   tomcat 日志配置文件，可通过该文件修改tomcat日志级别和日志路径

  conf/server.xml    tomcat服务器核心配置文件，用于配置tomcat的链接器、监听端口，处理请求的虚拟主机等

  conf/tomcat-users.xml  用于定义tomcat默认用户及角色映射信息，tomcat的manager模块即用该文件中定义的用户进行安全认证

  conf/web.xml  tomcat中所有应用默认的部署描述文件，主要定义了基础servlet和MIMI映射，如果应用中不包含web.xml，那么tomcat将使用此文件初始化部署描述，反之，tomcat会在启动时将默认部署描述与自定义配置进行合并。

- lib tomcat服务器依赖库目录，包含tomcat服务器运行环境依赖jar包
- logs tomcat默认的日志存放路径
- webapps tomcat默认的Web应用部署目录
- work Web应用JSP代码生成和编译临时目录

##Tomcat总体架构 
> 一个Server包含多个service(它们互相独立，只是共享一个JVM以及系统类库),一个Service负责维护多个Connector和一个Container,这样来自Connector的请求只能由它所属Service维护的Container处理。
> 一个Server可以包含多个Connector和Container.其中Connector负责开启Socket并监听客户端请求、返回响应数据；Container负责具体的请求处理。Connector和Container分别拥有自己的start()和stop()方法来加载和释放自己维护的资源。
>
> Container也可以命名为Engine,用于表示整个Servlet引擎
>
> 使用Context来表示一个Web应用，并且一个Engine可以包含多个Context。
>
> 在tomcat中，Servlet定义被称为Wrapper

我们使用Container来表示容器，container可以添加并维护子容器，因此Engine、Host、Context、Wrapper均继承自Container。它们之间是弱依赖关系，即它们之间的关系是通过Container的父子容器的概念来实现的。不过Service持有的是Engine接口。

#### Lifecycle 

所有组件均存在启动、停止等生命周期方法，拥有生命周期管理的特性，因此针对这些组件抽象了一个Lifecycle接口，该接口定义了生命周期管理的核心方法。

- Init()     初始化组件

- start()   启动组件

- stop()    停止组件

- destroy()   销毁组件

  同时，该接口支持组件状态以及状态之间的转换，支持添加事件监听器（LifecycleLister）用于监听组件的状态变化。如此，我们可以采用一致额机制来初始化、启动、停止以及销毁各个组件

tomcat默认提供了3个与状态无关的事件类型，其中PERIODIC_EVENT主要用于Container的后台定时处理，每次调用后触发该事件。

1. 周期事件（PERIODIC_EVENT）
2. 配置启动（CONFIGURE_START_EVENT）
3. 配置停止（CONFIGURE_STOP_EVENT）

#### Pipeline和Value

在tomcat中每个Container组件通过执行一个职责链来完成具体的请求处理。

Tomcat定义了Pipeline(管道)和Value(阀)两个接口。前者用于构造职责链，后者代表职责链上的每个处理器。

Pipeline中维护了一个基础的Value,它始终位于Pipeline的末端（即最后执行），封装了具体的请求处理和输出响应的过程，然后，通过addValue()方法，我们可以为Pipeline添加其他的Value.后添加的Value位于基础Value之前，并按照添加顺序执行.Pipeline通过获得首个Value来启动整个链条的执行。

tomcat每个层级的容器（Engine、Host、Context、Wrapper)均有对应的基础Value实现，同时维护了一个Pipeline实例。因此，我们可以在任何层级的容器上针对请求处理进行扩展。

#### Connector涉及

​	Connector将接受到的客户端请求交由与请求地址匹配的容器处理，至少要完成如下几项功能：

- 监听服务器端口，读取来自客户端的请求
- 将请求数据按照指定协议进行解析
- 根据请求地址匹配正确的容器进行处理
- 将相应返回客户端

在tomcat中，ProtocolHandler表示一个协议处理器，针对不同协议和I/0方式，提供不同的实现。tomcat支持多协议，默认支持HTTP和AJP.同时支持多种I/O方式，包括BIO、NIO、APR。在tomcat 8之后新增了对NIO2和HTTP/2协议的支持。

---

在Connector启动时，Endpoint会启动线程来监听服务器端口，并在接收到请求后调用Processor进行数据读取。

当Processor读取客户端请求后，需要按照请求地址映射到具体的容器进行处理，这个过程即为请求映射。由于Tomcat各个组件采用通用的生命周期管理，而且可以通过管理工具进行状态变更，因此请求映射除考虑映射规则的实现外，还要考虑组件的注册和销毁。

Tomcat通过Mapper和Mapperlister两个类实现上述功能。前者用于维护映射信息，同时按照映射规则（Servlet规范定义）查找容器，后者实现了ContainerListener和LifecycleListener，用于在容器组件状态发生变更时，注册或者取消对应的容器映射信息。为了实现上述功能，MapperListener实现了Lifecycle接口，当其启动时（在Service启动时启动），会自动作为监听器注册到各个容器组件上，同时将已创建的容器注册到Mapper.

Tomcat通过适配器模式（Adapter）实现了Connector与Mapper、Container的解耦。Tomcat默认的Connector实现（Coyote）对应的适配器为CoyoteAdapter.

---

#### Executer

Tomcat提供了Executor接口来表示一个可以在组件间共享的线程池（默认使用jdk5提供的线程池技术），该接口同样继承自Lifecycle，可按照通用的组件进行管理。

在Tomcat中Executor由Service维护，因此同一个Service中的组件可以共享一个线程池。

如果没有定义任何线程池，相关组件（如Endpoint）会自动创建线程池，此时，线程池不再共享。

在Tomcat中，Endpoint会启动一组线程来监听Socket端口，当接收到客户端请求后，会创建请求处理对象，并交由线程池处理，由此支持并发处理客户端请求。

#### Bootstrap和Catalina

Tomcat通过类Catalina提供了一个Shell程序，用于解析server.xml创建各个组件，同时，负责启动、停止应用服务器（只需要启动Tomcat顶层组件Server即可）

Tomcat提供了Bootstrap作为应用服务器启动入口。Bootsrtap负责创建Catalina实例，根据执行参数调用Catalina相关方法完成针对应用服务器的操作（启动、停止）。

Bootstrap与Tomcat应用服务器完全松耦合（通过反射调用Catalina实例），它可以直接依赖JRE运行并为Tomcat应用服务器创建共享类加载器，用于构造Catalina实例以及整个Tomcat服务器。

### Tomcat启动

首先调用Init()方法进行组件的逐级初始化，然后再调用start()方法进行启动。每次调用均伴随着生命周期状态变更事件的触发。

### 请求处理

应用服务器的请求处理开始于监听的Socket端口接收到数据，结束于将服务器处理结果写入Socket输出流

### 类加载器

**J2SE标准类加载器**

JVM默认提供了3个类加载器，它们以一种父子树的方式创建，同时使用委派模式确保应用程序可通过自身的类加载器（System）加载所有可见的类。

- BootStrap: 用于加载JVM提供的基础运行类，即位于%JAVA_HOME%/jre/lib目录下的核心类库
- Extension:Java提供的一个标准的扩展机制用于加载除核心类库外的Jar包。即只要复制到指定的扩展目录下的jar，JVM会自动加载（不需要通过-classpath指定）。默认的扩展目录是%JAVA_HOME%/jre/lib/ext。
- System:用于加载环境变量CLASSPATH(不推荐使用)指定目录下的或者-classpath运行参数指定的jar包，System类加载器通常用于加载应用程序jar包及其启动入口类（Tomcat的BootStrap类即由System类加载器加载）


**Tomcat加载器**

应用服务器通常要求会自行创建类加载器以实现更灵活的控制，有以下几点特性：

- 隔离性： Web应用类库相互隔离，避免依赖库或者应用包相互影响。
- 灵活性：既然Web应用之间的类加载器相互独立，那么我们只能只针对一个Web应用进行重新部署，此时该Web应用的类加载器将会重新创建，而不影响其他Web应用。
- 性能：由于灭个Web应用都有一个类加载器，因此Web应用在加载类时，不会搜索其他Web应用包含的jar包，性能自然高于应用服务器只有一个类加载器的情况。

除了每个Web应用的类加载器外，Tomcat也提供了3个基础的类加载器和Web应用类加载器，而且这3个类加载器指向的路径和包列表均可以由catalina,properties配置.

- Common：以System为父类加载器，是位于Tomcat应用服务器顶层的公用类加载器。其路径为common.loader，默认执行$CATALINA_HOME/lib下的包。
- Catalina：以Common为父加载器，是用于加载Tomcat应用服务器的类加载器，其路径为server.loader,默认为空，此时Tomcat使用Common类加载器加载应用服务器
- Shared：以Common为父加载器，是所有Web应用的父加载器，其路径为shared,loader.默认为空。此时Tomcat使用Common类加载器作为Web应用的父加载器。
- Web应用：以Shared为父加载器，加载/WEB-INF/classes目录下的未压缩的Class和资源文件以及/WEB-INF/lib目录下的jar包。该类加载器支队当前Web应用可见，对其他Web应用均不可见

默认情况下，这三个集成类加载器是同一个。

**Web应用类加载器**

Java默认的类加载机制是委派模式，委派过程如下：

1. 从缓存中加载
2. 如果缓存中没有，则从父类加载器中加载
3. 如果父类加载器没有，则从当前类加载器加载
4. 如果没有，泽抛出异常

Tomcat提供的Web应用类加载器与默认的委派模式稍有不同，当进行类加载时，除JVM基础类库外，它会首先尝试通过当前类加载器加载，然后才进行委派。所以，Web应用类加载器默认加载顺序如下：

1. 从缓存中加载，
2. 如果没有，则从JVM的Bootstrap类加载器加载
3. 如果没有，则从当前类加载器加载（按照WEB-INF/classes、WEB-INF/lib的顺序）
4. 如果没有，则从父类加载器加载，由于父类加载器采用默认的委派模式，所以加载顺序为System、Common、Shared

Tomcat提供了delegate属性用于控制是否启用Java委派模式，默认为false(不启用)。的那个配置为true时，Tomcat将使用java默认的委派模式，即那如下顺序加载：

1. 从缓存中加载
2. 如果没有，从JVM的Bootstrap类加载器加载
3. 如果没有，则从父类加载器加载（System、Common、Shared）
4. 如果没有，则从当前类加载器加载

## Catalina

Catalina包含了所有的容器组件，以及安全、会话、集群、部署、管理等Servlet容器架构的各个方面。它通过松耦合的方式集成Coyote，以完成按照请求协议进行数据读写，同时，它还包括我们的启动入口。Shell程序等。

Tomcat模块包含Catalina,Coyote,Jasper,JavaEL,Naming,Juli

Tomcat本质上是一款Servlet容器，因此Catalina是Tomcat的核心，其他模块均为Catalina提供支撑。通过Coyote模块提供链接通信，Jasper模块提供JSP引擎，Naming提供JNDI服务，Juli提供日志服务。

**Digester**

Catalina使用Digester解析XML（server.xml）配置文件并创建应用服务器，Tomcat在Catalina的创建过程中通过Digester结合LifecycleListener做了大量的初始化工作。Digester是一款将XML转换为Java对象的事件驱动型工具，是对SAX的高层次封装。

Digester及SAX的事件驱动，就是通过流读取XML文件，当识别出特定XML节点后便会执行特定的动作，或者创建Java对象，或者执行对象的某个方法。因此Digester的核心是匹配模式和处理规则。

Digester是非线程安全的。

**创建Server**

- server的解析
  1. 创建Server实例
  2. 创建全局J2EE企业命名上下文
  3. 为Server添加生命周期监听器
  4. 构造Service实例
  5. 为Service添加生命周期监听器
  6. 为Service添加Executor
  7. 为Service添加Connector
  8. 为Connector添加虚拟主机SSL配置
  9. 为Connector添加生命周期监听器
  10. 为Connector添加升级协议
  11. 添加子元素解析规则
- Engine的解析
  1. 创建Engine实例
  2. 为Engine添加集群配置
  3. 为Engine添加生命周期监听器
  4. 为Engine添加安全配置
- Host的解析
  1. 创建Host实例
  2. 为Host添加集群
  3. 为Host添加生命周期管理
  4. 为Host添加安全配置
- Context的解析
  1. Context实例化
  2. 为Context添加生命周期监听器
  3. 为Context指定类加载器
  4. 为Context添加会话管理器
  5. 为Context添加初始化参数
  6. 为Context添加安全配置以及Web资源配置
  7. 为Context添加资源链接
  8. 为Context添加Value
  9. 为Context添加守护资源配置
  10. 为Context添加Cookie处理器

**Web应用加载**

> Catalina对Web应用的加载主要由StandardHost、HostConfig、StandardContext、ContextConfig、StandardWeapper这5个类完成

1. StandardHost

   StandardHost加载Web应用的入口有两个，其中一个是在Catalina构造Server实例时，如果Host元素存在Context子元素（server.xml中），那么Context元素将会作为Host容器的子容器添加到Host实例中，并在Host启动时，由生命周期管理接口的start()方法启动（默认调用子容器的start()方法）。另一个入口则是由HostConfig自动扫描部署目录，创建Context实例并启动，这是大多数Web应用的加载方式。

2. HostConfig

   HostConfig处理的生命周期事件包括：START_EVENT、PERIODIC_EVENT、STOP_EVENT.其中，前两者都与Web应用部署密切相关，后者用于在Host停止时注销其对应的MBean.

   1. START_EVENT

      - Context描述文件部署

        Context描述文件的部署过程如下：

        ​	（1）扫描Host配置文件基础目录，即$CATALINA_BASE/conf/<Engine名称>/<Host名称>，对于该目录下的每个配置文件，由线程池完成解析部署

        ​	（2）对于每个文件的部署线程，进行如下操作：

        ​		① 使用Digester解析配置文件，创建Context实例

        ​		②更新Context实例的名称、路径，因此<Context>元素中配置的path属性无效

        ​		③为Context添加ContextConfig生命周期监听器

        ​		④通过Host的addChild()方法将Context实例添加到Host，该方法会判断Host是否已启动，			    如果是，则直接启动Context

        ​		⑤将Context描述文件、Web应用目录及web.xml等添加到守护资源，以便文件发生变更时（使用资源文件的上次修改时间进行判断）,重新部署或者加载Web应用。

      - Web目录部署

        部署Web应用目录的过程如下：

        （1）对于Host的appBase目录（默认为$CATALINA_BASE/webapps）下所有符合条件的目录（不符合deployIgnore的过滤规则、目录不为META-INF和WEB-INF）,由线程池完成部署

        （2）对于每个目录进行如下操作

        ​	①如果Host的deployXML属性值为true（即通过Context描述文件部署），并且存在META-INF/context.xml文件，则使用Digester解析context.xml文件创建Context对象。如果Context的copyXML属性为true，则将描述文件复制到$CATALINA_BASE/conf/<Engine名称>/<Host名称>目录下，文件名与Web应用目录名相同。

        ​	如果deployXML属性值为false,但是存在META-INF/context.xml文件，则构造FailedContext实例（Catalina的空模式，用于表示Context部署失败）

        ​	②为Context实例添加ContextConfig生命周期监听器

        ​	③通过Host的addChild()方法将Context实例添加到Host。该方法会判断Host是否已启动，如果是，则直接启动Context

        ​	④将Context描述文件、Web应用目录及web.xml等添加到守护资源，以便文件发生变更时重新部署或者加载Web应用。守护文件因deployXML和copyXML的配置稍有不同。

      - WAR包部署

        具体的部署过程如下：

        (1)对于Host的appBase目录（默认为$CATALINA_BASE/webapps）下所有符合条件的WAR包（不符合deployIgnore的过滤规则、文件名不为META-INF和WEB-INF、以war作为扩展名的文件），由线程池完成部署

        (2)对于每个WAR包进行如下操作：

        ​	①如果Host的deployXML属性为true,且在WAR包同名目录（去除扩展名）下存在META-INF/context.xml文件，同时Context的copyXML属性为false,则使用该描述文件创建Context实例（用于WAR包解压目录位于部署目录的情况）

        ​	如果Host的deployXML属性为true,且在WAR包压缩文件下存在META-INF/context.xml文件，则使用该描述文件创建Context对象

        ​	如果deployXML属性为false,但是在WAR包压缩文件下存在META-INF/context.xml文件，则构造FailedContext实例（Catalina的空模式，用于表示Context部署失败）

        ​	②如果deployXML为true,且META-INF/context.xml存在于WAR包中，同时Context的copyXML属性为true,则将context.xml文件复制到$CATALINA_BASE/conf/<Engine名称>/<Host名称>目录下，文件名称同WAR包名称（去除扩展名）

        ​	③为Context实例添加ContextConfig生命周期监听器

        ​	④通过Host的addChild()方法将Context实例添加到Host。该方法会判断Host是否已启动，如果是，则直接启动Context

        ​	⑤将Context描述文件、WAR包及web.xml等添加到守护资源，以便文件发生变更时重新部署或者加载Web应用

   2. PERIODIC_EVENT

      在HostConfig中通过DeployedApplication维护了两个守护资源列表：redeployResources和reloadResources,前者用于守护导致应用重新部署的资源，后者守护导致应用重新加载的资源。两个列表分别维护了资源极其最后修改时间

      当HostConfig接收到PERIODIC-event事件后，会检测守护资源的变更情况。如果发生变更，将重新加载或者部署应用以及更新资源的最后修改时间

      具体的部署过程如下：

      (1) 对于每一个已部署的Web应用，检查用于重新部署的守护资源。对于每一个守护的资源文件或者目录，如果发生变更，那么就有一下几种情况：

      - 如果资源对应为目录，则仅更新守护资源列表中的上次修改时间
      - 如果Web应用存在Context描述文件并且当前变更的是WAR包文件，则得到原Ccontext的docBase.如果docBase不以“.war”结尾（即Context指向的是WAR解压目录）。删除解压目录并重新加载，否则直接重新加载。更新守护资源
      - 其他情况下，直接卸载应用，并由接下来的处理步骤重新部署

      (2)对于每个已部署的Web应用，检查用于重新加载的守护资源，如果资源发生变更，则重新加载Context对象

      (3)如果Host配置为卸载旧版本应用，则检查并卸载

      (4)部署Web应用（新增以及处于卸载状态的描述文件、Web应用目录、WAR包），部署过程同上面叙述。

3. StandardContext

   StandardContext启动过程如下：

   1. 发布正在启动的JMX通知，这样可以通过添加NoticeficationListener来监听Web应用的启动
   2. 启动当前Context维护的JNDI资源
   3. 初始化当前Context使用的WebResourceRoot并启动，WebResourceRoot维护了Web应用所有的资源集合（Class文件、Jar包以及其他资源文件），主要用于类加载和按照路径查找资源文件
   4. 创建Web应用类加载器（WebappLoader）。WebappLoader继承自LifecycleMBeanBase,在其启动时创建Web应用类加载器（WebappClassLoader）.此外，该类还提供了background-Process，用于Context后台处理。当检测到Web应用的类文件、Jar包发生变更时，重新加载Context
   5. 如果没有设置Coookie处理器，则创建默认的Rfc6265CookieProcessor
   6. 设置字符集映射（CharsetMapper）,该映射主要用于根据Locale获取字符集编码
   7. 初始化临时目录，默认为$CATALINA_BASE/work/<Endine名称>/<Host名称>/<Context名称>
   8. Web应用的依赖检测，主要检测依赖扩展点完整性
   9. 如果当前Context使用JNDI,则为其添加NamingContextListener.
   10. 启动Web应用类加载器（WebappLoader.start）,此时才真正创建WebappClassLoader实例。
   11. 启动安全组件（Realm）
   12. 发布CONFIGURE_START_EVENT事件，ContextConfig监听该事件以完成Servlet的创建
   13. 启动Context子节点（Wrapper）
   14. 启动Context维护的Pipeline
   15. 创建会话管理器。如果配置了集群组件，则由集群组件创建，否则使用标准的会话管理器（StandardManager）.在集群环境下，需要将会话管理器注册到集群组件
   16. 将Context的Web资源集合（org.apache.catalina.WebResourceRoot）添加到ServletContext属性，属性名为org.apache.catalina.resources
   17. 创建实例管理器（InstanceManager）,用于创建对象实例，如Servlet、Filter等
   18. 将Jar包扫描器（JarScanner）添加到ServletContext属性，属性名为org.apache.tomcat.JarScanner
   19. 合并ServletContext初始化参数和Context组件中的ApplicationParameter。合并原则：ApplicationParameter配置为可以覆盖，那么只有当ServletContext没有相关参数或者相关参数为空时添加；如果配置为不可覆盖，则强制添加，此时即使ServletContext配置了同名参数也不会生效。
   20. 启动添加到当前Context的ServletContainerInitializer.该类的实例具体由ContextConfig查找并添加。
   21. 实例化应用监听器（ApplicationListener）,分为事件监听器以及生命周期监听器。这些监听器可以通过Context部署描述文件、可编程的方式（ServletContainerInitializer）或者Web.xml添加，并且触发ServletContextListener.contextInitializer
   22. 检测未覆盖的HTTP方法的安全约束
   23. 启动会话管理器
   24. 实例化FilterConfig(ApplicationFilterConfig),Filter,并调用Filter.init初始化
   25. 对于loadOnStartup ≥ 0 的Wrapper,调用Wrapper.load(),该方法负责实例化Servlet,并调用Servlet.init进行初始化
   26. 启动后台定时处理线程，只有当backgroundProcessorDelay > 0 时启动，用于监听守护文件的变更等。当backgroundProcessorDelay ≤ 0时，表示Context的后台任务由上级容器（Host）调度
   27. 发布正在运行的JMX通知
   28. 调用WebResourceRoot.gc()释放资源（WebResourceRoot加载资源时，为了提高性能会缓存某些信息，该方法用于清理这些资源，如关闭JAR文件）。
   29. 设置Context的状态，如果启动成功，设置为STARTING(其父类LifecycleBase会自动将状态转换为STARTED),否则设置为FAILED.

4. ContextConfig

   - AFTER_INIT_ENENT事件
   - BEFORE_START_EVENT事件
   - CONFIGURE_START_EVENT事件

   Web容器初始化

   > Tomcat解析时确保Web应用中的配置优先级最高，其次为Host级，最后为容器级

   Tomcat初始化Web容器的过程如下：

   1. 解析默认配置，生成WebXml对象（Tomcat使用该对象表示web.xml的解析结果）

   2. 解析Web应用的web.xml文件。如果StandardContext不为空，则将改属性指向的文件作为web.xml.否则使用默认路径，即WEB-INF/web.xml.解析结果通用伟WebXml对象，暂时将其称为“主WebXml”

   3. 扫描Web应用所有JAR包，如果包含META-INF/web-fragment.xml，则解析文件并创建WebXml对象。暂时称其为“片段WebXml”.

   4. 将web-fragment.xml创建的WebXml对象按照Servlet规范进行排序，同时将按照排序结果对应的JAR文件名列表设置到ServletContext属性中，属性名为“javax.servlet.context.orderedLibs”.这个排序决定了Filter的执行顺序

   5. 查找ServletContainerInitializer实现，并创建实例，查找范围分为 Web应用下的包  和  容器包。Tomcat返回查找结果列表时，确保Web应用的顺序在容器之后，因此容器中的实现将先加载

   6. 根据ServletContainerInitializer查询结果以及javax.servlet.annotation.HandlesTypes注解配置，初始化typeInitializerMap和initializerClass两个映射（主要用于后续的注解检测），前者表示类对应的ServletContainerInitializer集合，而后者表示每个ServletContainerinitlizer对应的类的集合，具体类由javax.servlet.annotation.HandlesTypes注解指定

   7. 当“主WebXml”的metadataComplete为false或者typeInitializerMap不为空时。

      ①处理WEB-INF/classes下的注解，对于该目录下的每个类做如下处理。

      - 检测javax.servlet.annotation.HandlesTypes注解
      - 当WebXml的metadataComplete为false，查找javax.servlet.annotation.WebServlet、javax.servlet.annotation.WebFilter、javax.servlet.annotation.WebListener注解配置，将其合并到“主WebXml”

      ②处理JAR包内的注解，只处理包含web-fragment.xml的JAR,对于JAR包中的每个类做如下处理

      - 检测javax.servlet.annotation.HandlesTypes注解；
      - 当“主WebXml”和“片段WebXml”的metadataComplete均为false,查找javax.servlet.annotation.WebServlet、javax.servlet.annotation.WebFilter、javax.servlet.annotation.WebListener注解配置，将其合并到“片段WebXml”。

   8. 如果“主WebXml”的metadataComplete为false,将所有的“片段WebXml”按照排序顺序合并到“主WebXml”

   9. 将“默认WebXml”合并到“主WebXml”

   10. 配置JspServlet.

   11. 使用“主WebXml”配置当前StandardContext，包括Servlet、Filter、Listener等Servlet规范中支持的组件。对于ServletContext层级的对象，直接由StandardContext维护，对于Servlet,则创建StandardWrapper子对象，并添加到StandardContext实例

   12. 将合并后的WebXml保存到ServletContext属性中，便于后续处理复用，属性名为org.apache.tomcat.util.scan.MergeWebXml

   13. 查找JAR包“MATA-INF/resources/”下的静态资源，并添加到StandardContext

   14. 将ServletContainerInitializer扫描结果添加到StandardContext,以便StandardContext启动时使用。

应用程序注解配置

当StandardContext的ignoreAnnotation为false时，Tomcat支持读取如下接口的Java命名服务注解配置，添加相关的JNDI资源引用，以便在实例化相关接口时，进行JNDI资源依赖注入。

支持读取的接口如下：

- Web应用程序监听器

  - javax.servlet.ServletContextAttributelistener
  - javax.servlet.ServletRequestListener
  - javax.servlet.ServletRequestAttributeListener
  - javax.servlet.http.HttpSessionAttributelistener
  - javax.servlet.http.HttpSessionAttributelistener
  - javax.servlet.http.HttpContextListener

- javax.servlet.Filter

- javax.servlet.Servlet

  支持读取的注解包括类注解、属相注解、方法注解，具体注解如下。

  - 类： javax.annotation.Resource、 javax.annotation.Resource
  - 属性和方法 : javax.annotation.Resource


5. StandardWrapper

   StandardWrapper的load过程具体如下：

   (1). 创建Servlet实例，如果添加了JNDI资源注入，将进行依赖注入

   (2).读取javax.servlet.annotation.MaltipartConfig注解配置，以用于multipart/form-data请求处理，包括临时文件存储路径、上传文件最大字节数、请求最大字节数、文件大小阈值

   (3).读取javax.servlet.annotation.ServletSecurity()注解配置，添加Servlet安全

   (4).调用javax.servlet.Servlet.init()方法进行初始化

6. Context命名规则

   Tomcat支持同时以相同的Context路径部署多个版本的Web应用，此时Tomcat将按照如下规则将请求匹配到对应版本的Context:

   - 如果请求中不包含session信息，将使用最新版本
   - 如果请求中包含session信息，检查每个版本中的会话管理器，如果会话管理器包含当前会话，则使用该版本
   - 如果请求中包含session信息，但是并未找到匹配的版本，则使用最新版本

**Web请求处理**

CoyoteAdapter.service处理请求的过程如下：

1. 根据Connector的请求和响应对象创建Servlt请求和响应
2. 转化请求参数并完成请求映射
   - 请求URL解码，初始化请求的路径参数
   - 检查URI是否合法，如果非法，则返回响应码400
   - 请求映射，映射结果保存到org.apache.catalina.connector.Request.mappingData,类型为org.apache.tomcat.util.http.mapper.MappingData,请求映射处理最终会根据URI定位到一个有效的Wrapper
   - 如果映射结果MappingData的redirectPath属性不为空（即为重定向请求），则调用org.apache.catalina.connector.Response.sendRedirect发送重定向并结束
   - 如果当前Connector不允许追踪（allowTrace为false）且当前请求的Method为TRACE，则返回响应码405
   - 执行连接器的认证及授权
3. 得到当前Engine的第一个Value并执行（invoke）,以完成客户端请求处理
4. 如果为异步请求：
   - 获得请求读取时间监听器（ReadListener）;
   - 如果请求读取已经结束，触发ReadListener.onAllDataRead
5. 如果为同步请求：
   - Flush并关闭请求输入流
   - Flush并关闭响应输出流

> 请求映射

> Catalina请求处理

**DefaultServlet和JspServlet**

1. DefaultServlet

   url-pattern为“/”，作为默认的Servlet,主要用于处理静态资源。

   还支持查看目录列表，默认情况下，Tomcat以HTML的形式输出文件目录列表（包括文件名、大小、最后修改时间）。此外，可以通过参数localXslFile、contextXslFile或globalXslFile指定一个XSL或XSLT文件。

2. JspServlet

   url-pattern为 *.jsp和 *.jspx, 负责处理所有JSP文件的请求

   主要完成以下工作：

   - 根据JSP文件生成对应Servlet的JAVA代码
   - 将JAVA代码编译为JAVA类，Tomcat支持Ant和JDT(eclipse提供的编译器)两种方式编译JSP类，默认JDT.
   - 构造Servlet类实例并执行请求



## Coyote

### 简介

Coyote----Tomcat链接器框架，是Tomcat服务器提供的供客户端访问的外部接口。客户端通过Coyote与服务器建立链接、发送请求并接收响应。

Coyote封装了底层的网络通信（Socket请求及响应处理），为Catalina容器提供了统一的接口，使Catalina容器与具体的请求协议及I/O方式解耦。Coyote将Socket输入转换为Request对象，交由Catalina容器进行处理，处理请求完成后，Catalina通过Coyote提供的Reponse对象将结果写入输出流。

在Coyote中，Tomcat支持以下3中传输协议

- HTTP/1.1协议：绝大多数Web应用采用的访问协议，主要用于Tomcat单独运行（不与Web服务器集成）的情况
- AJP协议：用于和Web服务器集成，以实现针对静态资源的优化以及集群部署，当前支持AJP/1.3
- HTTP/2.0: 下一代HTTP协议,自Tomcat8.5以及9.0版本开始支持

针对HTTP和AJP协议，Coyote又按照I/O方式提供了不同的选择方案：

- NIO: 采用Java NIO类库实现
- NIO2: 采用JDK7最新的NIO2类库实现
- APR: 采用APR实现（Apache可移植运行库，C/C++）

> 在8.0之前，Tomcat默认采用的I/O方式为BIO,之后改为NIO

### Web请求处理

几个核心概念：

- EndPoint : Coyote通信端点，即通信监听的接口，是具体的Socket接受处理类，是对传输层的抽象
- Processor: Coyote协议处理接口，负责构造Request和Response对象，并通过Adapter将其提交到Catalina容器处理，是对应用层的抽象、Processor是单线程的，Tomcat在同一次链接中复用Processor
- ProtocolHandler: Coyote协议接口，通过封装EndPoint和Processor，实现针对具体协议的处理功能。
- UpgradeProtocol: Tomcat采用UpgradeProtocol接口表示HTTP升级协议，当前只提供了一个实现(Http2Protocol)用于处理HTTP/2.0.

#### 请求处理

Connector请求处理过程：

1. 当Connector启动时，会启动其持有的Endpoint实例。EndPoint并行运行多个线程（由属性accptorThreadount）,每个线程运行一个AbstractEndPoint.Acceptor实例。在AbstarctEndPoint.Acceptor实例中监听端口通信（I/O方式不同，具体的处理方式也不同），而且只要EndPoint处于运行状态，始终循环监听。
2. 当监听到请求时，Acceptor将Socket封装为SocketWrapper实例（此时并未读取数据），并交由一个SocketProcessor对象处理（此过程也由线程池异步处理）。
3. SocketProcessor是一个线程池Worker实例，每个I/O方式均有自己的实现，它首先判断Socket的状态（如完成SSL握手），然后提交到ConnectionHandler处理。
4. ConnectionHandler是AbstarctProtocol的一个内部类，主要用于为链接选择一个合适的Processor实现进行请求处理。ConnectorHandler调用Processor.orocess()方法进行请求处理。如果不是协议协商的请求（如普通的HTTP/1.1请求或者AJP请求），那么Processor则会直接调用CoyoteAdapter.service()方法将其提交到Catalina容器处理。如果是协议协商请求，Processor会返回SocketState.UPGRADING,由ConnectionHandler进行协议升级
5. 协议升级时，ConnectionHandler会从当前Processor得到一个UpgradeToken对象（如果没有，则默认为HTTP/2），并构造一个升级Processor实例（如果是Tomcat支持的协议则会是UpgradeProcessorInternal,否则是UpgradeProcessorExternal）替换当前的Processor,并将当前的Processor释放回收。替换后，该链接的后续处理将由升级Processor完成.
6. 通过UpgradeToken中HttpUpgradeHandler对象的init()方法进行初始化，以便准备开始启用新协议。

#### 协议升级

HTTP/2.0通过8.5新增的UpgradeProtocol接口创建HttpUpgradeHandler以及UpgradeToken,而WebSocket则是通过过滤器WsFilter判断当前请求是否为WebSocket升级请求，如果是，则调用当前请求的upgrade()方法构造UpgradeToken并传递给Http11Protocol.

对于第一次协议协商的过程，HTTP/2.0是由链接器直接处理的，并未提交到Servlet容器，而WebSocket则提交到了Servlet容器。

由于HTTP/2.0是多路复用的协议，也就是多个HTTP请求通过一个链接完成，因此对于Http2UpgradeHandler,会将每次请求响应交给StreamProcessor处理。而StreamProcessor则会将请求提交到Servlet容器。

### HTTP

>  http协议特点: 
>
> 支持客户端/服务端模式、简单快速、灵活、无链接、无状态

Tomcat会自动检测当前服务是否安装了APR,如果安装了APR,那么Tomcat将自动使用APR处理HTTP（Http11AprProtocol）,否则使用NIO

### AJP















