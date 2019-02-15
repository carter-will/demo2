# Filter

*Filter 技术是servlet 2.3 新增加的功能。servlet2.3是sun公司于2000年10月发布的，它的开发者包括许多个人和公司团体，充分体现了sun公司所倡导的代码开放性原则。在众多参与者的共同努力下，servlet2.3比以往功能都强大了许多，而且性能也有了大幅提高。*

### 特色功能

**它新增加的功能包括:**

1. 应用程序生命周期事件控制;
2. 新的国际化;
3. 澄清了类的（装载规则）
4. 新的错误及安全属性;
5. 不赞成使用HttpUtils 类;
6. 各种有用的方法;
7. 阐明并扩展了几个servlet DTD;
8. filter功能.

### 功能介绍

其中最重要的就是filter功能.它使用户可以改变一个request和修改一个response. Filter 不是一个servlet，它不能产生一个response，它能够在一个request到达servlet之前预处理request，也可以在response离开servlet时处理response.换种说法，filter其实是一个"servlet chaining"(servlet 链).  包括:

1. 在servlet被调用之前截获;
2. 在servlet被调用之前检查servlet request;
3. 根据需要修改request头和request数据;
4. 根据需要修改response头和response数据;
5. 在servlet被调用之后截获.

你能够配置一个filter 到一个或多个servlet;单个servlet或servlet组能够被多个filter 使用。几个实用的filter 包括:用户辨认filter，日志filter，审核filter，加密filter，符号filter，能改变xml内容的XSLT filter等。

一个filter必须实现javax.servlet.Filter。

三个方法

1. void setFilterConfig(FilterConfig config) //设置filter 的配置对象;
2. FilterConfig getFilterConfig() //返回filter的配置对象;
3. void doFilter(ServletRequest req,ServletResponse res,FilterChain chain) //执行filter 的工作.

注:现setFilterConfig和getFilterConfig方法已取消，代之为init(FilterConfig config)和destory()方法。

（服务器)每次只调用setFilterConfig方法一次准备filter 的处理;调用doFilter方法多次以处理不同的请求.FilterConfig接口有方法可以找到filter名字及初始化参数信息.服务器可以设置FilterConfig为空来指明filter已经终结.

每一个filter从doFilter()方法中得到当前的request及response.在这个方法里，可以进行任何的针对request及response的操作.(包括收集数据，包装数据等).filter调用chain.doFilter()方法把控制权交给下一个filter.一个filter在doFilter()方法中结束.如果一个filter想停止request处理而获得对response的完全的控制，那它可以不调用下一个filter.

一个filter可以包装request 或response以改变几个方法和提供用户定制的属性.Api2.3提供了HttpServletRequestWrapper 和HttpServletResponseWrapper来实现.它们能分派最初的request和response.如果要改变一个方法的特性，必须继承wapper和重写方法.下面是一段简单的日志filter用来记录所有request的持续时间.

```
public class LogFilter implements Filter {

FilterConfig config;

public void setFilterConfig(FilterConfig config) {

this.config = config;

}

public FilterConfig getFilterConfig() {

return config;

}

public void doFilter(ServletRequest req,

ServletResponse res,

FilterChain chain) {

ServletContext context = getFilterConfig().getServletContext();

long bef = System.currentTimeMillis();

chain.doFilter(req,res); // no chain parameter needed here

long aft = System.currentTimeMillis();

context.log("Request to " + req.getRequestURI()

+ ": " + (aft-bef));

}

}
```

当server调用setFilterConfig(),filter保存config信息.在doFilter()方法中通过config信息得到servletContext.如果要运行这个filter，必须去配置到web.xml中.以tomcat4.01为例:

```
<filter>

<filter-name>

log //filter 名字

</filter-name>

<filter-class>

LogFilter //filter class(上例的servlet)

</filter-class>

</filter>

<filter-mapping>

<filter-name>log</filter-name>

< url-pattern>/*< /url-pattern>

</filter-mapping>

<servlet>

<servlet-name>servletname</servletname>

<servletclass>servletclass</servlet-class>

</servlet>

<servlet-mapping>

<servlet-name>servletname</servlet-name>

<url-pattern>*</url-pattern>

</servlet-mapping>
```

把这个web.xml放到web-inf中(详请参考tomcat帮助文档).

当每次请求一个request时(如index.jsp)，先到LogFilter中去并调用doFilter()方法，然后才到各自的servlet中去.如果是一个简单的servlet(只是一个页面，无任何输出语句)，那么可能的输出是:

Request to /index.jsp: 10

**参考**     [filter](https://baike.so.com/doc/5450367-5688736.html)

## Filter和Intercepor的区别

> Filter

 该过滤器的方法是创建一个类XXXFilter实现此接口，并在该类中的doFilter方法中声明过滤规则，然后在配置文件web.xml中声明他所过滤的路径

```
 <filter>
        <filter-name>XXXFilter</filter-name>
        <filter-class>
            com.web.util.XXXFilter
        </filter-class>
    </filter>
    
    <filter-mapping>
        <filter-name>XXXFilter</filter-name>
        <url-pattern>*.action</url-pattern>
    </filter-mapping>
```

> Intercepteor

 该过滤器的方法也是创建一个类XXXInterceptor实现此接口,在该类中intercept方法写过滤规则，不过它过滤路径的方法和Filter不同，它与strut.xml结合使用，
   创建一个strus.xml的子配置文件struts-l99-default.xml，它继承与struts2的struts-default，此配置文件是其他子配置文件的父类，只要是继承与该文件的配置文件所声明的路径都会被它过滤 如下

```
<package name="XXX-default" namespace="/" extends="struts-default">
        <interceptors>
            <interceptor name="authentication" class="com.util.XXXInterceptor" />
            
            <interceptor-stack name="user">
                <interceptor-ref name="defaultStack" />
                <interceptor-ref name="authentication" />
            </interceptor-stack>

            <interceptor-stack name="user-submit">
                <interceptor-ref name="user" />
                <interceptor-ref name="token" />
            </interceptor-stack>

            <interceptor-stack name="guest">
                <interceptor-ref name="defaultStack" />
            </interceptor-stack>

            <interceptor-stack name="guest-submit">
                <interceptor-ref name="defaultStack" />
                <interceptor-ref name="token" />
            </interceptor-stack>

        </interceptors>
        <default-interceptor-ref name="user" />
   </package>
```

**比较**

- 比较一,filter基于回调函数，我们需要实现的filter接口中doFilter方法就是回调函数，而interceptor则基于ｊａｖａ本身的反射机制,这是两者最本质的区别。

- 比较二,filter是依赖于servlet容器的，即只能在servlet容器中执行，很显然没有-servlet容器就无法来回调doFilter方法。而interceptor与servlet容器无关。
- 比较三，Filter的过滤范围比Interceptor大,Filter除了过滤请求外通过通配符可以保护页面，图片，文件等等，而Interceptor只能过滤请求。
- 比较四，Filter的过滤例外一般是在加载的时候在init方法声明,而Interceptor可以通过在xml声明是guest请求还是user请求来辨别是否过滤

