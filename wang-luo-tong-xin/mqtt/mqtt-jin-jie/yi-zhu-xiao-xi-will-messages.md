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

# 遗嘱消息(Will Messages)

## 遗嘱消息（Will Message）介绍与示例 | MQTT 5.0 特性详解

遗嘱消息是 MQTT 协议中的一个重要功能，它解决了只有服务端才能知道客户端是否在线的问题，使我们能够为意外离线的客户端优雅地完成善后事宜。

## 什么是 MQTT 遗嘱消息？ <a href="#shen-me-shi-mqtt-yi-zhu-xiao-xi" id="shen-me-shi-mqtt-yi-zhu-xiao-xi"></a>

在现实世界中，一个人可以制定一份遗嘱，声明在他去世后应该如何分配他的财产以及应该采取什么行动。在他去世后，遗嘱执行人会将这份遗嘱公开，并执行遗嘱中的指示。

在 MQTT 中，客户端可以在连接时在服务端中注册一个遗嘱消息，与普通消息类似，我们可以设置遗嘱消息的主题、有效载荷等等。当该客户端意外断开连接，服务端就会向其他订阅了相应主题的客户端发送此遗嘱消息。这些接收者也因此可以及时地采取行动，例如向用户发送通知、切换备用设备等等。

假设我们有一个传感器监控一个很少变化的值，普通的实现是定期发布最新数值，但更好的实现是仅在数值发生变化时以保留消息的形式发送它。这使得任何新的订阅者总能立即获得当前值，而不必等待传感器再一次发布。不过订阅者也因此没有办法根据是否及时收到消息来判断传感器是否离线。借助遗嘱消息，我们可以立即得知传感器保持活动超时，而且不必总是获取传感器发布的值。

### Will Message 还是 Last Will and Testament（LWT）？ <a href="#willmessage-hai-shi-lastwillandtestamentlwt" id="willmessage-hai-shi-lastwillandtestamentlwt"></a>

在一些博客或者代码中，我们可能会看到 Last Will and Testament 这个名字，或者是它的缩写：LWT。它指的就是 MQTT 中的 Will Message。导致这两种命名共存的原因可能是，MQTT 最早在 3.1 协议规范的摘要中，提到了 Last Will and Testament 这个概念。

虽然 MQTT 在协议的正文部分一直以来都是明确使用 Will Message 这个名字，但目前在用户群体中，这两个名字经常会被混用。

## MQTT 遗嘱消息如何运作 <a href="#mqtt-yi-zhu-xiao-xi-ru-he-yun-zuo" id="mqtt-yi-zhu-xiao-xi-ru-he-yun-zuo"></a>

### 客户端在连接时指定遗嘱消息 <a href="#ke-hu-duan-zai-lian-jie-shi-zhi-ding-yi-zhu-xiao-xi" id="ke-hu-duan-zai-lian-jie-shi-zhi-ding-yi-zhu-xiao-xi"></a>

遗嘱消息在客户端发起连接时指定，它和 Client ID、Clean Start 这些字段一起包含在客户端发送的 CONNECT 报文中。

与普通消息一样，我们可以为遗嘱消息设置主题（Will Topic）、保留消息标识位（Will Retain）、属性（Will Properties）、QoS（Will QoS）和有效载荷（Will Payload）。

