# RabbitMQ学习

## 安装与配置

- Rabbitmq管理插件启动

rabbitmq-plugins enable rabbitmq_management 启动

rabbitmq-plugins disable rabbitmq_management 关闭

或

```
"D:\devlenv\rabbitMQ\rabbitmq_server-3.7.8\sbin\rabbitmq-plugins.bat" enable rabbitmq_management
```

- 启动方式

  - 以应用方式启动

    >  rabbitmq-server -detached 后台启动
    >
    >  Rabbitmq-server 直接启动，如果你关闭窗口或者需要在改窗口使用其他命令时应用就会停止
    >
    >  关闭:rabbitmqctl stop

  - 以服务方式启动

    > rabbitmq-service install 安装服务
    >
    > rabbitmq-service start 开始服务
    >
    > Rabbitmq-service stop  停止服务
    >
    > Rabbitmq-service enable 使服务有效
    >
    > Rabbitmq-service disable 使服务无效
    >
    > rabbitmq-service help 帮助
    >
    > 关闭:rabbitmqctl stop

  net start RabbitMQ   启动              net stop RabbitMQ    关闭      

  `net stop RabbitMQ && net start RabbitMQ `重启MQ服务器

  *当rabbitmq-service install之后默认服务是enable的，如果这时设置服务为disable的话，rabbitmq-service start就会报错。当rabbitmq-service start正常启动服务之后，使用disable是没有效果的* 

- Rabbitmq节点管理方式

  rabbitmqctl list_queues name messages_ready messages_unacknowledge

> (主要用于检查 unacknowledged的队列排查问题) 

- 创建用户、密码、绑定角色

  **查看已有用户及用户的角色**  

  ```
  rabbitmqctl.bat list_users
  ```

  **新增一个用户**  

  ```
  rabbitmqctl.bat add_user username password
  ```

  rabbitmq用户角色

  | 角色                  | 含义                                       |
  | ------------------- | ---------------------------------------- |
  | 超级管理员（adminstrator） | 可登录管理控制台（弃用management plugin的情况下），可查看所有的信息，并且可以对用户，策略（policy）进行操作。 |
  | 监控者（monitoring）     | 可登录管理控制台，同事可以查看rabbitmq节点的相关信息（进程数，内存使用情况，磁盘使用情况等） |
  | 策略制定者（policymaker）  | 可登录管理控制台，同时可以对policy进行管理                 |
  | 普通管理员               | 仅可登录管理控制台，无法看到节点信息，也无法对策略进行管理            |
  | 其他的                 | 无法登录管理控制台，通常就是普通的生产者和消费者                 |

  **配置用户角色** 

  ```
  rabbitmqctl.bat set_user_tags username administrator
  ```

  也可以设置多个角色

  ```
  rabbitmqctl.bat  set_user_tags  username tag1 tag2 ...
  ```

  **更换用户密码** 

  ```
  rabbitmqctl change_password userName newPassword
  ```

  **删除用户**

  ```
  rabbitmqctl.bat delete_user username
  ```

  **测试rabbitmq是否安装成功**

  `rabbitmqctl status`

  **设置用户权限**

  `rabbitmqctl  set_permissions  -p  VHostPath  User  ConfP  WriteP  ReadP`

  **查看所有（指定hostpath）用户的权限信息**  

  `rabbitmqctl  list_permissions  [-p  VHostPath]`

  **查看指定用户的权限信息**

  `rabbitmqctl  list_user_permissions  User`

  **清除用户的权限信息**

  `rabbitmqctl  clear_permissions  [-p VHostPath]  User`


## 基本概念

- 虚拟主机：一个虚拟主机持有一组交换机、队列和绑定。为什么需要多个虚拟主机呢？很简单，RabbitMQ当中，用户只能在虚拟主机的粒度进行权限控制。 因此，如果需要禁止A组访问B组的交换机/队列/绑定，必须为A和B分别创建一个虚拟主机。每一个RabbitMQ服务器都有一个默认的虚拟主机“/”。

- 交换机：Exchange 用于转发消息，但是它不会做存储 ，如果没有 Queue bind 到 Exchange 的话，它会直接丢弃掉 Producer 发送过来的消息。

  **路由键** 。消息到交换机的时候，交互机会转发到对应的队列中，那么究竟转发到哪个队列，就要根据该路由键。

- 绑定：也就是交换机需要和队列相绑定，这其中如上图所示，是多对多的关系。

RabbitMQ 作用：

- 异步
- 解耦
- 缓冲
- 消息分发

**RabbitMQ 主要分为3个部分，生产者，交换机和队列，消费者** 

