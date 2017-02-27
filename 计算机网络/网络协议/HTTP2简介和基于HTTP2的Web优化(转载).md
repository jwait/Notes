# HTTP2简介和基于HTTP2的Web优化

本文定位入门级别，分作两大块：

- HTTP/2是什么
- 基于HTTP/2前端可以做什么优化

本文参考了一些*博文和资料*，后面已列出，感谢他们的分享。

## HTTP/2简介

> HTTP/2 is a replacement for how HTTP is expressed “on the wire.” It is not a ground-up rewrite of the protocol; HTTP methods, status codes and semantics are the same, and it should be possible to use the same APIs as HTTP/1.x (possibly with some small additions) to represent the protocol.

HTTP/2是现行HTTP协议（HTTP/1.x）的替代，但它不是重写，HTTP方法/状态码/语义都与HTTP/1.x一样。HTTP/2基于SPDY3，专注于**性能**，最大的一个目标是在用户和网站间只用一个连接（connection）。

HTTP/2由两个规范（Specification）组成：

1. Hypertext Transfer Protocol version 2 - RFC7540
2. HPACK - Header Compression for HTTP/2 - RFC7541

### 为什么需要HTTP/2

我们知道，影响一个HTTP网络请求的因素主要有两个：**带宽**和**延迟**。在今天的网络情况下，带宽一般不再是瓶颈，所以我们主要讨论下延迟。延迟一般有下面几个因素：

1. 浏览器阻塞（Head-Of-Line Blocking）：浏览器会因为一些原因阻塞请求。
2. DNS查询。
3. 建立连接（Initial connection）：HTTP基于 TCP 协议，TCP的3次握手和慢启动极大增加延迟。

说完背景，我们讨论下HTTP/1.x中到底存在哪些问题？

#### HTTP/1.x的缺陷

1. **连接无法复用**：连接无法复用会导致每次请求都经历三次握手和慢启动。三次握手在高延迟的场景下影响较明显，慢启动则对文件类大请求影响较大。
   - HTTP/1.0传输数据时，每次都需要重新建立连接，增加延迟。
   - 1.1虽然加入keep-alive可以复用一部分连接，但域名分片等情况下仍然需要建立多个connection，耗费资源，给服务器带来性能压力。
