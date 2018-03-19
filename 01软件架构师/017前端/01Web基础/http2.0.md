https://www.zhihu.com/question/34074946

在http1.1协议中，浏览器在一段时间，针对同一域名下的请求有一定数量限制，超过限制数目的请求会被阻塞。

这也是为何一些站点会有多个静态资源 CDN 域名的原因之一，拿 Twitter 为例，

Clients that use persistent connections SHOULD limit the number of simultaneous connections that they maintain to a given server. A single-user client SHOULD NOT maintain more than 2 connections with any server or proxy. A proxy SHOULD use up to 2*N connections to another server or proxy, where N is the number of simultaneously active users. These guidelines are intended to improve HTTP response times and avoid congestion.

https://www.w3.org/Protocols/rfc2616/rfc2616-sec8.html#sec8.1.4

因此HTTP/2可以很容易的去实现多流并行而不用依赖建立多个TCP连接，HTTP/2把HTTP协议通信的基本单位缩小为一个一个的帧。

关键之一就是在 应用层(HTTP/2)和传输层(TCP or UDP)之间增加一个二进制分帧层。

在过去， HTTP 性能优化的关键并不在于高带宽，而是低延迟。

TLS：安全传输层协议
TLS：Transport Layer Security
概况
安全传输层协议（TLS）用于在两个通信应用程序之间提供保密性和数据完整性。该协议由两层组成： TLS 记录协议（TLS Record）和 TLS 握手协议（TLS Handshake）。较低的层为 TLS 记录协议，位于某个可靠的传输协议（例如 TCP）上面，与具体的应用无关，所以，一般把TLS协议归为传输层安全协议。
协议结构

什么是长连接

 HTTP1.1规定了默认保持长连接（HTTP persistent connection ，也有翻译为持久连接），数据传输完成了保持TCP连接不断开（不发RST包、不四次握手），等待在同域名下继续用这个通道传输数据；相反的就是短连接。


TCP/IP协议是Internet最基本的协议，绝大数的应用层协议都是建立在TCP/IP之上的。在TCP/IP协议的网络通信中我们需要依赖IP地址和端口进行端到端的通信，而Http协议追求无状态，所以在早期的Http协议中，每个Http请求都要求打开一个TCP Socket连接，并且使用一次之后就断开这个TCP连接。

什么是长连接(过去的理解有很大的问题)
 HTTP1.1规定了默认保持长连接（HTTP persistent connection ，也有翻译为持久连接），数据传输完成了保持TCP连接不断开（不发RST包、不四次握手），等待在同域名下继续用这个通道传输数据；相反的就是短连接。
HTTP首部的Connection: Keep-alive是HTTP1.0浏览器和服务器的实验性扩展，当前的HTTP1.1 RFC2616文档没有对它做说明，因为它所需要的功能已经默认开启，无须带着它，但是实践中可以发现，浏览器的报文请求都会带上它。如果HTTP1.1版本的HTTP请求报文不希望使用长连接，则要在HTTP请求报文首部加上Connection: close。《HTTP权威指南》提到，有部分古老的HTTP1.0 代理不理解Keep-alive，而导致长连接失效：客户端-->代理-->服务端，客户端带有Keep-alive，而代理不认识，于是将报文原封不动转给了服务端，服务端响应了Keep-alive，也被代理转发给了客户端，于是保持了“客户端-->代理”连接和“代理-->服务端”连接不关闭，但是，当客户端第发送第二次请求时，代理会认为当前连接不会有请求了，于是忽略了它，长连接失效。书上也介绍了解决方案：当发现HTTP版本为1.0时，就忽略Keep-alive，客户端就知道当前不该使用长连接。其实，在实际使用中不需要考虑这么多，很多时候代理是我们自己控制的，如Nginx代理，代理服务器有长连接处理逻辑，服务端无需做patch处理，常见的是客户端跟Nginx代理服务器使用HTTP1.1协议&长连接，而Nginx代理服务器跟后端服务器使用HTTP1.0协议&短连接。

