# Zookeeper学习

参考：

[几句话了解Zookeeper工作原理](https://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247486974&idx=1&sn=4fac86401700b11970a194cbd5ccaabb&chksm=e91b68e2de6ce1f4e5c29bdcd1a026563c5dcd6b1f36cee4ec1b0c3e045ece81925e53a465a1&scene=0#rd) 

[ZooKeeper概念](https://mp.weixin.qq.com/s?__biz=MjM5ODI5Njc2MA==&mid=2655818907&idx=1&sn=990fb18f67f57672d5a5824455bc677b&chksm=bd74d94c8a03505af967fb8b2d59de9055b7b86ec7f5f114d2333f24a2642c014d0e4830cf43&mpshare=1&scene=24&srcid=0911N6oe8Dnn817zDaVUXfmj#rd)



## 概述

ZooKeeper是一种分布式协调服务，用于管理大型主机

分布式应用的优点：

- **可靠性** - 单个或几个系统的故障不会使整个系统出现故障。
- **可扩展性** - 可以在需要时增加性能，通过添加更多机器，在应用程序配置中进行微小的更改，而不会有停机时间。
- **透明性** - 隐藏系统的复杂性，并将其显示为单个实体/应用程序。

> Apache ZooKeeper是由集群（节点组）使用的一种服务，用于在自身之间协调，并通过稳健的同步技术维护共享数据。ZooKeeper本身是一个分布式应用程序，为写入分布式应用程序提供服务。

Zookeeper提供的常见服务：

- **命名服务** - 按名称标识集群中的节点。它类似于DNS，但仅对于节点。
- **配置管理** - 加入节点的最近的和最新的系统配置信息。
- **集群管理** - 实时地在集群和节点状态中加入/离开节点。
- **选举算法** - 选举一个节点作为协调目的的leader。
- **锁定和同步服务** - 在修改数据的同时锁定数据。此机制可帮助你在连接其他分布式应用程序（如Apache HBase）时进行自动故障恢复。
- **高度可靠的数据注册表** - 即使在一个或几个节点关闭时也可以获得数据。

> Zookeeper的好处：
>
> - **简单的分布式协调过程**
> - **同步** - 服务器进程之间的相互排斥和协作。此过程有助于Apache HBase进行配置管理。
> - **有序的消息**
> - **序列化** - 根据特定规则对数据进行编码。确保应用程序运行一致。这种方法可以在MapReduce中用来协调队列以执行运行的线程。
> - **可靠性**
> - **原子性** - 数据转移完全成功或完全失败，但没有事务是部分的。

## 基础知识

### ZooKeeper的架构Architecture

![架构图](https://7n.w3cschool.cn/attachments/day_161229/201612291344222238.jpg)

|     组件      |                    说明                    |
| :---------: | :--------------------------------------: |
| Client(客户端) | 客户端，我们的分布式应用集群的一个节点，从服务器访问信息。对于特定的时间间隔，每个客户端向服务器发送消息以使服务器知道客户端是活跃的。  类似的，当客户端连接时，服务器发送确认码。如果连接的服务器没有响应，客户端会自动将消息重定向到另一个服务器 |
| Server(服务器) | 服务器，我们的Zookeeper总体中的一个节点，为客户端提供所有的服务。向客户端发送确认码以告知服务器是活跃的 |
|  Ensemble   |    Zookeeper服务器组。形成ensemble所需的最小节点数为3    |
|   Leader    | 服务器节点。如果任何连接的节点失败，则执行自动恢复。Leader在服务启动时被选举 |
|  Follower   |             跟随leader指令的服务器节点             |

### 层次命名空间Hierarchical namespace

用于内存表示的ZooKeeper文件系统的树结构。ZooKeeper节点称为** znode **。每个znode由一个名称标识，并用路径(/)序列分隔。

![](https://7n.w3cschool.cn/attachments/day_161229/201612291345162031.jpg)

- 在图中，首先有一个由“/”分隔的znode。在根目录下，你有两个逻辑命名空间** config **和** workers **。
- **config **命名空间用于集中式配置管理，**workers **命名空间用于命名。
- 在config 命名空间下，每个znode最多可存储1MB的数据。这与UNIX文件系统相类似，除了父znode也可以存储数据。这种结构的主要目的是存储同步数据并描述znode的元数据。此结构称为 ZooKeeper数据模型。

ZooKeeper数据模型中的每个znode都维护着一个stat **结构。一个stat仅提供一个znode的**元数据。它由版本号，操作控制列表(ACL)，时间戳和数据长度组成。

- **版本号** - 每个znode都有版本号，这意味着每当与znode相关联的数据发生变化时，其对应的版本号也会增加。当多个zookeeper客户端尝试在同一znode上执行操作时，版本号的使用就很重要。
- **操作控制列表(ACL)** - ACL基本上是访问znode的认证机制。它管理所有znode读取和写入操作。
- **时间戳** - 时间戳表示创建和修改znode所经过的时间。它通常以毫秒为单位。ZooKeeper从“事务ID"(zxid)标识znode的每个更改。**Zxid **是唯一的，并且为每个事务保留时间，以便你可以轻松地确定从一个请求到另一个请求所经过的时间。
- **数据长度** - 存储在znode中的数据总量是数据长度。你最多可以存储1MB的数据。

#### Znode的类型

Znode被分为持久（persistent）节点，顺序（sequential）节点和临时（ephemeral）节点。

- **持久节点 ** - 即使在创建该特定znode的客户端断开连接后，持久节点仍然存在。默认情况下，除非另有说明，否则所有znode都是持久的。
- **临时节点 **- 客户端活跃时，临时节点就是有效的。当客户端与ZooKeeper集合断开连接时，临时节点会自动删除。因此，只有临时节点不允许有子节点。如果临时节点被删除，则下一个合适的节点将填充其位置。临时节点在leader选举中起着重要作用。
- **顺序节点 **- 顺序节点可以是持久的或临时的。当一个新的znode被创建为一个顺序节点时，ZooKeeper通过将10位的序列号附加到原始名称来设置znode的路径。例如，如果将具有路径 /myapp 的znode创建为顺序节点，则ZooKeeper会将路径更改为 /myapp0000000001 ，并将下一个序列号设置为0000000002。如果两个顺序节点是同时创建的，那么ZooKeeper不会对每个znode使用相同的数字。顺序节点在锁定和同步中起重要作用。

### Sessions（会话）

会话对于ZooKeeper的操作非常重要。会话中的请求按FIFO顺序执行。一旦客户端连接到服务器，将建立会话并向客户端分配**会话ID **。

客户端以特定的时间间隔发送**心跳**以保持会话有效。如果ZooKeeper集合在超过服务器开启时指定的期间（会话超时）都没有从客户端接收到心跳，则它会判定客户端死机。

会话超时通常以毫秒为单位。当会话由于任何原因结束时，在该会话期间创建的临时节点也会被删除。

### Watches（监视）

监视是一种简单的机制，使客户端收到关于ZooKeeper集合中的更改的通知。客户端可以在读取特定znode时设置Watches。Watches会向注册的客户端发送任何znode（客户端注册表）更改的通知。

Znode更改是与znode相关的数据的修改或znode的子项中的更改。只触发一次watches。如果客户端想要再次通知，则必须通过另一个读取操作来完成。当连接会话过期时，客户端将与服务器断开连接，相关的watches也将被删除。

## Zookeeper工作流

一旦ZooKeeper集合启动，它将等待客户端连接。客户端将连接到ZooKeeper集合中的一个节点。它可以是leader或follower节点。一旦客户端被连接，节点将向特定客户端分配会话ID并向该客户端发送确认。如果客户端没有收到确认，它将尝试连接ZooKeeper集合中的另一个节点。 一旦连接到节点，客户端将以有规律的间隔向节点发送心跳，以确保连接不会丢失。

- **如果客户端想要读取特定的znode，**它将会向具有znode路径的节点发送**读取请求**，并且节点通过从其自己的数据库获取来返回所请求的znode。为此，在ZooKeeper集合中读取速度很快。
- **如果客户端想要将数据存储在ZooKeeper集合中**，则会将znode路径和数据发送到服务器。连接的服务器将该请求转发给leader，然后leader将向所有的follower重新发出写入请求。如果只有大部分节点成功响应，而写入请求成功，则成功返回代码将被发送到客户端。 否则，写入请求失败。绝大多数节点被称为Quorum。

> 在ZooKeeper集合中拥有不同数量的节点的效果。
>
> - 如果我们有**单个节点**，则当该节点故障时，ZooKeeper集合将故障。它有助于“单点故障"，不建议在生产环境中使用。
> - 如果我们有**两个节点**而一个节点故障，我们没有占多数，因为两个中的一个不是多数。
> - 如果我们有**三个节点**而一个节点故障，那么我们有大多数，因此，这是最低要求。ZooKeeper集合在实际生产环境中必须至少有三个节点。
> - 如果我们有**四个节点**而两个节点故障，它将再次故障。类似于有三个节点，额外节点不用于任何目的，因此，最好添加奇数的节点，例如3，5，7。
>
> 我们知道写入过程比ZooKeeper集合中的读取过程要贵，因为所有节点都需要在数据库中写入相同的数据。因此，对于平衡的环境拥有较少数量（例如3，5，7）的节点比拥有大量的节点要好。

![](https://7n.w3cschool.cn/attachments/image/20161229/1482990578752713.png)

| 组件                         | 描述                                       |
| :------------------------- | :--------------------------------------- |
| 写入（write）                  | 写入过程由leader节点处理。leader将写入请求转发到所有的znode，并等待咋弄的的回复，如果一半的咋弄的回复，则写入过程完成。 |
| 读取（read）                   | 读取由特定连接的咋弄的在内部执行，因此不需要与集群进行交互            |
| 复制数据库（replocated database） | 它用于在zookeeper中存储数据。每个znode都有自己的数据库，每个znode在一致性的帮助下每次都有相同的数据。 |
| Leader                     | Leader是负责处理写入请求的Znode                    |
| Follower                   | follower从客户端接受写入请求，并将它们转发到leader znode   |
| 请求处理器（request processor）   | 只存在于leader节点，它管理来自follower节点的写入请求        |
| 原子广播（atomic broadcasts）    | 负责广播从leader节点到follower节点的变化              |

## Zookeeper leader选举

考虑一个集群中有N个节点。leader选举的过程如下：

- 所有节点创建具有相同路径 /app/leader_election/guid_ 的顺序、临时节点。
- ZooKeeper集合将附加10位序列号到路径，创建的znode将是 /app/leader_election/guid_0000000001，/app/leader_election/guid_0000000002等。
- 对于给定的实例，在znode中创建最小数字的节点成为leader，而所有其他节点是follower。
- 每个follower节点监视下一个具有最小数字的znode。例如，创建znode/app/leader_election/guid_0000000008的节点将监视znode/app/leader_election/guid_0000000007，创建znode/app/leader_election/guid_0000000007的节点将监视znode/app/leader_election/guid_0000000006。
- 如果leader关闭，则其相应的znode/app/leader_electionN会被删除。
- 下一个在线follower节点将通过监视器获得关于leader移除的通知。
- 下一个在线follower节点将检查是否存在其他具有最小数字的znode。如果没有，那么它将承担leader的角色。否则，它找到的创建具有最小数字的znode的节点将作为leader。
- 类似地，所有其他follower节点选举创建具有最小数字的znode的节点作为leader。

---

​:question: 简述leader的选举过程

首先服务器将自身状态转换为looking，并向集群中所有的服务器发起投票（myid,zxid）
接收其他服务器发送的投票，并进行处理，按照zxid最大的作为leader，如果相同按照myid最大的作为leader，更新投票信息，再次向集群发送投票。
统计投票结果，超过半数以上的投票即为leader                                                                                                                                                                  如果leader是自己，就把自己的状态变更为leading，否则变为follower

​:question: zk 的选举机制

(1)旧Leader宕机后，选举新Leader中，旧的Leader重启后不可能再次成为这次选举的新Leader。

(2)旧Leader宕机后，在剩下的Follower服务器选取新Leader的标准，一定是事务ID最大的那个Follower成为新Leader。（即数据同步最新的那台Follower服务器）

(3)事务ID（ZXID）是64位的数字。其中低32位可以靠做是一个简单的单调递增的计数器，高32位则代表一个Leader从生到死的epoch编号。

(4)新Leader选举出来，从事务proposal中分析出旧Leader的epoch编号，并递增1,作为新的事务ID的高32位，然后新事务ID的低32位从0位重新开始计数。

(5)新Leader通过事务ID和所有的Follower机器上的事务ID进行对比，确保数据同步。保证数据在所有的Follower上与之达成同步。旧Leader上新被提出的事务被抛弃。当数据达到同步，才将Follower服务器加入可用的Follower服务器列表。然后开始消息广播。

## Zookeeper的算法

zk的核心算法为ZAB(原子消息广播协议)，与Paxos不同，这是一种特别为zk设计的崩溃可恢复的原子消息广播算法

> **ZAB核心** ：所有事务请求必须由一个全局唯一的服务器来协调处理，这样的服务器称为Leader服务器，而余下的其他服务器则成为Follower服务器。Leader服务器负责将一个客户端事务请求转换成一个事务Proposal(提议)，并将该Proposal分发给集群中所有的Follower服务器。之后Leader服务器需要等待所有Follower服务器的反馈，一旦超过半数的Follower服务器进行了正确反馈后，那么Leader就会再次向所有的Follower服务器分发Commit消息，要求其将前一个Proposal进行提交

zab协议包括两个基本模式：消息广播以及崩溃恢复

### 消息广播

ZAB协议的消息广播过程使用的是一个原子广播协议，类似于一个二阶段提交的过程。针对客户端的事务请求，Leader服务器会为其生成对应的事务Proposal，并将其发送给集群中其余所有的机器，然后再分别手机各自的选票，最后进行事务提交。

### 奔溃恢复

一旦Leader服务器出现崩溃，或者由于网络原因导致Leader服务器失去了过半Follower的联系，那么就会进入崩溃恢复模式，并且恢复后需要一个新的leader，因此zab协议需要一个高效的选举方案

## zookeeper安装

### windows下安装

[安装教程](https://blog.csdn.net/yzy199391/article/details/80605195) 

