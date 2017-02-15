# HTTP 学习笔记

参考资料：《图解HTTP》、《HTTP权威指南》



## HTTP协议简析

* HTTP协议是Hyper Text Transfer Protocol（超文本传输协议）的缩写,是用于从万维网（WWW:World Wide Web ）服务器传输超文本到本地浏览器的传送协议。HTTP是一个基于TCP/IP通信协议来传递数据


* HTTP 协议用于客户端和服务器端之间的通信。HTTP 协议和 TCP/IP 协议簇内的其他众多的协议相同，用于客户和服务器之间的通信，请求访问文本或图像等资源的一端称为客户端，为提供资源响应的一端称为服务器端
* 通过请求和响应的交换达成通信。HTTP 协议规定，请求从客户端发出，最后服务器端响应该请求并返回。换句话说，肯定是先从客户端开始建立通信的，服务器端在没有接收到请求之前不会发送响应

### 请求报文与响应报文实例

![HTTP 请求报文实例](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/HTTP%E8%AF%B7%E6%B1%82%E6%8A%A5%E6%96%87%E5%AE%9E%E4%BE%8B.png)

​	起始行开头的 GET 表示请求访问服务器的类型，称为方法（method）。随后的字符串 /python-web-crawler/ 指明了请求访问的资源对象，也叫做请求 URI 。最后的 HTTP/1.1 ，即 HTTP 的版本号，用来提示客户端使用的 HTTP 协议功能。综合来看，这段请求的意思是：请求访问某台服务器上的 /python-web-crawler/ 对应的资源

​	请求报文是由请求方法、请求 URI 、协议版本、可选的请求首部字段和内容实体构成的



![HTTP 响应报文实例](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/HTTP%E5%93%8D%E5%BA%94%E6%8A%A5%E6%96%87%E5%AE%9E%E4%BE%8B.png)

​	在起始行开头的 HTTP/1.1 表示服务器对应的 HTTP 版本号。接着的 200 OK 表示请求的处理结果状态码（status code）和原因短语（reason-phrase）。

​	响应报文基本上是由协议版本、状态码、用以解释状态码的原因短语、可选的响应首部字段以及实体主体构成



## HTTP 协议详解

### HTTP 报文

​	用于 HTTP 协议交互的信息称为 HTTP 报文。请求端（客户端）的 HTTP 报文叫做请求报文，响应端（服务器端）的 HTTP 报文叫做响应报文。

#### HTTP 报文结构

- HTTP 报文本身是由多行（用CR + LF 作换行符）数据结构构成的字符串文本
- HTTP 报文大致可分为报文首部和报文主体两块。通常并不一定要有报文主体

HTTP 报文的结构如下：

![](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/HTTP%E6%8A%A5%E6%96%87%E6%A0%BC%E5%BC%8F.png)

#### 请求报文与响应报文结构

请求报文和响应报文的结构如下：

![请求报文与响应报文结构](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/HTTP%E8%AF%B7%E6%B1%82%E6%8A%A5%E6%96%87%E4%BB%A5%E5%8F%8A%E5%93%8D%E5%BA%94%E6%8A%A5%E6%96%87%E6%A0%BC%E5%BC%8F.PNG)

请求报文实例：

![请求报文实例](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/HTTP%E8%AF%B7%E6%B1%82%E6%8A%A5%E6%96%87%E5%B8%A6%E6%A0%87%E6%B3%A8.png)

响应报文实例：

![响应报文实例](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/HTTP%20%E5%93%8D%E5%BA%94%E6%8A%A5%E6%96%87%E5%B8%A6%E6%A0%87%E6%B3%A8.png)

请求报文和响应报文的首部由以下数据组成：

* 请求行：包含请求的方法，请求 URI 和 HTTP 版本
* 状态行：包含表明响应结果的状态码，原因短语和HTTP版本
* 首部字段：包含表示请求和响应的各种条件和属性的各类首部。一般有四种首部，分别是：通用首部，请求首部、响应首部和实体首部
* 其他：可能包含 HTTP 的 RFC 里未指定的首部（Cookie等）



#### 请求方法

​	请求报文中起始行的第一个字段表示请求访问服务器的类型，称为方法（method）