在实际使用中，HTTP头部有了Keep-Alive这个值并不代表一定会使用长连接，客户端和服务器端都可以无视这个值，也就是不按标准来，譬如我自己写的HTTP客户端多线程去下载文件，就可以不遵循这个标准，并发的或者连续的多次GET请求，都分开在多个TCP通道中，每一条TCP通道，只有一次GET，GET完之后，立即有TCP关闭的四次握手，这样写代码更简单，这时候虽然HTTP头有Connection: Keep-alive，但不能说是长连接。正常情况下客户端浏览器、web服务端都有实现这个标准，因为它们的文件又小又多，保持长连接减少重新开TCP连接的开销很有价值。
 以前使用libcurl做的上传/下载，就是短连接，抓包可以看到：1、每一条TCP通道只有一个POST；2、在数据传输完毕可以看到四次握手包。只要不调用curl_easy_cleanup，curl的handle就可能一直有效，可复用。这里说可能，因为连接是双方的，如果服务器那边关掉了，那么我客户端这边保留着也不能实现长连接。    
？什么叫做每一条TCP通道只有一个POST

长连接的过期时间
上图中的Keep-Alive: timeout=20，表示这个TCP通道可以保持20秒。另外还可能有max=XXX，表示这个长连接最多接收XXX次请求就断开。
对于客户端来说，如果服务器没有告诉客户端超时时间也没关系，服务端可能主动发起四次握手断开TCP连接，客户端能够知道该TCP连接已经无效；另外TCP还有心跳包来检测当前连接是否还活着，方法很多，避免浪费资源。
长连接的数据传送完成识别
 使用长连接之后，客户端、服务端怎么知道本次传输结束呢？两部分：1是判断传输数据是否达到了Content-Length指示的大小；2动态生成的文件没有Content-Length，它是分块传输（chunked），这时候就要根据chunked编码来判断，chunked编码的数据在最后有一个空chunked块，表明本次传输数据结束。更细节的介绍可以看这篇文章。

并发连接数的数量限制
在web开发中需要关注浏览器并发连接的数量，RFC文档说，客户端与服务器最多就连上两通道，但服务器、个人客户端要不要这么做就随人意了，有些服务器就限制同时只能有1个TCP连接，导致客户端的多线程下载（客户端跟服务器连上多条TCP通道同时拉取数据）发挥不了威力，有些服务器则没有限制。浏览器客户端就比较规矩，限制了同域名下能启动若干个并发的TCP连接去下载资源。并发数量的限制也跟长连接有关联，打开一个网页，很多个资源的下载可能就只被放到了少数的几条TCP连接里，这就是TCP通道复用（长连接）。如果并发连接数少，意味着网页上所有资源下载完需要更长的时间（用户感觉页面打开卡了）；并发数多了，服务器可能会产生更高的资源消耗峰值。浏览器只对同域名下的并发连接做了限制，也就意味着，web开发者可以把资源放到不同域名下，同时也把这些资源放到不同的机器上，这样就完美解决了。

容易混淆的概念——TCP的keep alive和HTTP的Keep-alive
 TCP的keep alive是检查当前TCP连接是否活着；HTTP的Keep-alive是要让一个TCP连接活久点。它们是不同层次的概念。
 TCP keep alive的表现：
    当一个连接“一段时间”没有数据通讯时，一方会发出一个心跳包（Keep Alive包），如果对方有回包则表明当前连接有效，继续监控。
这个“一段时间”可以设置。具体做法google吧。

HTTP keep-alive
Http是一个”请求-响应”协议，它的keep-alive主要是为了让多个http请求共享一个Tcp连接，以避免每个Http又新建一个Tcp连接。每个Http服务器默认的keep-alive时间可能是不一样的。
TCP keep-alive
Tcp的keep-alive是Tcp协议的一种保鲜装置，当Tcp请求响应结束后，经过tcp_keep-alive_time时间后，服务器会发出监测包去看看改Tcp连接是否还是继续连接的，是否已经出现了网络问题，是否客户端崩溃了等等问题。如果发现出现了问题，那么服务端就会去关闭连接，把这个Tcp连接关闭。

