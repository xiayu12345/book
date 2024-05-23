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

# 初识URl

HTTP 请求的内容通称为"资源"。”资源“这一概念非常宽泛，它可以是一份文档，一张图片，或所有其他你能够想到的格式。每个资源都由一个 (URI) 来进行标识。

一般情况下，资源的名称和位置由同一个 URL（统一资源定位符，它是 URI 的一种）来标识。也有某些特殊情况，资源的名称和位置由不同的 URI 进行标识：例如，待请求的资源希望客户端从另外一个位置访问它。我们可以使用一个特定的首部字段，`Alt-Svc`，来指示这种情况。

## URLs 与 URNs <a href="#urls-yu-urns" id="urls-yu-urns"></a>

#### URLs <a href="#urls" id="urls"></a>

URI 的最常见形式是统一资源定位符 (URL)，它也被称为 _Web 地址_。

```
https://developer.mozilla.org
https://developer.mozilla.org/zh-CN/docs/Learn/
https://developer.mozilla.org/zh-CN/search?q=URL
```

在浏览器的地址栏中输入上述任一地址，浏览器就会加载相应的网页（资源）。

URL 由多个必须或可选的组件构成。下面给出了一个复杂的 URL：

```
http://www.example.com:80/path/to/myfile.html?key1=value1&key2=value2#SomewhereInTheDocument
```

### [URNs](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics\_of\_HTTP/Identifying\_resources\_on\_the\_Web#urns) <a href="#urns" id="urns"></a>

URN 是另一种形式的 URI，它通过特定命名空间中的唯一名称来标识资源。

```
urn:isbn:9780141036144
urn:ietf:rfc:7230
```

上面两个 URN 标识了下面的资源：

* 乔治·奥威尔所著的《1984》
* IETF 规范 7230，超文本传输 协议 (HTTP/1.1)：Message Syntax and Routing.

## [统一资源标识符的语法 (URI)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics\_of\_HTTP/Identifying\_resources\_on\_the\_Web#%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E6%A0%87%E8%AF%86%E7%AC%A6%E7%9A%84%E8%AF%AD%E6%B3%95\_uri) <a href="#tong-yi-zi-yuan-biao-shi-fu-de-yu-fa-uri" id="tong-yi-zi-yuan-biao-shi-fu-de-yu-fa-uri"></a>

### [方案或协议](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics\_of\_HTTP/Identifying\_resources\_on\_the\_Web#%E6%96%B9%E6%A1%88%E6%88%96%E5%8D%8F%E8%AE%AE) <a href="#fang-an-huo-xie-yi" id="fang-an-huo-xie-yi"></a>

![Protocol](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics\_of\_HTTP/Identifying\_resources\_on\_the\_Web/mdn-url-protocol@x2.png)

`http://`告诉浏览器使用何种协议。对于大部分 Web 资源，通常使用 HTTP 协议或其安全版本，HTTPS 协议。另外，浏览器也知道如何处理其他协议。例如， `mailto:` 协议指示浏览器打开邮件客户端；`ftp:`协议指示浏览器处理文件传输。常见的方案有：

| 方案          | 描述                  |
| ----------- | ------------------- |
| data        | Data URIs           |
| file        | 指定主机上文件的名称          |
| ftp         | 文件传输协议              |
| http/https  | 超文本传输 协议／安全的超文本传输协议 |
| mailto      | 电子邮件地址              |
| ssh         | 安全 shell            |
| tel         | 电话                  |
| urn         | 统一资源名称              |
| view-source | 资源的源代码              |
| ws/wss      | （加密的）WebSocket 连接   |

### [主机](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics\_of\_HTTP/Identifying\_resources\_on\_the\_Web#%E4%B8%BB%E6%9C%BA) <a href="#zhu-ji" id="zhu-ji"></a>

![Domaine Name](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics\_of\_HTTP/Identifying\_resources\_on\_the\_Web/mdn-url-domain@x2.png)

`www.example.com` 既是一个域名，也代表管理该域名的机构。它指示了需要向网络上的哪一台主机发起请求。当然，也可以直接向主机的 IP address 地址发起请求。但直接使用 IP 地址的场景并不常见。

### 端口 <a href="#duan-kou" id="duan-kou"></a>

![Port](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics\_of\_HTTP/Identifying\_resources\_on\_the\_Web/mdn-url-port@x2.png)

`:80` 是端口。它表示用于访问 Web 服务器上资源的技术“门”。如果访问的该 Web 服务器使用 HTTP 协议的标准端口（HTTP 为 80，HTTPS 为 443）授予对其资源的访问权限，则通常省略此部分。否则端口就是 URI 必须的部分。

### [路径](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics\_of\_HTTP/Identifying\_resources\_on\_the\_Web#%E8%B7%AF%E5%BE%84) <a href="#lu-jing" id="lu-jing"></a>

![Path to the file](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics\_of\_HTTP/Identifying\_resources\_on\_the\_Web/mdn-url-path@x2.png)

`/path/to/myfile.html` 是 Web 服务器上资源的路径。在 Web 的早期，类似这样的路径表示 Web 服务器上的物理文件位置。现在，它主要是由没有任何物理实体的 Web 服务器抽象处理而成的。

### [查询](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics\_of\_HTTP/Identifying\_resources\_on\_the\_Web#%E6%9F%A5%E8%AF%A2) <a href="#cha-xun" id="cha-xun"></a>

![Parameters](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics\_of\_HTTP/Identifying\_resources\_on\_the\_Web/mdn-url-parameters@x2.png)

`?key1=value1&key2=value2` 是提供给 Web 服务器的额外参数。这些参数是用 & 符号分隔的键/值对列表。Web 服务器可以在将资源返回给用户之前使用这些参数来执行额外的操作。每个 Web 服务器都有自己的参数规则，想知道特定 Web 服务器如何处理参数的唯一可靠方法是询问该 Web 服务器所有者。

### [片段](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics\_of\_HTTP/Identifying\_resources\_on\_the\_Web#%E7%89%87%E6%AE%B5) <a href="#pian-duan" id="pian-duan"></a>

![Anchor](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics\_of\_HTTP/Identifying\_resources\_on\_the\_Web/mdn-url-anchor@x2.png)

`#SomewhereInTheDocument` 是资源本身的某一部分的一个锚点。锚点代表资源内的一种“书签”，它给予浏览器显示位于该“加书签”点的内容的指示。例如，在 HTML 文档上，浏览器将滚动到定义锚点的那个点上；在视频或音频文档上，浏览器将转到锚点代表的那个时间。值得注意的是 # 号后面的部分，也称为片段标识符，永远不会与请求一起发送到服务器。

## [示例](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics\_of\_HTTP/Identifying\_resources\_on\_the\_Web#%E7%A4%BA%E4%BE%8B) <a href="#shi-li" id="shi-li"></a>

```
https://developer.mozilla.org/zh-CN/docs/Learn
tel:+1-816-555-1212
git@github.com:mdn/browser-compat-data.git
ftp://example.org/resource.txt
urn:isbn:9780141036144
```