RabbitMQ 默认服务监听在 5672 端口上(带上 SSL 默认在 5671 上)

*持久化* , 服务重启时, 是否能恢复队列中的数据.

*调度策略* , 交换器如何把消息给到哪些队列, 是每个队列给一条, 或者把一条消息给多个队列.

*分配策略* , 队列面对消费者时, 如何把消息吐出去, 来一个消费者就把消息全给它, 还是只给一条.

*状态反馈* , 当消息从某一个队列中被提出后, 这个消息的生命周期就此结束, 还是说需要一个具体的信号以明确标识消息已被正确处理.

### 持久化

默认情况下, *消息*, *队列*, *交换器* 都不具有持久化的性质. 如果我们需要持久化功能, 那么在声明的时候就需要配置好.

```
channel.exchange_declare(exchange='first', type='fanout', durable=True)
channel.queue_declare(queue='hello', durable=True)
channel.basic_publish(exchange='first', routing_key='', body='Hello World!',  			                                                         properties=pika.BasicProperties(delivery_mode = 2,))
```

> 消息的持久化并不是一个很强的约束, 涉及数据落地的时机, 及系统层面的 *fsync*等问题, 不要认为消息完全不会丢. 如果要尽可能高地提高消息的持久化的有效性, 还需要配置其它的一些机制, 比如后面会谈到的 *状态反馈* 中的 *confirm mode*

> *交换器* , *队列*, *消息* 的持久化。前两者是一经声明, 则其性质无法再被更改, 即你不能先声明一个非持久化的队列, 再声明一个持久化的同名队列, 企图修改它, 这是不行的. 你重复声明时, 相关参数需要一致. 当然, 你可以删除它们再重新声明
>
> ```
> channel.queue_delete(queue='hello')
> channel.exchange_delete(exchange='first')
> ```

### 调度策略

- *fanout*

  把消息转发给所有绑定的队列上, 就是一个"广播"行为.

- *direct*

  行为是"先匹配, 再投送". 即在绑定时设定一个 *routing_key* , 消息的 *routing_key* 匹配时, 才会被交换器投送到绑定的队列中去.

- *topic*

  *topic* 和 *direct* 类似, 只是匹配上支持了"模式", 在"点分"的 *routing_key* 形式中, 可以使用两个通配符：

  - `*` 表示一个词.
  - `#` 表示零个或多个词.


- *headers*

  根据规则匹配, 相较于 *direct* 和 *topic* 固定地使用 *routing_key* , *headers*则是一个自定义匹配规则的类型.

  在队列与交换器绑定时, 会设定一组键值对规则, 消息中也包括一组键值对( `headers` 属性), 当这些键值对有一对, 或全部匹配时, 消息被投送到对应队列.

### 分配策略

调度策略是影响 *Exchange* 是不是要把消息给 *Queue* , 而分配策略影响队列如何把消息给 *Consuming* .

### 状态反馈

1. 消息发布的确认
2. 消息提取的确认

### 消息的BasicProperties

在 *AMQP* 协议中, 为消息预定了 14 个属性：

- *content_type* 标明消息的类型.
- *content_encoding* 标明消息的编码.
- *headers* 可扩展的信息对.
- *delivery_mode* 为 `2` 时表示该消息需要被持久化支持.
- *priority* 该消息的权重.
- *correlation_id* 用于"请求"与"响应"之间的匹配.
- *reply_to* "响应"的目标队列.
- *expiration* 有效期.
- *message_id* 消息的ID.
- *timestamp* 一个时间戳.
- *type* 消息的类型.
- *user_id* 用户的ID.
- *app_id* 应用的ID.
- *cluster_id* 服务集群ID.

## 应用

#### 多消费者, 并行处理

最常遇到的一种场景了. 消息产生之后堆到队列里, 有多个消费者的 *worker* 来共同处理这些消息, 以并行的方式提高处理效率.

 *fanout* 和 *direct* 都可以实现

#### 一条消息多种处理, 临时队列

*fanout* 典型的广播模式

实现上, 自然可以是当一个消费者被创建之后, 同时也创建一个自己的 *queue* , 然后绑定到指定的 *exchange* 上. 每个 *Consuming* 有自己的 *queue* , 那么于其自己做一套命名方法, 不如就忽略 *queue* 的名字, 让系统处理, 这就是 *临时队列* .

#### 发布订阅, 多种形式的实现

#### 远程调用, 信息流方向与角色转换

考虑远程调用的模型, "调用"本身是一个"请求/响应"的过程, 这是两个方向的信息流. 对应到队列中, 两个方向, 则至少需要两个队列.


















