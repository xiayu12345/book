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

# 创建连接时如何设置参数

## MQTT 连接的基本概念 <a href="#mqtt-lian-jie-de-ji-ben-gai-nian" id="mqtt-lian-jie-de-ji-ben-gai-nian"></a>

MQTT 连接由客户端向服务器端发起。任何运行了 MQTT 客户端库的程序或设备都是一个 MQTT 客户端，而 MQTT 服务器则负责接收客户端发起的连接，并将客户端发送的消息转发到另外一些符合条件的客户端。

客户端与服务器建立网络连接后，需要先发送一个 `CONNECT` 数据包给服务器。服务器收到 `CONNECT` 包后会回复一个 `CONNACK` 给客户端，客户端收到 `CONNACK` 包后表示 MQTT 连接建立成功。如果客户端在超时时间内未收到服务器的 `CONNACK` 数据包，就会主动关闭连接。

大多数场景下，MQTT 通过 TCP/IP 协议进行网络传输，但是 MQTT 同时也支持通过 WebSocket 或者 UDP 进行网络传输。

## MQTT over TCP <a href="#mqtt-over-tcp" id="mqtt-over-tcp"></a>

TCP/IP 应用广泛，是一种面向连接的、可靠的、基于字节流的传输层通信协议。它通过 ACK 确认和重传机制，能够保证发送的所有字节在接收时是完全一样的，并且字节顺序也是正确的。

MQTT 通常基于 TCP 进行网络通信，它继承了 TCP 的很多优点，能稳定运行在低带宽、高延时、及资源受限的环境下。

## MQTT over WebSocket <a href="#mqtt-over-websocket" id="mqtt-over-websocket"></a>

近年来随着 Web 前端的快速发展，浏览器新特性层出不穷，越来越多的应用可以在浏览器端通过浏览器渲染引擎实现，Web 应用的即时通信方式 WebSocket 也因此得到了广泛的应用。

很多物联网应用需要以 Web 的方式被使用，比如很多设备监控系统需要使用浏览器实时显示设备数据。但是浏览器是基于 HTTP 协议传输数据的，也就无法使用 MQTT over TCP。

MQTT 协议在创建之初便考虑到了 Web 应用的重要性，它支持通过 MQTT over WebSocket 的方式进行 MQTT 通信。

## MQTT 连接参数的使用 <a href="#mqtt-lian-jie-can-shu-de-shi-yong" id="mqtt-lian-jie-can-shu-de-shi-yong"></a>

#### 连接地址 <a href="#lian-jie-di-zhi" id="lian-jie-di-zhi"></a>

MQTT 的连接地址通常包含 ：服务器 IP 或者域名、服务器端口、连接协议。

**基于 TCP 的 MQTT 连接**

`mqtt` 是普通的 TCP 连接，端口一般为 1883。

`mqtts` 是基于 TLS/SSL 的安全连接，端口一般为 8883。

比如 `mqtt://broker.emqx.io:1883` 是一个基于普通 TCP 的 MQTT 连接地址。

**基于 WebSocket 的连接**

`ws` 是普通的 WebSocket 连接，端口一般为 8083。

`wss` 是基于 WebSocket 的安全连接，端口一般为 8084。

当使用 WebSocket 连接时，连接地址还需要包含 Path。比如 `ws://broker.emqx.io:8083/mqtt` 是一个基于 WebSocket 的 MQTT 连接地址。

### 客户端 ID（Client ID） <a href="#ke-hu-duan-idclientid" id="ke-hu-duan-idclientid"></a>

MQTT 服务器使用 Client ID 识别客户端，连接到服务器的每个客户端都必须要有唯一的 Client ID。Client ID 的长度通常为 1 至 23 个字节的 UTF-8 字符串。

**如果客户端使用一个重复的 Client ID 连接至服务器，将会把已使用该 Client ID 连接成功的客户端踢下线。**

### 用户名与密码（Username & Password） <a href="#yong-hu-ming-yu-mi-ma-usernameamppassword" id="yong-hu-ming-yu-mi-ma-usernameamppassword"></a>

MQTT 协议可以通过用户名和密码来进行相关的认证和授权，但是如果此信息未加密，则用户名和密码将以明文方式传输。如果设置了用户名与密码认证，那么最好要使用 `mqtts` 或 `wss` 协议。

大多数 MQTT 服务器默认为匿名认证，匿名认证时用户名与密码设置为空字符串即可。

### 连接超时（Connect Timeout） <a href="#lian-jie-chao-shi-connecttimeout" id="lian-jie-chao-shi-connecttimeout"></a>

连接超时时长，收到服务器连接确认前的等待时间，等待时间内未收到连接确认则为连接失败。