关于TCP心跳
在目前这两年的移动互联网火热的环境下，推送应用的非常多。推送实现的就是通过TCP长连接实现的，因为移动网络很多时候都会不稳定，另外NAT过一段时间就会刷新,所以而要如何保证客户端和服务器端连接就成了一个问题。TCP心跳包就是客户端监测连接的，它先发送一个心跳包到服务器，服务器再Ask，通过这种方式判断是否目前的长连接是否可用，如果断了，则通知上层应用，并关闭连接，另外发送心跳包也是避免一段时间都没有通信，NAT超时，NAT表被刷新，导致连接失效。

HTTP与TCP keep-alive联系
直接介绍一个场景就可能更容易明白了。客户端发送了一个Http请求，服务器响应后，判断这个Http是否是keep-alive的，如果不是则关闭连接。如果是keep-alive，则等待keep-alive time后再关闭，如果这期间再收到一个http 请求，则继续等待最后一个请求的keep-alive time时间，直到keep-alive time时间内没有收到请求，则关闭。
上面是HTTP keep-alive的，而TCP是它下一层的协议，本身TCP是长连接的，除非主动关闭。HTTP的keep-alive time一般是15ms, 30ms之类的，如果是超过了HTTP的keep-alive time时间，则HTTP会关闭TCP连接。本身TCP是不会关闭连接的，TCP的keep alive是TCP的保鲜装置，在keep alive timeout 后服务端发送一个监测包来判断连接是否仍保持着，如果还是可连接，则继续保持，它不会主动关闭连接的。而心跳包是为了防止NAT超时。

TCP/IP协议是Internet最基本的协议，绝大数的应用层协议都是建立在TCP/IP之上的。在TCP/IP协议的网络通信中我们需要依赖IP地址和端口进行端到端的通信，而Http协议追求无状态，所以在早期的Http协议中，每个Http请求都要求打开一个TCP Socket连接，并且使用一次之后就断开这个TCP连接。

HTTP的1.0版本中附加了一个Header：**keep-alive**，这个头信息表示连接可以被复用，也就是当第一个请求完成之后，socket连接不关闭，下一条请求继续使用这个链接。在HTTP/1.1中，默认情况下所有连接都被保持，除非在Header中指明要关闭：**Connection: Close**



keep-alive确实很高效，节约了大量时间，但是它并不是没有缺点的。我们都知道一个操作系统的能支撑的连接数是有限的，使用ulimit命令可以查看当前系统的最大连接数。

xionger$ ulimit -n
12544

也就说当一台服务器的连接数超过这个数量，后面的网络请求就无法到达服务器了。keep-alive会让client和server之间的服务器不断开，即使它们之间没有数据传输。所以Server端需要控制好这个链接闲置的时间（timeout），一旦闲置时间超出就立即关闭这个链接。

**所以对于这种长连接，在用户操作频繁的场景下是非常节约资源的。相对的，用户操作并不是特别频繁，那么短链接显然可以承担更大的并发量。**

HTTP2.0
在过去，HTTP性能优化的关键并不在于贷款，而是低延迟。TCP连接会时间进行自我调谐，期初会限制连接的最大速度，如果数据成功传输，会随时间的推移提高传输的速度。这种调谐则成为TCP慢启动，由于该原因，原本就具有突发性和短时性的HTTP连接变的非常低效

单链接多资源方式，减少服务端的链接压力，内存占用更少，连接吞吐更大

首部压缩Header Compression
HTTP/1.1并不支持HTTP首部压缩，为此SPDY和HTTP/2应运而生，SPDY使用的是通用的DEFLATE算啊。。。

服务器推送Server Push（？。。。）