* **GET**：GET 方法用来请求访问已被 URI 识别的资源。指定的资源经服务器端解析后返回解析内容
* **POST**：POST 方法用来传输实体的主体。虽然 GET 方法也可以用于传输实体的主体，但一般不使用 GET 进行传输，而是用 POST 方法。虽说 POST 的功能与 GET 类似，但 POST 的主要目的并不是获取响应的主体内容
* **PUT**：PUT 方法用来传输文件。就像 FTP 协议的文件上传一样，要求在请求报文的主体中包含文件内容，然后保存到请求 URI 指定的位置。但是 HTTP/1.1 的PUT 方法自身不带验证机制，存在安全问题，因此一般的 web 网站不使用该方法。
* **HEAD**：HEAD 方法和 GET 方法一样，只是不返回报文主体部分。用于确定 URI 的有效性及资源更新的日期时间等
* **DELETE**：DELETE 方法用来删除文件，是与 PUT 相反的方法，DELETE 方法按请求 URI 删除指定的资源。同样由于 HTTP/1.1 的 DELETE 方法不带验证机制，存在安全性问题
* **OPSTION**：OPTION 方法用来查询针对请求 URI 指定的资源支持的方法
* **TRACE**：TRACE 方法是让 Web 服务器端将之前的请求通信环回给客户端的方法
* **CONNECT**：CONNECTION 方法要求在与代理服务器通信时建立隧道，实现用隧道协议进行 TCP 通信。主要使用 SSL （security Socket Layer，安全套接字层） 和 TLS （Transport Layer Security，传输层安全）协议将通信内容加密后经网络隧道传输


#### 状态码

​	HTTP 状态码负责表示客户端 HTTP 请求的返回结果，标记服务器端的处理是否正常、通知出现的错误等工作。

​	状态码如 200 OK ，以 3 位数字和原因短语组成。

​	数字的第一位指定了响应的类别，后两位无分类。只要遵守了状态码类别的定义，及时改变 RFC 2616 中定义的状态码或服务器端自行改变自行创建状态码都没问题。

响应码的类别有以下 5 种：

|      | 类别                       | 原因短语           |
| ---- | ------------------------ | -------------- |
| 1XX  | Infomational（信息性状态码）     | 接收的请求正在处理      |
| 2XX  | Success（成功状态码）           | 请求正常处理完毕       |
| 3XX  | Redirection（重定向状态码）      | 需要进行附加操作得以完成请求 |
| 4XX  | Client Error（客户端错误状态码）   | 服务器无法处理请求      |
| 5XX  | Service Error（服务器端错误状态码） | 服务器处理请求错误      |



##### 2XX 成功

2XX 的响应结果表明请求被正常处理

* 200 OK：表示从客户端发来的请求在服务器端被正常处理
* 204 No Content ：该状态码代表服务器接收的请求已成功处理，但在返回的响应报文中不含实体的主体部分。另外，也不允许返回任何实体的主体。一般在只需要从客户端往服务器端发送信息，面对客户端不需要发送新信息内容的情况下使用。
* 206 Partial Content ：该状态码表示客户端进行了范围请求，为服务器成功执行了这部分的 GET 请求。响应报文中应该包含由 Content-Range 指定范围的实体内容

##### 3XX重定向

3XX 响应结果表明浏览器需要执行某些特殊的处理以正确处理请求

* 301 Moved Permanently：永久性重定向。该状态码表示请求的资源已被分配新的 URI ，以后应使用资源现在所指的 URI
* 302 Found：临时性重定向。该状态码表示请求的资源已被分配新的 URI。和 301 Moved Permanently 状态码类似，但 302 状态码代表资源不是永久移动，只是临时性质的
* 303 See Other：该状态码表示由于请求的资源存在另一个 URI，应使用 GET 方法定向获取请求的资源
* 304 Not Modified：该状态码表示客户端发送附加条件的请求时，服务器端允许访问资源，但未满足条件的情况。304 状态码返回时，不包含任何响应的主体部分
* 307 Temporary Redirect：临时性重定向。该状态码与 302 Found 有着相同的含义

#### 4XX 客户端错误

4XX 的响应结果表明客户端发生了错误

* 400 Bad Request：该状态码表示请求报文中存在语法错误。当错误发生时，需修改请求的内容再次发送请求。
* 401 Unauthorized：该状态码表示发送的请求需要有通过 HTTP 认证（BASIC 认证，DIGEST 认证）的认证信息，另外，若之前已经进行过一次请求，则表示用户认证失败
* 403 Forbidden：该状态码表明对请求资源的访问被服务器拒绝了
* 404 Not Found：该状态码表明服务器上无法找到请求的资源

#### 5XX 服务器错误

5XX 的响应结果表明服务器本身发送错误

* 500 Internal Server Error：该状态码表明服务器端在执行请求时发送错误，也有可能是 Web 应用存在 bug 或某些临时性的错误
* 503 Service Unavailable：该状态码表示服务器暂时处于超负载或正在进行停机维护，现在无法处理请求

