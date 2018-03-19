Rational AppScan

《白帽子讲web安全》



https://www.ibm.com/developerworks/downloads/r/appscan/


**CSRF**

Spring security 默认的org.springframework.security.web.csrf.CsrfFilter 对于HTTP请求有如下的判断：
private Pattern allowedMethods = Pattern.compile("^(GET|HEAD|TRACE|OPTIONS)$");


http://blog.csdn.net/behurry/article/details/38169279

1.什么是CSRF

CSRF 攻击简单来说，是多Tab页面浏览器的一个安全漏洞，比如你正在访问A网站，此时如果浏览器有你的cookie，并且session没有过期，此时你去访问B网站，那么B网站可以直接调用A网站的接口，而A网站则认为是你本人进行的操作。以下是图示：

2.如何进行防御

对CSRF进行防御，可以通过加Token.也就是当你访问A网站的时候，A会给你一个token，然后，接下去的post请求，你需要把token带上，不然服务器则拒绝接收这个请求。 
- 1. token的产生：spring-security 4.0之后默认开启csrf，可以直接产生csrf token。 
- 2. token的存储：这里存储是指服务端的存储，token是存储在session中。 
- 3. token的传送：token可以通过cookie，也可以放在header中自定义的属性中。 
- 4. token的接收和返回：前段收到http respon 之后，需要把相应的token返回回来。 
- 5. token校验：服务器端对自己持有的token和客户端反馈回来的token进行校验，决定是否拒绝服务（拒绝服务可以自定义）。







