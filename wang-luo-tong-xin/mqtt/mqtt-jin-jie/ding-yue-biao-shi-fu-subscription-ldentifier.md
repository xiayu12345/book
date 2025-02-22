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

# 订阅标识符（Subscription ldentifier）

## 为什么需要订阅标识符 <a href="#wei-shen-me-xu-yao-ding-yue-biao-shi-fu" id="wei-shen-me-xu-yao-ding-yue-biao-shi-fu"></a>

在大部分 MQTT 客户端的实现中，都会通过回调机制来实现对新到达消息的处理。

但是在回调函数中，我们只能知道消息的主题名是什么。如果是非通配符订阅，订阅时使用的主题过滤器将和消息中的主题名完全一致，所以我们可以直接建立订阅主题与回调函数的映射关系。然后在消息到达时，根据消息中的主题名查找并执行对应的回调函数。

但如果是通配符订阅，消息中的主题名和订阅时的主题过滤器将是两个不同的字符串，我们只有将消息中的主题名与原始的订阅挨个进行主题匹配，才能确定应该执行哪个回调函数。这显然极大地影响了客户端的处理效率。

![mqtt subscription identifier 01](https://assets.emqx.com/images/27648a4465bf3948af3a61e533fd8aad.png?imageMogr2/thumbnail/1520x)

另外，因为 MQTT 允许一个客户端建立多个订阅，那么当客户端使用通配符订阅时，一条消息可能同时与一个客户端的多个订阅匹配。

MQTT 允许服务端为这些重叠的订阅分别发送一次消息，也允许服务端只为这些重叠的订阅发送一条消息。

当服务端采用前一种实现时，客户端必须额外对这些消息进行去重才能保证回调不会被重复执行：

![mqtt subscription identifierm 02](https://assets.emqx.com/images/88ef650cac1ae4196fc008cda7d73279.png?imageMogr2/thumbnail/1520x)

## 订阅标识符的工作原理 <a href="#ding-yue-biao-shi-fu-de-gong-zuo-yuan-li" id="ding-yue-biao-shi-fu-de-gong-zuo-yuan-li"></a>

为了解决这个问题，MQTT 5.0 引入了订阅标识符。它的用法非常简单，客户端可以在订阅时指定一个订阅标识符，服务端则需要存储该订阅与订阅标识符的映射关系。当有匹配该订阅的 PUBLISH 报文要转发给此客户端时，服务端会将与该订阅关联的订阅标识符随 PUBLISH 报文一并返回给客户端。

![mqtt subscription identifierm 03](https://assets.emqx.com/images/e31a72810ff815d622b68f501094a44a.png?imageMogr2/thumbnail/1520x)

如果服务端选择为重叠的订阅分别发送一次消息，那么每个 PUBLISH 报文都应该包含与订阅相匹配的订阅标识符，而如果服务端选择为重叠的订阅只发送一条消息，那么 PUBLISH 报文将包含多个订阅标识符。

客户端只需要建立订阅标识符与回调函数的映射，就可以通过消息中的订阅标识符得知这个消息来自哪个订阅，以及应该执行哪个回调函数。

![mqtt subscription identifierm 04](https://assets.emqx.com/images/3ddfab45720fc724434c2edaf47662f6.png?imageMogr2/thumbnail/1520x)

在客户端中，订阅标识符并不属于会话状态的一部分，将订阅标识符和什么内容进行关联，完全由客户端决定。所以除了回调函数，我们也可以建立订阅标识符与订阅主题的映射，或者建立与 Client ID 的映射。后者在转发服务端消息给客户端的网关中非常有用。当消息从服务端到达网关，网关只要根据订阅标识符就能够知道应该将消息转发给哪个客户端，而不需要重新做一次主题的匹配和路由。

一个订阅报文只能包含一个订阅标识符，如果一个订阅报文中有多个订阅请求，那么这个订阅标识符将同时和这些订阅相关联。所以请尽量确保将多个订阅关联至同一个回调是您有意为之的。

## 如何使用订阅标识符 <a href="#ru-he-shi-yong-ding-yue-biao-shi-fu" id="ru-he-shi-yong-ding-yue-biao-shi-fu"></a>

1. 在 Web 浏览器上访问 MQTTX Web。
2.  创建一个使用 WebSocket 的 MQTT 连接，并且连接免费的 公共 MQTT 服务器：

    ![MQTT over WebSocket](https://assets.emqx.com/images/e1c10cbd018d0742f21f3b371ec89c6a.png?imageMogr2/thumbnail/1520x)
3.  连接成功后，我们先订阅主题 `mqttx_4299c767/home/+`，并指定 Subscription Identifier 为 1，然后订阅主题 `mqttx_4299c767/home/PM2_5`，并指定 Subscription Identifier 为 2。由于公共服务器可能同时被很多人使用，为了避免主题与别人重复，这里我们将 Client ID 作为主题前缀：

    ![New Subscription 1](https://assets.emqx.com/images/f3c0aed851e02f20aae69cf100b167d6.png?imageMogr2/thumbnail/1520x)

    ![New Subscription 2](https://assets.emqx.com/images/212728b6ae71b5baf73a860f75d4545a.png?imageMogr2/thumbnail/1520x)
4.  订阅成功后，我们向主题 `mqttx_4299c767/home/PM2_5` 发布一条消息。我们将看到当前客户端收到了两条消息，消息中的 Subscription Identifier 分别为 1 和 2。这是因为 EMQX 的实现是为重叠的订阅分别发送一条消息：

    ![Receive MQTT Messages](https://assets.emqx.com/images/fd38994dea83422bb31a85b5c14711b1.png?imageMogr2/thumbnail/1520x)
5.  而如果我们向主题 `mqttx_4299c767/home/temperature` 发布一条消息，我们将看到收到消息中的 Subscription Identifier 为 1：

    ![image.png](https://assets.emqx.com/images/f0a2dba909a1efa8fab0b07ea961a959.png?imageMogr2/thumbnail/1520x)