#### HTTP 首部字段

* HTTP 首部字段是构成 HTTP 报文的要素之一。在客户端与服务器之间的以 HTTP 协议进行通信的过程中，无论是请求还是响应都会使用首部字段，它能起到额外重要信息的作用

* HTTP 首部字段是由首部字段名和字段值构成的，中间用冒号“:”分隔

  ```
  首部字段：字段值
  ```

  例如，在 HTTP 首部中以　Content－Type　这个字段来表示报文主体的对象类型

  ```
  Content-Type: text/html
  ```

  另外，字段值对应单个 HTTP 首部字段可以由多个值，如下所示

  ```
  Keep-Alive : timeout=15,max=100
  ```

* HTTP 首部字段根据实际用途被分为以下 4 种类型

  * 通用首部字段：请求报文和响应报文双方都会使用的首部
  * 请求首部字段：从客户端向服务器端发送请求报文时使用的首部
  * 响应首部字段：从服务器端向客户端返回响应报文时使用的首部
  * 实体首部字段：针对请求报文和响应报文的实体部分使用的首部

* HTTP 首部字段按照定义缓存代理和非缓存代理的行为，分为 2 中类型

  * 端到端首部（End-to-end Header）：分在此类别中的首部会转发给请求/响应对应的接收目标，且必须保存在由缓存生成的响应中，另外规定它必须被转发
  * 逐跳首部（Hop-by-hop Header）：分在此类别中的首部只对单次转发有效，会因通过缓存或道理不再转发。HTTP/1.1 和之后的版本，如果要使用 Hop-by-hop 首部，需提供 Connection 首部字段。HTTP/1.1 中，Connection、Keep-Alive、Proxy-Authenticate、Proxy-Authorization、Trailer、TE、Transfer-Encoding 和 Upgrade 属于逐跳首部，其他均属于端到端首部

  ​

##### 通用首部字段

HTTP/1.1 规范中定义的通用首部字段

| 首部字段名             | 说明            |
| ----------------- | ------------- |
| Cache-Control     | 控制缓存的行为       |
| Connection        | 逐条首部、连接的管理    |
| Date              | 创建报文的日期时间     |
| Pragma            | 报文指令          |
| Trailer           | 报文末端的首部一览     |
| Transfer-Encoding | 指定报文主体的传输编码方式 |
| Upgrade           | 升级为其他协议       |
| Via               | 代理服务器的相关信息    |
| Warning           | 错误通知          |

##### 请求首部字段

HTTP/1.1 规范中定义的请求首部字段

| 首部字段                | 说明                              |
| ------------------- | ------------------------------- |
| Accept              | 用户代理可处理的媒体类型                    |
| Accept-Charset      | 优先的字符集                          |
| Accept-Encoding     | 优先的内容编码                         |
| Accept-Language     | 优先的语言（自然语言）                     |
| Authorization       | Web 认证信息                        |
| Expect              | 期待服务器的特定行为                      |
| From                | 用户电子邮箱地址                        |
| Host                | 请求资源所在的服务器                      |
| If-Match            | 比较实体标记（Etag）                    |
| If-Modified-Since   | 比较资源的更新时间                       |
| If-None-Match       | 比较实体标记（与If-Match相反）             |
| If-Range            | 资源未更新时发送实体Byte的范围请求             |
| If-Unmodifed-Since  | 比较资源的更新时间（与If-Modified-Since相反） |
| Max-Forwards        | 最大传输逐跳数                         |
| Proxy-Authorization | 代理服务器要求客户端的认证信息                 |
| Range               | 实体的字节请求范围                       |
| Referer             | 对请求中 URI 的原始获取方                 |
| TE                  | 传输编码的优先级                        |
| User-Agent          | HTTP 客户端程序的信息                   |

##### 响应首部字段

HTTP/1.1 规范中定义的响应首部字段

| 响应首部字段             | 说明              |
| ------------------ | --------------- |
| Accept-Range       | 是否接收字节范围请求      |
| Age                | 推算资源创建经过时间      |
| ETag               | 资源的匹配信息         |
| Location           | 令客户端重定向到指定的 URI |
| Proxy-Authenticate | 代理服务器对客户端的认证消息  |
| Retry-After        | 对再次发起请求的时机要求    |
| Server             | HTTP 服务器的安装信息   |
| Vary               | 代理服务器缓存的管理信息    |
| WWW-Authenticate   | 服务器对客户端认证信息     |

##### 实体首部字段

HTTP/1.1 规范中定义的实体首部字段

