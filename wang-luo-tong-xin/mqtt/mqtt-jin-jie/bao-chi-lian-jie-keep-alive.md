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

# 保持连接（Keep Alive）

## 为什么需要 Keep Alive <a href="#wei-shen-me-xu-yao-keepalive" id="wei-shen-me-xu-yao-keepalive"></a>

MQTT 协议是承载于 TCP 协议之上的，而 TCP 协议以连接为导向，在连接双方之间，提供稳定、有序的字节流功能。 但是，在部分情况下，TCP 可能出现半连接问题。所谓半连接，是指某一方的连接已经断开或者没有建立，而另外一方的连接却依然维持着。在这种情况下，半连接的一方可能会持续不断地向对端发送数据，而显然这些数据永远到达不了对端。为了避免半连接导致的通信黑洞，MQTT 协议提供了 **Keep Alive** 机制，使客户端和 MQTT 服务器可以判定当前是否存在半连接问题，从而关闭对应连接。

## MQTT Keep Alive 的机制流程与使用 <a href="#mqttkeepalive-de-ji-zhi-liu-cheng-yu-shi-yong" id="mqttkeepalive-de-ji-zhi-liu-cheng-yu-shi-yong"></a>

### 启用 Keep Alive <a href="#qi-yong-keepalive" id="qi-yong-keepalive"></a>

客户端在创建和 MQTT Broker 的连接时，只要将连接请求协议包内的 _Keep Alive_ 可变头部字段设置为非 0 值，就可以在通信双方间启用 **Keep Alive** 机制。 _Keep Alive_ 为 0\~65535 的一个整数，代表客户端发送两次 MQTT 协议包之间的最大间隔时间。

而 Broker 在收到客户端的连接请求后，会检查可变头部中的 _Keep Alive_ 字段的值，如果有值，则 Broker 将会启用 **Keep Alive** 机制。

### MQTT 5.0 Server Keep Alive <a href="#mqtt-5-0-server-keep-alive" id="mqtt-5-0-server-keep-alive"></a>

在 [MQTT 5.0](../mqtt5.0/mqtt-5.07-xiang-xin-gong-neng.md) 标准中，引入了 _Server Keep Alive_ 的概念，允许 Broker 根据自身的实现等因素，选择接受客户端请求中携带的 _Keep Alive_ 值，或者是覆盖这个值。如果 Broker 选择覆盖这个值，则需要将新值设置在连接确认包(**CONNACK**) 的 _Server Keep Alive_ 字段中，客户端如果在连接确认包中读取到了 _Server Keep Alive_，则需要使用该值，覆盖自己之前的 _Keep Alive_ 的值。

## Keep Alive 机制流程 <a href="#keepalive-ji-zhi-liu-cheng" id="keepalive-ji-zhi-liu-cheng"></a>

### **客户端流程**

在连接建立后，客户端需要确保, 自己任意两次 MQTT 协议包的发送间隔不超过 _Keep Alive_ 的值，如果客户端当前处于空闲状态，没有可发送的包，则可以发送 **PINGREQ** 协议包。

当客户端发送 **PINGREQ** 协议包后，Broker 必须返回一个 **PINGRESP** 协议包，如果客户端在一个可靠的时间内，没有收到服务器的 **PINGRESP** 协议包，则说明当前存在半连接、或者 Broker 已经下线、或者出现了网络故障，这个时候，客户端应当关闭当前连接。

### **Broker 流程**

在连接建立后，Broker 如果没有在 _Keep Alive_ 的 1.5 倍时间内，收到来自客户端的任何包，则会认为和客户端之间的连接出现了问题，此时 Broker 便会断开和客户端的连接。

如果 Broker 收到了来自客户端的 **PINGREQ** 协议包，需要回复一个 **PINGRESP** 协议包进行确认。

### **客户端接管机制**

当 Broker 里存在半连接时，如果对应的客户端发起了重连或新的连接，则 Broker 会启动客户端接管机制：关闭旧的半连接，然后与客户端建立新的连接。

这种机制保证了客户端不会因为 Broker 里存在的半连接，导致无法进行重连。

### Keep Alive 与遗嘱消息 <a href="#keepalive-yu-yi-zhu-xiao-xi" id="keepalive-yu-yi-zhu-xiao-xi"></a>

Keep Alive 通常还可以与遗嘱消息结合使用，通过遗嘱消息，设备可将自己的意外掉线情况及时通知第三方。

如下图，该客户端连接时设置了 Keep Alive 为 5 秒，并且设置了遗嘱消息。那么当服务器 7.5 秒（1.5 倍 Keep Alive）内未收到该客户端的任何报文时，即会向 `last_will` 主题发送 Payload 为 `offline` 的遗嘱消息。

![MQTT Keep Alive 与遗嘱消息](https://assets.emqx.com/images/3fc9e2c463bd38c21dc7f523520c7076.png?imageMogr2/thumbnail/1520x)

{% hint style="info" %}
更多关于遗嘱消息的介绍可查看：[MQTT 遗嘱消息（Will Message）的使用](yi-zhu-xiao-xi-will-messages.md)。
{% endhint %}