![MQTT will message fields](https://assets.emqx.com/images/0dcc740dabb41cfee950b1a0d71bc304.jpg?imageMogr2/thumbnail/1520x)

这些字段的用法与它们在普通消息中时完全相同，只是遗嘱消息可用的属性与普通应用消息略有不同，下表列出了它们的具体区别：

![MQTT 遗嘱消息](https://assets.emqx.com/images/74a81e6b50658612fa0045c1b867f693.jpg?imageMogr2/thumbnail/1520x)

遗嘱消息总是在客户端“死亡”后被发布，在某种意义上，它也是客户端发出的最后一个消息。所以主题别名在遗嘱消息中没有任何意义。

除此之外，遗嘱消息只是多了一个专属属性：**Will Delay Interval**。它是 MQTT 5.0 为遗嘱消息引入的一个重要改进

### 服务端在连接意外关闭时发布遗嘱消息 <a href="#fu-wu-duan-zai-lian-jie-yi-wai-guan-bi-shi-fa-bu-yi-zhu-xiao-xi" id="fu-wu-duan-zai-lian-jie-yi-wai-guan-bi-shi-fa-bu-yi-zhu-xiao-xi"></a>

如果客户端在连接时指定了遗嘱消息，那么服务端就会将该遗嘱消息存储在相应的会话中，直到以下任一条件满足时发布它：

1. 服务端检测到了一个 I/O 错误或者网络故障
2. 客户端在 Keep Alive 时间内未能通讯
3. 客户端在没有发送 Reason Code 为 0x00（正常关闭）的 DISCONNECT 报文的情况下关闭了网络连接
4. 服务端在没有收到 Reason Code 为 0x00（正常关闭）的 DISCONNECT 报文的情况下关闭了网络连接，例如客户端的报文或行为不符合协议要求而被服务端关闭连接。

简单起见，我们可以直接概括为，只要网络连接在服务端没有收到 Reason Code 为 0x00 的 DISCONNECT 报文的情况下关闭，那么服务端都需要发送遗嘱消息。

当客户端完成了预定的工作准备正常下线时，可以发送一个 Reason Code 为 0x00 的 DISCONNECT 报文然后关闭网络连接，避免服务端因此发布遗嘱消息。

### Will Delay Interval 与延迟发布 <a href="#willdelayinterval-yu-yan-chi-fa-bu" id="willdelayinterval-yu-yan-chi-fa-bu"></a>

默认情况下，服务端总是在网络连接意外关闭时立即发布遗嘱消息。但是很多时候，网络连接的中断是短暂的，所以客户端往往能够重新连接并继续之前的会话。这导致遗嘱消息可能被频繁地且无意义地发送。

所以 MQTT 5.0 专门为遗嘱消息增加了一个 Will Delay Interval 属性，这个属性决定了服务端将在网络连接关闭后延迟多久发布遗嘱消息，并以秒为单位。

如果没有指定 Will Delay Interval 或者将其设置为 0，服务端将仍然在网络连接关闭时立即发布遗嘱消息。

但如果将 Will Delay Interval 设置为一个大于 0 的值，并且客户端能够在 Will Delay Interval 到期前恢复连接，那么该遗嘱消息将不会被发布。

### 遗嘱消息与会话 <a href="#yi-zhu-xiao-xi-yu-hui-hua" id="yi-zhu-xiao-xi-yu-hui-hua"></a>

遗嘱消息是服务端会话状态的一部分，当会话结束，遗嘱消息也无法继续单独存在。

但是在遗嘱消息延迟发布期间，会话可能过期，也可能因为客户端在新的连接中设置 Clean Start 为 1 所以服务端需要丢弃之前的会话。

为了避免丢失遗嘱，此时服务端必须发布该遗嘱消息，即便 Will Delay Interval 还没有到期。

所以服务端最终何时发布遗嘱消息，取决于 Will Delay Interval 到期和会话结束这两种情况谁先发生。

## MQTT 3.1.1 中的遗嘱消息 <a href="#mqtt311-zhong-de-yi-zhu-xiao-xi" id="mqtt311-zhong-de-yi-zhu-xiao-xi"></a>

在 MQTT 3.1.1 中，只要网络连接在服务端没有收到 DISCONNECT 报文的情况下关闭，服务端都需要发布遗嘱消息。

由于 MQTT 3.1.1 没有 Will Delay Interval，也没有 Session Expiry Interval，所以遗嘱消息总是在网络连接关闭时立即发布。

### 为什么没有收到遗嘱消息？ <a href="#wei-shen-me-mei-you-shou-dao-yi-zhu-xiao-xi" id="wei-shen-me-mei-you-shou-dao-yi-zhu-xiao-xi"></a>

遗嘱消息的延迟发布和取消发布让订阅端最终是否会收到遗嘱消息这个问题变得稍显复杂。

我们对所有可能的情况进行了梳理，以便让大家更好地理解：

![MQTT 遗嘱消息流程](https://assets.emqx.com/images/33dbd295c2f56b83ebdf13d657ea59ce.jpg?imageMogr2/thumbnail/1520x)

1. 连接意外关闭且 Will Delay Interval 等于 0，遗嘱消息将在网络连接关闭时立即发布
2. 连接意外关闭且 Will Delay Interval 大于 0，遗嘱消息将被延迟发布，最大延迟时间取决于 Will Delay Interval 与 Session Expiry Interval 谁先到期：
   1. 客户端未能在 Will Delay Interval 或 Session Expiry Interval 到期前恢复连接，遗嘱消息将被发布。
   2. 在 Will Delay Interval 或 Session Expiry Interval 到期前
      1. 客户端指定 Clean Start 为 0 恢复连接，遗嘱消息将不会被发布。
      2. 客户端指定 Clean Start 为 1 恢复连接，遗嘱消息将因为 **现有会话结束** 而被立即发布。

如果现有网络连接尚未断开，但客户端使用相同 Client ID 发起新的连接，服务端会向现有的网络连接发送一个 Reason Code 为 0x8E（Session Taken Over）的 DISCONNECT 报文然后关闭它。这种情况在网络不佳时非常容易出现，但也属于连接意外关闭。

现在，请思考这样一个问题：如果现有的网络连接的 Session Expiry Interval 等于 0，Will Delay Interval 大于 0，那么当客户端指定 Clean Start 为 0 发起新的网络连接时服务端是否会发送遗嘱消息？

答案是遗嘱消息将在现有的网络连接断开时被立即发布。

当服务端关闭现有的网络连接，由于 Session Expiry Interval 为 0，会话也将立即结束。虽然 Clean Start 设置为 0，但服务端将为新的网络连接创建了一个新的会话。所以遗嘱消息将因为会话结束而被发布，即满足了上面所列情形中的 2.1 而不是 2.2.1。。

## 遗嘱消息使用技巧 <a href="#yi-zhu-xiao-xi-shi-yong-ji-qiao" id="yi-zhu-xiao-xi-shi-yong-ji-qiao"></a>

### 与保留消息一起使用 <a href="#yu-bao-liu-xiao-xi-yi-qi-shi-yong" id="yu-bao-liu-xiao-xi-yi-qi-shi-yong"></a>

服务端一旦发布了遗嘱消息，就会将它从会话中删除。如果关心此遗嘱消息的客户端不在线，那么它就错过了这条遗嘱消息。

为了避免这种情况，我们可以将遗嘱消息设置为保留消息，这样遗嘱消息在被发布后，还会以保留消息的形式存储在服务端中，客户端可以在任何时候获取这条遗嘱消息。

如果更进一步，我们还可以实现对指定客户端的状态监控。

让客户端 `myclient` 在每次连接时都指定一个主题为 `myclient/status`，有效载荷为 `offline` 并且设置了 Will Retain 标志的遗嘱消息。每当连接成功，就向主题 `myclient/status` 发布一个有效载荷为 `online` 的保留消息。这样，我们就可以随时订阅主题 `myclient/status`，来获取客户端 `myclient` 的最新状态。

### 会话过期通知 <a href="#hui-hua-guo-qi-tong-zhi" id="hui-hua-guo-qi-tong-zhi"></a>

通过设置一个大于 Session Expiry Interval 的 Will Delay Interval，服务端可以以遗嘱消息的形式发出会话过期通知。这对于一些更关心会话过期而不是网络连接中断的应用更加有用。即便是主动下线，客户端可以发送一个 Reason Code 为 0x04 的 DISCONNECT 报文要求服务端仍然发送遗嘱消息。

