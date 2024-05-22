# MQTT 5.0：7 项新功能

MQTT（Message Queuing Telemetry Transport）是一种为资源有限的设备和低带宽、高延迟的网络设计的轻量级消息传输协议。它特别适用于需要较小代码占用空间或网络带宽有限的远程连接。

MQTT 5.0 是该协议的最新版本，相比之前的版本有了很多改进。新增功能包括：原因代码、会话过期间隔、主题别名、用户属性、订阅选项、请求/响应功能和共享订阅等。本文将探讨这些新功能，介绍主流 Broker 和客户端 SDK 对 MQTT 5.0 协议的支持情况，以及从 MQTT 3.1.1 迁移到 MQTT 5.0 时的一些关键注意事项。

### MQTT 5.0 的发展历程 <a href="#mqtt50-de-fa-zhan-li-cheng" id="mqtt50-de-fa-zhan-li-cheng"></a>

MQTT 是在上世纪 90 年代末由 IBM 的 Andy Stanford-Clark 博士和 Arcom（现 Eurotech）的 Arlen Nipper 开发，用于通过卫星网络监测石油管道。最初的 MQTT 3.1 版本非常轻量级和易于实现，适用于各种物联网设备。

MQTT 3.1.1 于 2014 年发布，是一个 OASIS 标准，其中包括一些微调，增强了其清晰性和互操作性。它能够在资源有限的网络上高效地传输消息，因此在物联网应用中广受欢迎。

然而，随着物联网行业的发展，应用的需求也在不断变化。为了适应这些新的需求，在 2019 年发布了 MQTT 5.0，其中加入了一些新功能。MQTT 5.0 也因此能够更好地满足现代物联网应用的复杂需求。

### MQTT 5.0 的 7 个新功能 <a href="#mqtt50-de-7-ge-xin-gong-neng" id="mqtt50-de-7-ge-xin-gong-neng"></a>

#### 1. 原因代码：了解断开连接或失败原因 <a href="#id-1-yuan-yin-dai-ma-le-jie-duan-kai-lian-jie-huo-shi-bai-yuan-yin" id="id-1-yuan-yin-dai-ma-le-jie-duan-kai-lian-jie-huo-shi-bai-yuan-yin"></a>

MQTT 5.0 与之前的版本不同，它能够为每个确认报文提供一个原因代码，帮助我们了解断开连接或发生故障的原因。

这一改进有利于故障排除和更精细的错误处理。比如，如果客户端连接服务器失败，服务器会返回一个原因代码，解释连接不成功的原因。这可能是各种原因导致的，比如登录凭证错误或者服务器不在线。

> 详细请参考：[MQTT 5.0 Reason Code 介绍与使用速查表](../mqtt-jin-jie/yuan-yin-ma-biao-ji-su-cha-biao-reason-codes-quick-reference.md)

#### 2. 会话过期间隔：管理会话的生命周期 <a href="#id-2-hui-hua-guo-qi-jian-ge-guan-li-hui-hua-de-sheng-ming-zhou-qi" id="id-2-hui-hua-guo-qi-jian-ge-guan-li-hui-hua-de-sheng-ming-zhou-qi"></a>

这个功能允许客户端指定服务器在客户端断开连接后应将会话保持多长时间。在之前的 MQTT 版本中，会话要么在断开连接后立即结束，要么无限期地保持下去。使用 MQTT 5.0，您可以指定一个具体的时间段，在断开连接后，会话仍然有效。这样可以更灵活地管理会话的生命周期，并节省服务器的资源。

> 详细请参考：[Clean Start 与 Session Expiry Interval 介绍与示例](../mqtt-ru-men/hui-hua-xiang-jie.md)

#### 3. 主题别名：减少消息头部的开销 <a href="#id-3-zhu-ti-bie-ming-jian-shao-xiao-xi-tou-bu-de-kai-xiao" id="id-3-zhu-ti-bie-ming-jian-shao-xiao-xi-tou-bu-de-kai-xiao"></a>

MQTT 5.0 引入了主题别名，以减少消息头部的开销。在之前的版本中，每个消息都需要包含主题名称，导致数据包过大。

