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

# 主题别名(Topic Alias)

## 什么是主题别名 <a href="#shen-me-shi-zhu-ti-bie-ming" id="shen-me-shi-zhu-ti-bie-ming"></a>

主题别名（Topic Alias）是 MQTT v5.0 中新加入的与主题名（topic）相关的特性。它允许用户将主题长度较长且常用的主题名缩减为一个双字节整数来降低发布消息时的带宽消耗。

它是一个双字节整数，并将作为属性字段，编码在`PUBLISH`报文中可变报头部分。并且在实际应用中，将受到`CONNECT`报文和`CONNACK`报文中“主题别名最大长度”属性的限制。只要不超过该限制，任何主题名，都可以使用此特性缩减为编码长度2字节的整数。

## 为什么使用主题别名 <a href="#wei-shen-me-shi-yong-zhu-ti-bie-ming" id="wei-shen-me-shi-yong-zhu-ti-bie-ming"></a>

在使用 MQTT v3 协议时。如果客户端在某次连接中需要发布大量相同主题的消息，那么在每一条`PUBLISH`报文中写入相同的主题名，就造成了客户端和服务端之间带宽资源的浪费。同时对于服务端而言，每次对相同主题名的 UTF-8 字符串进行解析，都是对计算资源的浪费。

设想这样的场景，在位置 `A` 以固定频率报告温度湿度的传感器：

* 使用 `/position/A/temperature` 作为该位置温度消息的主题（长度23字节）；
* 使用 `/position/A/humidity` 作为该位置湿度消息的主题（长度20字节）。

除去第一次发布消息外，之后的每个`PUBLISH`报文，都需要将“主题名”这个已经传递过的信息再次通过网络传输。即便抛开客户端和服务端之间额外的带宽消耗不言，对服务端来说，面对成千上万的传感器发布的大量消息，对每个客户端的每条消息，都要将同样的主题名字符串进行解析，这将造成了计算资源的浪费。

此时使用 MQTT v5 中的主题别名特性，就可以有效降低资源消耗。当客户端或服务端发布频率较高，且主题名长度较大的情景下，使用主题别名可以将每条消息中主题名的带宽消耗缩减为 2 字节，同时因为计算机处理整数的效率高于处理字符串的效率，对于客户端或服务端在报文解析时消耗的计算资源也有了一定的节约。

## 怎样使用主题别名 <a href="#zen-yang-shi-yong-zhu-ti-bie-ming" id="zen-yang-shi-yong-zhu-ti-bie-ming"></a>

### 主题别名生命周期和作用范围 <a href="#zhu-ti-bie-ming-sheng-ming-zhou-qi-he-zuo-yong-fan-wei" id="zhu-ti-bie-ming-sheng-ming-zhou-qi-he-zuo-yong-fan-wei"></a>

该值由客户端和服务端**各自维护**，且生命周期和作用范围仅限于当前连接。连接断开后需要再次使用主题别名需要重新建立`主题别名<=>主题名`映射关系。

### 主题别名最大值（Topic Alias Maximum） <a href="#zhu-ti-bie-ming-zui-da-zhi-topicaliasmaximum" id="zhu-ti-bie-ming-zui-da-zhi-topicaliasmaximum"></a>

在 MQTT 客户端和服务端使用主题别名进行发布消息前，需要对可以使用的最大主题别名长度进行约定。这部份信息交换将在`CONNECT`报文和`CONNACK`报文中完成。“主题别名最大值”也将以报文属性的形式，用双字节整数值编码在`CONNECT`和`CONNACK`报文的可变报头中。 客户端的`CONNECT`报文中"主题别名最大值"指示了本客户端在此次连接中服务端可以使用的最大主题别名数量；同样地，服务端发送的`CONNACK`报文中，也通过此值表明了当前连接中对端（客户端）可以使用的最大主题别名数量。

![MQTT 主题别名最大值](https://assets.emqx.com/images/8e5825731ef375d0cf50b7fa8b45e348.png?imageMogr2/thumbnail/1520x)

双端各自设置对端可以使用的最大主题别名数量

### 设置与使用主题别名 <a href="#she-zhi-yu-shi-yong-zhu-ti-bie-ming" id="she-zhi-yu-shi-yong-zhu-ti-bie-ming"></a>

客户端（或服务端）在发送`PUBLISH`报文时，可以在可变报头的属性部分，用一个字节，值为`0x23`的标识符指示接下来2字节将是主题别名值。

但主题别名值不允许为0，也不允许大于服务端（客户端）发送的`CONNACK`（`CONNECT`）报文中设置的主题别名最大值。

对端接收到带有主题别名值和非空主题名的`PUBLISH`报文后，将建立主题别名和主题名的映射关系，在此之后发送的`PUBLISH`报文中，便可以仅用长度2字节的主题别名发布消息，对端将使用通过之前建立的`主题别名<=>主题名`映射关系来处理消息中的主题。并且由于这一映射关系由双端各自维护，所以客户端与服务端可以使用值相同的主题别名互相发布消息。

![设置与使用 MQTT 主题别名](https://assets.emqx.com/images/90b455343adc89bade35746b3bf71a88.png?imageMogr2/thumbnail/1520x)

MQTT Client 与 MQTT Broker 分别设置 topic\_alias

### 使用未设置的主题别名 <a href="#shi-yong-wei-she-zhi-de-zhu-ti-bie-ming" id="shi-yong-wei-she-zhi-de-zhu-ti-bie-ming"></a>

`PUBLISH`报文中使用的主题别名值如果在此前的报文中未进行设置，即对端并未建立当前主题别名到某个主题名的映射关系，而此条报文的可变报头中主题名字段为空，对端将使用包含原因码（REASON\_CODE）为`0x82`的`DISCONNECT`报文断开网络连接。

![使用未设置的 MQTT 主题别名](https://assets.emqx.com/images/a405b27fb7440c605450b87c44ede080.png?imageMogr2/thumbnail/1520x)

使用未建立映射关系的主题别名

### 重置主题别名 <a href="#zhong-zhi-zhu-ti-bie-ming" id="zhong-zhi-zhu-ti-bie-ming"></a>

当对端已经根据本次连接中某个`PUBLISH`报文创建了一个`主题别名<=>主题名`的映射关系时，可以在下一次发送`PUBLISH`报文时使用同样的主题别名值和非空的主题名来更新这个主题别名值到主题名的映射关系。

![重置 MQTT 主题别名](https://assets.emqx.com/images/264442ebc239f5a1cfbbf2f7ee990c1e.png?imageMogr2/thumbnail/1520x)

MQTT Client 与 MQTT Broker 分别更新主题别名所对应的主题名

### 总结 <a href="#zong-jie" id="zong-jie"></a>

主题别名作为 MQTT v5 新提供的特性，为 pub/sub 这一消息传递模型提供了更灵活的使用方式，对于主题名一致且数量大、重复性高的消息而言，可以有效节省带宽资源和计算资源。
