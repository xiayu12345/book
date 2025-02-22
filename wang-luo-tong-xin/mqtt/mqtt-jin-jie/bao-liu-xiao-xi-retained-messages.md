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

# 保留消息（Retained Messages）

## 什么是 MQTT 保留消息？ <a href="#shen-me-shi-mqtt-bao-liu-xiao-xi" id="shen-me-shi-mqtt-bao-liu-xiao-xi"></a>

发布者发布消息时，如果 Retained 标记被设置为 true，则该消息即是 MQTT 中的保留消息（Retained Message）。MQTT 服务器会为每个主题存储最新一条保留消息，以方便消息发布后才上线的客户端在订阅主题时仍可以接收到该消息。

如下图，当客户端订阅主题时，如果服务端存在该主题匹配的保留消息，则该保留消息将被立即发送给该客户端。

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## 何时使用 MQTT 保留消息？ <a href="#he-shi-shi-yong-mqtt-bao-liu-xiao-xi" id="he-shi-shi-yong-mqtt-bao-liu-xiao-xi"></a>

发布订阅模式虽然能让消息的发布者与订阅者充分解耦，但也存在一个缺点，即订阅者无法主动向发布者请求消息。订阅者何时收到消息完全依赖于发布者何时发布消息，这在某些场景中就产生了不便。

借助保留消息，新的订阅者能够立即获取最近的状态，而不需要等待无法预期的时间，例如：

* 智能家居设备的状态只有在变更时才会上报，但是控制端需要在上线后就能获取到设备的状态；
* 传感器上报数据的间隔太长，但是订阅者需要在订阅后立即获取到最新的数据；
* 传感器的版本号、序列号等不会经常变更的属性，可在上线后发布一条保留消息告知后续的所有订阅者。

## MQTT 保留消息的使用 <a href="#mqtt-bao-liu-xiao-xi-de-shi-yong" id="mqtt-bao-liu-xiao-xi-de-shi-yong"></a>

若要使用 MQTT 保留消息，只需在消息发布时将 Retained 状态设置为 true 即可。接下来我们以开源的跨平台 MQTT 5.0 桌面客户端工具 - MQTTX 为例，演示如何使用 MQTT 保留消息。

打开 MQTTX 后如下所示，需点击 `New Connection` 按钮创建一个 MQTT 连接。

![创建 MQTT 连接 1](https://assets.emqx.com/images/c3c89247952538c127839de49a398aec.png?imageMogr2/thumbnail/1520x)

创建页面如下，我们只需填写一个连接名称（Name），其他参数保持默认。Host 将默认为 EMQX Cloud 提供的公共 MQTT 服务器。连接参数填写完成后，点击右上角的 `Connect` 按钮创建 MQTT 连接。

![创建 MQTT 连接 2](https://assets.emqx.com/images/199e08891e0a7ca0ad78efa8f986dc21.png?imageMogr2/thumbnail/1520x)

连接成功后将会看到连接名称旁边的状态为绿色。然后我们在右下角消息输入框向主题 `sensor/t1` 发送一条普通的消息。

![MQTT 连接成功](https://assets.emqx.com/images/d66d61a3e507c9371f6665ac1f6be289.png?imageMogr2/thumbnail/1520x)

接下来我们选中右下角的 Retain 标记，并向主题 `sensor/t2` 发送两条保留消息。

![发送 MQTT 保留消息](https://assets.emqx.com/images/2c202c92516bb9d1394b65410b236dde.png?imageMogr2/thumbnail/1520x)

然后点击页面中间的 `New Subscription` 按钮创建订阅。

![订阅 MQTT 主题 1](https://assets.emqx.com/images/2e834540fa748f318f7a1f770070db64.png?imageMogr2/thumbnail/1520x)

如下，我们订阅通配符主题 `sensor/+`，该通配符主题将会匹配主题 `sensor/t1` 及 `sensor/t2`。

![订阅 MQTT 主题 2](https://assets.emqx.com/images/d7da8ae6e8cad9dffa82dee3b3014cc1.png?imageMogr2/thumbnail/1520x)

最后，我们将会看到该订阅能成功收到第二条保留消息，`sensor/t1` 的普通消息及 `sensor/t2` 的第一条保留消息都未收到。可见 MQTT 服务器只会为每个主题存储最新一条保留消息。

![接收 MQTT 保留消息](https://assets.emqx.com/images/a1a9d7e1ca32f77a8e54f09dccccee99.png?imageMogr2/thumbnail/1520x)

## 关于 MQTT 保留消息的 Q\&A <a href="#guan-yu-mqtt-bao-liu-xiao-xi-de-qampa" id="guan-yu-mqtt-bao-liu-xiao-xi-de-qampa"></a>

#### 如何判断一条消息是否是保留消息？ <a href="#ru-he-pan-duan-yi-tiao-xiao-xi-shi-fou-shi-bao-liu-xiao-xi" id="ru-he-pan-duan-yi-tiao-xiao-xi-shi-fou-shi-bao-liu-xiao-xi"></a>

当客户端订阅了有保留消息的主题后，即会收到该主题的保留消息，可通过消息中的保留标志位判断是否是保留消息。需要注意的是，在保留消息发布前订阅主题，将不会收到保留消息。**需要待保留消息发布后，重新订阅该主题，才会收到保留消息。**

如下图，我们先订阅主题 `sensor/t2`，然后向该主题发布一条保留消息，该订阅会立即收到一条消息，但是该消息并不是保留消息。当我们删除该订阅，再次重新订阅 `sensor/t2` 主题时，立即收到了刚刚发布的保留消息。

![MQTT 保留消息](https://assets.emqx.com/images/06d1e7ec9edfebccf2425c39a73b1e6e.png?imageMogr2/thumbnail/1520x)

#### 保留消息将保存多久？如何删除？ <a href="#bao-liu-xiao-xi-jiang-bao-cun-duo-jiu-ru-he-shan-chu" id="bao-liu-xiao-xi-jiang-bao-cun-duo-jiu-ru-he-shan-chu"></a>

服务器只会为每个主题保存最新一条保留消息，保留消息的保存时间与服务器的设置有关。若服务器设置保留消息存储在内存，则 MQTT 服务器重启后消息即会丢失；若存储在磁盘，则服务器重启后保留消息仍然存在。

保留消息虽然存储在服务端中，但它并不属于会话的一部分。也就是说，即便发布这个保留消息的会话已结束，保留消息也不会被删除。删除保留消息有以下几种方式：

* 客户端往某个主题发送一个 Payload 为空的保留消息，服务端就会删除这个主题下的保留消息；
* 在 MQTT 服务器上删除，比如 EMQX MQTT 服务器提供了在 Dashboard 上删除保留消息的功能；
* MQTT 5.0 新增了消息过期间隔属性，发布时可使用该属性设置消息的过期时间，不管消息是否为保留消息，都将会在过期时间后自动被删除。
