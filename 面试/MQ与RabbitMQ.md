## 为什么使用MQ？MQ的优点

简答

- 异步处理 - 相比于传统的串行、并行方式，提高了系统吞吐量。
- 应用解耦 - 系统间通过消息通信，不用关心其他系统的处理。
- 流量削锋 - 可以通过消息队列长度控制请求量；可以缓解短时间内的高并发请求。
- 日志处理 - 解决大量日志传输。
- 消息通讯 - 消息队列一般都内置了高效的通信机制，因此也可以用在纯的消息通讯。比如实现点对点消息队列，或者聊天室等。

详答

主要是：解耦、异步、削峰。

**解耦**：A 系统发送数据到 BCD 三个系统，通过接口调用发送。如果 E 系统也要这个数据呢？那如果 C 系统现在不需要了呢？A 系统负责人几乎崩溃…A 系统跟其它各种乱七八糟的系统严重耦合，A 系统产生一条比较关键的数据，很多系统都需要 A 系统将这个数据发送过来。如果使用 MQ，A 系统产生一条数据，发送到 MQ 里面去，哪个系统需要数据自己去 MQ 里面消费。如果新系统需要数据，直接从 MQ 里消费即可；如果某个系统不需要这条数据了，就取消对 MQ 消息的消费即可。这样下来，A 系统压根儿不需要去考虑要给谁发送数据，不需要维护这个代码，也不需要考虑人家是否调用成功、失败超时等情况。

就是一个系统或者一个模块，调用了多个系统或者模块，互相之间的调用很复杂，维护起来很麻烦。但是其实这个调用是不需要直接同步调用接口的，如果用 MQ 给它异步化解耦。

**异步**：A 系统接收一个请求，需要在自己本地写库，还需要在 BCD 三个系统写库，自己本地写库要 3ms，BCD 三个系统分别写库要 300ms、450ms、200ms。最终请求总延时是 3 + 300 + 450 + 200 = 953ms，接近 1s，用户感觉搞个什么东西，慢死了慢死了。用户通过浏览器发起请求。如果使用 MQ，那么 A 系统连续发送 3 条消息到 MQ 队列中，假如耗时 5ms，A 系统从接受一个请求到返回响应给用户，总时长是 3 + 5 = 8ms。

**削峰**：减少高峰时期对服务器压力。

## 消息队列有什么优缺点？RabbitMQ有什么优缺点？

优点上面已经说了，就是**在特殊场景下有其对应的好处**，**解耦**、**异步**、**削峰**。

缺点有以下几个：

**系统可用性降低**

本来系统运行好好的，现在你非要加入个消息队列进去，那消息队列挂了，你的系统不是呵呵了。因此，系统可用性会降低；

**系统复杂度提高**

加入了消息队列，要多考虑很多方面的问题，比如：一致性问题、如何保证消息不被重复消费、如何保证消息可靠性传输等。因此，需要考虑的东西更多，复杂性增大。

**一致性问题**

A 系统处理完了直接返回成功了，人都以为你这个请求就成功了；但是问题是，要是 BCD 三个系统那里，BD 两个系统写库成功了，结果 C 系统写库失败了，咋整？你这数据就不一致了。

所以消息队列实际是一种非常复杂的架构，你引入它有很多好处，但是也得针对它带来的坏处做各种额外的技术方案和架构来规避掉，做好之后，你会发现，妈呀，系统复杂度提升了一个数量级，也许是复杂了 10 倍。但是关键时刻，用，还是得用的。

## 你们公司生产环境用的是什么消息中间件？

这个首先你可以说下你们公司选用的是什么消息中间件，比如用的是RabbitMQ，然后可以初步给一些你对不同MQ中间件技术的选型分析。

举个例子：比如说ActiveMQ是老牌的消息中间件，国内很多公司过去运用的还是非常广泛的，功能很强大。

但是问题在于没法确认ActiveMQ可以支撑互联网公司的高并发、高负载以及高吞吐的复杂场景，在国内互联网公司落地较少。而且使用较多的是一些传统企业，用ActiveMQ做异步调用和系统解耦。

然后你可以说说RabbitMQ，他的好处在于可以支撑高并发、高吞吐、性能很高，同时有非常完善便捷的后台管理界面可以使用。

另外，他还支持集群化、高可用部署架构、消息高可靠支持，功能较为完善。

而且经过调研，国内各大互联网公司落地大规模RabbitMQ集群支撑自身业务的case较多，国内各种中小型互联网公司使用RabbitMQ的实践也比较多。

除此之外，RabbitMQ的开源社区很活跃，较高频率的迭代版本，来修复发现的bug以及进行各种优化，因此综合考虑过后，公司采取了RabbitMQ。

但是RabbitMQ也有一点缺陷，就是他自身是基于erlang语言开发的，所以导致较为难以分析里面的源码，也较难进行深层次的源码定制和改造，毕竟需要较为扎实的erlang语言功底才可以。

然后可以聊聊RocketMQ，是阿里开源的，经过阿里的生产环境的超高并发、高吞吐的考验，性能卓越，同时还支持分布式事务等特殊场景。

而且RocketMQ是基于Java语言开发的，适合深入阅读源码，有需要可以站在源码层面解决线上生产问题，包括源码的二次开发和改造。

另外就是Kafka。Kafka提供的消息中间件的功能明显较少一些，相对上述几款MQ中间件要少很多。

但是Kafka的优势在于专为超高吞吐量的实时日志采集、实时数据同步、实时数据计算等场景来设计。

因此Kafka在大数据领域中配合实时计算技术（比如Spark Streaming、Storm、Flink）使用的较多。但是在传统的MQ中间件使用场景中较少采用。

## Kafka、ActiveMQ、RabbitMQ、RocketMQ 有什么优缺点？