| 首部字段             | 说明             |
| ---------------- | -------------- |
| Allow            | 资源可支持的 HTTP    |
| Content-Encoding | 实体主体适用的编码方式    |
| Content-Language | 实体主体的自然语言      |
| Content-Length   | 实体主体的大小（单位：字节） |
| Content-Location | 替代对应资源的URI     |
| Content-MD5      | 实体主体的报文摘要      |
| Content-Range    | 实体主体的位置范围      |
| Content-Type     | 实体主体的媒体类型      |
| Expires          | 实体主体过期的日期时间    |
| Last_Modified    | 资源的最后修改时间      |

##### 非 HTTP/1.1 首部字段

​	在 HTTP 协议通信交互中适用到的首部字段，不限于 RFC 2616 中定义的 47 种首部字段。还有 Cookie、Set-Cookie 和 Content-Disposition 等在其他 RFC 中定义的首部字段，它们的适用频率也很高

#### HTTP 版本

* HTTP/0.9 : HTTP 问世与 1990 年，那时的 HTTP 并没有作为正式的标准被建立
* HTTP/1.0：HTTP 正式作为标准被公布于 1996 年，版本命名为 HTTP/1.0 ，并记载于 RFC 1945
* HTTP/1.1：1997 年公布的 HTTP/1.1是目前主流 HTTP 协议版本，当初的标准是 RFC 2068，之后发布的修订版 RFC 2616 就是当前的最新版本
* HTTP/2.0：新一代的 HTTP/2.0 正在制定

### HTTP 状态管理

​	HTTP 是无状态协议，它不对之前发生过的请求和响应的状态进行管理，也就是说，无法根据之前的状态进行本次的请求处理。这样的优点是：由于不必保存状态，可减少服务器的 CPU 及内存的消耗，同时这也使得 HTTP 本身非常的简单。

​	但是，HTTP 的无状态也使得无法实现状态的管理，如登陆认证。为了解决这个问题于是引入了 Cookie 技术。

​	Cookie 技术通过在请求和响应报文中写入 Cookie 信息来控制客户端的状态。Cookie 会根据从服务区端发送的响应报文内一个叫做 Set-Cookie 的首部字段信息，通知客户端保存 Cookie。当下一次客户端再往服务器端发送请求的时候，客户端会自动在请求报文中自动加入 Cookie 值后发送出去。服务器端发现客户端发送过来的连接请求，然后对比服务器上的记录，最后得到之前的状态信息

#### 设置 Cookie

![HTTP 状态管理-设置Cookie](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/HTTP%20%E7%8A%B6%E6%80%81%E7%AE%A1%E7%90%86-%E8%AE%BE%E7%BD%AECookie.png)

#### 自动发送 Cookie

![HTTP 状态管理-使用Cookie](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/HTTP%20%E7%8A%B6%E6%80%81%E7%AE%A1%E7%90%86-%E4%BD%BF%E7%94%A8Cookie.png)

### HTTP 连接管理

​	HTTP 协议的早期版本中，每进行一次 HTTP 通信就要断开一次 TCP 连接。

![HTTP 的连接建立与断开](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/HTTP%E8%BF%9E%E6%8E%A5%E7%AE%A1%E7%90%861.PNG)

​	由于早期的互联网通信都是一些容量较少的文本传输，所以即使这样也没有多大问题。但是随着 HTTP 的普及，文档中包含大量的图片和其他资源，就需要进行多个请求，因此每次的请求都会造成无谓的 TCP 连接建立和断开，增加通信量的开销

![大量HTTP的情况](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/HTTP%E8%BF%9E%E6%8E%A5%E7%AE%A1%E7%90%862.png)

#### 持久连接

​	为了解决上述 TCP 连接问题，HTTP/1.1 和一部分的 HTTP/1.0 提出了持久连接（HTTP Persistent Connections，也称为 HTTP Keep-Alive 或 HTTP connection reuse）的方法。持久连接的特点是，只要任意一端没有明确提出断开连接，则保持 TCP 连接状态

![HTTP 持久连接](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/HTTP%E8%BF%9E%E6%8E%A5%E7%AE%A1%E7%90%863.png)

* 持久连接的好处在于减少了 TCP 连接的重复建立和断开所造成的额外开销，减轻了服务器的负载。同时也可以提高 Web 网页的加载速度
* 在 HTTP/1.1 中，所有的连接默认是持久连接

#### 管线化

​	持久化连接使得多数请求以管线化（Pipelining）方式发送成为可能，从前发送请求后需等待并收到响应，才能发送下一个请求。管线化技术出现后，加之 TCP 的持久连接，不用等待响应亦可直接发送下一个请求，这样就可以做到同时发送多个请求，而不需要一个接一个地等待响应了

