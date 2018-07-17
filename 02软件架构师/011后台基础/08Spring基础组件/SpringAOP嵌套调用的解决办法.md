
    //从beanFactory取得AOP代理后的对象  
    SomeService someServiceProxy = (SomeService)beanFactory.getBean("someService");   
    //把AOP代理后的对象设置进去  
    someServiceProxy.setSelf(someServiceProxy);   
    //在someMethod里面调用self的someInnerMethod，这样就正确了  
    someServiceProxy.someMethod();  

    


参考资料
http://fyting.iteye.com/blog/109236