|            | ActiveMQ                                                | RabbitMQ                                                     | RocketMQ                                                     | Kafka                                                        | ZeroMQ               |
| ---------- | ------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------- |
| 单机吞吐量 | 比RabbitMQ低                                            | 2.6w/s（消息做持久化）                                       | 11.6w/s                                                      | 17.3w/s                                                      | 29w/s                |
| 开发语言   | Java                                                    | Erlang                                                       | Java                                                         | Scala/Java                                                   | C                    |
| 主要维护者 | Apache                                                  | Mozilla/Spring                                               | Alibaba                                                      | Apache                                                       | iMatix，创始人已去世 |
| 成熟度     | 成熟                                                    | 成熟                                                         | 开源版本不够成熟                                             | 比较成熟                                                     | 只有C、PHP等版本成熟 |
| 订阅形式   | 点对点(p2p)、广播（发布-订阅）                          | 提供了4种：direct, topic ,Headers和fanout。fanout就是广播模式 | 基于topic/messageTag以及按照消息类型、属性进行正则匹配的发布订阅模式 | 基于topic以及按照topic进行正则匹配的发布订阅模式             | 点对点(p2p)          |
| 持久化     | 支持少量堆积                                            | 支持少量堆积                                                 | 支持大量堆积                                                 | 支持大量堆积                                                 | 不支持               |
| 顺序消息   | 不支持                                                  | 不支持                                                       | 支持                                                         | 支持                                                         | 不支持               |
| 性能稳定性 | 好                                                      | 好                                                           | 一般                                                         | 较差                                                         | 很好                 |
| 集群方式   | 支持简单集群模式，比如’主-备’，对高级集群模式支持不好。 | 支持简单集群，'复制’模式，对高级集群模式支持不好。           | 常用 多对’Master-Slave’ 模式，开源版本需手动切换Slave变成Master | 天然的‘Leader-Slave’无状态集群，每台服务器既是Master也是Slave | 不支持               |
| 管理界面   | 一般                                                    | 较好                                                         | 一般                                                         | 无                                                           | 无                   |

综上，各种对比之后，有如下建议：

一般的业务系统要引入 MQ，最早大家都用 ActiveMQ，但是现在确实大家用的不多了，没经过大规模吞吐量场景的验证，社区也不是很活跃，所以大家还是算了吧，我个人不推荐用这个了；

后来大家开始用 RabbitMQ，但是确实 erlang 语言阻止了大量的 Java 工程师去深入研究和掌控它，对公司而言，几乎处于不可控的状态，但是确实人家是开源的，比较稳定的支持，活跃度也高；

不过现在确实越来越多的公司会去用 RocketMQ，确实很不错，毕竟是阿里出品，但社区可能有突然黄掉的风险（目前 RocketMQ 已捐给 [Apache](https://github.com/apache/rocketmq)，但 GitHub 上的活跃度其实不算高）对自己公司技术实力有绝对自信的，推荐用 RocketMQ，否则回去老老实实用 RabbitMQ 吧，人家有活跃的开源社区，绝对不会黄。

所以**中小型公司**，技术实力较为一般，技术挑战不是特别高，用 RabbitMQ 是不错的选择；**大型公司**，基础架构研发实力较强，用 RocketMQ 是很好的选择。

如果是**大数据领域**的实时计算、日志采集等场景，用 Kafka 是业内标准的，绝对没问题，社区活跃度很高，绝对不会黄，何况几乎是全世界这个领域的事实性规范。

## MQ 有哪些常见问题？如何解决这些问题？

MQ 的常见问题有：

1. 消息的顺序问题
2. 消息的重复问题

**消息的顺序问题**

消息有序指的是可以按照消息的发送顺序来消费。

假如生产者产生了 2 条消息：M1、M2，假定 M1 发送到 S1，M2 发送到 S2，如果要保证 M1 先于 M2 被消费，怎么做？

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS81LzQvMTZhODMwNzVlMjc1NTFiMA?x-oss-process=image/format,png)

解决方案：

（1）保证生产者 - MQServer - 消费者是一对一对一的关系

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS81LzQvMTZhODMwNjg3MDJhOTBiNw?x-oss-process=image/format,png)

缺陷：

- 并行度就会成为消息系统的瓶颈（吞吐量不够）
- 更多的异常处理，比如：只要消费端出现问题，就会导致整个处理流程阻塞，我们不得不花费更多的精力来解决阻塞的问题。 （2）通过合理的设计或者将问题分解来规避。
- 不关注乱序的应用实际大量存在
- 队列无序并不意味着消息无序 所以从业务层面来保证消息的顺序而不仅仅是依赖于消息系统，是一种更合理的方式。

**消息的重复问题**

造成消息重复的根本原因是：网络不可达。

所以解决这个问题的办法就是绕过这个问题。那么问题就变成了：如果消费端收到两条一样的消息，应该怎样处理？

消费端处理消息的业务逻辑保持幂等性。只要保持幂等性，不管来多少条重复消息，最后处理的结果都一样。保证每条消息都有唯一编号且保证消息处理成功与去重表的日志同时出现。利用一张日志表来记录已经处理成功的消息的 ID，如果新到的消息 ID 已经在日志表中，那么就不再处理这条消息。

## 什么是RabbitMQ？

RabbitMQ是一款开源的，Erlang编写的，基于AMQP协议的消息中间件

## rabbitmq 的使用场景

（1）服务间异步通信

（2）顺序消费

（3）定时任务

（4）请求削峰

## RabbitMQ基本概念

- Broker： 简单来说就是消息队列服务器实体
- Exchange： 消息交换机，它指定消息按什么规则，路由到哪个队列
- Queue： 消息队列载体，每个消息都会被投入到一个或多个队列
- Binding： 绑定，它的作用就是把exchange和queue按照路由规则绑定起来
- Routing Key： 路由关键字，exchange根据这个关键字进行消息投递
- VHost： vhost 可以理解为虚拟 broker ，即 mini-RabbitMQ server。其内部均含有独立的 queue、exchange 和 binding 等，但最最重要的是，其拥有独立的权限系统，可以做到 vhost 范围的用户控制。当然，从 RabbitMQ 的全局角度，vhost 可以作为不同权限隔离的手段（一个典型的例子就是不同的应用可以跑在不同的 vhost 中）。
- Producer： 消息生产者，就是投递消息的程序
- Consumer： 消息消费者，就是接受消息的程序
- Channel： 消息通道，在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务

由Exchange、Queue、RoutingKey三个才能决定一个从Exchange到Queue的唯一的线路。

## RabbitMQ的工作模式

**一.simple模式（即最简单的收发模式）**

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS84LzI4LzE2Y2Q3NTE3OGI3ZTVlMjA?x-oss-process=image/format,png)

1.消息产生消息，将消息放入队列