![HTTP 管线化](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/HTTP%E8%BF%9E%E6%8E%A5%E7%AE%A1%E7%90%864.png)

### HTTP 编码

​	HTTP 在传输数据时可以按照数据原貌直接传输，但也可以在传输过程中通过编码提升传输速率。通过在传输编码，能有效地处理大量的访问请求，但是将会消耗更多的 CPU 资源

#### 压缩传输的内容编码

​	HTTP 协议中的内容编码可以对报文实体的容量进行压缩，内容编码指明了应用在实体内容上的编码格式，并保持实体信息原样压缩。内容编码后的实体有客户端接收并负责解码

​	常用的内容编码有以下几种

* gzip
* compress
* deflate
* identity

#### 分隔发送的分块传输编码

​	在 HTTP 通信过程中，在传输大容量数据时，把数据分隔成多块让浏览器逐步显示页面，这种功能称为分块传输编码（Chunked Transfer Coding）。

​	分块传输编码会将实体主体分成多个部分（块）。每一块都会用十六进制来标记块的大小，而实体主体的最后一块会使用“0（CR + LF）”来标记。使用分块标记传输编码的实体主体会有接收的客户端负责解码，恢复到编码前的实体主体

### HTTP 内容协商

​	内容协商机制是指客户端和服务器端就响应的资源内容进行交涉，然后提供给客户端最合适的资源。内容协商会以响应资源的语言、字符集、编码方式等作为判断标准

​	包含在请求报文中某些首部字段就是判断的基准，如下：

* Accept
* Accept-Charset
* Accept-Encoding
* Accept-Language
* Content-Language

## HTTPS 协议

### HTTP 的缺点

HTTP 具有相当优秀和方便的一面，然而也由不足之处，HTTP 的主要不足如下：

* 通信使用明文（不加密），内容可能被窃听
* 不验证通信方的身份，因此可能遭遇伪装
* 无法证明报文的完整性，所以有可能已被纂改

### 什么是 HTTPS

​	如果在 HTTP 协议通信中使用未经加密的明文，如果通信线路被窃听，那么通信的内容将会被泄露。另外，对于 HTTP 来说，服务器也好，客户端也好，都是无法确定对方，很由可能并不是与预想的通信方在时机通信，并且还需要考虑接收到的报文在通信途中已经遭到纂改的可能性。

​	为了解决以上问题，需要在 HTTP 上加入加密处理和认证机制。我们把加入加密及认证机制的 HTTP 称为 HTTPS

```
HTTP + 加密 + 认证 + 完整性保护 = HTTPS
```

![使用HTTPS 通信](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/HTTPS-%E4%BD%BF%E7%94%A8%20HTTPS%20%E9%80%9A%E4%BF%A1.png)

​	HTTPS 并非是应用层的一种新协议，而是 HTTP 通信接口部分使用 SSL （Security Socket Layer） 和 TLS （Transfer Layer Security）协议代替。通常，HTTP 直接与 TCP 通信。当使用 SSL 时，则演变成先和 SSL 通信，在由 SSL 与 TCP 通信。简而言之，所谓的 HTTPS ，其实就是身披 SSL 协议的 HTTP 协议，并且 SSL 是独立于 HTTP 协议的

![HTTPS 协议栈](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/HTTPS-%E5%8D%8F%E8%AE%AE%E6%A0%88.png)

​	在采用 SSL 后，HTTP 就拥有了加密、证书和完整性保护的功能。



### 密钥加密技术

​	近代的加密方法是加密算法是公开的，为密钥确实保密的。通过这种方式得以保持加密方法的安全性。加密和解密都要用到密钥，没有密钥就无法对密码解密，相反，任何人持有了密钥就可以解密。如果密钥被攻击者获得，加密也就失去了意义

#### 共享密钥加密

​	加密和解密使用同一个密钥的方式称为共享密钥加密（Common Key Crypto System），也称为对称密钥加密。

​	以共享密钥的方式加密时必须将密钥也发给对方，但是在怎样才能安全地转交密钥是一个问题，在互联网中转发密钥时，如果通信被监听，那么密钥就可能落入攻击者手中，失去了加密的意义，同时，使用者也需要想方设法的保管接收到的密钥

#### 公开密钥加密

​	公开密钥加密方式可以很好地解决共享密钥加密的缺点。公开密钥加密使用一对非对称的密钥。一把叫做私有密钥（Private Key），另一把叫做公开密钥（Public Key）。顾名思义，私有密钥不能让其他人知道，而公开密钥则可以随意发布，任何人都可以获得。

