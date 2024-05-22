# PINGREQ与PINGRESP

除了用于连接、发布和订阅的控制报文，MQTT 还有一类报文用于在客户端和服务端之间模拟心跳，以达到保持连接的目的，它们分别是 PINGREQ 报文和 PINGRESP 报文，我们通常也会称它们为心跳报文。

客户端定期向服务端发送 PINGREQ 报文，服务端可以由此得知连接良好且客户端仍然活跃。每收到一个 PINGREQ 报文，服务端就会回复一个 PINGRESP 报文，因此客户端也可以由此得知连接良好且服务端仍然活跃。

## 报文示例 <a href="#bao-wen-shi-li" id="bao-wen-shi-li"></a>

我们使用 MQTTX CLI 向 公共 MQTT 服务器 发起一个客户端连接，不发布消息也不订阅主题，但我们仍然可以在 Wireshark 中看到客户端和服务端之间总是周期性地出现 MQTT 报文的往返，这些报文就是 PINGREQ 和 PINGRESP 报文。

以下命令将创建一个 Keep Alive 为 5 秒的客户端连接，这可以让我们尽快看到客户端发送 PINGREQ 报文：

```
mqttx conn --hostname broker.emqx.io --mqtt-version 5 --keepalive 5
```

我们会发现 PINGREQ 和 PINGRESP 报文总是只有 2 个字节的大小，并且它们的内容似乎也永远不会发生变化：

```
# PINGREQ
c0 00
# PINGRESP
d0 00
```

这是因为这两个报文有着非常简单的报文结构。

## PINGREQ & PINGRESP 报文结构 <a href="#pingreqamppingresp-bao-wen-jie-gou" id="pingreqamppingresp-bao-wen-jie-gou"></a>

PINGREQ 和 PINGRESP 报文的区别仅仅是固定报头中报文类型的不同，12（0xC）表示这是一个 PINGREQ 报文，13（0xD）则表示这是一个 PINGRESP 报文。

因为 PINGREQ 和 PINGRESP 报文均不包含**可变报头**和**有效载荷**，所以它们的**固定报头**中剩余长度字段的值永远都是 0。

![PING 报文.png](https://assets.emqx.com/images/eaf716b210700b04e6645307324b3f4c.png?imageMogr2/thumbnail/1520x)

这种报文结构让 PINGREQ 和 PINGRESP 报文的大小降到了最低，所以发送它们并不会占用太多的带宽。

### 总结 <a href="#zong-jie" id="zong-jie"></a>

PINGREQ 和 PINGRESP 是 MQTT 中最简单的报文类型，它们的内容固定不变。我们唯一可以改变的，就是通过连接时的 **Keep Alive 选项**，影响客户端发送 PINGREQ 报文的频率。

如果服务端没有在 1.5 倍 Keep Alive 时间内收到客户端发送的任何控制报文，就会认为客户端处于非活跃状态或网络异常而断开连接。在本文的报文示例中，我们在连接时 Keep Alive 设置为 5 秒，那么对服务端来说超时时间就是 7.5 秒。

对于客户端来说，如果在发送 PINGREQ 报文之后的一段时间内，没有收到服务端返回的 PINRESP 报文，那么它应该断开连接。这个时间的长短，主要取决于客户端对网络延迟的预期