2.消息的消费者(consumer) 监听 消息队列,如果队列中有消息,就消费掉,消息被拿走后,自动从队列中删除(隐患 消息可能没有被消费者正确处理,已经从队列中消失了,造成消息的丢失，这里可以设置成手动的ack,但如果设置成手动ack，处理完后要及时发送ack消息给队列，否则会造成内存溢出)。

**二.work工作模式(资源的竞争)**

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS84LzI4LzE2Y2Q3NTdiYzA1YzUyMjY?x-oss-process=image/format,png)

1.消息产生者将消息放入队列消费者可以有多个,消费者1,消费者2同时监听同一个队列,消息被消费。C1 C2共同争抢当前的消息队列内容,谁先拿到谁负责消费消息(隐患：高并发情况下,默认会产生某一个消息被多个消费者共同使用,可以设置一个开关(syncronize) 保证一条消息只能被一个消费者使用)。

**三.publish/subscribe发布订阅(共享资源)**

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS84LzI4LzE2Y2Q3Njk2NGMxMGM3NWI?x-oss-process=image/format,png)

1、每个消费者监听自己的队列；

2、生产者将消息发给broker，由交换机将消息转发到绑定此交换机的每个队列，每个绑定交换机的队列都将接收到消息。

**四.routing路由模式**

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS84LzI4LzE2Y2Q3ODFiZWFjOWMxNzQ?x-oss-process=image/format,png)

1.消息生产者将消息发送给交换机按照路由判断,路由是字符串(info) 当前产生的消息携带路由字符(对象的方法),交换机根据路由的key,只能匹配上路由key对应的消息队列,对应的消费者才能消费消息;

2.根据业务功能定义路由字符串

3.从系统的代码逻辑中获取对应的功能字符串,将消息任务扔到对应的队列中。

4.业务场景:error 通知;EXCEPTION;错误通知的功能;传统意义的错误通知;客户通知;利用key路由,可以将程序中的错误封装成消息传入到消息队列中,开发者可以自定义消费者,实时接收错误;

**五.topic 主题模式(路由模式的一种)**

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS84LzI4LzE2Y2Q3ODQ2ZjYxNjk2NjE?x-oss-process=image/format,png)

1.星号井号代表通配符

2.星号代表多个单词,井号代表一个单词

3.路由功能添加模糊匹配

4.消息产生者产生消息,把消息交给交换机

5.交换机根据key的规则模糊匹配到对应的队列,由队列的监听消费者接收消息消费

（在我的理解看来就是routing查询的一种模糊匹配，就类似sql的模糊查询方式）

## 如何保证RabbitMQ消息的顺序性？

拆分多个 queue，每个 queue 一个 consumer，就是多一些 queue 而已，确实是麻烦点；或者就一个 queue 但是对应一个 consumer，然后这个 consumer 内部用内存队列做排队，然后分发给底层不同的 worker 来处理。

## 消息如何分发？

若该队列至少有一个消费者订阅，消息将以循环（round-robin）的方式发送给消费者。每条消息只会分发给一个订阅的消费者（前提是消费者能够正常处理消息并进行确认）。通过路由可实现多消费的功能

## 消息怎么路由？

消息提供方->路由->一至多个队列消息发布到交换器时，消息将拥有一个路由键（routing key），在消息创建时设定。通过队列路由键，可以把队列绑定到交换器上。消息到达交换器后，RabbitMQ 会将消息的路由键与队列的路由键进行匹配（针对不同的交换器有不同的路由规则）；

常用的交换器主要分为一下三种：

fanout：如果交换器收到消息，将会广播到所有绑定的队列上

direct：如果路由键完全匹配，消息就被投递到相应的队列

topic：可以使来自不同源头的消息能够到达同一个队列。 使用 topic 交换器时，可以使用通配符

## 消息基于什么传输？

由于 TCP 连接的创建和销毁开销较大，且并发数受系统资源限制，会造成性能瓶颈。RabbitMQ 使用信道的方式来传输数据。信道是建立在真实的 TCP 连接内的虚拟连接，且每条 TCP 连接上的信道数量没有限制。

## 如何保证消息不被重复消费？或者说，如何保证消息消费时的幂等性？

先说为什么会重复消费：正常情况下，消费者在消费消息的时候，消费完毕后，会发送一个确认消息给消息队列，消息队列就知道该消息被消费了，就会将该消息从消息队列中删除；

但是因为网络传输等等故障，确认信息没有传送到消息队列，导致消息队列不知道自己已经消费过该消息了，再次将消息分发给其他的消费者。

针对以上问题，一个解决思路是：保证消息的唯一性，就算是多次传输，不要让消息的多次消费带来影响；保证消息等幂性；

比如：在写入消息队列的数据做唯一标示，消费消息时，根据唯一标识判断是否消费过；

假设你有个系统，消费一条消息就往数据库里插入一条数据，要是你一个消息重复两次，你不就插入了两条，这数据不就错了？但是你要是消费到第二次的时候，自己判断一下是否已经消费过了，若是就直接扔了，这样不就保留了一条数据，从而保证了数据的正确性。

## 如何确保消息正确地发送至 RabbitMQ？ 如何确保消息接收方消费了消息？

**发送方确认模式**

将信道设置成 confirm 模式（发送方确认模式），则所有在信道上发布的消息都会被指派一个唯一的 ID。

一旦消息被投递到目的队列后，或者消息被写入磁盘后（可持久化的消息），信道会发送一个确认给生产者（包含消息唯一 ID）。

如果 RabbitMQ 发生内部错误从而导致消息丢失，会发送一条 nack（notacknowledged，未确认）消息。

发送方确认模式是异步的，生产者应用程序在等待确认的同时，可以继续发送消息。当确认消息到达生产者应用程序，生产者应用程序的回调方法就会被触发来处理确认消息。

**接收方确认机制**

消费者接收每一条消息后都必须进行确认（消息接收和消息确认是两个不同操作）。只有消费者确认了消息，RabbitMQ 才能安全地把消息从队列中删除。

这里并没有用到超时机制，RabbitMQ 仅通过 Consumer 的连接中断来确认是否需要重新发送消息。也就是说，只要连接不中断，RabbitMQ 给了 Consumer 足够长的时间来处理消息。保证数据的最终一致性；

下面罗列几种特殊情况