​	使用公开密钥加密方式，发送密文的一方使用对方的公开密钥进行加密处理，对方收到被加密的信息后，再使用自己的私有密钥进行解密。利用这种方式，不需要发送用来解密的私有密钥，不必担心密钥被窃听而盗走。另外，要想根据密文和公开密钥，恢复到信息原文是异常困难的。

#### HTTPS 使用的加密技术

​	HTTPS 采用共享密钥加密和公开密钥加密两者并用的混合加密机制。因为公开密钥加密方式处理起来比共享密钥加密方式更为复杂，因此若在通信时使用公开密钥加密方式，效率就会很低。所以应该充分利用两者的优势，将多种方法组合起来用于通信，在交换密钥环节使用公开密钥加密方式，之后建立通信交换报文阶段则使用共享密钥加密方式



### 数字证书

​	上面提到，使用公开密钥加密方式可以实现加密交换通信报文加密使用的密钥，但是仍然存在一些困难，就是如何证明收到的公开密钥就是原本预想的那一台服务器发行的公开密钥。或许在公开密钥传输过程中，真正的公开密钥已经被替换掉了

​	为了解决上述的问题，可以使用由**数字证书认证机构（CA，Certificate Authority）**和其他相关机关颁发的**公开密钥证书**，公钥证书也可以叫做**数字证书**或者**证书**。数字证书认证机构处于客户端与服务器双方都可信赖的第三方机构的立场上。

​	服务器会将这份由数字证书认证机构颁发的公钥证书发送给客户端，以进行公开密钥加密方式通信。接收到证书的客户端可使用数字证书认证机构的公开密钥，对那张证书上的数字签名进行验证，一旦验证通过，客户端便可确认两件事：一、认证服务器的公开密钥是真实有效的数字证书认证机构。二、服务器的公开密钥是值得信赖的

​	需要注意的是，数字证书认证机构是实现对服务器发送的数字证书进行认证的关键，必须保证是在正确可信赖的认证机构进行证书的认证，防止攻击者伪造，这需要认证机关的公开密钥安全的转交到客户端，但是如可安全转交是很困难的事，故多数浏览器发布时都会事先在内部植入常用的认证机关的公开密钥



### HTTPS 的安全通信机制

HTTPS 的通信步骤如下：

![HTTPS 安全通信机制](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/HTTPS-%E5%AE%89%E5%85%A8%E9%80%9A%E4%BF%A1%E6%9C%BA%E5%88%B6.png)

1. 客户端通过发送 Client Hello 报文开始 SSL 通信。报文中包含客户端支持的 SSL 的指定版本、加密组件（Cipher Suite）列表（所使用的加密算法和密钥长度）
2. 服务器可进行 SSL 通信时，会以 Server Hello 报文作为应答。和客户端一样，在报文中包含 SSL 版本以及加密组件。服务器的加密组件内容是从接收到的客户端组件内筛选出来的
3. 之后服务器发送 Certificate 报文，报文中包含公开密钥证书
4. 最后服务器发送 Server Hello Done 报文通知客户端，最初阶段的 SSL 握手协商部分结束
5. SSL 第一次握手结束后，客户端以 Client Key Exchange 报文作为回应。报文中包含通信加密使用的一种称为 Pre-master secret 的随机密码串。该报文已使用步骤 3 接收到的数字证书中公开密钥进行加密
6. 接着客户端继续发送 Change Cipher Spec 报文。该报文会提示服务器，在此报文之后的通信会采用 Pre-master secret 密钥加密
7. 客户端发送 Finished 报文。该报文包含连接至今全部报文的整体校验值。这次握手协商是否能够成功，要以服务器是否能够解密该报文作为判定标准
8. 服务器同样发送 Change Cipher Spec 报文
9. 服务器同样发送 Finished 报文
10. 服务器和客户端的 Finished 报文交换完毕后，SSL 连接就算连接建立完成。同样，通信会收到 SSL 的保护，从此处开始进行应用层协议的通信，即发送 HTTP 响应
11. 应用层协议通信，即发送 HTTP 报文
12. 最后由客户端断开连接



![HTTP 安全通信过程](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/HTTPS-%E5%AE%89%E5%85%A8%E9%80%9A%E4%BF%A1%E8%BF%87%E7%A8%8B.png)

## HTTP 用户身份认证

HTTP/1.1 使用的认证方式如下所示：

* BASIC 认证（基本认证）
* DIGEST 认证（摘要认证）
* SSL 客户端认证
* FormBase 认证（基于表单认证）

### BASIC 认证

BASIC 认证是从 HTTP/1.1 就定义的认证方式。其认证步骤如下：

