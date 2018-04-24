https://www.imooc.com/article/13553


现代web开发框架如vue.js、react.js等，在设计的时候就考虑了XSS攻击对html插值进行了更进一步的抽象、过滤和转义，我们只要熟练正确地使用他们，就可以在大部分情况下避免XSS攻击


反射型
存储型
DOM型


1. Reflected XSS（基于反射的XSS攻击）
2. Stored XSS（基于存储的XSS攻击）
3. DOM-based or local XSS（基于DOM或本地的XSS攻击）

前端负责用户体验及xss，后台负责安全

1、如果万不得已，不要开放跨域请求。当然有些场景因为是访问移动端的离线h5文件，似乎不得不进行跨域请求。有条件的话，就在app客户端层面做请求拦截，对数据请求优化也会有所帮助。