2. **Head-Of-Line Blocking**：导致带宽无法被充分利用，以及后续健康请求被阻塞。[HOLB](http://stackoverflow.com/questions/25221954/spdy-head-of-line-blocking)是指一系列包（package）因为第一个包被阻塞；HTTP/1.x中，由于服务器必须按接受请求的顺序发送响应的规则限制，那么假设浏览器在一个（tcp）连接上发送了两个请求，那么服务器必须等第一个请求响应完毕才能发送第二个响应——[HOLB](https://en.wikipedia.org/wiki/HTTP_pipelining)。
   - 虽然现代浏览器允许每个origin建立6个connection，但大量网页动辄几十个资源，HOLB依然是主要问题。
3. **协议开销大**：HTTP/1.x中header内容过大（每次请求header基本不怎么变化），增加了传输的成本。
4. **安全因素**：HTTP/1.x中传输的内容都是明文，客户端和服务端双方无法验证身份。

### HTTP/2的新特性

因为HTTP/1.x的问题，人们提出了各种解决方案。比如希望复用连接的*长链接/http long-polling/websocket*等等，解决HOLB的`Domain Sharding（域名分片）/inline资源/css sprite`等等。

不过以上优化都绕开了协议，直到谷歌推出SPDY，才算是正式改造HTTP协议本身。降低延迟，压缩header等等，SPDY的实践证明了这些优化的效果，也最终带来HTTP/2的诞生。

**1. 新的二进制格式（Binary Format）**

http1.x诞生的时候是明文协议，其格式由三部分组成：start line（request line或者status line），header，body。要识别这3部分就要做协议解析，http1.x的解析是基于文本。基于文本协议的格式解析存在天然缺陷，文本的表现形式有多样性，要做到健壮性考虑的场景必然很多，二进制则不同，只认0和1的组合。基于这种考虑http2.0的协议解析决定采用二进制格式，实现方便且健壮。

[![image](https://cloud.githubusercontent.com/assets/8046480/21041065/a7779db0-be23-11e6-86b9-3c365a8d8033.png)](https://cloud.githubusercontent.com/assets/8046480/21041065/a7779db0-be23-11e6-86b9-3c365a8d8033.png)

http2的格式定义十分高效且精简。length定义了整个frame的大小，type定义frame的类型（一共10种），flags用bit位定义一些重要的参数，stream id用作流控制，payload就是request的正文。

[![image](https://cloud.githubusercontent.com/assets/8046480/21041167/411f0f84-be24-11e6-946e-126b1036dd15.png)](https://cloud.githubusercontent.com/assets/8046480/21041167/411f0f84-be24-11e6-946e-126b1036dd15.png)

**2. Header压缩**

http1.x的header由于cookie和user agent很容易膨胀，而且每次都要重复发送。

http2.0使用encoder来减少需要传输的header大小，通讯双方各自cache一份header fields表，既避免了重复header的传输，又减小了需要传输的大小。高效的压缩算法可以很大的压缩header，减少发送包的数量从而降低延迟。

**3. 流（stream）和多路复用（MultiPlexing）**

*什么是stream？*

> Multiplexing of requests is achieved by having each HTTP request/response exchange associated with its own stream. Streams are largely independent of each other, so a blocked or stalled request or response does not prevent progress on other streams.

> A "stream" is an independent, bidirectional sequence of frames exchanged between the client and server within an HTTP/2 connection.

> A client sends an HTTP request on a new stream, using a previously unused stream identifier. A server sends an HTTP response on the same stream as the request.

> Multiplexing of requests is achieved by having each HTTP request/response exchange associated with its own stream.

翻译下，stream就是在HTTP/2连接上的双向帧序列。每个http request都会新建自己的stream，response在同一个stream上返回。

多路复用（MultiPlexing），即连接共享。之所以可以复用，是因为每个stream高度独立，堵塞的stream不会影响其它stream的处理。一个连接上可以有多个stream，每个stream的frame可以随机的混杂在一起，接收方可以根据stream id将frame再归属到各自不同的request里面。

**3.1 流量控制（Flow Control）**

类似TCP协议通过sliding window的算法来做流量控制，http2.0使用 WINDOW_UPDATE frame 来做流量控制。每个stream都有流量控制，这保证了数据接收方可以只让自己需要的数据被传输。

![2016-12-09 7 39 28](https://cloud.githubusercontent.com/assets/8046480/21047939/909dad72-be47-11e6-84a7-8101f6cd4e39.png)

**3.2 流优先级（Stream Priority）**

每个流可以设置优先级。优先级的目标是允许终端高速对端（当对端处理并发流时）怎么分配资源。

更重要的是，当传输能力有限时，优先级可以用来挑选哪些流（高优先级）优先传输——这是优化浏览器渲染的关键，即服务端负责设置优先级，使重要的资源优先加载，加速页面渲染。

**4. Server Push**

Server Push即服务端能通过push的方式将客户端需要的内容预先推送过去，也叫“cache push”。

![2016-12-09 7 27 03](https://cloud.githubusercontent.com/assets/8046480/21047558/7fd34bb6-be45-11e6-869c-ae3136e17180.png)

 

## 基于HTTP/2的Web优化

### 从HTTP/1.x到HTTP/2的通用优化规则

尽管HTTP/2相比HTTP/1.x有很大的不同，但有几条优化规则是仍然适用的：

1. 减少DNS查询。DNS查询需要时间，没有resolved的域名会阻塞请求。
2. 减少TCP连接。HTTP/2只使用一个TCP连接。
3. 使用CDN。使用CDN分发资源可以减少延迟。
4. 减少HTTP跳转。特别是非同一域名的跳转，需要DNS，TCP，HTTP三种开销。
5. 消除不必要的请求数据。HTTP/2压缩了Header。
6. 压缩传输的数据。gzip压缩很高效。
7. 客户端缓存资源。缓存是必要的。
8. 消除不必要的资源。激进的提前获取资源对客户端和服务端都开销巨大。

### 因HTTP/2而不一样的优化规则

然后下面要介绍一些HTTP/1.x里推荐而HTTP/2禁止的优化。

1. Domain Sharding（域名分片）：HTTP/1.x中浏览器一般每个域名最多同时使用6个连接。每个连接都会经历3次握手，会消耗服务器资源，甚至会相互堵塞。然而1.x中我们仍然使用Domain Sharding（域名分片）来突破连接数的限制（提高并行加载能力）。

   但是，多少域名合适，每个连接的资源消耗，带宽竞争，DNS查询时间等等都是问题。在HTTP/2里，多路复用完美解决问题。所以请不要在HTTP/2里使用域名分片。

2. Concatenation（文件合并）：1.x中我们经常合并文件来减少请求。但是，

   - 大文件会延迟（delay）客户端的处理执行（必须等到整个文件下载完）。
   - 缓存失效的开销昂贵：很少量的数据更新会使整个大文件失效，然后需要重新下载整个大文件。
   - 破坏了颗粒化的资源的缓存，更新和重新生效。
   - 此外，文件合并也需要额外的构建处理（build step），增加项目复杂度。

   所以在HTTP/2里，请避免合并文件。使用小的颗粒化的资源，优化缓存政策。

3. Inline resource（内联资源）：内联资源也是1.x中常用的优化手段，可以减少请求。但是，内联资源无法独立缓存，也破坏了HTTP/2的多路复用和优先级策略（使用了父资源的优先级，也没法被客户端拒绝）。

   HTTP/2中不要再使用内联资源，直接利用Server push：

   - 颗粒化的资源可以被独立缓存。
   - 颗粒化的资源可以被正确地mux'ed（利用多路复用传输）和设置优先级。
   - 允许客户端更灵活地控制资源的下载和使用。

### HTTP/2需要特别考虑的优化规则

> With HTTP/2 the browser is **relying on the server** to deliver data in an optimal way --- this is critical

HTTP/1.1时，浏览器会通过维持一个优先级队列来给资源设置优先级，然后猜测可用TCP连接的最佳利用方式——这延迟了请求发出。

![2016-12-09 11 30 29](https://cloud.githubusercontent.com/assets/8046480/21054175/82a1b5cc-be67-11e6-83e4-7699288460ed.png)

HTTP/2中，浏览器根据`type/context`来给资源设置优先级，并且在发现资源后立即发出请求。优先级以权重+依赖（weights+dependencies）的形式传达给服务端。这时候，服务端必须给重要资源设置更高优先级，更早地传给浏览器，使浏览器能获取必要资源，尽早渲染。

![2016-12-09 11 35 41](https://cloud.githubusercontent.com/assets/8046480/21054334/3a40d55a-be68-11e6-9e54-5f88249c02e6.png)

 

## 参考

- [HTTP2主页](https://http2.github.io/)
- [HTTP,HTTP2.0,SPDY,HTTPS你应该知道的一些事](http://www.alloyteam.com/2016/07/httphttp2-0spdyhttps-reading-this-is-enough/) --- 朴实易懂！
- [Optimizing Application Delivery](https://hpbn.co/optimizing-application-delivery/)
- [HTTP2 is here, let's optimize!](https://docs.google.com/presentation/d/1r7QXGYOLCh4fcUq0jDdDwKJWNqWK1o4xMtYpKZCJYjM/present?slide=id.p19) --- 超棒的PPT！
- [HTTP 2.0的那些事](http://mrpeak.cn/blog/http2/)

**截图来自 HTTP2 is here, let's optimize!**