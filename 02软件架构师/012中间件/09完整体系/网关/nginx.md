http://nginx.org/en/docs/http/configuring_https_servers.html


upstream server和proxy_pass

upstream backend{


}

#OpenResty
OpenResty(又称：ngx_openresty) 是一个基于 NGINX 的可伸缩的 Web 平台，由中国人章亦春发起，提供了很多高质量的第三方模块。
OpenResty 是一个强大的 Web 应用服务器，Web 开发人员可以使用 Lua 脚本语言调动 Nginx 支持的各种 C 以及 Lua 模块,更主要的是在性能方面，OpenResty可以 快速构造出足以胜任**10K**以上并发连接响应的超高性能 Web 应用系统。
OpenResty 的目标是让你的 Web 服务直接跑在 Nginx 服务内部,充分利用 Nginx 的非阻塞 I/O 模型,不仅仅对 HTTP 客户端请求,甚至于对远程后端诸如 MySQL,PostgreSQL,~Memcaches 以及 ~Redis 等都进行一致的高性能响应。

meituan深度使用我们用openresty构建了美团的七层负载均衡接入层，美团所有HTTP流量都会经过OR，用Lua在接入层写了许多业务逻辑（包括流控、鉴权、waf、分流等），类似KONG的角色。美团云CDN的整个生态都围绕着OR进行开发的，包括文件拉取、存储、分发。我们利用OR开发了鼓捣了很多特性，如大文件分片，域名动态加载，keyless等等，很有搞头，哈哈。 CDN目前还在起步阶段，仅公司内部的服务接入，后续会在全国布点，对外服务。
美团：当时项目启动时并不知道有KONG，而且还这么强大，就自己造轮子了。而且我们的业务场不太一样。现在来看，建议用KONG，可扩展可定制性太强大了。

#Kong





#补充知识
Twemproxy 也叫 nutcraker。是 Twtter 开源的一个 Redis 和 Memcache 代理服务器，主要用于管理 Redis 和 Memcached 集群，减少与Cache 服务器直接连接的数量。

Twemproxy特性：

轻量级、快速
保持长连接
减少了直接与缓存服务器连接的连接数量
使用 pipelining 处理请求和响应
支持代理到多台服务器上
同时支持多个服务器池
自动分片数据到多个服务器上
实现完整的 memcached 的 ASCII 和再分配协议
通过 yaml 文件配置服务器池
支持多个哈希模式，包括一致性哈希和分布
能够配置删除故障节点
可以通过端口监控状态
支持 linux, *bsd,os x 和 solaris


[Twemproxy安装配置参考官网](https://github.com/twitter/twemproxy)



**参考资料**
[OpenResty 使用介绍](http://www.runoob.com/w3cnote/openresty-intro.html)
[Lua教程](http://www.runoob.com/lua/lua-tutorial.html)
[wireshark百度百科](https://baike.baidu.com/item/Wireshark/10876564?fr=aladdin)
[现在有哪些公司在用Openresty?](https://www.zhihu.com/question/40091533/answer/149646492)




