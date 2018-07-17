
#1.实现ApplicationContextAware，并注册Bean
public class SpringContextUtil implements ApplicationContextAware {

    private static ApplicationContext applicationContext; // Spring应用上下文环境

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        SpringContextUtil.applicationContext = applicationContext;
    }

    /**
     * 获取对象
     * 
     * @param name
     * @return Object 一个以所给名字注册的bean的实例
     * @throws BeansException
     */
    public static Object getBean(String name) throws BeansException {
        return applicationContext.getBean(name);
    }

}

  <bean id="springContextUtil" class="com.mycompany.yuanmeng.springdemo.aop.SpringContextUtil" />

public static void test() {
        
        new ClassPathXmlApplicationContext("spring.xml"); // 加载ApplicationContext（模拟启动web服务）, 当然如果是用web工程的话,可以直接在web.xml配置，容器会去加载此文件。
        
        UserReadService userReadService = (UserReadService) SpringContextUtil.getBean("userReadService");
        
        userReadService.getUserInfoById(null);
 }


#2.继承ApplicationObjectSupport