- 如果消费者接收到消息，在确认之前断开了连接或取消订阅，RabbitMQ 会认为消息没有被分发，然后重新分发给下一个订阅的消费者。（可能存在消息重复消费的隐患，需要去重）
- 如果消费者接收到消息却没有确认消息，连接也未断开，则 RabbitMQ 认为该消费者繁忙，将不会给该消费者分发更多的消息。

## 如何保证RabbitMQ消息的可靠传输？

消息不可靠的情况可能是消息丢失，劫持等原因；

丢失又分为：生产者丢失消息、消息列表丢失消息、消费者丢失消息；

**生产者丢失消息**：从生产者弄丢数据这个角度来看，RabbitMQ提供transaction和confirm模式来确保生产者不丢消息；

transaction机制就是说：发送消息前，开启事务（channel.txSelect()）,然后发送消息，如果发送过程中出现什么异常，事务就会回滚（channel.txRollback()）,如果发送成功则提交事务（channel.txCommit()）。然而，这种方式有个缺点：吞吐量下降；

confirm模式用的居多：一旦channel进入confirm模式，所有在该信道上发布的消息都将会被指派一个唯一的ID（从1开始），一旦消息被投递到所有匹配的队列之后；

rabbitMQ就会发送一个ACK给生产者（包含消息的唯一ID），这就使得生产者知道消息已经正确到达目的队列了；

如果rabbitMQ没能处理该消息，则会发送一个Nack消息给你，你可以进行重试操作。

**消息队列丢数据**：消息持久化。

处理消息队列丢数据的情况，一般是开启持久化磁盘的配置。

这个持久化配置可以和confirm机制配合使用，你可以在消息持久化磁盘后，再给生产者发送一个Ack信号。

这样，如果消息持久化磁盘之前，rabbitMQ阵亡了，那么生产者收不到Ack信号，生产者会自动重发。

那么如何持久化呢？

这里顺便说一下吧，其实也很容易，就下面两步

1. 将queue的持久化标识durable设置为true,则代表是一个持久的队列
2. 发送消息的时候将deliveryMode=2

这样设置以后，即使rabbitMQ挂了，重启后也能恢复数据

**消费者丢失消息**：消费者丢数据一般是因为采用了自动确认消息模式，改为手动确认消息即可！

消费者在收到消息之后，处理消息之前，会自动回复RabbitMQ已收到消息；

如果这时处理消息失败，就会丢失该消息；

解决方案：处理消息成功后，手动回复确认消息。

## 为什么不应该对所有的 message 都使用持久化机制？

首先，必然导致性能的下降，因为写磁盘比写 RAM 慢的多，message 的吞吐量可能有 10 倍的差距。

其次，message 的持久化机制用在 RabbitMQ 的内置 cluster 方案时会出现“坑爹”问题。矛盾点在于，若 message 设置了 persistent 属性，但 queue 未设置 durable 属性，那么当该 queue 的 owner node 出现异常后，在未重建该 queue 前，发往该 queue 的 message 将被 blackholed ；若 message 设置了 persistent 属性，同时 queue 也设置了 durable 属性，那么当 queue 的 owner node 异常且无法重启的情况下，则该 queue 无法在其他 node 上重建，只能等待其 owner node 重启后，才能恢复该 queue 的使用，而在这段时间内发送给该 queue 的 message 将被 blackholed 。

所以，是否要对 message 进行持久化，需要综合考虑性能需要，以及可能遇到的问题。若想达到 100,000 条/秒以上的消息吞吐量（单 RabbitMQ 服务器），则要么使用其他的方式来确保 message 的可靠 delivery ，要么使用非常快速的存储系统以支持全持久化（例如使用 SSD）。另外一种处理原则是：仅对关键消息作持久化处理（根据业务重要程度），且应该保证关键消息的量不会导致性能瓶颈。

## 如何保证高可用的？RabbitMQ 的集群

RabbitMQ 是比较有代表性的，因为是基于主从（非分布式）做高可用性的，我们就以 RabbitMQ 为例子讲解第一种 MQ 的高可用性怎么实现。RabbitMQ 有三种模式：单机模式、普通集群模式、镜像集群模式。

**单机模式**，就是 Demo 级别的，一般就是你本地启动了玩玩儿的?，没人生产用单机模式

**普通集群模式**，意思就是在多台机器上启动多个 RabbitMQ 实例，每个机器启动一个。你创建的 queue，只会放在一个 RabbitMQ 实例上，但是每个实例都同步 queue 的元数据（元数据可以认为是 queue 的一些配置信息，通过元数据，可以找到 queue 所在实例）。你消费的时候，实际上如果连接到了另外一个实例，那么那个实例会从 queue 所在实例上拉取数据过来。这方案主要是提高吞吐量的，就是说让集群中多个节点来服务某个 queue 的读写操作。

**镜像集群模式**：这种模式，才是所谓的 RabbitMQ 的高可用模式。跟普通集群模式不一样的是，在镜像集群模式下，你创建的 queue，无论元数据还是 queue 里的消息都会存在于多个实例上，就是说，每个 RabbitMQ 节点都有这个 queue 的一个完整镜像，包含 queue 的全部数据的意思。然后每次你写消息到 queue 的时候，都会自动把消息同步到多个实例的 queue 上。RabbitMQ 有很好的管理控制台，就是在后台新增一个策略，这个策略是镜像集群模式的策略，指定的时候是可以要求数据同步到所有节点的，也可以要求同步到指定数量的节点，再次创建 queue 的时候，应用这个策略，就会自动将数据同步到其他的节点上去了。这样的话，好处在于，你任何一个机器宕机了，没事儿，其它机器（节点）还包含了这个 queue 的完整数据，别的 consumer 都可以到其它节点上去消费数据。坏处在于，第一，这个性能开销也太大了吧，消息需要同步到所有机器上，导致网络带宽压力和消耗很重！RabbitMQ 一个 queue 的数据都是放在一个节点里的，镜像集群下，也是每个节点都放这个 queue 的完整数据。

## 如何解决消息队列的延时以及过期失效问题？消息队列满了以后该怎么处理？有几百万消息持续积压几小时，说说怎么解决？

消息积压处理办法：临时紧急扩容：

