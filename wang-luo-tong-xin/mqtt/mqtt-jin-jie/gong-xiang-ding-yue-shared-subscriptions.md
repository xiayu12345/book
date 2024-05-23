---
layout:
  title:
    visible: false
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 共享订阅（Shared Subscriptions）

## 什么是共享订阅 <a href="#shen-me-shi-gong-xiang-ding-yue" id="shen-me-shi-gong-xiang-ding-yue"></a>

在普通的订阅中，我们每发布一条消息，所有匹配的订阅端都会收到该消息的副本。当某个订阅端的消费速度无法跟上消息的生产速度时，我们没有办法将其中一部分消息分流到其他订阅端中来分担压力。这使订阅端容易成为整个消息系统的性能瓶颈。

所以 MQTT 5.0 引入了共享订阅特性，它使得 MQTT 服务端可以在使用特定订阅的客户端之间均衡地分配消息负载。这表示，当我们有两个客户端共享一个订阅时，那么每个匹配该订阅的消息都只会有一个副本投递给其中一个客户端。

共享订阅不仅为消费端带来了极佳的水平扩展能力，使我们可以应对更高的吞吐量，还为其带来了高可用性，即使共享订阅组中的一个客户端断开连接或发生故障，其他客户端仍然可以继续处理消息，在必要时还可以接管原先流向该客户端的消息流。

## 共享订阅如何工作 <a href="#gong-xiang-ding-yue-ru-he-gong-zuo" id="gong-xiang-ding-yue-ru-he-gong-zuo"></a>

使用共享订阅，我们不需要对客户端的底层代码进行任何改动，只需要在订阅时使用遵循以下命名规范的主题即可：

```
$share/{Share Name}/{Topic Filter}
```

其中 `$share` 是一个固定的前缀，以便服务端知道这是一个共享订阅主题。`{Topic Filter}` 则是我们实际想要订阅的主题。

中间的 `{Share Name}` 是一个由客户端指定的字符串，表示当前共享订阅使用的共享名。很多时候，`{Share Name}` 这个字段也会被叫作 Group Name 或者 Group ID，这确实会更容易理解一些。

需要共享同一个订阅的一组订阅会话，必须使用相同的共享名。所以 `$share/consumer1/sport/#` 和 `$share/consumer2/sport/#` 属于不同的共享订阅组。当一个消息同时与多个共享订阅组使用的过滤器匹配时，服务端会在每个匹配的共享订阅组中选择一个会话发送该消息的副本。这在某个主题的消息有多个不同类型的消费者时非常有用。

但是，两个订阅的共享名 `{Share Name}` 相同，并不表示它们一定是相同的共享订阅。只有 `{Share Name}/{Topic Filter}` 才能唯一地标识一个共享订阅组，下面这些订阅主题均属于不同的共享订阅组：

* `$share/consumer1/sport/tennis/+`
* `$share/consumer2/sport/tennis/+`
* `$share/consumer1/sport/#`
* `$share/comsumer1/finance/#`

共享订阅和普通订阅互不影响，当某个消息同时与共享订阅和普通订阅匹配时，服务端会向每个匹配的普通订阅的客户端发送该消息的副本，同时向每个匹配的共享订阅组中的其中一个会话发送该消息的副本。如果这些订阅来自同一个客户端，那么这个客户端可能会收到该消息的多个副本。

## 共享订阅的负载均衡策略 <a href="#gong-xiang-ding-yue-de-fu-zai-jun-heng-ce-lve" id="gong-xiang-ding-yue-de-fu-zai-jun-heng-ce-lve"></a>

共享订阅的核心在于服务端如何在客户端之间分配消息负载。比较常见的负载均衡策略有以下几种：

* 随机（Random），在共享订阅组内随机选择一个会话发送消息。
* 轮询（Round Robin），在共享订阅组内按顺序选择一个会话发送消息，循环往复。
* 哈希（Hash），基于某个字段的哈希结果来分配。
* 粘性（Sticky），在共享订阅组内随机选择一个会话发送消息，此后保持这一选择，直到该会话结束再重复这一过程。
* 本地优先（Local），随机选择，但优先选择与消息的发布者处于同一节点的会话，如果不存在这样的会话，则退化为普通的随机策略。

**随机** 和 **轮询** 这两种策略实现的均衡效果较为接近，所以它们在应用场景上的区别不大，但 **随机** 策略实际的均衡效果通常还会受到服务端采用的随机算法的影响。