http1.0被抱怨最多的就是连接无法复用，和head of line blocking这两个问题。理解这两个问题有一个十分重要的前提：客户端是依据域名来向服务器建立连接，一般PC端浏览器会针对单个域名的server同时建立6～8个连接，手机端的连接数则一般控制在4～6个。显然连接数并不是越多越好，资源开销和整体延迟都会随之增大。连接无法复用会导致每次请求都经历三次握手和慢启动。三次握手在高延迟的场景下影响较明显，慢启动则对文件类大请求影响较大。

对于http1.0的实现，在第一个请求没有收到回复之前，后续从应用层发出的请求只能排队，请求2，3，4，5只能等请求1的response回来之后才能逐个发出。网络通畅的时候性能影响不大，一旦请求1的request因为什么原因没有抵达服务器，或者response因为网络阻塞没有及时返回，影响的就是所有后续请求，问题就变得比较严重了。

串行请求，并发请求


**一段时间内的连接复用对PC端浏览器的体验帮助很大**，因为大部分的请求在集中在一小段时间以内。但对移动app来说，成效不大，app端的请求比较分散且时间跨度相对较大。所以移动端app一般会从应用层寻求其它解决方案，**长连接方案或者伪长连接方案：**

方案一：基于tcp的长链接现在越来越多的移动端app都会**建立一条自己的长链接通道**（自己是因为没有编码过app所以对该部分没有经验，很吃亏），通道的实现是基于tcp协议。基于tcp的socket编程技术难度相对复杂很多，而且需要自己制定协议，但带来的回报也很大。信息的上报和推送变得更及时，在请求量爆发的时间点还能减轻服务器压力（http短连接模式会频繁的创建和销毁连接）。不止是IM app有这样的通道，**像淘宝这类电商类app都有自己的专属长连接通道了**。现在业界也有不少成熟的方案可供选择了，google的protobuf就是其中之一。

方案二：long-polling
客户端在初始状态就会发送一个polling请求到服务器，服务器并不会马上返回业务数据，而是等待有新的业务数据产生的时候再返回。所以连接会一直被保持，一旦结束马上又会发起一个新的polling请求，如此反复，所以一直会有一个连接被保持。服务器有新的内容产生的时候，并不需要等待客户端建立一个新的连接。做法虽然简单，但有些难题需要攻克才能实现稳定可靠的业务框架：

* 和传统的http短链接相比，长连接会在用户增长的时候极大的增加服务器压力
* 移动端网络环境复杂，像wifi和4g的网络切换，进电梯导致网络临时断掉等，这些场景都需要考虑怎么重建健康的连接通道。
* 这种polling的方式稳定性并不好，需要做好数据可靠性的保证，比如重发和ack机制。
* polling的response有可能会被中间代理cache住，要处理好业务数据的过期机制。

long-polling方式还有一些缺点是无法克服的，比如**每次新的请求都会带上重复的header信息**，还有数据通道是单向的，主动权掌握在server这边，客户端有新的业务请求的时候无法及时传送。

方案三：http streaming
同long-polling不同的是，server并不会结束初始的streaming请求，而是持续的通过这个通道返回最新的业务数据。显然这个数据通道也是单向的。
streaming是通过在server response的头部里增加”Transfer Encoding: chunked”来告诉客户端后续还会有新的数据到来。除了和long－polling相同的难点之外，streaming还有几个缺陷：
有些代理服务器会等待服务器的response结束之后才会将结果推送到请求客户端。对于streaming这种永远不会结束的方式来说，客户端就会一直处于等待response的过程中。业务数据无法按照请求来做分割，所以客户端没收到一块数据都需要自己做协议解析，也就是说要做自己的协议定制。streaming不会产生**重复的header数据。**

方案四：web socketWebSocket和传统的tcp socket连接相似，也是基于tcp协议，提供双向的数据通道。
WebSocket优势在于提供了message的概念，比基于字节流的tcp socket使用更简单，同时又提供了传统的http所缺少的长连接功能。不过WebSocket相对较新，2010年才起草，并不是所有的浏览器都提供了支持。各大浏览器厂商最新的版本都提供了支持。


