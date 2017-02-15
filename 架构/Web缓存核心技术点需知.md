## Web缓存核心技术点需知

原创 *2016-11-13* *magedu-stanley* [运维部落](javascript:void(0);)

- [Web缓存核心技术点需知](undefined)

- - [5.1 HTTP首部控制](undefined)
  - [5.2 基于新鲜度检测机制：](undefined)
  - [2.1 特征1：时间局部性](undefined)
  - [2.2 特征2：空间局部性](undefined)
  - [2.3 缓存的优点](undefined)
  - [2.4 哪类数据应该被缓存](undefined)
  - [2.5 哪类数据可缓存但不应该被缓存](undefined)
  - [2.6 缓存命中率决定缓存有效性](undefined)
  - [2.7 缓存数据生命周期](undefined)
  - [2.8 缓存处理步骤](undefined)
  - [2.9 缓存和普通数据读取的区别](undefined)
  - [1. 完整请求流程中缓存设计](undefined)
  - [2. 缓存特征点](undefined)
  - [3. 智能DNS解析（CDN）](undefined)
  - [4. 常见的缓存工具实现](undefined)
  - [5. 常见缓存控制机制](undefined)
  - [6. 常见的HTTP首部字段类型](undefined)

# Web缓存核心技术点需知

------

**序：** 
该文共约9800字（6章），前5章建议阅读时长为8分钟。第6章为知识点整理，可后续检索回顾。 
**祝君愉快**

------

近些年，互联网基础设备和技术迅猛发展，互联网玩法日新月异，稍不留神就Out。整体网民的素质也在不断提升的同时，对互联网的体验也提出了新的高度和要求，众所周知智能背后意味着复杂，体验好背后也意味着互联网的架构越复杂。利益当先的前提下，最好的优化就是缓存，缓存在整个互联网的发展过程中作用可想而知。尤其在中国如此蹩脚的网络下，南电信北联通，中间坑的都是付费的用户和企业。越来越多的证明表明，网站访问速度越慢，用户流失的越快，要想加快网站访问速度，基于此背景条件下，缓存和反向代理更显的尤为重要。

## 1. 完整请求流程中缓存设计

从互联网起始至今缓存都无处不在，从最前端的用户侧开始到服务器架构设计领域均在缓存的设计。 
用户一次完整的访问简易流程图如下：

