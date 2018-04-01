## 深入学习 Node.js Http

<!-- TOC -->

- [深入学习 Node.js Http](#深入学习-nodejs-http)
    - [预备知识](#预备知识)
        - [HTTP](#http)
            - [请求报文示例](#请求报文示例)
            - [响应报文示例](#响应报文示例)
            - [Expect 请求头](#expect-请求头)
        - [FreeList](#freelist)
        - [IncomingMessage](#incomingmessage)
        - [ServerResponse](#serverresponse)
    - [Node.js Http](#nodejs-http)
        - [Http 基本使用](#http-基本使用)
        - [Http 服务器](#http-服务器)
    - [总结](#总结)
    - [参考资源](#参考资源)

<!-- /TOC -->

### 预备知识

#### HTTP

阅读本篇前，如果对 HTTP 协议还不了解的同学，建议先阅读[深入学习 Node.js Http 基础篇](https://github.com/semlinker/node-deep/blob/master/http/%E6%B7%B1%E5%85%A5%E5%AD%A6%E4%B9%A0%20Node.js%20Http%20%E5%9F%BA%E7%A1%80%E7%AF%87.md)这篇文章。

##### 请求报文示例

```
GET / HTTP/1.1
Host: www.baidu.com
Connection: keep-alive
Cache-Control: max-age=0
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.110 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
```

##### 响应报文示例

```
HTTP/1.1 200 OK
Server: bfe/1.0.8.18
Date: Thu, 30 Mar 2017 12:28:00 GMT
Content-Type: text/html; charset=utf-8
Connection: keep-alive
Cache-Control: private
Expires: Thu, 30 Mar 2017 12:27:43 GMT
```

##### Expect 请求头

Expect 是一个请求消息头，包含一个期望条件，表示服务器只有在满足此期望条件的情况下才能妥善地处理请求。规范中只规定了一个期望条件，即 `Expect: 100-continue`，对此服务器可以做出如下回应：

- [`100`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/100)：表示消息头中的期望条件可以得到满足，请求可以顺利进行。
- [`417`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/417) (Expectation Failed) 表示服务器不能满足期望的条件，也可以是其他任意表示客户端错误的状态码（4xx）。

常见的浏览器不会发送 `Expect` 消息头，但是其他类型的客户端如 cURL 默认会这么做。目前规范中只规定了 `Expect: 100-continue` 这一个期望条件。100-continue 握手的目的是允许客户端在发送包含请求体的消息前，判断源服务器是否愿意在客户端发送请求体前接收请求。

在实际开发过程中，需谨慎使用 Expect: 100-continue，因为如果遇到不支持 HTTP/1.1协议的服务器或代理服务器可能会引起问题。

#### FreeList

在 Node.js 中为了避免频繁创建和销毁对象，实现了一个通用的 FreeList 机制。在 http 模块中，就利用到了 FreeList 机制，即用来动态管理 HTTPParser 对象：

```javascript
var parsers = new FreeList('parsers', 1000, function() {
  var parser = new HTTPParser(HTTPParser.REQUEST);
  //...
}
```

是不是感觉很高大尚，其实 FreeList 的内部实现很简单，具体如下：

```javascript
class FreeList {
  constructor(name, max, ctor) {
    this.name = name; // 管理的对象名称
    this.ctor = ctor; // 管理对象的构造函数
    this.max = max; // 存储对象的最大值
    this.list = []; // 存储对象的数组
  }

  alloc() {
    return this.list.length ?
      this.list.pop() :
      this.ctor.apply(this, arguments);
  }

  free(obj) {
    if (this.list.length < this.max) {
      this.list.push(obj);
      return true;
    }
    return false;
  }
}
```

在处理 HTTP 请求的场景下，当新的请求到来时，我们通过调用 `parsers.alloc()` 方法来获取 HTTPParser 对象，从而解析 HTTP 请求。当完成 HTTP 解析任务后，我们可以通过调用 `parsers.free()` 方法来归还 HTTPParser 对象。

#### IncomingMessage

在 Node.js 服务器接收到请求时，会利用 HTTPParser 对象来解析请求报文，为了便于开发者使用，Node.js 会基于解析后的请求报文创建 IncomingMessage 对象，IncomingMessage 构造函数（代码片段）如下：

```javascript
function IncomingMessage(socket) {
  Stream.Readable.call(this);

  this.socket = socket;
  this.connection = socket;

  this.httpVersion = null;
  this.complete = false;
  this.headers = {}; // 解析后的请求头
  this.rawHeaders = []; // 原始的头部信息

  // request (server) only
  this.url = ''; // 请求url地址
  this.method = null; // 请求地址
}
util.inherits(IncomingMessage, Stream.Readable);
```

Http 协议是基于请求和响应，请求对象我们已经介绍了，那么接下来就是响应对象。在 Node.js 中，响应对象是 ServerResponse 类的实例。

#### ServerResponse

```javascript
function ServerResponse(req) {
  OutgoingMessage.call(this);

  if (req.method === 'HEAD') this._hasBody = false;

  this.sendDate = true;
  this._sent100 = false;
  this._expect_continue = false;

  if (req.httpVersionMajor < 1 || req.httpVersionMinor < 1) {
    this.useChunkedEncodingByDefault = chunkExpression.test(req.headers.te);
    this.shouldKeepAlive = false;
  }
}
util.inherits(ServerResponse, OutgoingMessage);
```

通过以上代码，我们可以发现 ServerResponse 继承于 OutgoingMessage。在 OutgoingMessage 对象中会包含用于生成响应报文的相关信息，这里就不详细展开，有兴趣的小伙伴可以查看 `_http_outgoing.js` 文件。

### Node.js Http

#### Http 基本使用

simple_server.js

```javascript
const http = require("http");

const server = http.createServer((req, res) => {
  res.end("Hello Semlinker!");
});

server.listen(3000, () => {
  console.log("server listen on 3000");
});
```

当运行完 `node simple_server.js ` 命令后，你可以通过  `http://localhost:3000/` 这个 url 地址来访问我们本地的服务器。不出意外的话，你将在打开的页面中看到 "Hello Semlinker!"。

虽然以上的示例很简单，但对于之前没有服务端经验或者刚接触 Node.js 的小伙伴来说，可能会觉得这是一个很神奇的事情。接下来我们来通过以上简单的示例，分析一下 Node.js 的 Http 模块。

#### Http 服务器

显而易见，`http.createServer()` 方法用来创建服务器，该方法的实现如下：

```javascript
function createServer(requestListener) {
  return new Server(requestListener);
}
```

在 `createServer` 函数内部，我们通过调用 Server 构造函数来创建服务器。因此，接下来的重点就是分析 Server 构造函数了，该函数的内部实现如下：

```javascript
function Server(options, requestListener) {
  if (!(this instanceof Server)) return new Server(options, requestListener);

  if (typeof options === 'function') {
    requestListener = options;
    options = {};
  } else if (options == null || typeof options === 'object') {
    options = util._extend({}, options);
  }

  net.Server.call(this, { allowHalfOpen: true });

  if (requestListener) {
    this.on('request', requestListener);
  }

  this.on('connection', connectionListener);
  this.timeout = 2 * 60 * 1000; // 设置超时时间
}
util.inherits(Server, net.Server);
```

看到  `this.on('request',requestListener)` 和 `this.on('connection',connectionListener)` 这两行，不知道小伙伴们有没有想起我们的 EventEmitter。如果对它还不了解的小伙伴，可以参考之前的文章 —— [深入学习 Node.js EventEmitter](https://github.com/semlinker/node-deep/blob/master/event/%E6%B7%B1%E5%85%A5%E5%AD%A6%E4%B9%A0%20Node.js%20EventEmitter.md)。

通过以上源码，目前我们得出了一个结论，在触发 `request` 事件后，就会调用我们设置的 requestListener 函数，即执行以下代码：

```javascript
(req, res) => {
  res.end("Hello Semlinker!");
}
```

那么什么时候会触发 request 事件呢？而 connection 事件和 connectionListener 又是什么？带着这些问题，我们来继续学习 Http 模块。

connection 事件，顾名思义用来跟踪网络连接。这里，我们重点来看一下 connectionListener 函数：

```javascript
function connectionListener(socket) {
  defaultTriggerAsyncIdScope(
    getOrSetAsyncId(socket), connectionListenerInternal, this, socket
  );
}
```

该函数内竟然还有一个 connectionListenerInternal，那只能继续往下分析了，connectionListenerInternal 函数（代码片段）的内部实现如下：

```javascript
function connectionListenerInternal(server, socket) {
  httpSocketSetup(socket);

  if (socket.server === null)
    socket.server = server;
  
  if (server.timeout && typeof socket.setTimeout === 'function')
    socket.setTimeout(server.timeout);
  socket.on('timeout', socketOnTimeout); // 处理超时情况

  var parser = parsers.alloc(); // 获取parser对象
  parser.reinitialize(HTTPParser.REQUEST);
  parser.socket = socket;
  socket.parser = parser;
  parser.incoming = null;

  var state = {
    outgoing: [],
    incoming: [], 
    //...
  };
  parser.onIncoming = parserOnIncoming.bind(undefined, server, socket, state);
}
```

在 connectionListenerInternal 函数内部，我们终于见到了 "预备知识" 章节中介绍的 parsers 对象（FreeList 实例）。现在是时候来目睹一下 HTTPParser 对象的芳容了：

```javascript
var parsers = new FreeList('parsers', 1000, function() {
  var parser = new HTTPParser(HTTPParser.REQUEST);

  parser._headers = [];
  parser._url = '';
  parser._consumed = false;

  parser.socket = null;
  parser.incoming = null;
  parser.outgoing = null;
  
  parser[kOnHeaders] = parserOnHeaders;
  parser[kOnHeadersComplete] = parserOnHeadersComplete;
  parser[kOnBody] = parserOnBody;

  return parser;
});
```

以 parser 开头的这些对象，都是定义在 `_http_common.js` 文件中的函数对象。这里我就不罗列出相关的代码了，只对它们的作用做一些简单的总结：

* parserOnHeaders：当请求头跨多个 TCP 数据包或者过大无法再一个运行周期内处理完才会调用该方法。
* kOnHeadersComplete：请求头解析完成后，会调用该方法。方法内部会创建 IncomingMessage 对象，填充相关的属性，比如 url、httpVersion、method 和 headers 等。
* parserOnBody：不断解析已接收的请求体数据。

这里需要注意的是，请求报文的解析工作是由 C++ 来完成，内部通过 binding 来实现，具体参考 `deps/http_parser` 目录。

```javascript
const { methods, HTTPParser } = process.binding('http_parser');
```

介绍完 HTTPParser 对象，我们继续回到 connectionListenerInternal 函数中，在最后一行我们设置 parser 对象的 onIncoming 属性为绑定后的 parserOnIncoming 函数，该函数的实现如下（代码片段）：

```javascript
function parserOnIncoming(server, socket, state, req, keepAlive) {
  state.incoming.push(req); // 缓冲IncomingMessage实例

  var res = new server[kServerResponse](req);

  if (socket._httpMessage) {
    state.outgoing.push(res); // 缓冲ServerResponse实例
  } else {
    res.assignSocket(socket);
  }

  // 判断请求头是否包含expect字段且http协议的版本为1.1
  if (req.headers.expect !== undefined &&
      (req.httpVersionMajor === 1 && req.httpVersionMinor === 1)) {
    // continueExpression: /(?:^|\W)100-continue(?:$|\W)/i
    // Expect: 100-continue
    if (continueExpression.test(req.headers.expect)) {
      res._expect_continue = true;

      if (server.listenerCount('checkContinue') > 0) {
        server.emit('checkContinue', req, res);
      } else {
        res.writeContinue();
        server.emit('request', req, res);
      }
    } else if (server.listenerCount('checkExpectation') > 0) {
      server.emit('checkExpectation', req, res);
    } else {
      // HTTP协议中的417Expectation Failed 状态码表示客户端错误，意味着服务器无法满足
      // Expect请求消息头中的期望条件。
      res.writeHead(417);
      res.end();
    }
  } else {
    server.emit('request', req, res);
  }
  return 0;  
}
```

通过观察上面的代码，我们终于发现了 `request` 事件的踪迹。在 parserOnIncoming 函数内，我们会基于 req 请求对象创建 ServerResponse 响应对象，在创建响应对象后，会判断请求头是否包含 expect 字段，然后针对不同的条件做出不同的处理。对于之前最早的示例来说，程序会直接走 `else`  分支，即触发 `request` 事件，并传递当前的请求对象和响应对象。

最后我们来回顾一下整个流程：

* 调用 `http.createServer()` 方法创建 server 对象，该对象创建完后，我们调用 `listen()` 方法执行监听操作。


* 当 server 接收到客户端的连接请求，在成功创建 socket 对象后，会触发 `connection` 事件。
* 当 `connection` 事件触发后，会执行对应的 `connectionListener` 回调函数。在函数内部会利用 HTTPParser 对象，对请求报文进行解析。
* 在完成请求头的解析后，会创建  IncomingMessage 对象，并填充相关的属性，比如 url、httpVersion、method 和 headers 等。
* 在配置完  IncomingMessage 对象后，会调用 parserOnIncoming 函数，在该函数内会构建 ServerResponse 响应对象，如果请求头不包含 expect 字段，则 server 就会触发 `request` 事件，并传递当前的请求对象和响应对象。
* `request` 事件触发后，就会执行我们设定的 `requestListener` 函数。

### 总结

本文基于一个简单的服务器示例，一步一步分析了 Node.js Http 模块中请求对象、响应对象内部的创建过程，此外还介绍了 Server 内部两个重要的事件：`connection` 与 `request`。

在文中我们只分析 `request` 事件的触发时机，并未介绍 `connection` 事件的触发时机。另外，我们也没有继续深入分析 server 对象 `listen()` 方法内部执行流程。这是为什么呢？其实这是为下一篇 "深入学习 Node.js Net" 文章留个小伏笔。

### 参考资源

* [深入理解Node.js：核心思想与源码分析](https://yjhjstz.gitbooks.io/deep-into-node/)
* [MDN - Expect 请求头](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Expect)