先修复 consumer 的问题，确保其恢复消费速度，然后将现有 cnosumer 都停掉。
新建一个 topic，partition 是原来的 10 倍，临时建立好原先 10 倍的 queue 数量。
然后写一个临时的分发数据的 consumer 程序，这个程序部署上去消费积压的数据，消费之后不做耗时的处理，直接均匀轮询写入临时建立好的 10 倍数量的 queue。
接着临时征用 10 倍的机器来部署 consumer，每一批 consumer 消费一个临时 queue 的数据。这种做法相当于是临时将 queue 资源和 consumer 资源扩大 10 倍，以正常的 10 倍速度来消费数据。
等快速消费完积压数据之后，得恢复原先部署的架构，重新用原先的 consumer 机器来消费消息。
MQ中消息失效：假设你用的是 RabbitMQ，RabbtiMQ 是可以设置过期时间的，也就是 TTL。如果消息在 queue 中积压超过一定的时间就会被 RabbitMQ 给清理掉，这个数据就没了。那这就是第二个坑了。这就不是说数据会大量积压在 mq 里，而是大量的数据会直接搞丢。我们可以采取一个方案，就是批量重导，这个我们之前线上也有类似的场景干过。就是大量积压的时候，我们当时就直接丢弃数据了，然后等过了高峰期以后，比如大家一起喝咖啡熬夜到晚上12点以后，用户都睡觉了。这个时候我们就开始写程序，将丢失的那批数据，写个临时程序，一点一点的查出来，然后重新灌入 mq 里面去，把白天丢的数据给他补回来。也只能是这样了。假设 1 万个订单积压在 mq 里面，没有处理，其中 1000 个订单都丢了，你只能手动写程序把那 1000 个订单给查出来，手动发到 mq 里去再补一次。

mq消息队列块满了：如果消息积压在 mq 里，你很长时间都没有处理掉，此时导致 mq 都快写满了，咋办？这个还有别的办法吗？没有，谁让你第一个方案执行的太慢了，你临时写程序，接入数据来消费，消费一个丢弃一个，都不要了，快速消费掉所有的消息。然后走第二个方案，到了晚上再补数据吧。

## 设计MQ思路

比如说这个消息队列系统，我们从以下几个角度来考虑一下：

首先这个 mq 得支持可伸缩性吧，就是需要的时候快速扩容，就可以增加吞吐量和容量，那怎么搞？设计个分布式的系统呗，参照一下 kafka 的设计理念，broker -> topic -> partition，每个 partition 放一个机器，就存一部分数据。如果现在资源不够了，简单啊，给 topic 增加 partition，然后做数据迁移，增加机器，不就可以存放更多数据，提供更高的吞吐量了？

其次你得考虑一下这个 mq 的数据要不要落地磁盘吧？那肯定要了，落磁盘才能保证别进程挂了数据就丢了。那落磁盘的时候怎么落啊？顺序写，这样就没有磁盘随机读写的寻址开销，磁盘顺序读写的性能是很高的，这就是 kafka 的思路。

其次你考虑一下你的 mq 的可用性啊？这个事儿，具体参考之前可用性那个环节讲解的 kafka 的高可用保障机制。多副本 -> leader & follower -> broker 挂了重新选举 leader 即可对外服务。

能不能支持数据 0 丢失啊？可以的，参考我们之前说的那个 kafka 数据零丢失方案。



# RabbitMQ集群原理与搭建

https://www.jianshu.com/p/6376936845ff

本篇主要介绍RabbitMQ集群方案的原理，如何搭建具备负载均衡能力的中小规模RabbitMQ集群，并最后给出生产环境构建一个能够具备高可用、高可靠和高吞吐量的中小规模RabbitMQ集群设计方案。

## 一、RabbitMQ集群方案的原理

RabbitMQ这款消息队列中间件产品本身是基于Erlang编写，Erlang语言天生具备分布式特性（通过同步Erlang集群各节点的magic cookie来实现）。因此，RabbitMQ天然支持Clustering。这使得RabbitMQ本身不需要像ActiveMQ、Kafka那样通过ZooKeeper分别来实现HA方案和保存集群的元数据。集群是保证可靠性的一种方式，同时可以通过水平扩展以达到增加消息吞吐量能力的目的。 下面先来看下RabbitMQ集群的整体方案：

