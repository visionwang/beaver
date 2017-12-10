


[ Java中httpClient中的三种超时设置小结（转）](http://blog.csdn.net/qq32933432/article/details/56277291)

[Apache HttpClient](http://hc.apache.org/httpclient-3.x/)


上传文件
FileEntity

StringEntity


 Gson gson = new GsonBuilder().create();
        ContentType contentType = ContentType.getOrDefault(entity);
        Charset charset = contentType.getCharset();
        Reader reader = new InputStreamReader(entity.getContent(), charset);
        return gson.fromJson(reader, MyJsonObject.class);


居然可以加拦截器

CloseableHttpClient httpclient = HttpClients.custom()
        .addInterceptorLast(new HttpRequestInterceptor() {

            public void process(
                    final HttpRequest request,
                    final HttpContext context) throws HttpException, IOException {
                AtomicInteger count = (AtomicInteger) context.getAttribute("count");
                request.addHeader("Count", Integer.toString(count.getAndIncrement()));
            }

        })
        .build();

AtomicInteger count = new AtomicInteger(1);
HttpClientContext localContext = HttpClientContext.create();
localContext.setAttribute("count", count);

HttpGet httpget = new HttpGet("http://localhost/");
for (int i = 0; i < 10; i++) {
    CloseableHttpResponse response = httpclient.execute(httpget, localContext);
    try {
        HttpEntity entity = response.getEntity();
    } finally {
        response.close();
    }
}

好帅哟


		HttpHost proxy = new HttpHost(QConfigHelper.getValue("qunar_proxy_url"),
				Integer.valueOf(QConfigHelper.getValue("qunar_proxy_port")));
		RequestConfig config = RequestConfig.custom().setProxy(proxy).setConnectTimeout(1000 * 6).build();
		logger.info("qunar_proxy_url: " + QConfigHelper.getValue("qunar_proxy_url"));
		logger.info("qunar_proxy_port: " + QConfigHelper.getValue("qunar_proxy_port"));
		request.setConfig(config);


代理
    HttpHost proxy = new HttpHost(proxy.ip, proxy.port); 


		SSLContext sslContext = SSLContexts.createSystemDefault();
		SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslContext, NoopHostnameVerifier.INSTANCE);
		// 设置SSL
		// 设置Proxy
		/*		HttpHost proxy = new HttpHost(QConfigHelper.getValue("qunar_proxy_url"),
						Integer.valueOf(QConfigHelper.getValue("qunar_proxy_port")));
				DefaultProxyRoutePlanner routePlanner = new DefaultProxyRoutePlanner(proxy);*/
		// return
		// HttpClients.custom().setSSLSocketFactory(sslsf).setRoutePlanner(routePlanner).build();
		return HttpClients.custom().setSSLSocketFactory(sslsf).build();


http://llying.iteye.com/blog/333644
httpClient通过代理（Http Proxy）进行请求 
在浏览一些网站的时候由于各种原因，无法进行访问。 
这时我们需要通过IE，FireFox进行Http的代理设置， 
当然httpClient也为我们提供这样的设置 
使用匿名代理 
Java代码  收藏代码
HttpClient httpClient = new HttpClient();  
//设置代理服务器的ip地址和端口  
httpClient.getHostConfiguration().setProxy("192.168.101.1", 5608);  
//使用抢先认证  
httpClient.getParams().setAuthenticationPreemptive(true);  

如果代理需要用户，密码进行验证 
Java代码  收藏代码
HttpClient httpClient = new HttpClient();  
httpClient.getHostConfiguration().setProxy("192.168.101.1", 5608);  
httpClient.getParams().setAuthenticationPreemptive(true);  
//如果代理需要密码验证，这里设置用户名密码  
httpClient.getState().setProxyCredentials(AuthScope.ANY, new UsernamePasswordCredentials("llying.iteye.com","llying"));  