1.4 解决head of line blockingHead of line blocking(以下简称为holb)是http2.0之前**网络体验的最大祸源**。正如图1中所示，健康的请求会被不健康的请求影响，而且这种体验的损耗受网络环境影响，出现随机且难以监控。为了解决holb带来的延迟，协议设计者设计了一种新的pipelining机制。http pipelining

不过pipelining并不是救世主，它也存在不少缺陷：

* pipelining只能适用于http1.1，一般来说，支持http1.1的server都要求支持pipelining。
* 只有幂等的请求（GET，HEAD）能使用pipelining，非幂等请求比如POST不能使用，因为请求之间可能会存在先后依赖关系。
* head of line blocking并没有完全得到解决，server的response还是要求依次返回，遵循FIFO(first in first out)原则。也就是说如果请求1的response没有回来，2，3，4，5的response也不会被送回来。
* 绝大部分的http代理服务器不支持pipelining。
* 和不支持pipelining的老服务器协商有问题。
* 可能会导致新的Front of queue blocking问题。

1.5 其它奇技淫巧
为了解决延迟带来的苦恼，永远都会有聪明的探索者找出新的捷径来。互联网的蓬勃兴盛催生出了各种新奇技巧，我们来依次看下这些“捷径”及各自的优缺点。

**Spriting（图片合并）**
Spriting指的是将多个小图片合并到一张大的图片里，这样多个小的请求就被合并成了一个大的图片请求，然后再利用js或者css文件来取出其中的小张图片使用。好处显而易见，请求数减少，延迟自然低。坏处是文件的粒度变大了，有时候我们可能只需要其中一张小图，却不得不下载整张大图，cache处理也变得麻烦，在只有一张小图过期的情况下，为了获得最新的版本，不得不从服务器下载完整的大图，即使其它的小图都没有过期，显然浪费了流量。

**Inlining（内容内嵌）**
Inlining的思考角度和spriting类似，是将额外的数据请求通过base64编码之后内嵌到一个总的文件当中。比如一个网页有一张背景图，我们可以通过如下代码嵌入：background: url(data:image/png;base64,)data部分是base64编码之后的字节码，这样也避免了一次多余的http请求。但这种做法也有着和spriting相同的问题，资源文件被绑定到了其它文件，粒度变得难以控制。

**Concatenation（文件合并）**
Concatenation主要是针对js这类文件，现在前端开发交互越来越多，零散的js文件也在变多。将多个js文件合并到一个大的文件里在做一些压缩处理也可以减小延迟和传输的数据量。但同样也面临着粒度变大的问题，一个小的js代码改动会导致整个js文件被下载。

**Domain Sharding（域名分片）**
前面我提到过很重要的一点，浏览器或者客户端是根据domain（域名）来建立连接的。比如针对Example Domain只允许同时建立2个连接，但mobile.example.com被认为是另一个域名，可以再建立两个新的连接。依次类推，如果我再多建立几个sub domain（子域名），那么同时可以建立的http请求就会更多，这就是Domain Sharding了。连接数变多之后，受限制的请求就不需要等待前面的请求完成才能发出了。这个技巧被大量的使用，一个颇具规模的网页请求数可以超过100，使用domain sharding之后同时建立的连接数可以多到50个甚至更多。

2.1 SPDY的目标(阿里的网关的相关分享)
降低延迟，客户端的单连接单请求，server的FIFO响应队列都是延迟的大头。http最初设计都是客户端发起请求，然后server响应，server无法主动push内容到客户端。压缩http header，http1.x的header越来越膨胀，cookie和user agent很容易让header的size增至1kb大小，甚至更多。而且由于http的无状态特性，header必须每次request都重复携带，很浪费流量。

所以SPDY的手术刀对准的是http。