![img](https://upload-images.jianshu.io/upload_images/4325076-c555bfd584d9a323.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/551/format/webp)

RabbitMQ集群的方案1.jpg


 上面图中采用三个节点组成了一个RabbitMQ的集群，**Exchange A**（交换器，对于RabbitMQ基础概念不太明白的童鞋可以看下基础概念）**的元数据信息在所有节点上是一致的，而Queue（存放消息的队列）的完整数据则只会存在于它所创建的那个节点上。**，其他节点只知道这个queue的metadata信息和一个指向queue的owner node的指针。



## （1）RabbitMQ集群元数据的同步

RabbitMQ集群会始终同步四种类型的内部元数据（类似索引）：
 a.队列元数据：队列名称和它的属性；
 b.交换器元数据：交换器名称、类型和属性；
 c.绑定元数据：一张简单的表格展示了如何将消息路由到队列；
 d.vhost元数据：为vhost内的队列、交换器和绑定提供命名空间和安全属性；
 因此，当用户访问其中任何一个RabbitMQ节点时，通过rabbitmqctl查询到的queue／user／exchange/vhost等信息都是相同的。

## （2）为何RabbitMQ集群仅采用元数据同步的方式

我想肯定有不少同学会问，想要实现HA方案，那将RabbitMQ集群中的所有Queue的完整数据在所有节点上都保存一份不就可以了么？（**可以类似MySQL的主主模式嘛**）这样子，任何一个节点出现故障或者宕机不可用时，那么使用者的客户端只要能连接至其他节点能够照常完成消息的发布和订阅嘛。
 我想RabbitMQ的作者这么设计主要还是基于集群本身的性能和存储空间上来考虑。第一，存储空间，如果每个集群节点都拥有所有Queue的完全数据拷贝，那么每个节点的存储空间会非常大，集群的消息积压能力会非常弱（**无法通过集群节点的扩容提高消息积压能力**）；第二，性能，消息的发布者需要将消息复制到每一个集群节点，对于持久化消息，网络和磁盘同步复制的开销都会明显增加。

## （3）RabbitMQ集群发送/订阅消息的基本原理

RabbitMQ集群的工作原理图如下：



![img](https:////upload-images.jianshu.io/upload_images/4325076-b93fd8387a8c045a.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/539/format/webp)

RabbitMQ集群工作原理.jpg

### 场景1、客户端直接连接队列所在节点

如果有一个消息生产者或者消息消费者通过amqp-client的客户端连接至节点1进行消息的发布或者订阅，那么此时的集群中的消息收发只与节点1相关，这个没有任何问题；如果客户端相连的是节点2或者节点3（队列1数据不在该节点上），那么情况又会是怎么样呢？

### 场景2、客户端连接的是非队列数据所在节点

如果消息生产者所连接的是节点2或者节点3，此时队列1的完整数据不在该两个节点上，那么在发送消息过程中这两个节点主要起了一个路由转发作用，根据这两个节点上的元数据（也就是上文提到的：**指向queue的owner node的指针**）转发至节点1上，最终发送的消息还是会存储至节点1的队列1上。
 同样，如果消息消费者所连接的节点2或者节点3，那这两个节点也会作为路由节点起到转发作用，将会从节点1的队列1中拉取消息进行消费。

## 二、RabbitMQ集群的搭建

## （1）搭建RabbitMQ集群所需要安装的组件

在搭建RabbitMQ集群之前有必要在每台虚拟机上安装如下的组件包，分别如下：
 a.**Jdk 1.8**
 b.**Erlang运行时环境**，这里用的是otp_src_19.3.tar.gz (200MB+)
 c.**RabbitMq的Server组件**，这里用的rabbitmq-server-generic-unix-3.6.10.tar.gz
 关于如何安装上述三个组件的具体步骤，已经有不少博文对此进行了非常详细的描述，那么本文就不再赘述了。有需要的同学可以具体参考这些步骤来完成安装。

## （2）搭建10节点组成的RabbitMQ集群

该节中主要展示的是集群搭建，需要确保每台机器上正确安装了上述三种组件，并且每台虚拟机上的RabbitMQ的实例能够正常启动起来。
 a.编辑每台RabbitMQ的cookie文件，以确保各个节点的cookie文件使用的是同一个值,可以scp其中一台机器上的cookie至其他各个节点，cookie的默认路径为/var/lib/rabbitmq/.erlang.cookie或者$HOME/.erlang.cookie，节点之间通过cookie确定相互是否可通信。
 b.配置各节点的hosts文件( vim /etc/hosts)



```css
xxx.xxx.xxx.xxx rmq-broker-test-1
xxx.xxx.xxx.xxx rmq-broker-test-2
xxx.xxx.xxx.xxx rmq-broker-test-3
......
xxx.xxx.xxx.xxx rmq-broker-test-10
```

c.逐个节点启动RabbitMQ服务



```undefined
rabbitmq-server -detached
```

d.查看各个节点和集群的工作运行状态



```undefined
rabbitmqctl status, rabbitmqctl cluster_status
```

e.以rmq-broker-test-1为主节点，在rmq-broker-test-2上：



```css
rabbitmqctl stop_app 
rabbitmqctl reset 
rabbitmqctl join_cluster rabbit@rmq-broker-test-2 
rabbitmqctl start_app 
```

在其余的节点上的操作步骤与rmq-broker-test-2虚拟机上的一样。
 d.在RabbitMQ集群中的节点只有两种类型：内存节点/磁盘节点，单节点系统只运行磁盘类型的节点。而在集群中，可以选择配置部分节点为内存节点。
 内存节点将所有的队列，交换器，绑定关系，用户，权限，和vhost的元数据信息保存在内存中。而磁盘节点将这些信息保存在磁盘中，但是内存节点的性能更高，为了保证集群的高可用性，必须保证集群中有两个以上的磁盘节点，来保证当有一个磁盘节点崩溃了，集群还能对外提供访问服务。在上面的操作中，可以通过如下的方式，设置新加入的节点为内存节点还是磁盘节点：



```css
#加入时候设置节点为内存节点（默认加入的为磁盘节点）
[root@mq-testvm1 ~]# rabbitmqctl join_cluster rabbit@rmq-broker-test-1 --ram
```



```csharp
#也通过下面方式修改的节点的类型
[root@mq-testvm1 ~]# rabbitmqctl changeclusternode_type disc | ram
```

e.最后可以通过“rabbitmqctl cluster_status”的方式来查看集群的状态，上面搭建的10个节点的RabbitMQ集群状态（3个节点为磁盘即诶但，7个节点为内存节点）如下：



```bash
Cluster status of node 'rabbit@rmq-broker-test-1'
[{nodes,[{disc,['rabbit@rmq-broker-test-1','rabbit@rmq-broker-test-2',
                'rabbit@rmq-broker-test-3']},
         {ram,['rabbit@rmq-broker-test-9','rabbit@rmq-broker-test-8',
               'rabbit@rmq-broker-test-7','rabbit@rmq-broker-test-6',
               'rabbit@rmq-broker-test-5','rabbit@rmq-broker-test-4',
               'rabbit@rmq-broker-test-10']}]},
 {running_nodes,['rabbit@rmq-broker-test-10','rabbit@rmq-broker-test-5',
                 'rabbit@rmq-broker-test-9','rabbit@rmq-broker-test-2',
                 'rabbit@rmq-broker-test-8','rabbit@rmq-broker-test-7',
                 'rabbit@rmq-broker-test-6','rabbit@rmq-broker-test-3',
                 'rabbit@rmq-broker-test-4','rabbit@rmq-broker-test-1']},
 {cluster_name,<<"rabbit@mq-testvm1">>},
 {partitions,[]},
 {alarms,[{'rabbit@rmq-broker-test-10',[]},
          {'rabbit@rmq-broker-test-5',[]},
          {'rabbit@rmq-broker-test-9',[]},
          {'rabbit@rmq-broker-test-2',[]},
          {'rabbit@rmq-broker-test-8',[]},
          {'rabbit@rmq-broker-test-7',[]},
          {'rabbit@rmq-broker-test-6',[]},
          {'rabbit@rmq-broker-test-3',[]},
          {'rabbit@rmq-broker-test-4',[]},
          {'rabbit@rmq-broker-test-1',[]}]}]
```

## （3）配置HAProxy

HAProxy提供高可用性、负载均衡以及基于TCP和HTTP应用的代理，支持虚拟主机，它是免费、快速并且可靠的一种解决方案。根据官方数据，其最高极限支持10G的并发。HAProxy支持从4层至7层的网络交换，即覆盖所有的TCP协议。就是说，Haproxy 甚至还支持 Mysql 的均衡负载。为了实现RabbitMQ集群的软负载均衡，这里可以选择HAProxy。
 关于HAProxy如何安装的文章之前也有很多同学写过，这里就不再赘述了，有需要的同学可以参考下网上的做法。这里主要说下安装完HAProxy组件后的具体配置。
 HAProxy使用单一配置文件来定义所有属性，包括从前端IP到后端服务器。下面展示了用于7个RabbitMQ节点组成集群的负载均衡配置（另外3个磁盘节点用于保存集群的配置和元数据，不做负载）。同时，HAProxy运行在另外一台机器上。HAProxy的具体配置如下：



```bash
#全局配置
global
        #日志输出配置，所有日志都记录在本机，通过local0输出
        log 127.0.0.1 local0 info
        #最大连接数
        maxconn 4096
        #改变当前的工作目录
        chroot /apps/svr/haproxy
        #以指定的UID运行haproxy进程
        uid 99
        #以指定的GID运行haproxy进程
        gid 99
        #以守护进程方式运行haproxy #debug #quiet
        daemon
        #debug
        #当前进程pid文件
        pidfile /apps/svr/haproxy/haproxy.pid

#默认配置
defaults
        #应用全局的日志配置
        log global
        #默认的模式mode{tcp|http|health}
        #tcp是4层，http是7层，health只返回OK
        mode tcp
        #日志类别tcplog
        option tcplog
        #不记录健康检查日志信息
        option dontlognull
        #3次失败则认为服务不可用
        retries 3
        #每个进程可用的最大连接数
        maxconn 2000
        #连接超时
        timeout connect 5s
        #客户端超时
        timeout client 120s
        #服务端超时
        timeout server 120s

        maxconn 2000
        #连接超时
        timeout connect 5s
        #客户端超时
        timeout client 120s
        #服务端超时
        timeout server 120s

#绑定配置
listen rabbitmq_cluster
        bind 0.0.0.0:5672
        #配置TCP模式
        mode tcp
        #加权轮询
        balance roundrobin
        #RabbitMQ集群节点配置,其中ip1~ip7为RabbitMQ集群节点ip地址
        server rmq_node1 ip1:5672 check inter 5000 rise 2 fall 3 weight 1
        server rmq_node2 ip2:5672 check inter 5000 rise 2 fall 3 weight 1
        server rmq_node3 ip3:5672 check inter 5000 rise 2 fall 3 weight 1
        server rmq_node4 ip4:5672 check inter 5000 rise 2 fall 3 weight 1
        server rmq_node5 ip5:5672 check inter 5000 rise 2 fall 3 weight 1
        server rmq_node6 ip6:5672 check inter 5000 rise 2 fall 3 weight 1
        server rmq_node7 ip7:5672 check inter 5000 rise 2 fall 3 weight 1

#haproxy监控页面地址
listen monitor
        bind 0.0.0.0:8100
        mode http
        option httplog
        stats enable
        stats uri /stats
        stats refresh 5s
```

在上面的配置中“listen rabbitmq_cluster bind 0.0.0.0：5671”这里定义了客户端连接IP地址和端口号。这里配置的负载均衡算法是roundrobin—加权轮询。与配置RabbitMQ集群负载均衡最为相关的是“ server rmq_node1 ip1:5672 check inter 5000 rise 2 fall 3 weight 1”这种，它标识并且定义了后端RabbitMQ的服务。主要含义如下:
 (a)“server <name>”部分：定义HAProxy内RabbitMQ服务的标识；
 (b)“ip1:5672”部分：标识了后端RabbitMQ的服务地址；
 (c)“check inter <value>”部分：表示每隔多少毫秒检查RabbitMQ服务是否可用；
 (d)“rise <value>”部分：表示RabbitMQ服务在发生故障之后，需要多少次健康检查才能被再次确认可用；
 (e)“fall <value>”部分：表示需要经历多少次失败的健康检查之后，HAProxy才会停止使用此RabbitMQ服务。



```csharp
#启用HAProxy服务
[root@mq-testvm12 conf]# haproxy -f haproxy.cfg
```

启动后，即可看到如下的HAproxy的界面图：



![img](https:////upload-images.jianshu.io/upload_images/4325076-4b3d00f2cffc7b0f.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

RabbitMQ集群Haproxy部署的UI界面.jpg

## （4）RabbitMQ的集群架构设计图

经过上面的RabbitMQ10个节点集群搭建和HAProxy软弹性负载均衡配置后即可组建一个中小规模的RabbitMQ集群了，然而为了能够在实际的生产环境使用还需要根据实际的业务需求对集群中的各个实例进行一些性能参数指标的监控，从性能、吞吐量和消息堆积能力等角度考虑，可以选择Kafka来作为RabbitMQ集群的监控队列使用。因此，这里先给出了一个中小规模RabbitMQ集群架构设计图：



![img](https:////upload-images.jianshu.io/upload_images/4325076-5f863f419723664c.png?imageMogr2/auto-orient/strip|imageView2/2/w/966/format/webp)

RabbitMQ小规模集群的架构设计图(附加了监控部分).png



对于消息的生产和消费者可以通过HAProxy的软负载将请求分发至RabbitMQ集群中的Node1～Node7节点，其中Node8～Node10的三个节点作为磁盘节点保存集群元数据和配置信息。鉴于篇幅原因这里就不在对监控部分进行详细的描述的，会在后续篇幅中对如何使用RabbitMQ的HTTP API接口进行监控数据统计进行详细阐述。

## 三、总结

本文主要详细介绍了RabbitMQ集群的工作原理和如何搭建一个具备负载均衡能力的中小规模RabbitMQ集群的方法，并最后给出了RabbitMQ集群的架构设计图。限于笔者的才疏学浅，对本文内容可能还有理解不到位的地方，如有阐述不合理之处还望留言一起探讨。



# Rabbitmq镜像集群部署

本篇接上面这篇安装文章的步骤，来搭建rabbitmq集群：

地址 ： https://blog.csdn.net/winy_lm/article/details/81070494

 

环境：两台服务器作为两个节点，把node-003加入node-002

192.168.95.129 node-002

192.168.95.130 node-003

### 一、集群搭建

\1. 配置环境host，两个节点的host都需要包含每个节点的信息，信息要一致。下面是在node-002中的操作。node-003中也需要做相同一样的操作(把hostname改为 node-003,多个节点同理)

a. [root@node-002 sbin]# vi /etc/hosts

192.168.95.129 node-002
192.168.95.130 node-003

b.  保存a后：

[root@node-002 sbin]# vi /etc/hostname

node-002

c. 保存b

[root@node-002 sbin]# vi /etc/sysconfig/network

NETWORKING=yes
HOSTNAME=node-002

d. 保存c

[root@node-002 sbin]# service network restart

e. 重启服务器使host修改生效， 查询hostname是否已修改

[root@node-002 sbin]# reboot

[root@node-002 sbin]# hostname
node-002
 

\2. 因为rabbitmq集群是基于erlang同步的，所以先配置使各个节点 中 .erlang.cookie文件一致。

[root@node-002 sbin]# find / -name .erlang.cookie
/root/.erlang.cookie

[root@node-002 sbin]# scp /root/.erlang.cookie root@node-003:/root

\3. [root@node-002 sbin]# cd /opt/rabbitmq_server-3.7.7/sbin

[root@node-002 sbin]# ./rabbitmq-server -detached

[root@node-002 sbin]# ps aux|grep rabbitmq

\4. 切换到node-003服务器, 确定下两台服务器互相能否ping通，ping node-002

![img](https://img-blog.csdn.net/20180720152034523?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dpbnlfbG0=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

[root@node-003 sbin]# cd /opt/rabbitmq_server-3.7.7/sbin

[root@node-003 sbin]# ./rabbitmq-server -detached
[root@node-003 sbin]# ps aux|grep rabbitmq
[root@node-003 sbin]# ./rabbitmqctl stop_app

//默认是磁盘节点，如果是内存节点的话，需要加--ram参数【./rabbitmqctl join_cluster --ram rabbit@node-002】

[root@node-003 sbin]# ./rabbitmqctl join_cluster rabbit@node-002  

![img](https://img-blog.csdn.net/20180720111028488?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dpbnlfbG0=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


[root@node-003 ~]# rabbitmqctl start_app

[root@node-003 sbin]# ./rabbitmqctl cluster_status
![img](https://img-blog.csdn.net/20180720111340905?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dpbnlfbG0=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

\5. 因为两台服务器host已经改掉了，已经部署localhost了，所以需要加上对应的用户，先前的username用户已经登录不上

控制台了。添加一个admin用户如下：【node-003也可以直接用这个用户访问】

[root@node-002 sbin]#  ./rabbitmqctl -n rabbit@node-002 add_user admin 123456
Adding user "admin" ...
[root@node-002 sbin]# ./rabbitmqctl -n rabbit@node-002 set_user_tags admin administrator
Setting tags for user "admin" to [administrator] ...
[root@node-002 sbin]#  ./rabbitmqctl -n rabbit@node-002 set_permissions -p / admin '.*' '.*' '.*'
Setting permissions for user "admin" in vhost "/" ...
[root@node-002 sbin]# ./rabbitmq-plugins enable rabbitmq_management

\6. 通过 [http://192.168.95.129:15672](http://192.168.95.129:15672/#/)  ，输入 admin 123456 即可登录。

![img](https://img-blog.csdn.net/20180720112032224?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dpbnlfbG0=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 二、 集群模式，rabbitmq集群有2种模式(上面的就是普通模式)，详细如下：

**普通模式：**默认模式，以两个节点（rabbit01、rabbit02）为例来进行说明。对于Queue来说，消息实体只存在于其中一个节点rabbit01（或者rabbit02），rabbit01和rabbit02两个节点仅有相同的元数据，即队列的结构。当消息进入rabbit01节点的Queue后，consumer从rabbit02节点消费时，RabbitMQ会临时在rabbit01、rabbit02间进行消息传输，把A中的消息实体取出并经过B发送给consumer。所以consumer应尽量连接每一个节点，从中取消息。即对于同一个逻辑队列，要在多个节点建立物理Queue。否则无论consumer连rabbit01或rabbit02，出口总在rabbit01，会产生瓶颈。当rabbit01节点故障后，rabbit02节点无法取到rabbit01节点中还未消费的消息实体。如果做了消息持久化，那么得等rabbit01节点恢复，然后才可被消费；如果没有持久化的话，就会产生消息丢失的现象。

**镜像模式:** 把需要的队列做成镜像队列，存在与多个节点属于RabbitMQ的HA方案。该模式解决了普通模式中的问题，其实质和普通模式不同之处在于，消息实体会主动在镜像节点间同步，而不是在客户端取数据时临时拉取。该模式带来的副作用也很明显，除了降低系统性能外，如果镜像队列数量过多，加之大量的消息进入，集群内部的网络带宽将会被这种同步通讯大大消耗掉。所以在对可靠性要求较高的场合中适用。

### 二、 镜像模式

首先镜像模式要依赖policy模块，这个模块是做什么用的呢？

policy中文来说是政策，策略的意思，那么他就是要设置，那些Exchanges或者queue的数据需要复制，同步，如何复制同步？对就是做这些的。

\1. 直接用管控台添加策略如下：

![img](https://img-blog.csdn.net/20180720155801930?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dpbnlfbG0=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

参数解释：

name: 为策略名称 ，我命名为 winy-ha

Pattern: 为匹配符，只有一个^代表匹配所有，^zlh为匹配名称为zlh的exchanges或者queue,这里我设置为匹配全部

Apply to：使用对象

Priority：配置了多个策略时候的优先级，值越大，优先级越高。(单个策略配置意义不大)

注意：没有指定优先级的消息会将优先级以0对待。 对于超过优先级队列所定最大优先级的消息，优先级以最大优先级对待

Definition： 为匹配类型，他分为3种模式：all-所有（所有的queue），exctly-部分（需配置ha-params参数，此参数为int类型比如3，众多集群中的随机3台机器），nodes-指定（需配置ha-params参数，此参数为数组类型比如["3rabbit@F","rabbit@G"]这样指定为F与G这2台机器。）

\2. 测试一下上面策略是否生效

控制台queue tab页面添加一个测试用的queue , **install-test-queue**

![img](https://img-blog.csdn.net/20180720161219118?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dpbnlfbG0=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

 

### 三、负载均衡 HAProxy

略

 