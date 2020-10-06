# 消息队列 

#### 消息队列

消息队列，是**分布式系统**中不可缺少的**中间件**。可实现**高性能**，**高可用**，**可伸缩和最终一致性架构**

目前主流的消息队列有

- Kafka：大数据、实时计算，kafka是业内标准。
- RabbitMQ：开源，Erlang实现，Spring Cloud 支持上不错，社区活跃
- RocketMQ ：Java实现，阿里出品。
- ActiveMQ ：不清楚，没经过大规模吞吐量场景的验证，社区不活跃

![image-20200928164858644](https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200928164900.png)

#### 角色

- 生产者（Producer）

- 消费者（Consumer

- 消息代理（Message Broker）：负责**存储消息**和**转发消息**两件事情。

  其中，**转发消息**分为推送和拉取两种方式。

  - **拉取**（Pull）
  - **推送**（Push）

#### 应用场景

- 应用**解耦**
  - 解耦：从主动调用的方式，变成了消息的订阅发布
- **异步**处理
  - 串行调用变为异步，提升速度。
  - 前提，返回的结果不依赖于处理的结果。
- 流量**削峰**
  - 转发到消息队列中，慢慢处理。
  - 生产中，如果对时效没有要求，可以允许短暂的高峰期积压
- 消息**通讯**
  - 消息通讯
- 日志处理（异步处理）
  - 解决大量**日志传输**的问题（ELK+kafka）

#### 缺点

- **复杂度提高**
  - 1）消息怎么不**重复**消息。
  - 2）消息怎么保证不**丢失**。
  - 3）需要消息**顺序**的业务场景，怎么处理。
- **可用性降低**：万一消息队列挂了。
- **一致性问题**。**一定要达到数据的最终一致性**

#### 消费语义

1. 消息**至多被消费一次**（At most once）：消息可能会丢失，但**绝不重传**。

   - 吞吐量大，容忍消息丢失。
   - Message Broker**不**对接收到的消息**响应**确认
   - Message Broker转发**不**要求**持久**性，也不关心消费者是否真的收到
   - Message Broker消息被拿走就删除，**不关心消费**者，消费情况

2. 消息**至少被消费一次**（At least once）：消息可以**重传**，但绝不丢失。

   - Message Broker 必须**响应**对消息的确认，并提供**持久**性保障
   - **接收到消费者通知**后，才删除消息。

3. 消息**仅被消费一次**（Exactly once）：每一条消息只被传递一次。

   - Message Broker 上存储的消息被 Consumer 仅消费一次。

     - Message Broker**不**对接收到的消息**响应**确认

     - Message Broker **提供持久性**保障
     - 消息有唯一标识，**消费者记录唯一标准**，防止重复消费

   - Producer 上产生的消息被 Consumer 仅消费一次。

     - Message Broker对接收到的**消息响应确认**。 **Producer 负责为该消息产生唯一标识**，以防止 Consumer 重复消费
     - Message Broker 提供**持久性**保障，消息队列里有唯一标识
     - **消费者记录唯一标识**，防止重复消费

#### 消息队列有几种投递方式？分别有什么优缺点

**push**

- 优点：**即时**
- 缺点：就是受限于**消费者**的消费能力，可能造成消息的**堆积**

**pull**

- 优点：消费者自己掌控进度，消费者不会堆积，但消息队列可能会堆积。
- 缺点：**延迟、忙等**

**目前的消息队列，基于 push + pull 模式结合的方式**，Broker 仅仅告诉 Consumer 有新的消息，具体的消息拉取，还是 Consumer 自己主动拉取。

##### 消费者实现**幂等性**？消息不被重复消费？

**消息仅被消费一次**，且**每条消息从 Producer 保证被送达，并且被 Consumer 仅消费一次**。

重复消费：offset，消费者挂了

方式：

1. **业务层自己确认是否是否消费过**
2. **事物**，Zookeeper 和 Redis 在获取分布式锁

##### 如何保证生产者的发送消息的**可靠性**？

###### **RabbitMQ** 

![image-20200928174648240](https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200928174649.png)

1. 生产者弄丢了数据

   1. 提供事物（同步），会降低吞吐

      ```java
      // 开启事务
      channel.txSelect
      try {
          // 这里发送消息
      } catch (Exception e) {
          channel.txRollback
      
          // 这里再次重发这条消息
      }
      
      // 提交事务
      channel.txCommit
      ```

   2. 开启生产者 `confirm` 模式（消息确认机制）：

      - 开启 `confirm` 模式之后，你每次写的消息都会分配一个唯一的 id，然后如果写入了 RabbitMQ 中，RabbitMQ 会给你回传一个 `ack` 消息，告诉你说这个消息 ok 了。
      - 如果 `RabbitMQ` 没能处理这个消息，会**回调你的**一个 `nack` 接口，告诉你这个消息接收失败，你可以重试。
      - 而且你可以结合这个机制**自己在内存里维护每个消息 id 的状态**，如果超过一定时间还没接收到这个消息的回调，那么你可以重发。

2. RabbitMQ 弄丢了数据

   1. **开启 RabbitMQ 的持久化**
      - 创建 queue 的时候将其设置为持久化
      - 发送消息的时候将消息的 `deliveryMode` 设置为 `2`（持久化了才返回）
      - 持久化可以跟生产者那边的 `confirm` 机制配合

