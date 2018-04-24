

请求类型

Content-Type:application/x-www-form-urlencoded

Content-Type:application/json


      首先，@RequestBody需要接的参数是一个String化的json，这里直接使用JSON.stringify(json)这个方法来转化；其次，@RequestBody从名称上来看也就是说要读取的数据在请求体里，所以要发post请求；第三，要设置请求的内容类型contentType为 contentType:"application/json，明确的告诉服务器发送的内容是json数据，而默认的contentType是application/x-www-form-urlencoded; charset=UTF-8那么可以开始写出代码了。


var ds  = {


}
Springboot支持Rest风格给编码带来了很好的便捷性，@RequestBody让我们可以直接以application/json请求并在到达controller层获得已反序列化的对象，当有拦截的需求，这种方式却不再奏效。

application/x-www-form-urlencoded通过表单提交，在sevlet实现中，mutipart/form-data和application/x-www-form-urlencoded会被特殊处理，请求参数将被放置于request.paramter，这是一个map。我们可以从map中获取参数进行验证，或者其他拦截需求，map的获取类似hibernate的延迟加载，当调用request.getparamter(）方法，servlet才会从请求流中读取请求参数加载入map。InputStream也会存有这份数据，但如果这份数据被读取，那么到了controller层将无法读出数据，同样，拦截之后到达controller层时请求数据已经被加载入了controller层方法实参，实参对象需要有set方法，框架会以反射的方式调用属性的set方法注入数据，数据只会被注入到已有的属性。

当以application/json的content-type传送数据，被传送的对象只需被json序列化。当以application/x-www-form-urlencoded的方式传送数据。请求的内容需要以..=..&..=..的格式提交，在请求体内内容将会以”&”和“ = ”进行拆分。

当以application/json的content-type传送数据，被传送的对象只需被json序列化。当以application/x-www-form-urlencoded的方式传送数据。请求的内容需要以..=..&..=..的格式提交，在请求体内内容将会以”&”和“ = ”进行拆分。

但你需要使用content-type=application/json且后台使用@RequestBody，你无法再从paramter中获取请求数据。
选择application/x-www-form-urlencoded还是application/json，得看你是否有从request.paramter获取请求数据的需求。

大体明白了

      <!-- okhttp -->
            <dependency>
                <groupId>com.squareup.okhttp3</groupId>
                <artifactId>okhttp</artifactId>
                <version>${okhttp.version}</version>
            </dependency>


上车，下车

网络效应，如果某件事物具有网络效应，你就别下车

