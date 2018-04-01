## 深入学习 Node.js Http 基础篇

<!-- TOC -->

- [深入学习 Node.js Http 基础篇](#深入学习-nodejs-http-基础篇)
    - [B/S 结构定义](#bs-结构定义)
    - [URI (统一资源标志符)](#uri-统一资源标志符)
        - [URI 文法](#uri-文法)
    - [MIME](#mime)
        - [文件格式](#文件格式)
    - [HTTP 协议](#http-协议)
    - [HTTP 协议主要特点](#http-协议主要特点)
    - [HTTP 请求报文](#http-请求报文)
        - [请求报文示例](#请求报文示例)
        - [请求行](#请求行)
        - [请求头](#请求头)
        - [空行（请求）](#空行请求)
        - [请求体](#请求体)
    - [HTTP 响应报文](#http-响应报文)
        - [响应报文示例](#响应报文示例)
        - [状态行](#状态行)
        - [响应头](#响应头)
        - [空行（响应）](#空行响应)
        - [响应体](#响应体)
    - [HTTP 请求方法](#http-请求方法)
    - [HTTP 状态码](#http-状态码)
    - [参考资源](#参考资源)

<!-- /TOC -->

### B/S 结构定义

> **浏览器-服务器（Browser/Server）结构**，简称[B/S结构](https://zh.wikipedia.org/wiki/B/S%E7%BB%93%E6%9E%84)，与[C/S结构](https://zh.wikipedia.org/wiki/C/S%E7%BB%93%E6%9E%84)不同，其客户端不需要安装专门的[软件](https://zh.wikipedia.org/wiki/%E8%BD%AF%E4%BB%B6)，只需要[浏览器](https://zh.wikipedia.org/wiki/%E6%B5%8F%E8%A7%88%E5%99%A8)即可，浏览器通过[Web](https://zh.wikipedia.org/wiki/Web)[服务器](https://zh.wikipedia.org/wiki/%E6%9C%8D%E5%8A%A1%E5%99%A8)与[数据库](https://zh.wikipedia.org/wiki/%E6%95%B0%E6%8D%AE%E5%BA%93)进行交互，可以方便的在不同平台下工作；服务器端可采用高性能[计算机](https://zh.wikipedia.org/wiki/%E8%AE%A1%E7%AE%97%E6%9C%BA)，并安装[Oracle](https://zh.wikipedia.org/wiki/Oracle)、[Sybase](https://zh.wikipedia.org/wiki/Sybase)、[Informix](https://zh.wikipedia.org/w/index.php?title=Informix&action=edit&redlink=1)等大型数据库。B/S结构简化了客户端的工作，它是随着[Internet](https://zh.wikipedia.org/wiki/Internet)技术兴起而产生的，对C/S技术的改进，但该结构下服务器端的工作较重，对服务器的性能要求更高。—— [维基百科](https://zh.wikipedia.org/wiki/%E6%B5%8F%E8%A7%88%E5%99%A8-%E6%9C%8D%E5%8A%A1%E5%99%A8)

![B/S结构](./images/http-resource-1.png)

(图片资源来源于网络)

### URI (统一资源标志符)

> 在[电脑](https://zh.wikipedia.org/wiki/%E9%9B%BB%E8%85%A6)术语中，**统一资源标识符**（英语：Uniform Resource Identifier，或**URI**)是一个用于[标识](https://zh.wikipedia.org/wiki/%E6%A0%87%E8%AF%86)某一[互联网](https://zh.wikipedia.org/wiki/%E4%BA%92%E8%81%94%E7%BD%91)[资源](https://zh.wikipedia.org/wiki/%E8%B5%84%E6%BA%90)名称的[字符串](https://zh.wikipedia.org/wiki/%E5%AD%97%E7%AC%A6%E4%B8%B2)。 该种标识允许用户对网络中（一般指[万维网](https://zh.wikipedia.org/wiki/%E4%B8%87%E7%BB%B4%E7%BD%91)）的资源通过特定的[协议](https://zh.wikipedia.org/wiki/%E5%8D%8F%E8%AE%AE)进行交互操作。URI的最常见的形式是[统一资源定位符](https://zh.wikipedia.org/wiki/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E5%AE%9A%E4%BD%8D%E7%AC%A6)（URL），经常指定为非正式的网址。更罕见的用法是[统一资源名称](https://zh.wikipedia.org/wiki/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E5%90%8D%E7%A7%B0)（URN），其目的是通过提供一种途径。用于在特定的[命名空间](https://zh.wikipedia.org/wiki/%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4)资源的标识，以补充网址。—— 维基百科

#### URI 文法

> URI文法由[URI协议](https://zh.wikipedia.org/w/index.php?title=URI%E5%8D%8F%E8%AE%AE&action=edit&redlink=1)名（例如“[`http`](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)”，“[`ftp`](https://zh.wikipedia.org/wiki/%E6%96%87%E4%BB%B6%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)”，“[`mailto`](https://zh.wikipedia.org/wiki/%E7%94%B5%E5%AD%90%E9%82%AE%E4%BB%B6)”或“`file`”），一个[冒号](https://zh.wikipedia.org/wiki/%E5%86%92%E5%8F%B7)，和协议对应的内容所构成。特定的协议定义了协议内容的语法和[语义](https://zh.wikipedia.org/wiki/%E8%AF%AD%E4%B9%89)，而所有的协议都必须遵循一定的URI文法通用规则，亦即为某些专门目的保留部分特殊字符。—— [维基百科](https://zh.wikipedia.org/wiki/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E6%A0%87%E5%BF%97%E7%AC%A6)

下面展示了 URI 例子及它们的组成部分：



```shell
                       权限                 路径
        ┌───────────────┴───────────────┐┌───┴────┐
  abc://username:password@example.com:123/path/data?key=value&key2=value2#fragid1
  └┬┘   └───────┬───────┘ └────┬────┘ └┬┘           └─────────┬─────────┘ └──┬──┘
  协议        用户信息         主机名    端口                  查询参数          片段
```

### MIME

> MIME (Multipurpose Internet Mail Extensions) 多用途互联网邮件扩展类型。是设定某种[扩展名](http://baike.baidu.com/item/%E6%89%A9%E5%B1%95%E5%90%8D)的[文件](http://baike.baidu.com/item/%E6%96%87%E4%BB%B6)用一种[应用程序](http://baike.baidu.com/item/%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F)来打开的方式类型，当该扩展名文件被访问的时候，[浏览器](http://baike.baidu.com/item/%E6%B5%8F%E8%A7%88%E5%99%A8)会自动使用指定应用程序来打开。多用于指定一些[客户端](http://baike.baidu.com/item/%E5%AE%A2%E6%88%B7%E7%AB%AF)[自定义](http://baike.baidu.com/item/%E8%87%AA%E5%AE%9A%E4%B9%89)的[文件名](http://baike.baidu.com/item/%E6%96%87%E4%BB%B6%E5%90%8D)，以及一些媒体文件打开方式。 —— [百度百科](https://baike.baidu.com/item/MIME/2900607)

#### 文件格式

每个 MIME 类型由两部分组成，前面是数据的大类别，例如声音 audio、图象 image 等，后面定义具体的种类。

常见的 MIME 类型有：

| 资源名称      | 后缀    | 类型              |
| --------- | ----- | --------------- |
| 超文本标记语言文本 | .html | text/html       |
| xml文档     | .xml  | text/xml        |
| 普通文本      | .txt  | text/plain      |
| PNG图像     | .png  | image/png       |
| PDF文档     | .pdf  | application/pdf |

了解更多的 MIME 类型 - [互联网媒体类型](https://zh.wikipedia.org/wiki/%E4%BA%92%E8%81%94%E7%BD%91%E5%AA%92%E4%BD%93%E7%B1%BB%E5%9E%8B)。

### HTTP 协议

> **超文本传输协议**（[英文](https://zh.wikipedia.org/wiki/%E8%8B%B1%E6%96%87)：**HyperText Transfer Protocol**，[缩写](https://zh.wikipedia.org/wiki/%E7%B8%AE%E5%AF%AB)：**HTTP**）是[互联网](https://zh.wikipedia.org/wiki/%E7%B6%B2%E9%9A%9B%E7%B6%B2%E8%B7%AF)上应用最为广泛的一种[网络协议](https://zh.wikipedia.org/wiki/%E7%BD%91%E7%BB%9C%E5%8D%8F%E8%AE%AE)。设计HTTP最初的目的是为了提供一种发布和接收[HTML](https://zh.wikipedia.org/wiki/HTML)页面的方法。通过HTTP或者HTTPS协议请求的资源由[统一资源标识符](https://zh.wikipedia.org/wiki/%E7%B5%B1%E4%B8%80%E8%B3%87%E6%BA%90%E6%A8%99%E8%AD%98%E7%AC%A6)（Uniform Resource Identifiers，URI）来标识。—— 维基百科

HTTP 协议是基于请求与相应，具体如下图所示：

![http-protocol](./images/http-resource-2.png)

(图片资源来源于网络)

### HTTP 协议主要特点

- 简单快速：当客户端向服务器端发送请求时，只是简单的填写请求路径和请求方法即可，然后就可以通过浏览器或其他方式将该请求发送就行了。
- 灵活：HTTP 协议允许客户端和服务器端传输任意类型任意格式的数据对象。
- 无连接：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接，采用这种方式可以节省传输时间。（当今多数服务器支持 Keep-Alive 功能，使用服务器支持长连接，解决无连接的问题）。
- 无状态：无状态是指协议对于事务处理没有记忆能力，服务器不知道客户端是什么状态。即客户端发送 HTTP请求后，服务器根据请求，会给我们发送数据，发送完后，不会记录信息。（使用 cookie 机制可以保持 session，解决无状态的问题）。

### HTTP 请求报文

HTTP 请求报文由**请求行**、**请求头**、**空行** 和 **请求体(请求数据)** 4 个部分组成，如下图所示：

![request-message](./images/http-resource-3.png)

(图片资源来源于网络)

#### 请求报文示例

```
GET / HTTP/1.1
Host: www.baidu.com
Connection: keep-alive
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.110 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Encoding: gzip, deflate, sdch, br
Accept-Language: zh-CN,zh;q=0.8,en;q=0.6,id;q=0.4
Cookie: PSTM=1490844191; BIDUPSID=2145FF54639208435F60E1E165379255; BAIDUID=CFA344942EE2E0EE081D8B13B5C847F9:FG=1;
```

#### 请求行

请求行由请求方法、URL 和 HTTP 协议版本组成，它们之间用空格分开。

```shell
GET / HTTP/1.1
```

#### 请求头

请求头由 `key-value` 对组成，每行一对，key (键) 和 value (值)用英文冒号 `:` 分隔。请求头通知服务器有关于客户端请求的信息，典型的请求头有：

- User-Agent：用户代理信息 - Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_3) AppleWebKit/537.36 ...
- Accept：客户端可识别的内容类型列表 - text/html,application/xhtml+xml,application/xml
- Accept-Language：客户端可接受的自然语言 -  zh-CN,zh;q=0.8,en;q=0.6,id;q=0.4
- Accept-Encoding：客户端可接受的编码压缩格式 - gzip, deflate, sdch, br
- Host：请求的主机名，允许多个域名同处一个IP地址，即虚拟主机 - `www.baidu.com`
- connection：连接方式 
  - close：告诉WEB服务器或代理服务器，在完成本次请求的响应后，断开连接
  - keep-alive：告诉WEB服务器或代理服务器。在完成本次请求的响应后，保持连接，以等待后续请求
- Cookie：存储于客户端扩展字段，向同一域名的服务端发送属于该域的cookie - PSTM=1490844191; BIDUPSID=2145FF54639208435F60E1E165379255;

#### 空行（请求）

最后一个请求头之后是一个空行，发送回车符和换行符，通知服务器以下不再有请求头。

#### 请求体

请求数据不在 GET 方法中使用，而是在 POST 方法中使用。与请求数据相关的最常使用的请求头是 Content-Type和 Content-Length。

### HTTP 响应报文

HTTP响应报文由**状态行、响应头、空行和响应体** 4 个部分组成，如下图所示：

![response-message](./images/http-resource-4.png)

(图片资源来源于网络)

#### 响应报文示例

```
HTTP/1.1 200 OK
Server: bfe/1.0.8.18
Date: Thu, 30 Mar 2017 12:28:00 GMT
Content-Type: text/html; charset=utf-8
Connection: keep-alive
Cache-Control: private
Expires: Thu, 30 Mar 2017 12:27:43 GMT
Set-Cookie: BDSVRTM=0; path=/
```

#### 状态行

状态行格式： HTTP-Version Status-Code Reason-Phrase CRLF

- HTTP-Version：HTTP 协议版本。
- Status-Code：状态码。
- Reason-Phrase：状态码描述。
- CRLF：回车/换行符。

#### 响应头

响应头由 `key-value` 对组成，每行一对，key (键) 和 value (值)用英文冒号 `:` 分隔。响应头域允许服务器传递不能放在状态行的附加信息，这些域主要描述服务器的信息和Request-URI进一步的信息，典型的响应头有：

- Server：包含处理请求的原始服务器的软件信息。
- Date：服务器日期。
- Content-Type：返回的资源类型 （MIME）。
- Connection：连接方式
  - close：连接已经关闭。
  - keep-alive：连接已保持，在等待本次连接的后续请求。
- Cache-Control：缓存控制。
- Expires：设置过期时间。
- Set-Cookie：设置 Cookie 信息。

#### 空行（响应）

最后一个响应头之后是一个空行，发送回车符和换行符，通知浏览器以下不再有响应头。

#### 响应体

服务器返回给浏览器的响应信息，下面是百度首页的响应体片段：

```html
<!DOCTYPE html>
<!--STATUS OK-->
<html>
<head>
    <meta http-equiv="content-type" content="text/html;charset=utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=Edge">
    <link rel="icon" sizes="any" mask href="//www.baidu.com/img/baidu.svg">
    <title>百度一下，你就知道</title>
</head>
<body>
  ...
</body>
</html>
```

### HTTP 请求方法

HTTP 协议的请求方法有：GET、POST、HEAD、PUT、DELETE、OPTIONS、TRACE、CONNECT、PATCH、HEAD。

HTTP 常用的请求方法：

- GET：获取资源，使用 URL 方式传递参数，大小为 2KB
  - `http://www.example.com/users` - 获取所有用户
- POST：传输资源，HTTP Body，大小默认 8M
  - `http://www.example.com/users/a-unique-id ` - 新增用户
- PUT：资源更新
  - `http://www.example.com/users/a-unique-id` - 更新用户
- DELETE：删除资源
  - `http://www.example.com/users/a-unique-id` - 删除用户

### HTTP 状态码

状态代码由三位数字组成，第一个数字定义了响应的类别，且有五种可能取值：

- 1xx：指示信息 – 表示请求已接收，继续处理。
- 2xx：成功 – 表示请求已被成功接收。
- 3xx：重定向 – 要完成请求必须进行更进一步的操作。
- 4xx：客户端错误 – 请求有语法错误或请求无法实现。
- 5xx：服务器错误 – 服务器未能实现合法的请求。

常见状态代码、状态描述的说明如下：

- 200 OK：客户端请求成功。
- 204 No Content：没有新文档，浏览器应该继续显示原来的文档。
- 206 Partial Content：客户发送了一个带有 Range 头的 GET 请求，服务器完成了它。
- 301 Moved Permanently：所请求的页面已经转移至新的 url。
- 302 Found：所请求的页面已经临时转移至新的 url。
- 304 Not Modified：客户端有缓冲的文档并发出了一个条件性的请求，服务器告诉客户，原来缓冲的文档还可以继续使用。
- 400 Bad Request：客户端请求有语法错误，不能被服务器所理解。
- 401 Unauthorized：请求未经授权，这个状态代码必须和 WWW-Authenticate 报头域一起使用。
- 403 Forbidden：对被请求页面的访问被禁止。
- 404 Not Found：请求资源不存在。
- 500 Internal Server Error：服务器发生不可预期的错误。
- 503 Server Unavailable：请求未完成，服务器临时过载或当机，一段时间后可能恢复正常。

### 参考资源

- [HTTP请求报文和HTTP响应报文以及工作原理](http://www.kannng.com/http/2016/10/26/http-packets.html)
- [百度百科 - HTTP](http://baike.baidu.com/link?url=Z2p_PetqY7-ujWiMn15wvG6z4K7ORksVt8Dy47sWI2KF_p4VEZKZrN_7vfwuF1HLBfzLs_vJJNyyF-xAQ4AyuK#1)



