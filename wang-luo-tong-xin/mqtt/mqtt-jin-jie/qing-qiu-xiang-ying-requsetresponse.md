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

# 请求响应(Requset/Response)

## MQTT 5.0 之前的“请求 / 响应” <a href="#mqtt50-zhi-qian-de-qing-qiu-xiang-ying" id="mqtt50-zhi-qian-de-qing-qiu-xiang-ying"></a>

MQTT 的发布订阅机制，使消息的发送者和接收者完全解耦，所以消息可以被异步地传递。但这也带来了一个问题，即便是使用了 QoS 1 或 2 的消息，发布端也只能确保消息到达服务端，而无法知晓订阅端最终是否收到了消息。在执行一些请求或命令时，发布端还会想要知道对端的执行结果。最直接的做法是，让订阅端为请求返回响应。

在 MQTT 中，这不难实现，只需要通信双方提前协商好请求主题和响应主题，然后订阅端在收到请求后向响应主题返回响应即可。这也是 MQTT 5.0 之前的客户端普遍采用的做法。

但在这个方案中，响应主题必须提前确定，无法灵活地变更。当存在多个不同的请求方时，由于只能订阅相同的响应主题，所以所有请求方都会收到响应，并且**它们无法分辨响应是否属于自己**：

![01requestresponsebeforemqtt5.jpg](https://assets.emqx.com/images/67cf464afec7fb78ba1676358bdfd6aa.jpg?imageMogr2/thumbnail/1520x)

{% hint style="info" %}
存在多个请求方时，容易出现响应混乱的问题
{% endhint %}

虽然有不少办法可以避免这一问题，但这也导致各个厂商可能有着完全不同的实现，这使用户在集成不同厂商设备时的难度和工作量极大增加。

为了解决以上问题，MQTT 5.0 引入了响应主题 (Response Topic)、关联数据 (Correlation Data) 和响应信息 (Response Information) 这些属性使 MQTT 中的“请求 / 响应”机制标准化、规范化。

## MQTT 5.0 的“请求 / 响应”如何运作? <a href="#mqtt50-de-qing-qiu-xiang-ying-ru-he-yun-zuo" id="mqtt50-de-qing-qiu-xiang-ying-ru-he-yun-zuo"></a>

### 响应主题 <a href="#xiang-ying-zhu-ti" id="xiang-ying-zhu-ti"></a>

在 MQTT 5.0 中，请求方可以在请求消息中指定一个自己期望的响应主题 (Response Topic)。响应方根据请求内容采取适当的操作后，向请求中携带的响应主题发布响应消息。如果请求方订阅了该响应主题，那么就会收到响应。

![02responsetopic2.jpg](https://assets.emqx.com/images/790d8c87fe2670dd6454d8456bf41ab0.jpg?imageMogr2/thumbnail/1520x)

请求方可以将自己的 Client ID 作为响应主题的一部分，这可以有效避免不同的请求方不小心使用了相同的响应主题而造成冲突。

### 关联数据 <a href="#guan-lian-shu-ju" id="guan-lian-shu-ju"></a>

请求方还可以在请求中携带关联数据 (Correlation Data)，响应方**必须**在响应中将关联数据**原封不动**地返回，请求方因此可以识别响应所属的原始请求。

这可以避免在响应方没有按请求顺序返回响应或者由于网络连接断开导致丢失了某个响应 (QoS 0) 时，请求方不正确地关联响应与原始请求。

另一方面，请求方可能需要与多个响应方交互，譬如我们通过手机控制家中的各种智能设备，关联数据可以让请求方仅订阅一个响应主题就能管理从多个响应方异步返回的响应。

![03correlationdata.png](https://assets.emqx.com/images/94f045c13ac06e422e8730928d3a51d5.png?imageMogr2/thumbnail/1520x)

在以上“请求 / 响应”的过程中，MQTT 服务端不会对响应主题、关联数据进行任何更改，它仅起到转发的作用。

### 响应信息 <a href="#xiang-ying-xin-xi" id="xiang-ying-xin-xi"></a>

出于安全考虑，MQTT 服务端通常会限制客户端可以发布和订阅的主题。请求方可以指定一个随机的响应主题，但无法保证自己有订阅该主题的权限，也无法保证响应方有向该响应主题发布消息的权限。

所以 MQTT 5.0 还引入了响应信息 (Response Information) 属性，通过在 CONNECT 报文中将请求响应信息(Request Response Information) 标识符设置为 1，客户端可以请求服务端在 CONNACK 报文中返回响应信息。客户端可以将响应信息的内容作为响应主题的某个特定部分，以便通过服务端的权限检查。

![04responseinformation.png](https://assets.emqx.com/images/5d299bd028d7d3d9851589e9eaa0477a.png?imageMogr2/thumbnail/1520x)

MQTT 并未进一步约定这部分的细节，比如响应信息的内容格式以及客户端如何根据响应信息创建响应主题，所以不同服务端和客户端的实现可能有所不同。例如，服务端可以使用响应信息 “FRONT,mytopic” 既指示响应主题中特定部分的具体内容，又指示该特定部分在响应主题中的位置，也可以与客户端提前约定如何使用该特定部分，然后使用响应信息 “mytopic” 仅指示该特定部分的具体内容。

以智能家居场景为例，智能设备不会跨用户使用，我们可以让 MQTT 服务端将设备所属用户的 ID 作为响应信息返回，客户端统一使用该用户 ID 作为响应主题的前缀。MQTT 服务端只需要确保这些客户端在它们会话的生命周期内，都拥有该用户 ID 开头的主题的发布和订阅权限即可。

## MQTT “请求 / 响应”的使用建议 <a href="#mqtt-qing-qiu-xiang-ying-de-shi-yong-jian-yi" id="mqtt-qing-qiu-xiang-ying-de-shi-yong-jian-yi"></a>

以下是在 MQTT 中使用“请求 / 响应”时的一些建议，遵循这些建议将有助于你实施最佳实践：

1. MQTT 的 QoS 1 或 2 只能确保消息到达服务端，如果想要确认消息是否到达订阅端，可以借助“请求 / 响应”机制。
2. 在发送请求前订阅响应主题以免错过响应。
3. 确保响应方和请求方拥有发布和订阅响应主题的必要权限，响应信息 (Response Information) 可以帮助我们构建符合权限要求的响应主题。
4. 存在多个请求方时，请求方需要使用不同的响应主题以免响应混淆，使用 Client ID 作为主题的一部分是常见的做法。
5. 存在多个响应方时，请求方最好在请求中设置关联数据 (Correlation Data) 以免响应混淆。
6. 遗嘱消息也可以使用“请求 / 响应”，只需要在连接时为遗嘱消息设置响应主题即可。这可以帮助客户端知道在自己离线期间遗嘱消息是否被消费，以便做出适当的调整。