在实际应用中，消息之间可能存在关联，比如属于同一张图片的多个分片显然不适合分发给多个订阅者。在这种情况下，我们就需要基于 Client ID 或者 Topic 的 **哈希** 策略来选择会话。这可以保证来自同一个发布端或者主题的消息始终由共享订阅组中的同一个会话处理。当然，**粘性** 策略也有相同的效果。

**本地优先** 策略比 **随机** 策略更合适在集群中使用，优先选择本地订阅端的策略可以有效降低消息的延迟。不过使用这一策略的前提是我们可以确保发布端和订阅端比较均衡地分布在每个节点上，以免不同订阅端上的消息负载差别过大。

## 共享订阅使用场景 <a href="#gong-xiang-ding-yue-shi-yong-chang-jing" id="gong-xiang-ding-yue-shi-yong-chang-jing"></a>

以下是几个典型的共享订阅的使用场景：

* 后端消费能力与消息的生产能力不匹配时，我们可以借助共享订阅让更多的客户端一起分担负载。
* 系统需要保证高可用性，特别是在大量消息流入的关键业务上，我们可以通过共享订阅来避免单点故障。
* 消息的流入量可能会在未来快速增长，需要消费端能够水平扩展，我们可以通过共享订阅来提供高扩展性。

## 共享订阅使用建议 <a href="#gong-xiang-ding-yue-shi-yong-jian-yi" id="gong-xiang-ding-yue-shi-yong-jian-yi"></a>

### 在共享订阅组内使用相同的 QoS <a href="#zai-gong-xiang-ding-yue-zu-nei-shi-yong-xiang-tong-de-qos" id="zai-gong-xiang-ding-yue-zu-nei-shi-yong-xiang-tong-de-qos"></a>

MQTT 虽然允许一个共享订阅组内的会话使用不同的 QoS 等级，但这可能会使消息在投递给同一个组内的不同会话时存在不同的质量保证。相应地，在出现一些问题的时候，我们的调试也将变得困难重重。所以我们最好在共享订阅组内使用相同的 QoS。

### 合理地设置会话过期时间 <a href="#he-li-di-she-zhi-hui-hua-guo-qi-shi-jian" id="he-li-di-she-zhi-hui-hua-guo-qi-shi-jian"></a>

持久会话与共享订阅一起使用是非常常见的。但需要注意，即便共享订阅组中的某个客户端离线，但只要它的会话与订阅仍在存在时，MQTT 服务端仍然会向此会话分发消息。考虑到客户端可能因为故障等原因长时间离线，如果会话的过期时间过长，那么这段时间内将有很多消息因为被投递给离线客户端而无法得到处理。

一个更好的选择可能是，一旦订阅端离线，即便会话没有过期，MQTT 服务端在分配消息负载时也不再考虑这个订阅端。虽然与普通订阅的行为不同，但这是 MQTT 协议允许的。

## 演示 <a href="#yan-shi" id="yan-shi"></a>

为了更好地展示共享订阅的效果，这里我们直接使用 MQTTX CLI 这个 MQTT 命令行客户端工具来进行演示。

启动三个终端窗口，使用以下命令创建三个连接至 免费的公共 MQTT 服务器 的客户端并分别订阅主题 `$share/consumer1/sport/+`、`$share/consumer1/sport/+` 和 `$share/consumer2/sport/+`：

```
mqttx sub -h 'broker.emqx.io' --topic '$share/consumer1/sport/+'
```

然后启动一个新的终端窗口，使用以下命令向主题 `sport/tennis` 发布 6 条消息。这里我们使用了 `--multiline` 选项，以每次键入回车的方式发送多条消息：

```
mqttx pub -h 'broker.emqx.io' --topic sport/tennis -s --stdin --multiline
```

EMQX 默认为共享订阅使用的负载均衡策略为 Round Robin，所以我们将看到 `consumer1` 组内的两个订阅端交替收到我们发布的消息，而共享订阅组 `consumer2` 中只有一个订阅端，所以它将收到所有消息：

![05demo.png](https://assets.emqx.com/images/878d6ebddf34b8cfa5144d5b8577e524.png?imageMogr2/thumbnail/1520x)

这只是一个非常简单的示例，你还可以尝试随时加入或退出共享订阅组，观察 EMQX 是否及时地按最新的订阅分配负载，或者自行安装 EMQX 然后观察不同负载均衡策略的表现。
