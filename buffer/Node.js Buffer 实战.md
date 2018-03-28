## Node.js Buffer 实战

<!-- TOC -->

- [Node.js Buffer 实战](#nodejs-buffer-实战)
    - [预备知识](#预备知识)
        - [Base64](#base64)
        - [Data URLs](#data-urls)
            - [语法](#语法)
            - [示例](#示例)
            - [常见问题](#常见问题)
    - [Node.js Buffer 实战](#nodejs-buffer-实战-1)
        - [实战一  Buffer 转换为其他格式](#实战一--buffer-转换为其他格式)
        - [实战二  使用 Buffers 来修改字符串编码](#实战二--使用-buffers-来修改字符串编码)
        - [实战三  处理 data URLs](#实战三--处理-data-urls)
        - [实战四  创建自己的网络协议](#实战四--创建自己的网络协议)
            - [讨论](#讨论)
    - [总结](#总结)
    - [参考资源](#参考资源)

<!-- /TOC -->

### 预备知识

#### Base64

> **Base64**是一种基于64个可打印字符来表示[二进制数据](https://zh.wikipedia.org/wiki/%E4%BA%8C%E8%BF%9B%E5%88%B6)的表示方法。由于![{\displaystyle 2^{6}=64}](https://wikimedia.org/api/rest_v1/media/math/render/svg/c4becc8d811901597b9807eccff60f0897e3701a)，所以每6个[比特](https://zh.wikipedia.org/wiki/%E4%BD%8D%E5%85%83)为一个单元，对应某个可打印字符。3个[字节](https://zh.wikipedia.org/wiki/%E5%AD%97%E8%8A%82)有24个比特，对应于4个Base64单元，即3个字节可表示4个可打印字符。它可用来作为[电子邮件](https://zh.wikipedia.org/wiki/%E7%94%B5%E5%AD%90%E9%82%AE%E4%BB%B6)的传输[编码](https://zh.wikipedia.org/wiki/%E5%AD%97%E7%AC%A6%E7%BC%96%E7%A0%81)。在Base64中的可打印字符包括[字母](https://zh.wikipedia.org/wiki/%E6%8B%89%E4%B8%81%E5%AD%97%E6%AF%8D)`A-Z`、`a-z`、[数字](https://zh.wikipedia.org/wiki/%E6%95%B0%E5%AD%97)`0-9`，这样共有62个字符，此外两个可打印符号在不同的系统中而不同。一些如[uuencode](https://zh.wikipedia.org/wiki/Uuencode)的其他编码方法，和之后[BinHex](https://zh.wikipedia.org/w/index.php?title=BinHex&action=edit&redlink=1)的版本使用不同的64字符集来代表6个二进制数字，但是不被称为 Base64。 
>
> Base64常用于在通常处理文本[数据](https://zh.wikipedia.org/wiki/%E6%95%B0%E6%8D%AE)的场合，表示、传输、存储一些二进制数据，包括[MIME](https://zh.wikipedia.org/wiki/MIME)的[电子邮件](https://zh.wikipedia.org/wiki/%E7%94%B5%E5%AD%90%E9%82%AE%E4%BB%B6)及[XML](https://zh.wikipedia.org/wiki/XML)的一些复杂数据。 —— [维基百科](https://zh.wikipedia.org/wiki/Base64)

#### Data URLs

Data URLs，即为前缀为 `data：scheme ` 的 URL，其允许内容创建者向文档中嵌入小文件。

##### 语法

```
data:[<mediatype>][;base64],<data>
```

`mediatype ` 是个 MIME 类型的字符串，例如 "`image/jpeg`" 表示 JPEG 图像文件。如果被省略，则默认值为 `text/plain;charset=US-ASCII`。

如果数据是文本类型，你可以直接将文本嵌入 (根据文档类型，使用合适的实体字符或转义字符)。如果是二进制数据，你可以将数据进行 base64 编码之后再进行嵌入。

##### 示例

```
data:text/plain;base64,SGVsbG8sIFdvcmxkIQ%3D%3D
```

##### 常见问题

* 长度限制：虽然 Firefox 支持无限长度的 `data` URLs，但是标准中并没有规定浏览器必须支持任意长度的 `data` URIs。比如，Opera 11浏览器限制 URLs 最长为 65535 个字符，这意外着 data URLs 最长为 65529 个字符（如果你使用纯文本 data:，而不是指定一个 MIME 类型的话，那么 65529 字符长度是编码后的长度，而不是源文件）。
* 缺乏错误处理：MIME 类型错误或者 base64 编码错误，都会造成 `data` URIs 无法被正常解析, 但不会有任何相关错误提示。

### Node.js Buffer 实战

在[深入学习 Node.js Buffer](https://github.com/semlinker/node-deep/blob/master/buffer/%E6%B7%B1%E5%85%A5%E5%AD%A6%E4%B9%A0Node.js%20Buffer.md)中我们介绍了 `Buffer.from()` 方法，然后通过简单示例分析了 `buffer.js` 文件内的 `fromString()`  函数，最后介绍了 Array#slice() 与 Buffer#slice() 方法之间的区别。

这篇文章我们将参考 "Node.js硬实战：115个核心技巧" 这本书中的示例，来实战一下 Buffer 对象的相关 API。

在 Node.js 中如果没提供编码格式，那么文件操作以及很多网络操作就会将数据作为 Buffer 类型返回，以 fs.readFile 为例：

```javascript
const fs = require("fs");
fs.readFile("./names.txt", (err, buf) => {
  console.log(`It's buffer object? ${Buffer.isBuffer(buf) ? "Yes" : "No"}`);
});
```

#### 实战一  Buffer 转换为其他格式

默认情况下，没有指定编码格式，Node 的一些核心 API 都会返回 Buffer 对象。比如你需要把一个 Buffer 转换为文本类型，此时你可以利用 Buffer API 提供的 toString() 方法：

```javascript
const fs = require("fs");
fs.readFile("./names.txt", (err, buf) => {
  console.log(buf.toString());
});
```

toString() 方法默认使用 utf8 的编码格式进行解码。但是，如果我们知道这个数据仅仅包含了ascii 字符时，我们也可以把编码改为 ascii 来提升性能。为了实现这个，我们需要提供编码类型作为 toString() 方法的第一个参数：

```javascript
const fs = require("fs");
fs.readFile("./names.txt", (err, buf) => {
  console.log(buf.toString('ascii'));
});
```

#### 实战二  使用 Buffers 来修改字符串编码

除了把 Buffer 转换为字符串，你也可以利用 Buffer 来进行字符串的编码格式转换。有时候，创建一个字符串数据后修改它的编码格式很有用的。例如，当你需要从一个使用基础验证的服务器请求数据时，你需要发送 Base64 编码的用户名和密码：

```
Authorization: Basic c2VtbGlua2VyOmxvdmVfbm9kZQ==
```

在进行 Base64 编码前，基础验证需要把用户名和密码拼接到一起，用 `:` 分隔开来，例如，我们使用 semlinker 作为用户名，而 love_node 作为密码：

```javascript
let authstring = 'semlinker:love_node';
```

现在我们需要把这个字符串转为 Buffer 对象，然后修改它的编码格式：

```javascript
let encoded = Buffer.from(authstring).toString('base64'); // c2VtbGlua2VyOmxvdmVfbm9kZQ==
```

在浏览器环境中，我们可以通过 `window.btoa("semlinker:love_node")` 来实现 Base64 编码。

#### 实战三  处理 data URLs

Data URLs 是说明 Buffer  API 非常有用的例子：

```javascript
const fs = require("fs");
const mime = "image/png";
const encoding = "base64";
const data = fs.readFileSync("./logo.png").toString(encoding);
const url = "data:" + mime + ";" + encoding + "," + data;
```

上面示例中，我们利用 `fs.readFileSync()` 方法读取本地的 `logo.png` 文件，然后使用 `Buffer` 对象的 `toString()` 方法把内容转换为 Base64 编码的字符串，最后使用 Data URLs 的语法，生成 Data URLs。

作为前端在日常开发过程中，我们有时候会使用 FileReader 对象的 `readAsDataURL()` 方法，把文件转换为 Data URLs 的形式，比如：

```javascript
const reader = new FileReader();

reader.onload = function (e) {  
  let dataURL = fileReader.result;
};

reader.readAsDataURL(file);
```

在获取到 Data URLs 数据后，我们就可以把该数据提交到后台服务器，当服务器接收到请求参数后，就可以对数据进行解析，然后把内容保存为相应 MIME 类型的文件，这里就不详细展开。

接下来，我们来看一个简单的解析 base64 字符串并输出文件的示例：

```javascript
const fs = require('fs');
const uri = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUg...';
const data = uri.split(',')[1];
const buf = Buffer(data, 'base64');
fs.writeFileSync('./new-logo.png', buf);
```

#### 实战四  创建自己的网络协议

有些时候，你需要一种在进程或者网络之间进行高效的数据通信。针对这种场景，你就可以利用 Node.js 的 Buffer API 可以来创建自定义二进制协议。

##### 讨论

在这个例子中，我们要开发一个简洁的数据库协议，主要内容有：

- 使用掩码来确定数据存放于哪个数据库。
- 数据保存一个在 0~255 范围之内的无符号正数（单字节）的键值来标识。
- 存储通过 zlib 压缩的任意长度的数据。

| Byte | 内容         |                         |
| ---- | ---------- | ----------------------- |
| 0    | 1 byte     | 决定数据要写入到哪个数据库           |
| 1    | 1 byte     | 一个字节的无符号整数用做数据库键存储      |
| 2-n  | o -n bytes | 存储的数据，任意通过 zlib 进行压缩的字节 |

- 使用比特来表示选择哪个数据库

在我们定义的协议中，第一个字节用于表示选择哪个数据库来存储传输的数据。在数据接收端，主数据库就像一个简单的多维数组，支配着 8 个数据分库（恰好一个字节等于 8 比特）。在 JavaScript 中可以简单地使用数组字面量来表示：

```javascript
var database = [[],[],[],[],[],[],[],[]];
```

比特对应位置的数据库会将存储接收到的数据。例如，数字 8 对应的二进制 `00001000`。我们会把数据信息存储在第 4 个数据库中，因为第 4 个比特位是 1（比特按照从右往左的顺序）。

通常，数值的二进制不仅仅只有一位为 1。例如，20 的二进制表示是 `00010100`，在应用中意味着我们要把数据存储在第 3 个和第 5 个数据库。所以得想想办法，把给定的任意一个数值转换为只有一位为 1 的二进制。这里需要用到位掩码。

位掩码可以达到我们想要的效果。例如，我们想要确定是否要把数据存放在第 5 个数据库，可以创建一个第 5 位为 1 的位掩码。用二进制的格式表示为 00010000，即是数值 32（或者十六进制的 0x20）。之后我们就可以使用这个掩码去测试某个数值是否满足要求，在 JavaScript 中我们可以利用 `&` 位与来实现该功能。比如：

```
 00010100
&00010000
----------
 00010000
```

我们可以看到，如果一个值和对应的位掩码相同，或者本身为 1 时，通过运算后的值依旧还是对应的位掩码。利用 `&` 位与运算符的特性，我们就可以设置一个简单的条件来判断：

```javascript
if((value & bitmask) === bitmask) { // ... }
```

为了测试接收到的二进制协议中的第一个字节，我们需要建立一个掩码列表，用来对应数据库的索引。如果匹配到对应的掩码，那么可以确定是哪个数据库需要写入数据。用一个数组来记录每一个二进制不同位为 1 的值，对应的掩码表如下：

* 1 - 0000 0001
* 2 - 0000 0010
* 4 - 0000 0100
* 8 - 0000 1000
* 16 - 0001 0000
* 32 - 0010 0000
* 64 - 0100 0000
* 128 - 1000 0000

此时我们可以设置一个简单的循环来测试每一个掩码对应的第一个字节的值：

```javascript
var database = [[],[],[],[],[],[],[],[]];
var bitmasks = [1, 2, 4, 8, 16, 32, 64, 128];

function store(buf){
  var db = buf[0]; // 决定数据要写入到哪个数据库
  bitmasks.forEach(function(bitmask, index) {
     if((db & bitmask) === bitmask) {
        // 执行匹配后的操作
     }
   });
}
```

接下来我们来看一下如何找出数据存储的键值。

* 找出数据存储的键值

在数据库协议中，数据字节位为 1 的值是一个无符号整数（0~255），用于表示数据库中存储数据的键值。我们特意把数据库设置为一个多维数据，第一维用于表示数据分库，第二维用于存储键值和数据，由于键值为数字，使用数组便可以了。

举个例子，数据 `foo` 存放在第一个和第三个数据库中的 0 键值，数据结构看来是这样的：

```javascript
[
  ['foo'],
  [],
  ['foo'],
  [],
  [],
  [],
  [],
  []
]
```

从字节位 1 中获取键值，可以使用 `readUInt8()` 方法：

```javascript
var key = buf.readUInt8(1); // 也可以使用buf[1]
```

我们把上述的代码加入到之前的代码中：

```javascript
var database = [[],[],[],[],[],[],[],[]];
var bitmasks = [1, 2, 4, 8, 16, 32, 64, 128];

function store(buf){
  var db = buf[0]; // 决定数据要写入到哪个数据库
  var key = buf.readUInit8(1); // 一个字节的无符号整数用做数据库键存储
  bitmasks.forEach(function(bitmask, index) {
     if((db & bitmask) === bitmask) {
        database[index][key] = 'foo';
     }
   });
}
```

现在我们已经完成数据库协议中两个环节，接下来完成最后一个环节的内容，即使用 zlib 实现解压缩。

* 使用 zlib 解压缩

在传输字符串 ASCII/UTF-8 数据时进行压缩绝对是一个好主意，这可以大大减少传输使用的带宽。在简介的数据库协议中，我们假定接收到的数据是已经压缩过的。在 Node.js 中内置的 `zlib` 模块提供了 deflate（压缩）、inflate（解压缩）的方法，它也包括了 gzip 的压缩方法。为了防止获取到错误的信息，需要检查接收到的数据是否经过了正确的压缩，如果不是，则不进行解压。通常，zlib 压缩的数据结果第一个字节为 0x78，我们根据以下条件来判断：

```javascript
if(buf[2] === 0x78) { //...}
```

需要注意的是，我们从第 2 个字节位开始，因为前面已经处理过数据库索引和键值。

确定处理的是压缩过的数据后，可以使用 zlib.inflate 方法来解压缩。我们还需要使用 buf.slice() 来截取数据那一个部分，更新后的代码如下：

```javascript
var zlib = require('zlib');
var database = [[],[],[],[],[],[],[],[]];
var bitmasks = [1, 2, 4, 8, 16, 32, 64, 128];

function store(buf){
  var db = buf[0]; // 决定数据要写入到哪个数据库
  var key = buf.readUInt8(1); // 一个字节的无符号整数用做数据库键存储
  
  if(buf[2] === 0x78){ // 一般来说，zlib压缩的数据结果第一个字节为0x78
    zlib.inflate(buf.slice(2), function(er, inflatedBuf){
      if(er) return console.error(er);
      var data = inflatedBuf.toString();
      bitmasks.forEach(function(bitmask, index) {
        if((db & bitmask) === bitmask) {
          database[index][key] = data;
        }
      })
    });
  }
}
```

以上代码中，因为 zlib.inflate 方法返回的是一个 Buffer 对象，我们需要将其解码为字符串来进行存储。

现在我们的代码可以用来存储数据了。万事俱备只欠东风，接下来我们来生成测试数据，具体代码如下：

```javascript
var header = Buffer.alloc(2);

header[0] = 8; // 存放在第4个数据库(00001000)
header[1] = 0; // 存放在0键值

zlib.deflate('my message', function(er, deflateBuf) { // 压缩'my message'
  if(er) return console.error(er);
  // 把头部信息和数据打包在一起
  var message = Buffer.concat([header, deflateBuf]);
  store(message); // 存储信息
});
```

至此，简洁的数据库协议已基本开发完成。以下是本人基于书中的示例做了些简单的调整，有兴趣的小伙伴可以参考一下：

```javascript
const zlib = require("zlib");
const database = [[], [], [], [], [], [], [], []];
const bitmasks = [1, 2, 4, 8, 16, 32, 64, 128];

function store(buf) {
  return new Promise((resolve, reject) => {
    let db = buf[0]; // 决定数据要写入到哪个数据库
    let key = buf.readUInt8(1); // 一个字节的无符号整数用做数据库键存储
    let matched = false;

    if (buf[2] === 0x78) {
      // 一般来说，zlib压缩的数据结果第一个字节为0x78
      zlib.inflate(buf.slice(2), function(err, inflatedBuf) {
        if (err) {
          console.error(err);
          reject(err);
        } else {
          let data = inflatedBuf.toString();
          bitmasks.forEach(function(bitmask, index) {
            if ((db & bitmask) === bitmask) {
              database[index][key] = data;
              matched = true;
              resolve({
                code: 0,
                status: "success"
              });
            }
          });
          if (!matched) reject(new Error("Match failed"));
        }
      });
    }
  });
}

/***************数据库协议测试***************/
const header = Buffer.alloc(2);

header[0] = 8; // 存放在第4个数据库(00001000)
header[1] = 0; // 存放在0键值

zlib.deflate("my message", async function(err, deflateBuf) {
  // 压缩'my message'
  if (err) return console.error(err);
  // 把头部信息和数据打包在一起
  let message = Buffer.concat([header, deflateBuf]);
  try {
    const result = await store(message);
    if (result && result.status === "success") {
      console.dir(database);
    }
  } catch (error) {
    console.log("Save failed!");
  }
});
```

### 总结

本文基于 "Node.js硬实战：115个核心技巧" 这本书中的部分示例，介绍了 Buffer 对象中常用的 API，最后还介绍了如何利用 Buffer API 实现简洁的数据库协议。个人感觉 "Node.js硬实战：115个核心技巧" 这本书挺不错的，推荐感兴趣的小伙伴可以阅读一下。

### 参考资源

* [MDN - Data URIs](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/data_URIs)
* Node.js硬实战：115个核心技巧