![img](http://mmbiz.qpic.cn/mmbiz_jpg/rtibSseGoBickqQUvEflCy2eB1sPTkCpIes3kk0VDYjhnDGEfbkOeED61aTlnh5N9VDZicBuYHu41b1JPe8XVvHng/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

------

> 1.用户端

**系统** 
基于DNS缓存: 缓解DNS解析压力，提高解析响应速度

**浏览器** 
基于内容缓存: 根据已有规则与服务器交互过程中，非过期文件不再从服务器请求全新内容

**DNS（CDN）** 
基于DNS：基于DNS的智能解析，将用户的请求解析至距离用户最近的CDN缓存服务器，图片，样式等静态和常期不变的资源交由CDN响应，缓解RealServer压力和高峰访问拥塞。

------

> 2.企业端

**硬LB** 
请求经防火墙过滤后，传递至后端硬LB服务器，再根据不同场景分发至后端** 软LB\**

**软LB** 
软LB再次过滤拆分请求，动静分离，静态请求分离至静态集群，动态请求分发至应用集群。

**NoSQL（Cache）** 
NoSQL：Redis,Memcache,ZeroMQ等NoSQL的出现大大缓解了前端直接穿透对数据库和分布式文件集群服务器的压力。静态文件读取方面，Squid,Memcache,Varnish等缓存率理想情况下更高达80%以上。

*大型成熟的高可用网站拓扑可参考如下-摘自网络*

![img](http://mmbiz.qpic.cn/mmbiz_jpg/rtibSseGoBickqQUvEflCy2eB1sPTkCpIec5SDrVCu4MbLrPk5pan9bFhtWkTaSZXzEVuGAEEjStNciagFPzbVuPA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

## 2. 缓存特征点

并非所有的数据被缓存或需要缓存，缓存是为了解决20%数据被80%的人频繁访问的问题而生。数据如希望被缓存往往具备变化缓慢的特征。被缓存的数据往往具备如下特性：

### 2.1 特征1：时间局部性

缓存的数据往往被打有时间缀，具有定期失效的特征，过期后会从源服务器检验请求验证是否需要重新拉取数据。 
某数据被访问后，该数据往往会再次在短时间内被访问到。

### 2.2 特征2：空间局部性

被访问数据的周边数据被访问的概率会比其它常规数据访问大很多，所以这些访问数据和其它周边有可能被访问的数据通过某种方式集中在一起，以提高数据的被访问速度，减少数据查找时长。 
完成这类功能的工具往往称为Cache。

### 2.3 缓存的优点

缓存的优点无需赘述： 
\1. 节约带宽 
\2. 缓存后端服务器请求穿透压力 
\3. 降低时延，加速响应

### 2.4 哪类数据应该被缓存

1. 热（区）数据：所谓热（区）数据就是指经常被访问到的数据，这类数据被缓存最有价值，缓存命中率高

### 2.5 哪类数据可缓存但不应该被缓存

1. 用户账号密码信息等数据，该类数据不仅不应该被缓存，反而要被着重保护，这些年发生的撞库，密码破解等恶性事件，往往都是因为用户个人不当心或企业安全意味不足，导致用户敏感信息流失。

### 2.6 缓存命中率决定缓存有效性

缓存命中率=hit/(hit+mixx) 
hit表示缓存被命中，miss表示没有命中，也就是缓存项中没有对应的资源 
文档命中率：从文档命中的个数进行衡量 
字节命中率：从内容命中的大小(字节)进行衡量

### 2.7 缓存数据生命周期

数据被缓存数并非永久缓存。根据业务所需和服务器容量及用户行为，各类数据会定期被清理。如王宝强离婚事件，据说因此微博缓存用品横向扩展一倍服务器来响应该突发热点事件，但该热点事情结束后这批服务器是需要下架以节约成本，同时因为热点事件的结束，该时期的热点词及缓存数据将不再被缓存以做清理，该类清理往往需要手动清除，而对于静态文件图片，css等文件，则会设置定期清理，总结如下：

```
缓存清理策略：
1. 缓存项过期：缓存资源往往会被设置有效时长，过期自动清理或失效
2. 缓存空间用尽：缓存空间用尽时，会根据LRU（最近最小使用）算法清理缓存
3. 清理策略设置过长过短都不好，过长数据容易陈旧，过短起不到缓存效果

```

### 2.8 缓存处理步骤

Start接收请求解析请求查询缓存新鲜度检测构建响应报文发送响应报文记录日记End

### 2.9 缓存和普通数据读取的区别

缓存数据是**K/V**数据，**即key/value**，缓存的存储方式一般是以文件到的MD5码的前某些字符为键，做为存储目录，存储目录一般是**多级的**，**其值则可能是整个MD5码，也可能时MD5码减去key所用的字符**

**k/v**存储的好处是检索速度速度快，具有幂等性，此处的幂等性指的是，**无论查找多少次所消耗的时间都是一样的**。

## 3. 智能DNS解析（CDN）

**“缓存最好的实现方式莫过于CDN”** 
各软件缓存功能实则已是用户请求的最后一环，要想尤佳的体验，CDN是无上之选，智能解析可以做到将用户的请求解析到离用户最近的缓存服务器，缓存命中则直接返回给客户端，若不命中，有可能请求离自己最近的缓存服务器，而不是直接请求真实的服务器，因为缓存服务器往往是形成一个网络

对于国内的分裂网络(电信，联通，移动)，CDN可以判断用户来自于哪各运营商的网络，解析请求到相应网络的服务器主机，前提时需要提供web服务运营商需要在不同的的电信运营商部署服务器

**GSLB**：Globle Servive Load Balance 全局服务负载均衡 
部署专用的一台主机，调度用户请求到离用户最近缓存服务器。 
有全局也有就局部负载均衡，局部服务负载均很器作用是调度本地缓存服务器

## 4. 常见的缓存工具实现

**squid&varnish** 
基于http协议提专业的供缓存控制的程序有squid和varnish，squid比较重量级，是个老家伙，不过，老当益壮，而varnish比较轻量，性能和稳定性于squid不相上下

**nginx&httpd** 
nginx和httpd做为提供web服务的程序，同时也提供了简单的缓存功能，一般在条件允许都因该使用专业的缓存控制软件，而不使用自带的缓存功能

## 5. 常见缓存控制机制

### 5.1 HTTP首部控制

**过期日期（Expires）:**

> 首部格式：Expires:Fri, 20 May 2016 02:03:18 GMT

```
HTTP1.0版本使用的缓存控制，在响应报文加上Expire首部定义资源的有效日期，其定义的时间为绝对时间，此中控制方式会出现时区差产生的问题

```

**Cache-Control:**

> 首部格式：Cache-Control: max-age=600

```
HTTP1.1版本才添加的缓存控制机制，其在请求报文或响应报文首部添加一个cache-control的首部，用于定义资源的缓存最大时长，是相对于响应报文首部中的date首部定义的时间。
一般响应报文首部会同时有Expires首部和Cache-control首部

```

> Cache-Control首部相关指令：

```
cache-request-directive=no-cache 不接受缓存响应
                                no-store 不缓存在本地
                                max-age  缓存最大有效时长       
                                min-fresh 
        cache-response-directive=public
                                 private
                                 no-cache 
                                 no-store
                                 must-revalidate
                                 max-age
        no-cache：可缓存，但用户每次请求都需要先到上游服务器做缓存检验

```

> 请求报文缓存控制首部

```
用于发送请求报文时，明确提示所请求的服务器自己是否接收缓存数据或只接收哪类数据（这种情况可能无效，因为缓存代理可能时多级的，可能到了第一个缓存服务器就结束了）

```

> 响应报文缓存控制首部

```
用于发送响应报文时，明确标示响应的数据是属于哪种类型

```

> `小结：`基于过期时间的缓存控制，只要缓存命中且在有效期内，都将以缓存数据响应用户，否则就请求上游服务器获取资源

### 5.2 基于新鲜度检测机制：

> 有效性再验证：revalidate

用户第一次请求的资源被缓存之后，当用户再次请求时，缓存服务器会先与后端缓存服务器做缓存有效性校验，就有如下情况： 
**(1)如果原始内容没有发生改变，则仅响应首部，不响应body部分(响应码304)** 
**(2)如果原始内容发生改变，则正常响应(响应码200)** 
**(3)如果原始内容消失，则响应404**

> 条件式请求首部

**If-Modifified-Since:** 基于原始内容的最近一次修改的时间戳进行验证，如果发生改变，则200响应，否则304响应

**If-Unmodified-Since:** 基于原始内容的最近一次修改的时间戳进行验证，如果没有发生改变，则304响应，否则200响应

**If-Match:** 使用标签(Etag)进行有效性检验，标签吻合，说明内容没有发生改变，以304响应此次的标签可以理解为文件的校验码，只要文件发生改变，则校验码必定发生改变

**If-None-Match:** 使用标签(Etag)进行有效性检验，标签不吻合，则以200正常 
响应，否则以304响应

```
********小结********
*(1)基于新鲜度检测机制的缓存控制有个缺点，缓存服务器对于客户端的每次请求，都要做有效性验证，这样虽然保证了响应客户端的内容的新鲜度，但也造成了麻烦，所以就有了过期时间控制和新鲜度控制两种控制机制的结合：

*(2)只要缓存没有过期，都以缓存内容响应用户，若缓存过期，则需要向后端服务器做有效性验证，如果原始内容没有发生改变，则继续以缓存内容响应用户，若原始内容发生改变，则取得原始内容先缓存再响应客户端

*(3)其实浏览器也会将内容缓存在本地，用户再次对同一资源请求时，浏览器和缓存服务器之间会基于标签(**Etag**)进行有效性检测，请求的内容没有发生，则由浏览器缓直接响应，否则就以上述的各缓存控制机制给予响应

```

## 6. 常见的HTTP首部字段类型

- **通用首部字段（General Header Fields）:** 请求报文和响应报文两方都会使用的首部；
- **请求首部字段（Request Header Fields）:** 从客户端向服务器发送请求报文时使用的首部。补充了请求的附加内容，客户端的信息，响应内容相关的优先级等信息。
- **响应首部字段（Response Header Fields）:** 从服务器向客户端返回响应报文时使用的首部。补充了响应的附加内容，也会要求客户端附加额外的内容信息。
- **实体首部字段（Entity Header Fields）:** 针对请求报文和响应报文的实体部分使用的首部。补充了资源内容更新时间等与实体有关的信息。

**HTTP常用首部字段预览表**

> 通用首部字段

| 首部字段名             | 说明            |
| ----------------- | ------------- |
| Cache-Control     | 控制缓存的行为       |
| Connection        | 逐跳首部，连接的管理    |
| Date              | 创建报文的日期时间     |
| Pragna            | 报文指令          |
| Trailer           | 报文末端的首部一览     |
| Transfer-Encoding | 指定报文主体的传输编码方式 |
| Upgrade           | 升级为其他协议       |
| Via               | 代理服务器的相关信息    |
| Warning           | 错误通知请求首部字段    |

> 请求首部字段

| 首部字段名               | 说明                              |
| ------------------- | ------------------------------- |
| Accept              | 用户代理可处理的媒体类型                    |
| Accept—Charset      | 优先的字符集                          |
| Accept-Encoding     | 优先的内容编码                         |
| Accept-Language     | 优先的语言（自然语言）                     |
| Authorization       | Web认证信息                         |
| Expect              | 期待服务器的指定行为                      |
| From                | 用户的电子邮箱地址                       |
| Host                | 请求资源所在服务器                       |
| if-Match            | 比较实体标记（ETag）                    |
| if-Modified-Since   | 比较资源的更新时间                       |
| if-None-Match       | 比较实体标记（与if-Match相反）             |
| if-Range            | 资源为更新时发送实体Byte的范围请求             |
| if-Unmodified-Since | 比较资源的更新时间（与if-Modified-Since相反） |
| Max-Forwards        | 最大传输逐跳数                         |
| Proxy-Authorization | 代理服务器要求客户端的认证信息                 |
| Range               | 实体字节范围请求                        |
| Referer             | 对请求中的URL的原始获取方法                 |
| TE                  | 传输编码的优先级                        |
| User-Agent          | HTTP客户端程序的信息                    |

> 响应首部字段

| 首部字段名              | 说明             |
| ------------------ | -------------- |
| Accept-Ranges      | 是否接受字节范围请求     |
| Age                | 推算资源创建经过时间     |
| ETag               | 资源的匹配信息        |
| Location           | 令客户端重定向至指定的URL |
| Proxy-Authenticate | 代理服务器对客户端的认证信息 |
| Rety-After         | 对再次发起请求的时机要求   |
| Server             | HTTP服务器的安装信息   |
| Vary               | 代理服务器缓存的管理信息   |
| WWW-Authenticate   | 服务器对客户端的认证信息   |

> 实体首部字段

| 首部字段名            | 说明             |
| ---------------- | -------------- |
| Allow            | 资源科支持的HTTP方法   |
| Content-Encoding | 实体主体适用的编码方式    |
| Content-Language | 实体主体的自然语言      |
| Content-Length   | 实体主体的大小（单位：字节） |
| Content-Location | 替代对资源的URL      |
| Content-MD5      | 实体主体的报文摘要      |
| Content-Range    | 实体主体的位置范围      |
| Content-Type     | 实体主体的媒体类型      |
| Expires          | 实体主体过期的日期时间    |
| Last-Modified    | 资源的最后修改日期时间    |
| 为Cookie服务的首部字段   |                |

> cookie服务首部字段

| 首部字段名      | 说明                |
| ---------- | ----------------- |
| Set-Cookie | 开始状态管理所有的Cookie信息 |
| Cookie     | 服务器接收到的Cookie信息   |

> set-cookie字段属性

| 属性           | 说明                                       |
| ------------ | ---------------------------------------- |
| NAME=VALUE   | 赋予Cookie的名称和其值                           |
| expires=DATE | Cookie的有效期（若不mingque指定则默认为浏览器关闭前为止）      |
| path=PATH    | 将服务器上的文件目录作为Cookie的适用对象（若不指定则默认为文档所在的目录） |
| domain=域名    | 作为Cookie适用对象的域名（若不指定则默认为创建Cookie的服务器的域名） |
| Scure        | 仅在HTTPS安全通信时才会发送Cookie                   |
| HttpOnly     | 加以限制，使Cookie不能被JavaScript脚本访问            |

------

![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAIAAACQd1PeAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyBpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMC1jMDYwIDYxLjEzNDc3NywgMjAxMC8wMi8xMi0xNzozMjowMCAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RSZWY9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZVJlZiMiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENTNSBXaW5kb3dzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOkJDQzA1MTVGNkE2MjExRTRBRjEzODVCM0Q0NEVFMjFBIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOkJDQzA1MTYwNkE2MjExRTRBRjEzODVCM0Q0NEVFMjFBIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6QkNDMDUxNUQ2QTYyMTFFNEFGMTM4NUIzRDQ0RUUyMUEiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6QkNDMDUxNUU2QTYyMTFFNEFGMTM4NUIzRDQ0RUUyMUEiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz6p+a6fAAAAD0lEQVR42mJ89/Y1QIABAAWXAsgVS/hWAAAAAElFTkSuQmCC)

