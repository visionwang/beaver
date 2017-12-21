

https://www.cnblogs.com/cswuyg/p/3653263.html

HTTP的长连接和短连接

    本文总结&分享网络编程中涉及的长连接、短连接概念。
    关键字：Keep-Alive，并发连接数限制，TCP，HTTP
一、什么是长连接
     HTTP1.1规定了默认保持长连接（HTTP persistent connection ，也有翻译为持久连接），数据传输完成了保持TCP连接不断开（不发RST包、不四次握手），等待在同域名下继续用这个通道传输数据；相反的就是短连接。


很多时候代理是我们自己控制的，如Nginx代理，代理服务器有长连接处理逻辑，服务端无需做patch处理，**常见的是客户端跟Nginx代理服务器使用HTTP1.1协议&长连接，而Nginx代理服务器跟后端服务器使用HTTP1.0协议&短连接。**









