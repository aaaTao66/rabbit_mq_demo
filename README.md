## 订阅模型分类:
1、一个生产者多个消费者
2、每个消费者都有一个自己的队列
3、生产者没有将消息直接发送给队列，而是发送给exchange(交换机、转发器)
4、每个队列都需要绑定到交换机上
5、生产者发送的消息，经过交换机到达队列，实现一个消息被多个消费者消费
例子：注册->发邮件、发短信

X（Exchanges）：交换机一方面：接收生产者发送的消息。另一方面：知道如何处理消息，例如递交给某个特别队列、递交给所有队列、或是将消息丢弃。到底如何操作，取决于Exchange的类型。

Exchange类型有以下几种：

Fanout：广播，将消息交给所有绑定到交换机的队列

Direct：定向，把消息交给符合指定routing key 的队列

Topic：通配符，把消息交给符合routing pattern（路由模式） 的队列

Header：header模式与routing不同的地方在于，header模式取消routingkey，使用header中的 key/value（键值对）匹配队列。

Header模式参考这篇文章https://blog.csdn.net/zhu_tianwei/article/details/40923131

Exchange（交换机）只负责转发消息，不具备存储消息的能力，因此如果没有任何队列与Exchange绑定，或者没有符合路由规则的队列，那么消息会丢失！
// 声明exchange，指定类型为fanout
channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
思考：
1、publish/subscribe与work queues有什么区别。

### 区别：

1）work queues不用定义交换机，而publish/subscribe需要定义交换机。

2）publish/subscribe的生产方是面向交换机发送消息，work queues的生产方是面向队列发送消息(底层使用默认交换机)。

3）publish/subscribe需要设置队列和交换机的绑定，work queues不需要设置，实际上work queues会将队列绑定到默认的交换机 。

相同点：

所以两者实现的发布/订阅的效果是一样的，多个消费端监听同一个队列不会重复消费消息。

2、实际工作用 publish/subscribe还是work queues。

建议使用 publish/subscribe，发布订阅模式比工作队列模式更强大（也可以做到同一队列竞争），并且发布订阅模式可以指定自己专用的交换机。

---

## Routing 路由模型（交换机类型：direct）
P：生产者，向Exchange发送消息，发送消息时，会指定一个routing key。

X：Exchange（交换机），接收生产者的消息，然后把消息递交给 与routing key完全匹配的队列

C1：消费者，其所在队列指定了需要routing key 为 error 的消息

C2：消费者，其所在队列指定了需要routing key 为 info、error、warning 的消息
```java
// 生产者
// 发送消息，并且指定routing key 为：sms，只有短信服务能接收到消息
channel.basicPublish(EXCHANGE_NAME, "sms", null, message.getBytes());

// 消费者1
 // 声明队列
channel.queueDeclare(QUEUE_NAME, false, false, false, null);

// 绑定队列到交换机，同时指定需要订阅的routing key。可以指定多个
channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "sms");//指定接收发送方指定routing key为sms的消息

// 消费者2
// 绑定队列到交换机，同时指定需要订阅的routing key。可以指定多个
channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "email");//指定接收发送方指定routing key为email的消息

```
发送sms的RoutingKey,运行后发现只有消费者1,消费到了消息

---
## Topics 通配符模式（交换机类型：topics）

 每个消费者监听自己的队列，并且设置带统配符的routingkey,生产者将消息发给broker，由交换机根据routingkey来转发消息到指定的队列。

Routingkey一般都是有一个或者多个单词组成，多个单词之间以“.”分割，例如：inform.sms

通配符规则：

#：匹配一个或多个词

*：匹配不多不少恰好1个词

举例：

audit.#：能够匹配audit.irs.corporate 或者 audit.irs

audit.*：只能匹配audit.irs