![BASIC认证](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/HTTP-BASIC%E8%AE%A4%E8%AF%81.png)

1. 当请求的资源需要BASIC 认证时，服务器会随状态码 401 Authorization Required，返回带 WWW-Authorizate 首部字段的响应，该字段内包含认证的方式 BASIC 及 Request-URI 安全域字符串（realm）
2. 接收到状态码 401 的客户端为了通过 BASIC 认证，需要将用户 ID 及密码发送给服务器。发送字符串的内容是由用户 ID 和密码构成的，两者中间以冒号（:）连接后，再经过 Base64 编码处理
3. 接收到包含首部字段 Authorization 请求的服务器，会对认证信息的正确性进行验证，如验证通过，则返回一条包含 Request-URI 资源的响应

BASIC认证的缺点：

* BASIC 认证虽然采用了 Base64 编码方式，但这不是加密处理。不需要任何附加的信息即可进行解码，如果被人窃听，被盗的可能性极高
* 一般的浏览器无法实现认证的注销操作

BASIC 认证使用上的不够便捷灵活，且达不到多数 Web 网站的安全性等级，因此并不常用

### DIGEST 认证

​	为了弥补 BASIC 认证存在的弱点，从 HTTP/1.1 起就有了 DIGEST 认证。DIGEST 同样使用了质询/响应的方式，但不会像 BASIC 认证那样直接发送明文密码

​	所谓的质询响应方式是指，一开始一方会发送认证要求给另一方，接着使用另一方那里接收到的质询码计算生成响应码，最后将响应码返回给对方的认证方式。

DIGEST 认证步骤

![DIGEST 认证步骤](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/HTTP-DIGEST%E8%AE%A4%E8%AF%81.png)

1. 请求需认证的资源时，服务器会随状态码 401 Authorization Required，返回带 WWW-Authorizate 首部字段的响应，该字段内包含质询响应方式认证所需的临时质询码（随机数，nonce）

   首部字段 WWW-Authorizate 中必须包含 realm 和nonce 这两个字段的信息。客户端就是依靠向服务器回送的这两个值进行认证的

2. 接收到 401 状态码的客户端，返回的响应中包含 DIGEST 认证必须的首部字段 Authorization 信息

   首部字段 Authorization 内必须包含 username、realm、nonce、uri 和 response 的字段信息。其中，realm 和 nonce 就是从服务器接收到的响应中的字段。username 就是 realm 限定范围内可进行认证的用户名。uri 即 Request-URI 的值，但考虑到经代理转发后 Request-URI 的值可能被修改，因此会事先复制一份副本保存在 uri 内。response 也可以叫做 Request-Digest ，存放经过 MD5 运算后的密码字符串，形成响应码

3. 接收到包含首部字段 Authorizaton 请求的服务器，会确认认证信息的正确性，认证通过后则返回包含 Request-URI 资源的响应。并且在此时会在首部字段 Authorization-Info 写入一些认证成功的相关信息

DIGEST 认证的缺点：

* DIGEST 认证提供了高于 BASIC 认证的安全等级，但是和 HTTPS 的客户端认证相比仍旧很弱。
* DIGEST 认证提供了防止密码被窃听的保护机制，但是并不存在防止用户伪装的保护机制

DIGEST 认证和 BASIC 认证一样，使用上不那么便捷灵活，且仍旧达不到多数 Web 网站对高度安全等级的要求

### SSL 客户端认证

​	SSL 客户端认证时借由 HTTPS 客户端证书完成的方式，凭借客户端证书认证，服务器可确认访问是否来自已登陆的客户端

SSL 客户端认证的认证步骤：

​	为达到 SSL 客户端认证的目的，需要事先将客户端证书发送给客户端，且客户端必须安装此证书

1. 接收到需要认证资源的请求，服务器会发送 Certificate Request 报文，要求客户端提供客户端证书
2. 用户选择将发送的客户端证书后，客户端会把客户端证书信息以Client Certificate 报文方式发送给服务器
3. 服务器端验证客户端证书验证通过后方可领取证书内客户端的公开密钥，然后开始 HTTPS 加密通信

SSL 认证方式的缺点：

* 使用 SSL 认证方式需要用到客户端证书，而客户端证书需要支付一定费用才能使用。

### 基于表单认证

​	基于表单的认证方式并不是 HTTP 协议中定义的。客户端会向服务器上的 Web 程序发送登陆信息，按登陆信息的验证结果认证

​	根据 Web程序的时机安装，提供的用户界面及认证方式也各不相同。