### 保活周期（Keep Alive） <a href="#bao-huo-zhou-qi-keepalive" id="bao-huo-zhou-qi-keepalive"></a>

保活周期，是一个以秒为单位的时间间隔。客户端在无报文发送时，将按 Keep Alive 设定的值定时向服务端发送心跳报文，确保连接不被服务端断开。

在连接建立成功后，如果服务器没有在 Keep Alive 的 1.5 倍时间内收到来自客户端的任何包，则会认为和客户端之间的连接出现了问题，此时服务器便会断开和客户端的连接。

更多细节可查看：[MQTT 协议中的 Keep Alive 机制](../mqtt-jin-jie/bao-chi-lian-jie-keep-alive.md)。

### 清除会话（Clean Session） <a href="#qing-chu-hui-hua-cleansession" id="qing-chu-hui-hua-cleansession"></a>

为 `false` 时表示创建一个持久会话，在客户端断开连接时，会话仍然保持并保存离线消息，直到会话超时注销。为 `true` 时表示创建一个新的临时会话，在客户端断开时，会话自动销毁。

持久会话避免了客户端掉线重连后消息的丢失，并且免去了客户端连接后重复的订阅开销。这一功能在带宽小，网络不稳定的物联网场景中非常实用。

服务器为持久会话保存的消息数量取决于服务器的配置。

> **注意：** 持久会话恢复的前提是客户端使用固定的 Client ID 再次连接，如果 Client ID 是动态的，那么连接成功后将会创建一个新的持久会话。

### 遗嘱消息（Last Will） <a href="#yi-zhu-xiao-xi-lastwill" id="yi-zhu-xiao-xi-lastwill"></a>

遗嘱消息是 MQTT 为那些可能出现**意外断线**的设备提供的将**遗嘱**优雅地发送给其他客户端的能力。设置了遗嘱消息消息的 MQTT 客户端异常下线时，MQTT 服务器会发布该客户端设置的遗嘱消息。

> **意外断线包括**：因网络故障，连接被服务端关闭；设备意外掉电；设备尝试进行不被允许的操作而被服务端关闭连接等。

遗嘱消息可以看作是一个简化版的 MQTT 消息，它也包含 Topic、Payload、QoS、Retain 等信息。

* 当设备意外断线时，遗嘱消息将被发送至遗嘱 Topic；
* 遗嘱 Payload 是待发送的消息内容；
* 遗嘱 QoS 与普通 MQTT 消息的 QoS 一致，详细请见[MQTT QoS（服务质量）介绍](li-jie-qos.md)。
* 遗嘱 Retain 为 `true` 时表明遗嘱消息是保留消息。MQTT 服务器会为每个主题存储最新一条保留消息，以方便消息发布后才上线的客户端在订阅主题时仍可以接收到该消息。详细请见[MQTT 保留消息是什么？如何使用？](../mqtt-jin-jie/bao-liu-xiao-xi-retained-messages.md)

更多关于遗嘱消息的介绍可查看博客：[MQTT 遗嘱消息（Will Message）的使用](../mqtt-jin-jie/yi-zhu-xiao-xi-will-messages.md)。

### 协议版本 <a href="#xie-yi-ban-ben" id="xie-yi-ban-ben"></a>

使用较多的 MQTT 协议版本有 MQTT v3.1、MQTT v3.1.1 及 MQTT v5.0。目前，MQTT 5.0 已成为绝大多数物联网企业的首选协议，我们建议初次接触 MQTT 的开发者直接使用该版本。

## MQTT 5.0 新增连接参数 <a href="#mqtt50-xin-zeng-lian-jie-can-shu" id="mqtt50-xin-zeng-lian-jie-can-shu"></a>

### **Clean Start & Session Expiry Interval**

MQTT 5.0 中将 Clean Session 拆分成了 Clean Start 与 Session Expiry Interval。

Clean Start 用于指定连接时是创建一个全新的会话还是尝试复用一个已存在的会话。为 `true` 时表示必须丢弃任何已存在的会话，并创建一个全新的会话；为 `false` 时表示必须使用与 Client ID 关联的会话来恢复与客户端的通信（除非会话不存在）。

Session Expiry Interval 用于指定网络连接断开后会话的过期时间。设置为 0 或未设置，表示断开连接时会话即到期；设置为大于 0 的数值，则表示会话在网络连接关闭后会保持多少秒；设置为 0xFFFFFFFF 表示会话永远不会过期。

### **连接属性（Connect Properties）**

MQTT 5.0 还新引入了连接属性的概念，进一步增强了协议的可扩展性。更多细节可查看：[MQTT 5.0 连接属性](../mqtt5.0/mqtt-5.0-lian-jie-shu-xing.md)。