3. 消费端弄丢了数据

   - RabbitMQ 提供的 `ack` 机制，简单来说，就是**你必须关闭 RabbitMQ 的自动 ack** ，可以通过一个 api 来调用就行，然后每次你自己代码里确保处理完的时候，再在程序里 `ack` 一把。
   - [为防止任务执行期间出错](https://www.cnblogs.com/julygift/p/9445107.html)，设置NO_ACK=FALSE标志，这样、一旦任务没有应答的话，相应的任务就会被RabbitMQ自动Re-Queue,避免丢失任务
     - 问题-超时一直Re-Queue，死循环。
   - 绑定死信队列

###### **Kafka**

1. 消费端弄丢了数据
   - **关闭自动提交** offset，在处理完之后自己**手动提交 offset**，就可以保证数据不会丢，**可能会有重复消费**，自己保证幂等性
2. Kafka 弄丢了数据
   - broker 宕机，然后重新选举 partition 的 leader，没来得及同步，丢失数据，
   - 为了防止leader切换不丢数据
     - 给 topic 设置 `replication.factor` 参数：这个值必须大于 1，要求每个**partition 必须有至少 2 个副本**。
     - 在 Kafka 服务端设置 `min.insync.replicas` 参数：这个值必须大于 1，这个是**要求一个 leader 至少感知到有至少一个 follower 还跟自己保持联系，没掉队**，这样才能确保 leader 挂了还有一个 follower 吧。
     - 在 **producer 端**设置 `acks=all` ：这个是要求每条数据，必须是**写入所有 replica 之后，才能认为是写成功了**。
     - 在 **producer 端**设置 `retries=MAX` （很大很大很大的一个值，无限次重试的意思）：这个是**要求一旦写入失败，就无限重试**，卡在这里了。
3. 生产者会不会弄丢数据？
   - 设置了 `acks=all`、`retries=Ingeter.Max` ,生产者会自动不断的重试，重试无限次。

##### 如何保证消息的顺序性？

场景： mysql `binlog` 日志，到MQ，再出来，顺序得一致。多个消费者，多个线程消费。

解决顺序型，得同步.

###### 解决方案

**RabbitMQ**

1. 拆分多个 queue，每个 queue 一个 consumer

2. 一个 queue 但是对应一个 consumer，然后这个 consumer 内部用**内存队列做排队**，然后分发给底层不同的 worker 来处理。

   ![rabbitmq-order-02](https://github.com/doocs/advanced-java/raw/master/docs/high-concurrency/images/rabbitmq-order-02.png)

**Kafka**

1. 一个 topic，每个个 partition，对应一个 consumer，内部单线程消费

2. 再写 **N 个内存 queue**，具有相同 key 的数据都到同一个内存 queue；然后对于 N 个线程，**每个线程分别消费一个内存 queue** 即可，这样就能保证顺序性。

   ![kafka-order-02](https://github.com/doocs/advanced-java/raw/master/docs/high-concurrency/images/kafka-order-02.png)

##### 如何解决消息积压的问题？

1. 先修复 consumer ，将现有 consumer 都停掉。
2. **新建一个 topic**，`partition` 是原来的 `10` 倍，临时建立好原先 10 倍的 queue 数量。
3. 然后写一个临时的**分发数据的 consumer 程序**，这个程序部署上去消费积压的数据，**消费之后不做耗时的处理**，直接均匀轮询写入临时建立好的 10 倍数量的 queue。
4. 接着临时征用 10 倍的机器来`部署 consumer`，**每一批 consumer 消费一个临时 queue 的数据**。这种做法相当于是临时将 queue 资源和 consumer 资源扩大 10 倍，以正常的 10 倍速度来消费数据。
5. 等快速消费完积压数据之后，**得恢复原先部署的架构**，**重新**用原先的 consumer 机器来消费消息。

##### 如何解决消息过期的问题？

大量积压，丢弃，**趁没人的时候，丢弃批量重导**

##### 消息队列如何实现高可用？

###### RabbitMQ ：

1. 单机：DEMO
2. 普通集群：创建的**创建的 queue，只会放在一个 RabbitMQ 实例上**，每个实例放队列的元数据。集群间可能产生大量的数据传输，没备份，可用性无法保障。就是为了提高吞吐量。
3. 镜像集群模式（高可用性）：基于主从做高可用。你创建的 queue，无论元数据还是 queue 里的消息都会**存在于多个实例上**，也就是镜像
   - 开销大，**不是分布式**，所有数据放到一个节点里，**不能线性扩展**。

###### Kafka 的高可用性

由多个 `broker` 组成，每个 broker 是一个节点；你创建一个 topic，这个 topic 可以划分为多个 `partition`，每个 partition 可以存在于不同的 broker 上，每个 partition 就放一部分数据。

这就是**天然的分布式消息队列**，就是说一个 topic 的数据，是**分散放在多个机器上的，每个机器就放一部分数据**。

- Kafka 0.8 以后，提供了 HA 机制，就是 replica（复制品） 副本机制
- 所有 replica 会选举一个 leader 出来，那么生产和消费都跟这个 **leader 打交道**，然后其他 replica 就是 follower。
- **写数据**的时候，生产者就写 leader，然后 leader 将数据落地写本地磁盘，接着其他 follower 自己**主动从 leader 来 pull 数据**
- **消费**的时候，只会从 leader 去读，但是只有当一个消息已经被所有 follower 都同步成功返回 ack 的时候，这个消息才会被消费者读到。
-  broker 宕机：会从 follower 中**重新选举**一个新的 leader 出来

#### [设计一个消息队列](https://www.cnblogs.com/expiator/p/10021298.html)

- 可伸缩，broker -> topic -> partition
- 落地磁盘：顺序写
- 可用性：多副本 -> leader & follower -> broker 挂了重新选举 leader 即可对外服务
- 0丢失

# RabbitMQ 

## RabbitMQ 是什么？

RabbitMQ 是一个由 Erlang 语言开发的 AMQP 的开源实现。

> AMQP ：Advanced Message Queue，高级消息队列协议。它是应用层协议的一个开放标准，为面向消息的中间件设计，基于此协议的客户端与消息中间件可传递消息，并不受产品、开发语言等条件的限制。

RabbitMQ具体特点包括：

- 1、可靠性（Reliability）

- 2、灵活的路由（Flexible Routing）

  > 在消息进入队列之前，通过 Exchange 来路由消息的。对于典型的路由功能，RabbitMQ 已经提供了一些内置的 Exchange 来实现。针对更复杂的路由功能，可以将多个 Exchange 绑定在一起，也通过插件机制实现自己的 Exchange 。

- 3、消息集群（Clustering）

- 4、高可用（Highly Available Queues）

- 6、多语言客户端（Many Clients）

  > RabbitMQ 几乎支持所有常用语言，比如 Java、.NET、Ruby 等等。

- 7、管理界面（Management UI）

  > RabbitMQ 提供了一个易用的用户界面，使得用户可以监控和管理消息 Broker 的许多方面。

- 8、**跟踪机制**（Tracing）

  > 如果消息异常，RabbitMQ 提供了消息跟踪机制，使用者可以找出发生了什么。

- 9、插件机制（Plugin System）

## RabbitMQ 中的 Broker 是指什么？Cluster 又是指什么？

- Broker（代理） ，是指**一个或多个 erlang node 的逻辑分组**，且 node 上运行着 RabbitMQ 应用程序。
- Cluster（集群） ，是在 Broker 的基础之上，增加了 node 之间共享元数据的约束。

🦅 **vhost 是什么？起什么作用？**

`vhost` 可以理解为`虚拟 Broker` ，即 mini-RabbitMQ server 。其内部均含有独立的 queue、exchange 和 binding 等，但最最重要的是，其拥有独立的权限系统，可以做到 vhost 范围的用户控制。当然，从 RabbitMQ 的全局角度，vhost 可以作为不同权限隔离的手段（一个典型的例子就是不同的应用可以跑在不同的 vhost 中）。

> 这个，和 Tomcat、Nginx、Apache 的 vhost 是一样的概念。

## 什么是元数据？元数据分为哪些类型？包括哪些内容？

在非 Cluster 模式下，元数据主要分为：

- **Queue 元数据**（queue 名字和属性等）
- **Exchange 元数据**（exchange 名字、类型和属性等）
- **Binding 元数据**（存放路由关系的查找表）
- **Vhost 元数据**（vhost 范围内针对前三者的名字空间约束和安全属性设置）。

🦅 **与 Cluster 相关的元数据有哪些？元数据是如何保存的？元数据在 Cluster 中是如何分布的？**

在 Cluster 模式下，还包括 Cluster 中 **node 位置信息和 node 关系信息**。

元数据根据 erlang node 的类型确定是仅保存于 RAM 中，还是同时保存在 RAM 和 disk 上。元数据在 Cluster 中是全 node 分布的。

下图所示为 queue 的元数据在单 node 和 cluster 两种模式下的分布图：

[![分布图](http://static.iocoder.cn/c7566feb7a61cb647296a7224ba7f39d)](http://static.iocoder.cn/c7566feb7a61cb647296a7224ba7f39d)分布图

🦅 **RAM node 和 Disk node 的区别？**

- RAM node 仅将 fabric（即 queue、exchange 和 binding等 RabbitMQ基础构件）相关元数据保存到内存中，但 Disk node 会在内存和磁盘中均进行存储。
- RAM node 上**唯一会存储到磁盘上的元数据是 Cluster 中使用的 Disk node 的地址**。并且要求在 RabbitMQ Cluster 中至少存在一个 Disk node 。

## RabbitMQ 概念里的 channel、exchange 和 queue 是什么？

- `queue` 具有自己的 erlang 进程；
- `exchange` 内部实现为保存 `binding` 关系的查找表；
- `channel` 是实际进行**路由工作的实体**，即负责按照 routing_key 将 message 投递给 queue 。

由 AMQP 协议描述可知，channel 是真实 TCP 连接之上的虚拟连接，所有 AMQP 命令都是通过 channel 发送的，且每一个 channel 有唯一的 ID 。

- 一个 channel 只能被单独一个操作系统线程使用，故投递到特定 channel 上的 message 是有顺序的。但一个操作系统线程上允许使用多个 channel 。

- channel 号为 0 的 channel 用于处理所有对于当前 connection 全局有效的帧，而 1-65535 号 channel 用于处理和特定 channel 相关的帧。

- AMQP 协议给出的 channel 复用模型如下：

  ![channel 复用模型](http://static.iocoder.cn/0cb69bbcda6043bc0c9d7733de251d76)

  channel 复用模型

  - 其中每一个 channel 运行在一个独立的线程上，多线程共享同一个 socket 。

🦅 **消息基于什么传输？**

由于 TCP 连接的**创建和销毁开销较大**，且并发数受系统资源限制，会造成性能瓶颈。RabbitMQ 使用信道(`channel` )的方式来传输数据。信道是**建立在真实的 TCP 连接内的虚拟连接**，且每条 TCP连接上的信道数量没有限制。

🦅 **RabbitMQ 上的一个 queue 中存放的 message 是否有数量限制？**

可以认为是无限制，因为限制取决于机器的内存，但是消息过多会导致处理效率的下降。

> 下面的几个问题，Cluster 相关。

🦅 **在单 node 系统和多 node 构成的 cluster 系统中声明 queue、exchange ，以及进行 binding 会有什么不同？**

- 当你在单 node 上声明 queue 时，只要该 node 上相关元数据进行了变更，你就会得到 `Queue.Declare-ok` 回应；而在 cluster 上声明 queue ，则要求 cluster 上的**全部 node 都要进行元数据成功更新**，才会得到 `Queue.Declare-ok` 回应。
- 另外，若 **node 类型为 RAM node** 则变更的数据仅保存在内存中，若类型为 Disk node 则还要变更保存在磁盘上的数据。

🦅 **客户端连接到 Cluster 中的任意 node 上是否都能正常工作？**

是的。客户端感觉不到有何不同。

🦅 **若 Cluster 中拥有某个 queue 的 owner node 失效了，且该 queue 被声明具有 durable 属性(持久化属性)，是否能够成功从其他 node 上重新声明该 queue ？**

- 不能，在这种情况下，将得到 404 NOT_FOUND 错误。只能等 queue 所属的 node 恢复后才能使用该 queue 。
- 但若该 queue 本身不具有 durable 属性，则可在其他 node 上重新声明。

🦅 **Cluster 中 node 的失效会对 consumer 产生什么影响？若是在 cluster 中创建了 mirrored queue（镜像队列） ，这时 node 失效会对 consumer 产生什么影响？**

- 若是 consumer 所连接的那个 node 失效（无论该 node 是否为 consumer 所订阅 queue 的 owner node），则 **consumer 会在发现 TCP 连接断开时，按标准行为执行重连逻辑，并根据 “Assume Nothing（不做任何假设）” 原则重建相应的 fabric 即可**。
- 若是失效的 node 为 consumer 订阅 queue 的 owner node，**则 consumer 只能通过 Consumer Cancellation Notification（消费者取消通知） 机制来检测与该 queue 订阅关系的终止，否则会出现傻等却没有任何消息来到的问题。**

🦅 **Consumer Cancellation Notification 机制用于什么场景？**

用于保证当镜像 queue 中 `master 挂掉`时，连接到 slave 上的 consumer 可以**收到自身 consume 被取消的通知**，进而可以重新执行 consume 动作从新选出的 master 出获得消息。

若不采用该机制，连接到 slave 上的 consumer 将不会感知 master 挂掉这个事情，导致后续无法再收到新 master 广播出来的 message 。

另外，因为在**镜像 queue 模式**下，存在将 message 进行 requeue 的可能，所以实现 consumer 的逻辑时需要能够正确处理出现重复 message 的情况。

🦅 **能够在地理上分开的不同数据中心使用 RabbitMQ cluster 么？**

不能。

- 第一，你**无法控制所创建的 queue 实际分布在 cluster 里的哪个 node 上**（一般使用 HAProxy + cluster 模型时都是这样），这可能会导致各种跨地域访问时的常见问题。
- 第二，Erlang 的 OTP 通信框架对**延迟的容忍度有限**，这可能会**触发各种超时**，导致业务疲于处理。
- 第三，在广域网上的**连接失效问题将导致经典的“脑裂”问题**，而 RabbitMQ 目前无法处理。（该问题主要是说 Mnesia）

## 如何确保消息正确地发送至 RabbitMQ？

RabbitMQ 使用**发送方确认模式**，确保消息正确地发送到 RabbitMQ。

- 发送方确认模式：将**信道设置成 confirm（确认） 模式（发送方确认模式）**，则所有在信道上发布的消息都会被指派一个唯一的 ID 。一旦消息被投递到目的队列后，或者消息被写入磁盘后（可持久化的消息），信道会发送一个确认给生产者（包含消息唯一ID）。如果 RabbitMQ 发生内部错误从而导致消息丢失，会发送一条 nack（not acknowledged，未确认）消息。
- 发送方确认模式是**异步的**，生产者应用程序在等待确认的同时，可以继续发送消息。当确认消息到达生产者应用程序，生产者应用程序的回调方法就会被触发来处理确认消息。

具体的代码实现，可以看看 [《芋道 Spring Boot 消息队列 RabbitMQ 入门》](http://www.iocoder.cn/Spring-Boot/RabbitMQ/?vip)的[「14. 生产者的发送确认」](http://svip.iocoder.cn/RabbitMQ/Interview/#) 小节

🦅 **向不存在的 exchange 发 publish 消息会发生什么？向不存在的 queue 执行 consume 动作会发生什么？**

都会收到 `Channel.Close` 信令告之不存在（内含原因 404 NOT_FOUND）。

🦅 **什么情况下会出现 blackholed 问题？**

> blackholed ，对应中文为“黑洞”。

blackholed 问题是指，**向 exchange 投递了 message ，而由于各种原因导致该 message 丢失，但发送者却不知道**。可导致 blackholed 的情况：

- 1、向未绑定 queue 的 exchange 发送 message 。
- 2、exchange 以 binding_key key_A 绑定了 queue queue_A，但向该 exchange 发送 message 使用的 routing_key 却是 key_B 。

🦅 **如何防止出现 blackholed 问题？**

没有特别好的办法，只能在具体实践中通过各种方式保证相关 fabric 的存在。另外，如果在执行 `Basic.Publish` 时设置 `mandatory=true` ，则在遇到可能出现 blackholed 情况时，服务器会通过返回 `Basic.Return` 告之当前 message 无法被正确投递（内含原因 312 NO_ROUTE）。

具体的代码实现，可以看看 [《芋道 Spring Boot 消息队列 RabbitMQ 入门》](http://www.iocoder.cn/Spring-Boot/RabbitMQ/?vip)的[「14.3 ReturnCallback」](http://svip.iocoder.cn/RabbitMQ/Interview/#) 小节

🦅 **routing_key 和 binding_key 的最大长度是多少？**

**255 字节。**

🦅 **消息怎么路由？**

从概念上来说，消息路由必须有三部分：交换器、路由、绑定。

- 生产者把消息发布到**交换器**上；

- **绑定**决定了消息如何从路由器路由到特定的队列；

  > 如果一个路由绑定了两个队列，那么发送给该路由时，这两个队列都会增加一条消息。

- 消息最终到达**队列**，并被消费者接收。

详细来说，就是：

- 消息发布到交换器时，消息将拥有一个路由键（routing key），在消息创建时设定。
- 通过队列路由键，可以把队列绑定到交换器上。
- 消息到达交换器后，RabbitMQ 会将消息的路由键与队列的路由键进行匹配（针对不同的交换器有不同的路由规则）。如果能够匹配到队列，则消息会投递到相应队列中；如果不能匹配到任何队列，消息将进入 “黑洞”。

常用的**交换器**主要分为一下三种：

- direct：如果路由键**完全匹配**，消息就被投递到相应的队列。
- fanout：如果交换器收到消息，将会**广播**到所有绑定的队列上。
- topic：可以使来自不同源头的消息能够到达同一个队列。 使用 topic 交换器时，可以使用通配符，比如：`“*”` 匹配特定位置的任意文本， `“.”` 把路由键分为了几部分，`“#”` 匹配所有规则等。特别注意：发往 topic 交换器的消息不能随意的设置选择键（routing_key），必须是由 `"."` 隔开的一系列的标识符组成。

具体的代码实现，可以看看 [《芋道 Spring Boot 消息队列 RabbitMQ 入门》](http://www.iocoder.cn/Spring-Boot/RabbitMQ/?vip)的[「3. 快速入门」](http://svip.iocoder.cn/RabbitMQ/Interview/#) 小节

## 如何确保消息接收方消费了消息？

RabbitMQ 使用**接收方消息确认机制**，确保消息接收方消费了消息。

- 接收方消息确认机制：消费者接收每一条消息后都必须进行确认（消息接收和消息确认是两个不同操作）。只有消费者确认了消息，RabbitMQ 才能安全地把消息从队列中删除。

这里并没有用到超时机制，RabbitMQ 仅通过 Consumer 的连接中断来确认是否需要重新发送消息。也就是说，只要连接不中断，RabbitMQ 给了 Consumer 足够长的时间来处理消息。

下面罗列几种特殊情况：

- 如果消费者接收到消息，在确认之前断开了连接或取消订阅，RabbitMQ 会认为消息没有被分发，然后重新分发给下一个订阅的消费者。（可能存在消息重复消费的隐患，需要根据 bizId 去重）
- 如果消费者接收到消息却没有确认消息，连接也未断开，则 RabbitMQ 认为该消费者繁忙，将不会给该消费者分发更多的消息。

具体的代码实现，可以看看 [《芋道 Spring Boot 消息队列 RabbitMQ 入门》](http://www.iocoder.cn/Spring-Boot/RabbitMQ/?vip)的[「13. 消费者的消息确认」](http://svip.iocoder.cn/RabbitMQ/Interview/#) 小节。

🦅 **如何避免消息重复投递或重复消费？**

在**消息生产时**，**MQ 内部针对每条生产者发送的消息生成一个 inner-msg-id** ，作为去重和幂等的依据（消息投递失败并重传），**避免重复的消息进入队列**

在**消息消费时**，要求消息体中必须要**有一个 bizId**（对于同一业务全局唯一，如支付 ID、订单 ID、帖子 ID 等）作为**去重和幂等**的依据，避免同一条消息被重复消费。

🦅 **消息如何分发？**

若该队列至少有一个消费者订阅，消息将以循环（round-robin）的方式发送给消费者。每条消息只会分发给一个订阅的消费者（前提是消费者能够正常处理消息并进行确认）。

🦅 **RabbitMQ 有几种消费模式？**

RabbitMQ 有 pull 和 push 两种消费模式。具体的使用，可参见 [《RabbitMQ 之 Consumer 消费模式（Push & Pull）》](https://blog.csdn.net/u013256816/article/details/62890189) 文章。

## 为什么不应该对所有的 message 都使用持久化机制？

首先，必然导致性能的下降，因为写磁盘比写 RAM 慢的多，message 的吞吐量可能有 10 倍的差距。

其次，message 的持久化机制用在 RabbitMQ 的内置 Cluster 方案时会出现“坑爹”问题。矛盾点在于：

- 若 message 设置了 persistent 属性，但 queue 未设置 durable 属性，那么当该 queue 的 owner node 出现异常后，在未重建该 queue 前，发往该 queue 的 message 将被 blackholed 。
- 若 message 设置了 persistent 属性，同时 queue 也设置了 durable 属性，那么当 queue 的 owner node 异常且无法重启的情况下，则该 queue 无法在其他 node 上重建，只能等待其 owner node 重启后，才能恢复该 queue 的使用，而在这段时间内发送给该 queue 的 message 将被 blackholed 。

所以，是否要对 message 进行持久化，需要综合考虑性能需要，以及可能遇到的问题。若想达到 100,000 条/秒以上的消息吞吐量（单 RabbitMQ 服务器），则要么使用其他的方式来确保 message 的可靠 delivery ，要么使用非常快速的存储系统以支持全持久化（例如使用 SSD）。另外一种处理原则是：仅对关键消息作持久化处理（根据业务重要程度），且应该保证关键消息的量不会导致性能瓶颈。

🦅 **RabbitMQ 允许发送的 message 最大可达多大？**

根据 AMQP 协议规定，消息体的大小由 64-bit 的值来指定，所以你就可以知道到底能发多大的数据了。

🦅 **为什么说保证 message 被可靠持久化的条件是 queue 和 exchange 具有 durable 属性，同时 message 具有 persistent 属性才行？**

binding 关系可以表示为 exchange–binding–queue 。从文档中我们知道，若要求投递的 message 能够不丢失，要求 message 本身设置 persistent 属性，同时要求 exchange 和 queue 都设置 durable 属性。

- 其实这问题可以这么想，若 exchange 或 queue 未设置 durable 属性，则在其 crash 之后就会无法恢复，那么即使 message 设置了 persistent 属性，仍然存在 message 虽然能恢复但却无处容身的问题。
- 同理，若 message 本身未设置 persistent 属性，则 message 的持久化更无从谈起。

🦅 **如何确保消息不丢失？**

消息持久化的前提是：将交换器/队列的 durable 属性设置为 true ，表示交换器/队列是持久交换器/队列，在服务器崩溃或重启之后不需要重新创建交换器/队列（交换器/队列会自动创建）。

如果消息想要从 RabbitMQ 崩溃中恢复，那么消息必须：

- 在消息发布前，通过把它的 “投递模式” 选项设置为2（持久）来把消息标记成持久化
- 将消息发送到持久交换器
- 消息到达持久队列

RabbitMQ 确保持久性消息能从服务器重启中恢复的方式是，将它们写入磁盘上的一个持久化日志文件。

- 当发布一条持久性消息到持久交换器上时，RabbitMQ 会在消息提交到日志文件后才发送响应（如果消息路由到了非持久队列，它会自动从持久化日志中移除）。
- 一旦消费者从持久队列中消费了一条持久化消息，RabbitMQ 会在持久化日志中把这条消息标记为等待垃圾收集。如果持久化消息在被消费之前 RabbitMQ 重启，那么 RabbitMQ 会自动重建交换器和队列（以及绑定），并重播持久化日志文件中的消息到合适的队列或者交换器上。

## 什么是死信队列？

DLX，Dead-Letter-Exchange。利用 DLX ，当消息在一个队列中变成死信（dead message）之后，它能被重新 publish 到另一个 Exchange ，这个 Exchange 就是DLX。消息变成死信一向有一下几种情况：

- 消息被拒绝（basic.reject / basic.nack）并且 `requeue=false` 。
- 消息 TTL 过期（参考：RabbitMQ之TTL（Time-To-Live 过期时间））。
- 队列达到最大长度。

详细的，可以看看 [《RabbitMQ 之死信队列》](http://www.iocoder.cn/RabbitMQ/dead-letter-queue/?vip) 文章。

🦅 **“dead letter”queue 的用途？**

当消息被 RabbitMQ server 投递到 consumer 后，但 consumer 却通过 `Basic.Reject` 进行了拒绝时（同时设置 `requeue=false`），那么该消息会被放入 “dead letter” queue 中。该 queue 可用于排查 message 被 reject 或 undeliver 的原因。

🦅 **`Basic.Reject` 的用法是什么？**

该信令可用于 consumer 对收到的 message 进行 reject 。

- 若在该信令中设置 `requeue=true` ，则当 RabbitMQ server 收到该拒绝信令后，会将该 message 重新发送到下一个处于 consume 状态的 consumer 处（理论上仍可能将该消息发送给当前 consumer）。
- 若设置 `requeue=false` ，则 RabbitMQ server 在收到拒绝信令后，将直接将该 message 从 queue 中移除。

另外一种移除 queue 中 message 的小技巧是，consumer 回复 `Basic.Ack` 但不对获取到的 message 做任何处理。而 `Basic.Nack`是对 Basic.Reject 的扩展，以支持一次拒绝多条 message 的能力。

## RabbitMQ 中的 cluster、mirrored queue，以及 warrens 机制分别用于解决什么问题？

🦅 **1）cluster**

- cluster 是为了解决当 cluster 中的任意 node 失效后，producer 和 consumer 均可以通过其他 node 继续工作，即提高了可用性；另外可以通过增加 node 数量增加 cluster 的消息吞吐量的目的。
- cluster 本身不负责 message 的可靠性问题（该问题由 producer 通过各种机制自行解决）；cluster 无法解决跨数据中心的问题（即脑裂问题）。
- 另外，在cluster 前使用 HAProxy 可以解决 node 的选择问题，即业务无需知道 cluster 中多个 node 的 ip 地址。可以利用 HAProxy 进行失效 node 的探测，可以作负载均衡。下图为 HAProxy + cluster 的模型：[HAProxy + cluster 的模型](https://img-blog.csdnimg.cn/20181219191433895)

🦅 **2）Mirrored queue**

- Mirrored queue 是为了解决使用 cluster 时所创建的 queue 的完整信息仅存在于单一 node 上的问题，从另一个角度增加可用性。
- 若想正确使用该功能，需要保证：1）consumer 需要支持 Consumer Cancellation Notification 机制；2）consumer 必须能够正确处理重复 message 。

🦅 **3）Warrens**

Warrens 是为了解决 cluster 中 message 可能被 blackholed 的问题，即不能接受 producer 不停 republish message 但 RabbitMQ server 无回应的情况。

Warrens 有两种构成方式：

- 一种模型，是两台独立的 RabbitMQ server + HAProxy ，其中两个 server 的状态分别为 active 和 hot-standby 。该模型的特点为：两台 server 之间无任何数据共享和协议交互，两台 server 可以基于不同的 RabbitMQ 版本。如下图所示：[模型一](https://img-blog.csdnimg.cn/20181219191433914)
- 另一种模型，为两台共享存储的 RabbitMQ server + keepalived ，其中两个 server 的状态分别为 active 和 cold-standby 。该模型的特点为：两台 server 基于共享存储可以做到完全恢复，要求必须基于完全相同的 RabbitMQ 版本。如下图所示：[模型二](https://img-blog.csdnimg.cn/20181219191433931)

Warrens 模型存在的问题：

- 对于第一种模型，虽然理论上讲不会丢失消息，但若在该模型上使用持久化机制，就会出现这样一种情况，即若作为 active 的 server 异常后，持久化在该 server 上的消息将暂时无法被 consume ，因为此时该 queue 将无法在作为 hot-standby 的 server 上被重建，所以，只能等到异常的 active server 恢复后，才能从其上的 queue 中获取相应的 message 进行处理。而对于业务来说，需要具有：a.感知 AMQP 连接断开后重建各种 fabric 的能力；b.感知 active server 恢复的能力；c.切换回 active server 的时机控制，以及切回后，针对 message 先后顺序产生的变化进行处理的能力。
- 对于第二种模型，因为是基于共享存储的模式，所以导致 active server 异常的条件，可能同样会导致 cold-standby server 异常；另外，在该模型下，要求 active 和 cold-standby 的 server 必须具有相同的 node 名和 UID ，否则将产生访问权限问题；最后，由于该模型是冷备方案，故无法保证 cold-standby server 能在你要求的时限内成功启动。

## RabbitMQ 如何实现高可用？

> 这个问题，和 [「RabbitMQ 中的 cluster、mirrored queue，以及 warrens 机制分别用于解决什么问题？」](http://svip.iocoder.cn/RabbitMQ/Interview/#) 会比较类似。

RabbitMQ 的高可用，是基于**主从**做高可用性的。它有三种模式：

- 单机模式
- 普通集群模式
- 镜像集群模式

🦅 **1）单机模式**

单机模式，就是启动单个 RabbitMQ 节点，一般用于本地开发或者测试环境。实际生产环境下，基本不会使用。

🦅 **普通集群模式（无高可用性）**

> 这种方式，就是上面问题的 **cluster** 。

普通集群模式，意思就是在多台机器上启动多个 RabbitMQ 实例，每个机器启动一个。

- 你**创建的 queue，只会放在一个 RabbitMQ 实例上**，但是每个实例都同步 queue 的元数据（元数据可以认为是 queue 的一些配置信息，通过元数据，可以找到 queue 所在实例）。
- 你消费的时候，实际上如果连接到了另外一个实例，那么那个实例会从 queue 所在实例上拉取数据过来。

[![架构图](http://static2.iocoder.cn/75d36ed17c91932e28b5eeba681ad8ec)](http://static2.iocoder.cn/75d36ed17c91932e28b5eeba681ad8ec)架构图

这种方式确实很麻烦，也不怎么好，**没做到所谓的分布式**，就是个普通集群。因为这导致你要么消费者每次随机连接一个实例然后拉取数据，要么固定连接那个 queue 所在实例消费数据，前者有**数据拉取的开销**，后者导致**单实例性能瓶颈**。

而且如果那个放 queue 的实例宕机了，会导致接下来其他实例就无法从那个实例拉取，如果你**开启了消息持久化**，让 RabbitMQ 落地存储消息的话，**消息不一定会丢**，得等这个实例恢复了，然后才可以继续从这个 queue 拉取数据。

所以这个事儿就比较尴尬了，这就**没有什么所谓的高可用性**，**这方案主要是提高吞吐量的**，就是说让集群中多个节点来服务某个 queue 的读写操作。

🦅 **镜像集群模式（高可用性）**

> 艿艿：请教了下胖友，他们采用这种方式。
>
> 这种方式，就是上面问题的 **Mirrored queue** 。

这种模式，才是所谓的 RabbitMQ 的高可用模式。跟普通集群模式不一样的是，在镜像集群模式下，你创建的 queue，无论元数据还是 queue 里的消息都会**存在于多个实例上**，就是说，每个 RabbitMQ 节点都有这个 queue 的一个**完整镜像**，包含 queue 的全部数据的意思。然后每次你写消息到 queue 的时候，都会自动把**消息同步**到多个实例的 queue 上。

[![架构图](http://static2.iocoder.cn/20a6c4d82a08becf7e8d913662b00357)](http://static2.iocoder.cn/20a6c4d82a08becf7e8d913662b00357)架构图

那么**如何开启这个镜像集群模式**呢？其实很简单，RabbitMQ 有很好的管理控制台，就是在后台新增一个策略，这个策略是**镜像集群模式的策略**，指定的时候是可以要求数据同步到所有节点的，也可以要求同步到指定数量的节点，再次创建 queue 的时候，应用这个策略，就会自动将数据同步到其他的节点上去了。

这样的话，好处在于，你任何一个机器宕机了，没事儿，其它机器（节点）还包含了这个 queue 的完整数据，别的 consumer 都可以到其它节点上去消费数据。坏处在于，第一，这个性能开销也太大了吧，消息需要同步到所有机器上，导致网络带宽压力和消耗很重！第二，这么玩儿，不是分布式的，就**没有扩展性可言**了，如果某个 queue 负载很重，你加机器，新增的机器也包含了这个 queue 的所有数据，并**没有办法线性扩展**你的 queue。你想，如果这个 queue 的数据量很大，大到这个机器上的容量无法容纳了，此时该怎么办呢？

## 如何使用 RabbitMQ 实现 RPC

基于 [RabbitMQ reply_to](https://www.rabbitmq.com/direct-reply-to.html) 特性，可以很轻易使用 RabbitMQ 实现 RPC 功能。

具体的代码实现，可以看看 [《芋道 Spring Boot 消息队列 RabbitMQ 入门》](http://www.iocoder.cn/Spring-Boot/RabbitMQ/?vip)的[「15. RPC 远程调用」](http://svip.iocoder.cn/RabbitMQ/Interview/#) 小节。

🦅 **使用 RabbitMQ 实现 RPC 有什么好处？**

- 1、将客户端和服务器解耦：客户端只是发布一个请求到 MQ 并消费这个请求的响应。并不关心具体由谁来处理这个请求，MQ 另一端的请求的消费者可以随意替换成任何可以处理请求的服务器，并不影响到客户端。

  > 相当于 RPC 的注册发现功能，交给 RabbitMQ 来实现了。

- 2、减轻服务器的压力：传统的 RPC 模式中如果客户端和请求过多，服务器的压力会过大。由 MQ 作为中间件的话，过多的请求而是被 MQ 消化掉，服务器可以控制消费请求的频次，并不会影响到服务器。

- 3、服务器的横向扩展更加容易：如果服务器的处理能力不能满足请求的频次，只需要增加服务器来消费 MQ 的消息即可，MQ会帮我们实现消息消费的负载均衡。

- 4、可以看出 RabbitMQ 对于 RPC 模式的支持也是比较友好地，

  ```
  amq.rabbitmq.reply-to
  ```

  ,

   

  ```
  reply_to
  ```

  ,

   

  ```
  correlation_id
  ```

   

  这些特性都说明了这一点，再加上

   

  spring-rabbit

   

  的实现，可以让我们很简单的使用消息队列模式的 RPC 调用。

  > 例如说：[`rabbitmq-jsonrpc`](https://github.com/rabbitmq/rabbitmq-jsonrpc) 的实现。

当然，虽然有这些优点，实际场景下，我们并不会这么做。😈

🦅 **为什么 heavy RPC 的使用场景下不建议采用 disk node ？**

heavy RPC 是指在业务逻辑中高频调用 RabbitMQ 提供的 RPC 机制，导致不断创建、销毁 reply queue ，进而造成 disk node 的性能问题（因为会针对元数据不断写盘）。所以在使用 RPC 机制时需要考虑自身的业务场景，一般来说不建议。

## RabbitMQ 是否会弄丢数据？

> 艿艿：这个问题，基本是我们前面看到的几个问题的总结合并。

[![弄丢消息的几种情况](http://static.iocoder.cn/6303e69011255831c54d605250a6aa67)](http://static.iocoder.cn/6303e69011255831c54d605250a6aa67)弄丢消息的几种情况

🦅 **生产者弄丢了数据？**

生产者将数据发送到 RabbitMQ 的时候，可能数据就在半路给搞丢了，因为网络问题啥的，都有可能。

> 方案一：事务功能

🚀 此时可以选择用 RabbitMQ 提供的【事务功能】，就是生产者**发送数据之前**开启 RabbitMQ 事务`channel.txSelect`，然后发送消息，如果消息没有成功被 RabbitMQ 接收到，那么生产者会收到异常报错，此时就可以回滚事务`channel.txRollback`，然后重试发送消息；如果收到了消息，那么可以提交事务`channel.txCommit`。代码如下：

```
// 开启事务
channel.txSelect
try {
    // 这里发送消息
} catch (Exception e) {
    channel.txRollback

    // 这里再次重发这条消息
}

// 提交事务
channel.txCommit
```

- 但是问题是，RabbitMQ 事务机制（同步）一搞，基本上**吞吐量会下来，因为太耗性能**。

具体的代码实现，可以看看 [《芋道 Spring Boot 消息队列 RabbitMQ 入门》](http://www.iocoder.cn/Spring-Boot/RabbitMQ/?vip)的[「12. 事务消息」](http://svip.iocoder.cn/RabbitMQ/Interview/#) 小节。

> 方案二：confirm 功能。

🚀 所以一般来说，如果你要确保说写 RabbitMQ 的消息别丢，可以开启 【`confirm` 模式】，在生产者那里设置开启 `confirm` 模式之后，你每次写的消息都会分配一个唯一的 id，然后如果写入了 RabbitMQ 中，RabbitMQ 会给你回传一个 `ack` 消息，告诉你说这个消息 ok 了。如果 RabbitMQ 没能处理这个消息，会回调你的一个 `nack` 接口，告诉你这个消息接收失败，你可以重试。而且你可以结合这个机制自己在内存里维护每个消息 id 的状态，如果超过一定时间还没接收到这个消息的回调，那么你可以重发。

具体的代码实现，可以看看 [《芋道 Spring Boot 消息队列 RabbitMQ 入门》](http://www.iocoder.cn/Spring-Boot/RabbitMQ/?vip)的[「14. 生产者的发送确认」](http://svip.iocoder.cn/RabbitMQ/Interview/#) 小节。

> 对比总结

🚀 事务机制和 `confirm` 机制最大的不同在于，**事务机制是同步的**，你提交一个事务之后会**阻塞**在那儿，但是 `confirm` 机制是**异步**的，你发送个消息之后就可以发送下一个消息，然后那个消息 RabbitMQ 接收了之后会异步回调你的一个接口通知你这个消息接收到了。

所以一般在生产者这块**避免数据丢失**，都是用 `confirm` 机制的。

> 😈 不过 confirm 功能，也可能存在丢消息的情况。举个例子，如果回调到 `nack` 接口，此时 JVM 挂掉了，那么此消息就丢失了。（这个是艿艿的猜想，还在找胖友探讨中。关于这块，欢迎星球讨论。）

🦅 **Broker 弄丢了数据**

就是 Broker 自己弄丢了数据，这个你必须**开启 Broker 的持久化**，就是消息写入之后会持久化到磁盘，哪怕是 Broker 自己挂了，**恢复之后会自动读取之前存储的数据**，一般数据不会丢。除非极其罕见的是，Broker 还没持久化，自己就挂了，**可能导致少量数据丢失**，但是这个概率较小。

设置持久化有**两个步骤**：

- 创建 queue 的时候将其设置为持久化
  这样就可以保证 RabbitMQ 持久化 queue 的元数据，但是它是不会持久化 queue 里的数据的。
- 第二个是发送消息的时候将消息的 `deliveryMode` 设置为 2
  就是将消息设置为持久化的，此时 RabbitMQ 就会将消息持久化到磁盘上去。

必须要同时设置这两个持久化才行，Broker 哪怕是挂了，再次重启，也会从磁盘上重启恢复 queue，恢复这个 queue 里的数据。

注意，哪怕是你给 Broker 开启了持久化机制，也有一种可能，就是这个消息写到了 Broker 中，但是还没来得及持久化到磁盘上，结果不巧，此时 Broker 挂了，就会导致内存里的一点点数据丢失。

所以，持久化可以跟生产者那边的 `confirm` 机制配合起来，只有消息被持久化到磁盘之后，才会通知生产者 `ack` 了，所以哪怕是在持久化到磁盘之前，Broker 挂了，数据丢了，生产者收不到 `ack`，你也是可以自己重发的。

🦅 **消费端弄丢了数据？**

RabbitMQ 如果丢失了数据，主要是因为你消费的时候，**刚消费到，还没处理，结果进程挂了**，比如重启了，那么就尴尬了，RabbitMQ 认为你都消费了，这数据就丢了。

这个时候得用 RabbitMQ 提供的 `ack` 机制，简单来说，就是你必须关闭 RabbitMQ 的自动 `ack`，可以通过一个 api 来调用就行，然后每次你自己代码里确保处理完的时候，再在程序里 `ack` 一把。这样的话，如果你还没处理完，不就没有 `ack` 了？那 RabbitMQ 就认为你还没处理完，这个时候 RabbitMQ 会把这个消费分配给别的 consumer 去处理，消息是不会丢的。

🦅 **总结**

[![总结](http://static2.iocoder.cn/83213e2ac79cd8899b09a66a5cf71669)](http://static2.iocoder.cn/83213e2ac79cd8899b09a66a5cf71669)总结

## RabbitMQ 如何保证消息的顺序性？

和 Kafka 与 RocketMQ 不同，Kafka 不存在类似类似 Topic 的概念，而是真正的一条一条队列，并且每个队列可以被多个 Consumer 拉取消息。这个，是非常大的一个差异。

🚀 **来看看 RabbitMQ 顺序错乱的场景**：

一个 queue，多个 consumer。比如，生产者向 RabbitMQ 里发送了三条数据，顺序依次是 data1/data2/data3，压入的是 RabbitMQ 的一个内存队列。有三个消费者分别从 MQ 中消费这三条数据中的一条，结果消费者 2 先执行完操作，把 data2 存入数据库，然后是 data1/data3。这不明显乱了。😈 也就是说，乱序消费的问题。

[![乱序](http://static2.iocoder.cn/29ea655479f826f9a6a5d9d005f46c7c)](http://static2.iocoder.cn/29ea655479f826f9a6a5d9d005f46c7c)乱序

🚀 **解决方案**：

- 方案一，拆分多个 queue，每个 queue 一个 consumer，就是多一些 queue 而已，确实是麻烦点。

  > 这个方式，有点模仿 Kafka 和 RocketMQ 中 Topic 的概念。例如说，原先一个 queue 叫 `"xxx"` ，那么多个 queue ，我们可以叫 `"xxx-01"`、`"xxx-02"` 等，相同前缀，不同后缀。

- 方案二，或者就一个 queue 但是对应一个 consumer，然后这个 consumer 内部用内存队列做排队，然后分发给底层不同的 worker 来处理。

  > 这种方式，就是讲一个 queue 里的，相同的“key” 交给同一个 worker 来执行。因为 RabbitMQ 是可以单条消息来 ack ，所以还是比较方便的。这一点，也是和 RocketMQ 和 Kafka 不同的地方。

[![解决乱序](http://static2.iocoder.cn/f1bd6c79deaa6fae89a62d0e53f1ad43)](http://static2.iocoder.cn/f1bd6c79deaa6fae89a62d0e53f1ad43)解决乱序

实际上，我们会发现上述的两个方案，前提都是一个 queue 只能启动一个 consumer 对应。

具体的代码实现，可以看看 [《芋道 Spring Boot 消息队列 RabbitMQ 入门》](http://www.iocoder.cn/Spring-Boot/RabbitMQ/?vip)的[「11. 顺序消息」](http://svip.iocoder.cn/RabbitMQ/Interview/#) 小节



## 待整理资料

https://blog.csdn.net/mingongge/article/details/99512557

https://www.jianshu.com/p/78847c203b76



# Kafka 

## Apache Kafka 是什么?

Kafka 是基于**发布与订阅**的**消息系统**。Kafka 是一个分布式的，高吞吐、可分区的，冗余备份的持久性的日志服务。它主要用于处理活跃的流式数据。

- Kafka运行在JVM上

🦅 **Kafka 的主要特点？**

- 1、高吞吐。生产25 w/s（50MB），消费 55 w/s（110MB）。
- 2、持久化。
- 3、分布式系统，无需停机、易于横向扩展。
- 4、消息被处理的状态是在 Consumer 端维护。
- 5、支持 online（在线） 和 offline （离线）的场景。

🦅 **聊聊 Kafka 的设计要点？**

1）吞吐量

- 1、数据磁盘**持久化**：磁盘的**顺序读写性能**。

- 2、`zero-copy`

- 3、数据**批量发送**

- 4、数据**压缩**

- 5、Topic 划分为多个 `Partition` ，提高并行度。

  > 数据在磁盘上存取代价为 `O(1)`。
  >
  > - Kafka 以 Topic 来进行消息管理，每个 Topic 包含多个 Partition ，每个 Partition 对应一个**逻辑 log** ，由**多个 segment 文件组成**。
  > - 每个 segment 中存储多条消息（见下图），消息 id 由其逻辑位置决定，即**从消息 id 可直接定位到消息的存储位置**，避免 id 到位置的额外映射。
  > - **每个 Partition 在内存中对应一个 index** ，记录每个 segment 中的第一条消息偏移。
  >
  > 发布者发到某个 Topic 的消息会被均匀的分布到多个 Partition 上（随机或根据用户指定的回调函数进行分布），Broker 收到发布消息往对应 Partition 的最后一个 segment 上添加该消息。

- **内存文件映射:内存映射文件将磁盘上的文件内容与内存映射起来**，我们往内存里写入数据，操作系统会在稍后把数据冲刷到磁盘上。

  > 直接使用 Linux 文件系统的 Cache ，来高效缓存数据。
  >
  > 当某个 `segment`上 的消息条数达到配置值或消息发布时间超过阈值时，segment上 的消息会**被 flush 到磁盘**，**只有 flush 到磁盘上的消息订阅者才能订阅到**，segment 达到一定的大小后将不会再往该 segment 写数据，Broker 会创建新的 segment 文件。

2）**负载均衡**

- 1、Producer 根据用户**指定的算法，将消息发送到指定的 Partition 中**。
- 2、Topic 存在多个 `Partition` ，每个 Partition 有自己的`replica` ，每个 replica 分布在不同的 `Broker` 节点上。多个Partition 需要选取出 Leader partition ，**Leader Partition 负责读写**，并由 Zookeeper 负责 fail over （**失效转移**）。
- 3、相同 Topic 的多个 Partition 会分配给不同的 Consumer 进行拉取消息，进行消费。

3）**拉取系统**

由于 Kafka Broker 会持久化数据，Consumer 非常适合采取 pull 的方式消费数据，具有以下几点好处：

- 1、简化 Kafka 设计。
- 2、Consumer 自主控制消息拉取速度。
- 3、Consumer 自主选择消费模式，例如批量，重复消费，从尾端开始消费等。

4）可扩展性

> 通过 Zookeeper 管理 Broker 与 Consumer 的动态加入与离开。

- 当需要增加 Broker 节点时，新增的 Broker 会向 Zookeeper 注册，而 Producer 及 Consumer 会根据注册在 Zookeeper 上的 watcher 感知这些变化，并及时作出调整。
- 当新增和删除 Consumer 节点时，相同 Topic 的多个 Partition 会分配给剩余的 Consumer 们。

另外，推荐阅读 [《为什么 Kafka 这么快？》](https://mp.weixin.qq.com/s/pzVS7r3QaQPFwob-fY8b4A) 文章，写的更加细致。

## Kafka 的架构是怎么样的？

**Kafka拓扑结构**

![img](https://img2018.cnblogs.com/blog/907596/201907/907596-20190708110113755-1652522197.jpg)

的整体架构非常简单，是分布式架构，Producer、Broker 和Consumer 都可以有多个。

- Producer，Consumer 实现 Kafka 注册的接口。
- 数据从 Producer 发送到 Broker 中，Broker 承担一个中间缓存和分发的作用。
- Broker 分发注册到系统中的 Consumer。Broker 的作用类似于缓存，即活跃的数据和离线处理系统之间的缓存。
- 客户端和服务器端的通信，是基于简单，高性能，且与编程语言无关的 **TCP 协议**。

几个重要的基本概念：

- Topic：特指 Kafka 处理的消息源（feeds of messages）的不同分类。

- Partition：Topic 物理上的分组（分区），一个 Topic 可以分为多个 Partition 。每个 Partition 都是一个有序的队列。Partition 中的每条消息都会被分配一个有序的 id（offset）。

  > - replicas：Partition 的副本集，保障 Partition 的高可用。
  > - leader：replicas 中的一个角色，Producer 和 Consumer 只跟 Leader 交互。
  > - follower：replicas 中的一个角色，从 leader 中复制数据，作为副本，一旦 leader 挂掉，会从它的 followers 中选举出一个新的 leader 继续提供服务。

- Message：消息，是通信的基本单位，每个 Producer 可以向一个Topic（主题）发布一些消息。

- Producers：消息和数据生产者，向 Kafka 的一个 Topic 发布消息的过程，叫做 producers 。

- Consumers：消息和数据消费者，订阅 Topic ，并处理其发布的消息的过程，叫做 consumers 。

  > Consumer group：每个 Consumer 都属于一个 Consumer group，每条消息只能被 Consumer group 中的一个 Consumer 消费，但可以被多个 Consumer group 消费。

- Broker：缓存代理，Kafka 集群中的一台或多台服务器统称为 broker 。

  > Controller：Kafka 集群中，通过 Zookeeper 选举某个 Broker 作为 Controller ，用来进行 leader election 以及 各种 failover 。

- ZooKeeper：Kafka 通过 ZooKeeper 来存储集群的 Topic、Partition 等元信息等。

😈 单纯角色来说，Kafka 和 RocketMQ 是基本一致的。比较明显的差异是：

> RocketMQ 从 Kafka 演化而来。

- 1、Kafka 使用 Zookeeper 作为命名服务；RocketMQ 自己实现了一个轻量级的 Namesrver 。

- 2、Kafka Broker 的每个分区都有一个首领分区；RocketMQ 每个分区的“首领”分区，都在 Broker Master 节点上。

  > RocketMQ 没有首领分区一说，所以打上了引号。

- 3、Kafka Consumer 使用 **poll 的方式**拉取消息；RocketMQ Consumer 提供 poll 的方式的同时，封装了一个 push 的方式。

  > RocketMQ 的 push 的方式，也是基于 poll 的方式的封装。

- … 当然还有其它 …

🦅 **Kafka 为什么要将 Topic 进行分区？**

正如我们在 [「聊聊 Kafka 的设计要点？」](http://svip.iocoder.cn/Kafka/Interview/#) 问题中所看到的，是为了负载均衡，从而能够水平拓展。

- Topic 只是逻辑概念，面向的是 Producer 和 Consumer ，而 Partition 则是物理概念。如果 Topic 不进行分区，而将 Topic 内的消息存储于一个 Broker，那么关于该 Topic 的所有读写请求都将由这一个 Broker 处理，吞吐量很容易陷入瓶颈，这显然是不符合高吞吐量应用场景的。
- 有了 Partition 概念以后，假设一个 Topic 被分为 10 个 Partitions ，Kafka 会根据一定的算法将 10 个 Partition 尽可能均匀的分布到不同的 Broker（服务器）上。
- 当 Producer 发布消息时，Producer 客户端可以采用 random、key-hash 及轮询等算法选定目标 Partition ，若不指定，Kafka 也将根据一定算法将其置于某一分区上。
- 当 Consumer 拉取消息时，Consumer 客户端可以采用 Range、轮询 等算法分配 Partition ，从而从不同的 Broker 拉取对应的 Partition 的 leader 分区。

所以，Partiton 机制可以极大的提高吞吐量，并且使得系统具备良好的水平扩展能力。

## Kafka 的应用场景有哪些？

[![Kafka 的应用场景](http://static2.iocoder.cn/3636ff4bd554ee1dfcfb92448073b5b8)](http://static2.iocoder.cn/3636ff4bd554ee1dfcfb92448073b5b8)Kafka 的应用场景

1）消息队列

比起大多数的消息系统来说，Kafka 有更好的吞吐量，内置的分区，冗余及容错性，这让 Kafka 成为了一个很好的大规模消息处理应用的解决方案。消息系统一般吞吐量相对较低，但是需要更小的端到端延时，并常常依赖于 Kafka 提供的强大的持久性保障。在这个领域，Kafka 足以媲美传统消息系统，如 ActiveMQ 或 RabbitMQ 。

2）行为跟踪

Kafka 的另一个应用场景，是跟踪用户浏览页面、搜索及其他行为，以发布订阅的模式实时记录到对应的 Topic 里。那么这些结果被订阅者拿到后，就可以做进一步的实时处理，或实时监控，或放到 Hadoop / 离线数据仓库里处理。

3）元信息监控

作为操作记录的监控模块来使用，即汇集记录一些操作信息，可以理解为运维性质的数据监控吧。

4）日志收集

日志收集方面，其实开源产品有很多，包括 Scribe、Apache Flume 。很多人使用 Kafka 代替日志聚合（log aggregation）。日志聚合一般来说是从服务器上收集日志文件，然后放到一个集中的位置（文件服务器或 HDFS）进行处理。

然而， Kafka 忽略掉文件的细节，将其更清晰地抽象成一个个日志或事件的消息流。这就让 Kafka 处理过程延迟更低，更容易支持多数据源和分布式数据处理。比起以日志为中心的系统比如 Scribe 或者 Flume 来说，Kafka 提供同样高效的性能和因为复制导致的更高的耐用性保证，以及更低的端到端延迟。

5）流处理

这个场景可能比较多，也很好理解。保存收集流数据，以提供之后对接的 Storm 或其他流式计算框架进行处理。很多用户会将那些从原始 Topic 来的数据进行阶段性处理，汇总，扩充或者以其他的方式转换到新的 Topic 下再继续后面的处理。

例如一个文章推荐的处理流程，可能是先从 RSS 数据源中抓取文章的内容，然后将其丢入一个叫做“文章”的 Topic 中。后续操作可能是需要对这个内容进行清理，比如回复正常数据或者删除重复数据，最后再将内容匹配的结果返还给用户。这就在一个独立的 Topic 之外，产生了一系列的实时数据处理的流程。Strom 和 Samza 是非常著名的实现这种类型数据转换的框架。

6）事件源

事件源，是一种应用程序设计的方式。该方式的状态转移被记录为按时间顺序排序的记录序列。Kafka 可以存储大量的日志数据，这使得它成为一个对这种方式的应用来说绝佳的后台。比如动态汇总（News feed）。

7）持久性日志（Commit Log）

Kafka 可以为一种外部的持久性日志的分布式系统提供服务。这种日志可以在节点间备份数据，并为故障节点数据回复提供一种重新同步的机制。Kafka 中日志压缩功能为这种用法提供了条件。在这种用法中，Kafka 类似于 Apache BookKeeper 项目。

## Kafka 消息发送和消费的简化流程是什么？

[![Kafka 消息发送和消费](http://static2.iocoder.cn/456f3adab70714c0a1b1fdbd8a686732)](http://static2.iocoder.cn/456f3adab70714c0a1b1fdbd8a686732)Kafka 消息发送和消费

- 1、Producer ，根据指定的 partition 方法（round-robin、hash等），将消息发布到指定 Topic 的 Partition 里面。
- 2、Kafka 集群，接收到 Producer 发过来的消息后，将其持久化到硬盘，并保留消息指定时长（可配置），而不关注消息是否被消费。
- 3、Consumer ，从 Kafka 集群 pull 数据，并控制获取消息的 offset 。至于消费的进度，可手动或者自动提交给 Kafka 集群。

🦅 **1）Producer 发送消息**

Producer 采用 push 模式将消息发布到 Broker，每条消息都被 append 到 Patition 中，属于顺序写磁盘（顺序写磁盘效率比随机写内存要高，保障 Kafka 吞吐率）。Producer 发送消息到 Broker 时，会根据分区算法选择将其存储到哪一个 Partition 。

其路由机制为：

- 1、指定了 Partition ，则直接使用。
- 2、未指定 Partition 但指定 key ，通过对 key 进行 hash 选出一个 Partition 。
- 3、Partition 和 key 都未指定，使用轮询选出一个 Partition 。

写入流程：

- 1、Producer 先从 ZooKeeper 的 `"/brokers/.../state"` 节点找到该 Partition 的 leader 。

  > 注意噢，Producer 只和 Partition 的 leader 进行交互。

- 2、Producer 将消息发送给该 leader 。

- 3、leader 将消息写入本地 log 。

- 4、followers 从 leader pull 消息，写入本地 log 后 leader 发送 ACK 。

- 5、leader 收到所有 ISR 中的 replica 的 ACK 后，增加 HW（high watermark ，最后 commit 的 offset） 并向 Producer 发送 ACK 。

🦅 **2）Broker 存储消息**

物理上把 Topic 分成一个或多个 Patition，每个 Patition 物理上对应一个文件夹（该文件夹存储该 Patition 的所有消息和索引文件）。

🦅 **3）Consumer 消费消息**

high-level Consumer API 提供了 consumer group 的语义，一个消息只能被 group 内的一个 Consumer 所消费，且 Consumer 消费消息时不关注 offset ，最后一个 offset 由 ZooKeeper 保存（下次消费时，该 group 中的 Consumer 将从 offset 记录的位置开始消费）。

注意：

- 1、如果消费线程大于 Patition 数量，则有些线程将收不到消息。
- 2、如果 Patition 数量大于消费线程数，则有些线程多收到多个 Patition 的消息。
- 3、如果一个线程消费多个 Patition，则无法保证你收到的消息的顺序，而一个 Patition 内的消息是有序的。

Consumer 采用 pull 模式从 Broker 中读取数据。

- push 模式，很难适应消费速率不同的消费者，因为消息发送速率是由 Broker 决定的。它的目标是尽可能以最快速度传递消息，但是这样很容易造成 Consumer 来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而 pull 模式，则可以根据 Consumer 的消费能力以适当的速率消费消息。
- 对于 Kafka 而言，pull 模式更合适，它可简化 Broker 的设计，Consumer 可自主控制消费消息的速率，同时 Consumer 可以自己控制消费方式——即可批量消费也可逐条消费，同时还能选择不同的提交方式从而实现不同的传输语义。

🦅 **Kafka Producer 有哪些发送模式？**

Kafka 的发送模式由 Producer 端的配置参数 `producer.type`来设置。

- 这个参数指定了在后台线程中消息的发送方式是同步的还是异步的，默认是同步的方式，即 `producer.type=sync` 。
- 如果设置成异步的模式，即 `producer.type=async` ，可以是 Producer 以 batch 的形式 push 数据，这样会极大的提高 Broker的性能，但是这样会增加丢失数据的风险。
- 如果需要确保消息的可靠性，必须要将 `producer.type`设置为 sync 。

对于异步模式，还有 4 个配套的参数，如下：

[![参数](http://static2.iocoder.cn/792a59a9b2b4f271c179e81cf4278322)](http://static2.iocoder.cn/792a59a9b2b4f271c179e81cf4278322)参数

- 以 batch 的方式推送数据可以极大的提高处理效率，Kafka Producer 可以将消息在内存中累计到一定数量后作为一个 batch 发送请求。batch 的数量大小可以通过 Producer 的参数（`batch.num.messages`）控制。通过增加 batch 的大小，可以减少网络请求和磁盘 IO 的次数，当然具体参数设置需要在效率和时效性方面做一个权衡。

- 在比较新的版本中还有

   

  ```
  batch.size
  ```

   

  这个参数。Producer 会尝试批量发送属于同一个 Partition 的消息以减少请求的数量. 这样可以提升客户端和服务端的性能。默认大小是 16348 byte (16k).

  - 发送到 Broker 的请求可以包含多个 batch ，每个 batch 的数据属于同一个 Partition 。
  - 太小的 batch 会降低吞吐. 太大会浪费内存。

具体的代码实现，可以看看 [《芋道 Spring Boot 消息队列 Kafka 入门》](http://www.iocoder.cn/Spring-Boot/RabbitMQ/?vip)的[「3. 快速入门」](http://svip.iocoder.cn/Kafka/Interview/#)和[「4. 批量发送消息」](http://svip.iocoder.cn/Kafka/Interview/#)小节。

🦅 **Kafka Consumer 是否可以消费指定的分区消息？**

Consumer 消费消息时，向 Broker 发出“fetch”请求去消费特定分区的消息，Consumer 指定消息在日志中的偏移量(offset)，就可以消费从这个位置开始的消息，Consumer 拥有了 offset 的控制权，可以向后回滚去重新消费之前的消息，这是很有意义的。

🦅 **Kafka 的 high-level API 和 low-level API 的区别？**

High Level API

- 屏蔽了每个 Topic 的每个 Partition 的 offset 管理（自动读取Zookeeper 中该 Consumer group 的 last offset）、Broker 失败转移、以及增减 Partition 时 Consumer 时的负载均衡（Kafka 自动进行负载均衡）。
- 如果 Consumer 比 Partition 多，是一种浪费。一个 Partition 上是不允许并发的，所以 Consumer 数不要大于 Partition 数。

Low Level API

> Low-level API 也就是 Simple Consumer API ，实际上非常复杂。

- API 控制更灵活，例如消息重复读取，消息 offset 跳读，Exactly Once 原语。
- API 更复杂，offset 不再透明，需要自己管理，Broker 自动失败转移需要处理，增加 Consumer、Partition、Broker 需要自己做负载均衡。

## Kafka 的网络模型是怎么样的？

Kafka 基于高吞吐率和效率考虑，并没有使用第三方网络框架，而且自己基于 Java NIO 封装的。

🦅 **1）KafkaClient ，单线程 Selector 模型。**

[![KafkaClient](http://static2.iocoder.cn/00e8ec59cf40c62db53b4d66dc45e17c)](http://static2.iocoder.cn/00e8ec59cf40c62db53b4d66dc45e17c)KafkaClient

> 实际上，就是 NettyClient 的 NIO 方式。

- 单线程模式适用于并发链接数小，逻辑简单，数据量小。
- 在 Kafka 中，Consumer 和 Producer 都是使用的上面的单线程模式。这种模式不适合 Kafka 的服务端，在服务端中请求处理过程比较复杂，会造成线程阻塞，一旦出现后续请求就会无法处理，会造成大量请求超时，引起雪崩。而在服务器中应该充分利用多线程来处理执行逻辑。

🦅 **2）KafkaServer ，多线程 Selector 模型。**

> KafkaServer ，指的是 Kafka Broker 。

[![KafkaServer](http://static.iocoder.cn/8493bc98876609462fba617d520d1b9a)](http://static.iocoder.cn/8493bc98876609462fba617d520d1b9a)KafkaServer

Broker 的内部处理流水线化，分为多个阶段来进行(SEDA)，以提高吞吐量和性能，尽量避免 Thead 盲等待，以下为过程说明。

> 实际上，就是 NettyServer 的 NIO 方式。

- Accept Thread 负责与客户端建立连接链路，然后把 Socket 轮转交给Process Thread 。

  > 相当于 Netty 的 Boss EventLoop 。

- Process Thread 负责接收请求和响应数据，Process Thread 每次基于 Selector 事件循环，首先从 Response Queue 读取响应数据，向客户端回复响应，然后接收到客户端请求后，读取数据放入 Request Queue 。

  > 相当于 Netty 的 Worker EventLoop 。

- Work Thread 负责业务逻辑、IO 磁盘处理等，负责从 Request Queue 读取请求，并把处理结果放入 Response Queue 中，待 Process Thread 发送出去。

  > 相当于业务线程池。

😈 实际上，艿艿的想法，如果自己实现 MQ ，完全可以直接使用 Netty 作为网络通信框架。包括，RocketMQ 就是如此实现的。

🦅 **解释如何提高远程用户的吞吐量?**

如果 Producer、Consumer 位于与 Broker 不同的数据中心，则可能需要调优套接口缓冲区大小，以对长网络延迟进行摊销。

## Kafka 的数据存储模型是怎么样的?

Kafka 每个 Topic 下面的所有消息都是以 Partition 的方式分布式的存储在多个节点上。同时在 Kafka 的机器上，每个 Partition 其实都会对应一个日志目录，在目录下面会对应多个日志分段（LogSegment）。

```
MacBook-Pro-5:test-0 yunai$ ls
00000000000000000000.index	00000000000000000000.timeindex	leader-epoch-checkpoint
00000000000000000000.log	00000000000000000004.snapshot
```

- Topic 为 `test` ，Partition 为 0 ，所以文件目录是 `test-0` 。

LogSegment 文件由两部分组成，分别为 `.index` 文件和 `.log` 文件，分别表示为 segment **索引**文件和**数据**文件。这两个文件的命令规则为：Partition 全局的第一个 segment 从 0 开始，后续每个 segment 文件名为上一个 segment 文件最后一条消息的 offset 值，数值大小为 64 位，20 位数字字符长度，没有数字用 0 填充，如下，假设有 1000 条消息，每个 LogSegment 大小为 100 ，下面展现了 900-1000 的 `.index` 和 `.log` 文件：

[`.index` 和 `.log` 文件](https://user-gold-cdn.xitu.io/2018/7/24/164ca4f7e778be7c?imageView2/0/w/1280/h/960/format/jpeg/ignore-error/1)

- 由于 Kafka 消息数据太大，如果全部建立索引，即占了空间又增加了耗时，所以 Kafka 选择了稀疏索引的方式（通过 `.index` 索引 `.log` 文件），这样的话索引可以直接进入内存，加快偏查询速度。

🦅 **简单介绍一下如何读取数据？**

如果我们要读取第 911 条数据。

- 首先第一步，找到它是属于哪一段的，根据二分法查找到他属于的文件，找到 `0000900.index` 和 `00000900.log` 之后。

- 然后，去 `.index` 中去查找 `(911-900) =11` 这个索引或者小于 11 最近的索引，在这里通过二分法我们找到了索引是 `[10,1367]` 。

  > - 10 表示，第 10 条消息开始。
  > - 1367 表示，在 `.log` 的第 1367 字节开始。
  >
  > 😈 所以，本图的第 911 条的“1360”是错的，相比“1367” 反倒小了。

- 然后，我们通过这条索引的物理位置 1367 ，开始往后找，直到找到 911 条数据。

上面讲的是如果要找某个 offset 的流程，但是我们大多数时候并不需要查找某个 offset ，只需要按照顺序读即可。而在顺序读中，操作系统会对内存和磁盘之间添加 page cahe ，也就是我们平常见到的预读操作，所以我们的顺序读操作时速度很快。但是 Kafka 有个问题，如果分区过多，那么日志分段也会很多，写的时候由于是批量写，其实就会变成随机写了，随机 I/O 这个时候对性能影响很大。所以一般来说 Kafka 不能有太多的Partition 。针对这一点，RocketMQ 把所有的日志都写在一个文件里面，就能变成顺序写，通过一定优化，读也能接近于顺序读。

> 并且，截止到 RocketMQ4 版本，索引文件，对每个数据文件中的消息，都有对应的索引。这个是和 Kafka 的稀疏索引不太一样的地方。

更详尽的，推荐阅读 [《Kafka 之数据存储》](http://matt33.com/2016/03/08/kafka-store/) 文章。

🦅 **为什么不能以 Partition 作为存储单位？**

如果就以 Partition 为最小存储单位，可以想象，当 Kafka Producer 不断发送消息，必然会引起 Partition 文件的无限扩张，将对消息文件的维护以及已消费的消息的清理带来严重的影响，因此，需以 segment 为单位将 Partition 进一步细分。

每个 Partition（目录）相当于一个巨型文件，被平均分配到多个大小相等的 segment（段）数据文件中（每个 segment 文件中消息数量不一定相等），这种特性也方便 old segment 的删除，即方便已被消费的消息的清理，提高磁盘的利用率。每个 Partition 只需要支持顺序读写就行，segment 的文件生命周期由服务端配置参数（`log.segment.bytes`，`log.roll.{ms,hours}` 等若干参数）决定。

## Kafka 的消息格式是怎么样的？

message 中的物理结构为：

[![message 物理结构](http://static2.iocoder.cn/a034ab2df1d5b67520432355e0bbf95b)](http://static2.iocoder.cn/a034ab2df1d5b67520432355e0bbf95b)message 物理结构

参数说明：

| 关键字              | 解释说明                                                     |
| :------------------ | :----------------------------------------------------------- |
| 8 byte offset       | 在parition(分区)内的每条消息都有一个有序的id号，这个id号被称为偏移(offset),它可以唯一确定每条消息在parition(分区)内的位置。即offset表示partiion的第多少message |
| 4 byte message size | message大小                                                  |
| 4 byte CRC32        | 用crc32校验message                                           |
| 1 byte “magic”      | 表示本次发布Kafka服务程序协议版本号                          |
| 1 byte “attributes” | 表示为独立版本、或标识压缩类型、或编码类型                   |
| 4 byte key length   | 表示key的长度,当key为-1时，K byte key字段不填                |
| K byte key          | 可选                                                         |
| value bytes payload | 表示实际消息数据                                             |

不过，这是早期 Kafka 的版本，最新版本的格式，推荐阅读如下两篇文章：

- [《一文看懂 Kafka 消息格式的演变》](http://www.iocoder.cn/Kafka/message-format/?vip)
- [《Kafka 消息格式中的变长字段（Varints）》](http://www.iocoder.cn/Kafka/varints/)

当然，看懂这个数据格式，基本也能知道消息的大体格式。

## Kafka 的副本机制是怎么样的？

Kafka 的副本机制，是多个 Broker 节点对其他节点的 Topic 分区的日志进行复制。当集群中的某个节点出现故障，访问故障节点的请求会被转移到其他正常节点(这一过程通常叫 Reblance)，Kafka 每个主题的每个分区都有一个主副本以及 0 个或者多个副本，副本保持和主副本的数据同步，当主副本出故障时就会被替代。

[![副本机制](http://static2.iocoder.cn/1f42986437f76c108007e86a51ae1287)](http://static2.iocoder.cn/1f42986437f76c108007e86a51ae1287)副本机制

> 注意哈，下面说的 Leader 指的是每个 Topic 的某个分区的 Leader ，而不是 Broker 集群中的【集群控制器】。

在 Kafka 中并不是所有的副本都能被拿来替代主副本，所以在 Kafka 的Leader 节点中维护着一个 ISR（In sync Replicas）集合，翻译过来也叫正在同步中集合，在这个集合中的需要满足两个条件:

- 1、节点必须和 Zookeeper 保持连接。
- 2、在同步的过程中这个副本不能落后主副本太多。

另外还有个 AR（Assigned Replicas）用来标识副本的全集，OSR 用来表示由于落后被剔除的副本集合，所以公式如下：

- ISR = Leader + 没有落后太多的副本。
- AR = OSR + ISR 。

这里先要说下两个名词：HW 和 LEO 。

- HW（高水位 HighWatermark），是 Consumer 能够看到的此 Partition 的位置。
- LEO（logEndOffset），是每个 Partition 的 log 最后一条 Message 的位置。
- HW 能保证 Leader 所在的 Broker 失效，该消息仍然可以从新选举的Leader 中获取，不会造成消息丢失。

当 Producer 向 Leader 发送数据时，可以通过`request.required.acks` 参数来设置数据可靠性的级别：

- 1（默认）：这意味着 Producer 在 ISR 中的 Leader 已成功收到的数据并得到确认后发送下一条 message 。如果 Leader 宕机了，则会丢失数据。
- 0：这意味着 Producer 无需等待来自 Broker 的确认而继续发送下一批消息。这种情况下数据传输效率最高，但是数据可靠性确是最低的。
- -1：Producer 需要等待 ISR 中的所有 Follower 都确认接收到数据后才算一次发送完成，可靠性最高。但是这样也不能保证数据不丢失，比如当 ISR 中只有 Leader 时(其他节点都和 Zookeeper 断开连接，或者都没追上)，这样就变成了 `acks=1` 的情况。

关于这块详详细的内容，推荐阅读

- [《Kafka 数据可靠性深度解读》](https://blog.csdn.net/u013256816/article/details/71091774) 的 [「3 高可靠性存储分析」](http://svip.iocoder.cn/Kafka/Interview/#) 小节
- [《Kafka 集群内复制功能深入剖析》](https://www.jianshu.com/p/03d6a335237f)

## ZooKeeper 在 Kafka 中起到什么作用？

关于 ZooKeeper 是什么，不了解的胖友，直接去看 [《精尽 Zookeeper 面试题》](http://svip.iocoder.cn/Zookeeper/Interview/) 。

在基于 Kafka 的分布式消息队列中，ZooKeeper 的作用有：

- 1、Broker 在 ZooKeeper 中的注册。

- 2、Topic 在 ZooKeeper 中的注册。

- 3、Consumer 在 ZooKeeper 中的注册。

- 4、Producer 负载均衡。

  > 主要指的是，Producer 从 Zookeeper 拉取 Topic 元数据，从而能够将消息发送负载均衡到对应 Topic 的分区中。

- 5、Consumer 负载均衡。

- 6、记录消费进度 Offset 。

  > Kafka 已推荐将 consumer 的 Offset 信息保存在 Kafka 内部的 Topic 中。

- 7、记录 Partition 与 Consumer 的关系。

其实，总结起来，就是两类功能：

- Broker、Producer、Consumer 和 Zookeeper 的交互。

  > 对应 1、2、3、5 。

- 相应的状态存储到 Zookeeper 中。

  > 对应 4、6、7 。

详细的每一点，看 [《再谈基于 Kafka 和 ZooKeeper 的分布式消息队列原理》](https://gitbook.cn/books/5bc446269a9adf54c7ccb8bc/index.html) 的 [「Kafka 架构中 ZooKeeper 以怎样的形式存在？」](http://svip.iocoder.cn/Kafka/Interview/#) 小节。

## Kafka 如何实现高可用？

在 [「Kafka 的架构是怎么样的？」](http://svip.iocoder.cn/Kafka/Interview/#) 问题中，已经基本回答了这个问题。

[Kafka 集群](https://segmentfault.com/img/bVbcmpm?w=776&h=436)

- Zookeeper 部署 2N+1 节点，形成 Zookeeper 集群，保证高可用。

- Kafka Broker 部署集群。每个 Topic 的 Partition ，基于【副本机制】，在 Broker 集群中复制，形成 replica 副本，保证消息存储的可靠性。每个 replica 副本，都会选择出一个 leader 分区（Partition），提供给客户端（Producer 和 Consumer）进行读写。

- Kafka Producer 无需考虑集群，因为和业务服务部署在一起。Producer 从 Zookeeper 拉取到 Topic 的元数据后，选择对应的 Topic 的 leader 分区，进行消息发送写入。而 Broker 根据 Producer 的 `request.required.acks` 配置，是写入自己完成就响应给 Producer 成功，还是写入所有 Broker 完成再响应。这个，就是胖友自己对消息的可靠性的选择。

- Kafka Consumer 部署集群。每个 Consumer 分配其对应的 Topic Partition ，根据对应的分配策略。并且，Consumer 只从 leader 分区（Partition）拉取消息。另外，当有新的 Consumer 加入或者老的 Consumer 离开，都会将 Topic Partition 再均衡，重新分配给 Consumer 。

  > 注意噢，此处说的都是同一个 Kafka Consumer group 。

总的来说，Kafka 和 RocketMQ 的高可用方式是比较类似的，主要的差异在 Kafka Broker 的副本机制，和 RocketMQ Broker 的主从复制，两者的差异，以及差异带来的生产和消费不同。😈 当然，实际上，都是和“主” Broker 做消息的发送和读取不是？！

## 什么是 Kafka 事务？

推荐阅读 [《Kafka 事务简介》](https://blog.csdn.net/ransom0512/article/details/78840042) 文章。

😈 和想象中的，是不是有点差别？！

具体的代码实现，可以看看 [《芋道 Spring Boot 消息队列 Kafka 入门》](http://www.iocoder.cn/Spring-Boot/RabbitMQ/?vip)的[「11. 事务消息」](http://svip.iocoder.cn/Kafka/Interview/#)小节。

## Kafka 是否会弄丢数据？

> 艿艿：注意，Kafka 是否会丢数据，主要取决于我们如何使用。这点，非常重要噢。

🦅 **消费端弄丢了数据？**

唯一可能导致消费者弄丢数据的情况，就是说，你消费到了这个消息，然后消费者那边自动提交了 offset ，让 Broker 以为你已经消费好了这个消息，但其实你才刚准备处理这个消息，你还没处理，你自己就挂了，此时这条消息就丢咯。

这不是跟 RabbitMQ 差不多吗，大家都知道 Kafka 会自动提交 offset ，那么只要关闭自动提交 offset ，在处理完之后自己手动提交 offset ，就可以保证数据不会丢。但是此时确实还是可能会有重复消费，比如你刚处理完，还没提交 offset ，结果自己挂了，此时肯定会重复消费一次，自己保证幂等性就好了。

> RocketMQ push 模式下，在确认消息被消费完成，才会提交 Offset 给 Broker 。

生产环境碰到的一个问题，就是说我们的 Kafka 消费者消费到了数据之后是写到一个内存的 queue 里先缓冲一下，结果有的时候，你刚把消息写入内存 queue ，然后消费者会自动提交 offset 。然后此时我们重启了系统，就会导致内存 queue 里还没来得及处理的数据就丢失了。

具体的代码实现，可以看看 [《芋道 Spring Boot 消息队列 Kafka 入门》](http://www.iocoder.cn/Spring-Boot/RabbitMQ/?vip)的[「12. 消费进度的提交机制」](http://svip.iocoder.cn/Kafka/Interview/#)小节。

🦅 **Broker 弄丢了数据？**

这块比较常见的一个场景，就是 Kafka 某个 Broker 宕机，然后重新选举 Partition 的 leader。大家想想，要是此时其他的 follower 刚好还有些数据没有同步，结果此时 leader 挂了，然后选举某个 follower 成 leader 之后，不就少了一些数据？这就丢了一些数据啊。

生产环境也遇到过，我们也是，之前 Partition 的 leader 机器宕机了，将 follower 切换为 leader 之后，就会发现说这个数据就丢了。

所以此时一般是要求起码设置如下 4 个参数：

- 给 Topic 设置 `replication.factor` 参数：这个值必须大于 1，要求每个 partition 必须有至少 2 个副本。

- 在 Kafka 服务端设置 `min.insync.replicas` 参数：这个值必须大于 1 ，这个是要求一个 leader 至少感知到有至少一个 follower 还跟自己保持联系，没掉队，这样才能确保 leader 挂了还有一个 follower 吧。

- 在 Producer 端设置 `acks=all`：这个是要求每条数据，必须是**写入所有 replica 之后，才能认为是写成功了**。

  > 不过这个也不一定能够绝对保证，例如说，Broker 集群里，所有节点都挂了，只剩下一个节点。此时，`acks=all` 和 `acks=1` 就等价了。当然，也可以通过设置 `min.insync.replics` 参数，每次写入要求最小的同步副本数。
  >
  > 这块也和朋友交流了下，他们金融场景下，`acks=all` 也是这么配置的。原因嘛，因为他们是金融场景呀。

- 在 Producer 端设置 `retries=MAX`（很大很大很大的一个值，无限次重试的意思）：这个是**要求一旦写入失败，就无限重试**，卡在这里了。

我们生产环境就是按照上述要求配置的，这样配置之后，至少在 Kafka broker 端就可以保证在 leader 所在 Broker 发生故障，进行 leader 切换时，数据不会丢失。

🦅 **生产者会不会弄丢数据？**

如果按照上述的思路设置了 `acks=all` ，一定不会丢，要求是，你的 leader 接收到消息，所有的 follower 都同步到了消息之后，才认为本次写成功了。如果没满足这个条件，生产者会自动不断的重试，重试无限次。

- 关于 Kafka Producer 重试发送消息的逻辑的源码解析，可以看看 [《Kafka 重试机制解读》](https://www.jianshu.com/p/5bf9c7c02ada) 。

------

😈 另外，在推荐一篇文章 [《360 度测试：KAFKA 会丢数据么？其高可用是否满足需求？》](https://juejin.im/post/5bf8f2bee51d4507c12bc722) ，提供了一些测试示例。

关于这一块，可以重点看看 [《Kafka 权威指南》](https://u.jd.com/jJpbF8) 的 [「6.4 在可靠的系统里使用生产者」](http://svip.iocoder.cn/Kafka/Interview/#) 和 [「6.5 在可靠的系统里使用消费者」](http://svip.iocoder.cn/Kafka/Interview/#) 小节。

## Kafka 如何保证消息的顺序性？

Kafka 本身，并不像 RocketMQ 一样，提供顺序性的消息。所以，提供的方案，都是相对有损的。如下：

> 这里的顺序消息，我们更多指的是，单个 Partition 的消息，被顺序消费。

- 方式一，Consumer ，对每个 Partition 内部单线程消费，单线程吞吐量太低，一般不会用这个。

- 方式二，Consumer ，拉取到消息后，写到 N 个内存 queue，具有相同 key 的数据都到同一个内存 queue 。然后，对于 N 个线程，每个线程分别消费一个内存 queue 即可，这样就能保证顺序性。

  > 这种方式，相当于对【方式一】的改进，将相同 Partition 的消息进一步拆分，保证相同 key 的数据消费是顺序的。
  >
  > 不过这种方式，消费进度的更新会比较麻烦。

当然，实际情况也不太需要考虑消息的顺序性，基本没有业务需要。

具体的代码实现，可以看看 [《芋道 Spring Boot 消息队列 Kafka 入门》](http://www.iocoder.cn/Spring-Boot/RabbitMQ/?vip)的[「10. 顺序消息」](http://svip.iocoder.cn/Kafka/Interview/#)小节。

## 666. 彩蛋

😈 略显仓促的一篇文章，后续会重新在梳理一下。如果胖友对 Kafka 有什么疑惑，一定要在星球里提出，我们一起在讨论和解答一波，然后整理到这篇文章中。

同时，期待下厮大的 Kafka 新书。

参考与推荐如下文章：

- [《高并发面试必问：分布式消息系统 Kafka 简介》](https://blog.csdn.net/caisini_vc/article/details/48007297)

- [《14 个最常见的 Kafka 面试题及答案》](https://blog.csdn.net/yjh314/article/details/77568580)

  > 这篇博客，有点傻逼。。。。

- [《你需要知道的 Kafka》](https://juejin.im/post/5b573eafe51d45198f5c80a4)

- [《Kafka 内部网络框架模型分析》](https://blog.csdn.net/lizhitao/article/details/52332749)

- [《再谈基于 Kafka 和 ZooKeeper 的分布式消息队列原理》](https://gitbook.cn/books/5bc446269a9adf54c7ccb8bc/index.html)

- [《如何保证消息的可靠性传输？（如何处理消息丢失的问题）》](https://github.com/doocs/advanced-java/blob/master/docs/high-concurrency/how-to-ensure-the-reliable-transmission-of-messages.md)

**文章目录**

1. [Apache Kafka 是什么?](http://svip.iocoder.cn/Kafka/Interview/#Apache-Kafka-是什么)
2. [Kafka 的架构是怎么样的？](http://svip.iocoder.cn/Kafka/Interview/#Kafka-的架构是怎么样的？)
3. [Kafka 的应用场景有哪些？](http://svip.iocoder.cn/Kafka/Interview/#Kafka-的应用场景有哪些？)
4. [Kafka 消息发送和消费的简化流程是什么？](http://svip.iocoder.cn/Kafka/Interview/#Kafka-消息发送和消费的简化流程是什么？)
5. [Kafka 的网络模型是怎么样的？](http://svip.iocoder.cn/Kafka/Interview/#Kafka-的网络模型是怎么样的？)
6. [Kafka 的数据存储模型是怎么样的?](http://svip.iocoder.cn/Kafka/Interview/#Kafka-的数据存储模型是怎么样的)
7. [Kafka 的消息格式是怎么样的？](http://svip.iocoder.cn/Kafka/Interview/#Kafka-的消息格式是怎么样的？)
8. [Kafka 的副本机制是怎么样的？](http://svip.iocoder.cn/Kafka/Interview/#Kafka-的副本机制是怎么样的？)
9. [ZooKeeper 在 Kafka 中起到什么作用？](http://svip.iocoder.cn/Kafka/Interview/#ZooKeeper-在-Kafka-中起到什么作用？)
10. [Kafka 如何实现高可用？](http://svip.iocoder.cn/Kafka/Interview/#Kafka-如何实现高可用？)
11. [什么是 Kafka 事务？](http://svip.iocoder.cn/Kafka/Interview/#什么是-Kafka-事务？)
12. [Kafka 是否会弄丢数据？](http://svip.iocoder.cn/Kafka/Interview/#Kafka-是否会弄丢数据？)
13. [Kafka 如何保证消息的顺序性？](http://svip.iocoder.cn/Kafka/Interview/#Kafka-如何保证消息的顺序性？)
14. [666. 彩蛋](http://svip.iocoder.cn/Kafka/Interview/#666-彩蛋)

© 2014 - 2020 芋道源码 | 

总访客数 806807 次 && 总访问量 3936675 次

[回到首页](http://svip.iocoder.cn/index)