使用主题别名，可以为主题分配一个简短的数字别名。这个别名可以在后续的消息中替代完整的主题名称，大大减少了 MQTT 头部的大小，从而节省了网络带宽。

> 详细请参考：[主题别名 - MQTT 5.0 新特性](../mqtt-jin-jie/zhu-ti-bie-ming-topic-alias.md)

#### 4. 用户属性：MQTT 头部中的自定义元数据 <a href="#id-4-yong-hu-shu-xing-mqtt-tou-bu-zhong-de-zi-ding-yi-yuan-shu-ju" id="id-4-yong-hu-shu-xing-mqtt-tou-bu-zhong-de-zi-ding-yi-yuan-shu-ju"></a>

这个功能让用户可以在 MQTT 报文的头部添加自定义的元数据。这对于需要在 MQTT 消息中携带额外信息的应用非常有用，比如消息的时间戳、设备位置或其他应用相关的数据。用户属性增加了 MQTT 消息传输的灵活性和控制力。

> 详细请参考：[用户属性 - MQTT 5.0 新特性](../mqtt-jin-jie/yong-hu-shu-xing-user-properties.md)

#### 5. 订阅选项：细粒度的订阅控制 <a href="#id-5-ding-yue-xuan-xiang-xi-li-du-de-ding-yue-kong-zhi" id="id-5-ding-yue-xuan-xiang-xi-li-du-de-ding-yue-kong-zhi"></a>

MQTT 5.0 让客户端可以指定如何接收每个订阅主题的消息。比如，客户端可以指定他们是否接收某个订阅的保留消息，或者是否接收和订阅具有相同 QoS（服务质量）级别的消息。

> 详细教程请参考：[MQTT 订阅选项介绍与示例](../mqtt-jin-jie/ding-yue-xuan-xiang-subscription-options.md)

#### 6. 请求/响应：允许客户端回复指定主题 <a href="#id-6-qing-qiu-xiang-ying-yun-xu-ke-hu-duan-hui-fu-zhi-ding-zhu-ti" id="id-6-qing-qiu-xiang-ying-yun-xu-ke-hu-duan-hui-fu-zhi-ding-zhu-ti"></a>

请求/响应功能让客户端可以指定一个主题，供服务器直接回复。

在早期的 MQTT 版本中，如果客户端想要回复一条消息，它必须把回复发布到一个主题，而原始发送者必须订阅那个主题才能收到回复。使用 MQTT 5.0 的请求/响应功能，客户端和服务器之间的通信变得更高效和简洁。

> 详细请参考：[请求 / 响应介绍与示例](../mqtt-jin-jie/qing-qiu-xiang-ying-requsetresponse.md)

#### 7. 共享订阅：订阅者负载均衡功能 <a href="#id-7-gong-xiang-ding-yue-ding-yue-zhe-fu-zai-jun-heng-gong-neng" id="id-7-gong-xiang-ding-yue-ding-yue-zhe-fu-zai-jun-heng-gong-neng"></a>

这个功能让多个客户端可以共享一个订阅。当一条消息发布到一个共享主题时，服务器会把消息分发给共享订阅中的某个客户端，从而实现消息的负载均衡。

这个功能在有多个服务实例运行，并且想要平均分配工作量的场景中非常有用。

> 详细请参考：[共享订阅介绍与示例](../mqtt-jin-jie/gong-xiang-ding-yue-shared-subscriptions.md)



#### 8. 处理安全问题 <a href="#id-3-chu-li-an-quan-wen-ti" id="id-3-chu-li-an-quan-wen-ti"></a>

MQTT 5.0 在带来多项改进的同时，也引入了一些新的安全风险。比如，有了新的用户属性功能，客户端可以向 Broker 发送自定义数据。这是一个强大的功能，但如果使用不当可能会造成问题。因此，要从安全的角度检查所有的新功能。

可以采取以下措施来提高安全性：使用新的增强验证功能来加强安全性，限制客户端只能发送必要的用户属性，以及持续监测任何可疑的活动。

> 要了解更多信息，请参考详细的[ MQTT 安全指南](../mqtt-an-quan-zhi-nan.md)。