​	由于使用上的便利性及安全性问题，HTTP 协议标准提供的 BASIC 认证和 DIGEST 认证几乎不怎么使用。另外，SSL 客户端认证虽然具有高度的安全等级，但因导入及维护费用问题，还尚未普及，故当前多半使用的是基于表单的认证

## 基于 HTTP 的功能追加协议

### HTTP 瓶颈

​	随着时代的发展，Web 的用途多样化，网站追求的功能越来越多，这些网站所追求的功能可通过 Web 应用和脚本程序实现，即使这些功能已经满足需求，在性能上未必是最优的，这是由于 HTTP 协议上的限制以及自身性能有限，以下这些 HTTP 标准就会成为瓶颈

* 一条连接上只可发送一个请求
* 请求只能从客户端开始。客户端不可以接收除响应之外的指令
* 请求/响应首部未经压缩就发送。首部信息越多延迟越大
* 发送冗余的首部。每次互相发送相同的首部造成浪费较多
* 可任意选择数据压缩格式。非强制压缩发送

### spdy

​	Google 在2010 年发布了 SPDY ，其开发目标旨在解决 HTTP 的性能瓶颈，缩短 Web 页面的加载时间。期间，陆续出现的 Ajax 和 Comet 等提高易用性的技术，一定程度上使 HTTP 得到改善，但 HTTP 协议本身限制令人束手无策，SPDY 为了在协议级别消除 HTTP 所遭遇的瓶颈，SPDY 并没有完全改写 HTTP 协议，而是在 TCP/IP 的应用层与运输层之间添加新会话层的形式运作

使用 SPDY 后，HTTP 协议获得额外以下功能：

* 多路复用流。通过单一的 TCP 连接，可以无限制处理多个 HTTP 请求
* 赋予请求优先级。SPDY 不仅可以无限制地并发处理请求，还可以给请求逐个分配优先级顺序，这样使为了在发送多个请求时，解决因带宽低而导致响应变慢的问题
* 压缩 HTTP 首部。压缩 HTTP 请求和响应的首部，通信的数据包数量和发送的字节数会更少
* 推送功能。支持服务器主动向客户端推送数据的功能，服务器可直接发送数据，而不必等待客户端请求
* 服务器提示功能。服务器可以主动提示客户端请求所需的资源

### WebSocket

​	WebSocket，即 Web 浏览器与 Web 服务器之间全双工通信标准，是 HTML5 标准的一部分，WebSocket API 由 W3C 定为标准。

​	一旦 Web 服务器与客户端之间建立起 WebSocket 协议的通信连接，之后所有的通信都依靠于这个专用的协议进行。通常过程中可项目发送 JSON 、XML、HTML 或图片等任意格式的数据

​	由于建立在 HTTP 基础上的协商，因此连接的发起方仍然是客户端，而一旦确认 WebSocket 通信连接，不论服务器还是客户端，任意一方都可直接向对方发送报文

WebSocket 协议的主要特点：

* 推送功能。支持由服务器向客户端推送数据的推送功能。
* 减少通信量。只要建立 WebSocket 连接，就可以一致保持连接状态。和 HTTP 相比，每次连接时的总开销减少，而且由于 WebSocket 的首部信息小，通信量也相应减少



#### 建立 WebSocket 连接

​	为了实现 WebSocket 通信，在 HTTP 连接建立之后，需要完成一次“握手步骤”

##### 握手 · 请求

​	为了实现 WebSocket 通信，需要使用到 HTTP 的 Upgrade 首部字段，告知服务器通信协议发生改变，已达到握手的目的

![WebSocket 握手请求报文](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/WebSocket%20%E6%8F%A1%E6%89%8B%E8%AF%B7%E6%B1%82%E6%8A%A5%E6%96%87.png)

​	Sec-WebSocket-Key 字段内记录者握手过程中必不可少的键值。Sec-WebSocket	-Protocol 字段记录使用的子协议

##### 握手 · 相应

​	对于之前的请求，返回状态码 101 Switching Protocol 的响应

![WebSocket 握手响应报文](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/WebSocket%20%E6%8F%A1%E6%89%8B%E5%93%8D%E5%BA%94%E6%8A%A5%E6%96%87.png)

​	Sec-WebSocket-Accept 的字段值是由握手请求中的 Sec-WebSocket-Key 的字段生成。成功握手确立 WebSocket 连接之后，通信是不再使用 HTTP 的数据组，而是使用 WebSocket 独立的数据帧



WebSocket 连接的建立过程如下：

![WebSocket 通信过程](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2017-2-1/WebSocket%20%E9%80%9A%E4%BF%A1%E8%BF%87%E7%A8%8B.png)