* 降低延迟，客户端的单连接单请求，server的FIFO响应队列都是延迟的大头。
* http最初设计都是客户端发起请求，然后server响应，server无法主动push内容到客户端。
* 压缩http header，http1.x的header越来越膨胀，**cookie和user agent**很容易让header的size增至1kb大小，甚至更多。而且由于http的无状态特性，header必须每次request都重复携带，很浪费流量。

多路复用（multiplexing）。多路复用通过多个请求stream共享一个tcp连接的方式，解决了http1.x holb（head of line blocking）的问题，降低了延迟同时提高了带宽的利用率。

SPDY高级功能
* server推送（server push）。http1.x只能由客户端发起请求，然后服务器被动的发送response。开启server push之后，server通过X-Associated-Content header（X-开头的header都属于非标准的，自定义header）告知客户端会有新的内容推送过来。在用户第一次打开网站首页的时候，server将资源主动推送过来可以极大的提升用户体验。
* server暗示（server hint）。和server push不同的是，server hint并不会主动推送内容，只是告诉有新的内容产生，内容的下载还是需要客户端主动发起请求。server hint通过X-Subresources header来通知，一般应用场景是客户端需要先查询server状态，然后再下载资源，可以节约一次查询请求。

**页面加载时间相比于http1.x减少了64%**

我们来看下HTTP2.0一些重要的设计前提：

* 客户端向server发送request这种基本模型不会变。
* 老的scheme不会变，使用http://和https://的服务和应用不会要做任何更改，不会有http2://。
* 使用http1.x的客户端和服务器可以无缝的通过代理方式转接到http2.0上。不识别http2.0的代理服务器可以将请求降级到http1.x。

**为什么经常谈到代理服务器？比较懵逼**
google制定SPDY的时候也遇到了这个问题，他们的办法是强制SPDY走https，在**SSL层**完成这个协商过程。ssl层的协商在http协议通信之前，所以是最适合的载体。
HTTP
SSL
TCP

另一个tls的拓展叫ALPN（Application Layer Protocol Negotiation）
各浏览器（除了IE）之所以只实现了基于SSL的HTTP2.0，另一个原因是走SSL请求的成功率会更高，被SSL封装的request不会被监听和修改，这样网络中间的网络设备就无法基于http1.x的认知去干涉修改request，http2.0的request如果被意外的修改，请求的成功率自然会下降。
对于app开发者来说，他们可以坚持使用没有ssl的http2.0，不过要承担一个多余的RTT延迟和请求可能被破坏的代价。

新的二进制格式（Binary Format）http1.x诞生的时候是明文协议，其格式由三部分组成：start line（request line或者status line），header，body。要识别这3部分就要做协议解析，http1.x的解析是基于文本。基于文本协议的格式解析存在天然缺陷，文本的表现形式有多样性，要做到健壮性考虑的场景必然很多，二进制则不同，只认0和1的组合。基于这种考虑http2.0的协议解析决定采用二进制格式，实现方便且健壮。

实际上现在很多请求都是走https了，要调试https请求必须有私钥才行。http2.0的绝大部分request应该都是走https，所以调试方便无法作为一个有力的考虑因素了。curl，tcpdump，wireshark这些工具会更适合http2.0的调试。

重置连接表现更好很多app客户端都有取消图片下载的功能场景，对于http1.x来说，是通过设置tcp segment里的reset flag来通知对端关闭连接的。这种方式会直接断开连接，下次再发请求就必须重新建立连接。http2.0引入RST_STREAM类型的frame，可以在不断开连接的前提下取消某个request的stream，表现更好。


Server PushServer Push的功能前面已经提到过，http2.0能通过push的方式将客户端需要的内容预先推送过去，所以也叫“cache push”。另外有一点值得注意的是，客户端如果退出某个业务场景，出于流量或者其它因素需要取消server push，也可以通过发送RST_STREAM类型的frame来做到。
流量控制（Flow Control）
TCP协议通过sliding window的算法来做流量控制。发送方有个sending window，接收方有receive window。



[HTTP/2.0 相比1.0有哪些重大改进？](https://www.zhihu.com/question/34074946)











