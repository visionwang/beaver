在Java代码中可以使用 @Resource或者@Autowired注解方式来经行注入。虽然@Resource和@Autowired都可以来完成注入依赖，但它们之间是有区 别的。
 a。@Resource默认是按照名称来装配注入的，只有当找不到与名称匹配的bean才会按照类型来装配注入；
@Resource有两个属性是比较重要的，分是name和type，Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。所以如果使用name属性，则使用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。如果既不指定name也不指定type属性，这时将通过反射机制使用byName自动注入策略。
　　@Resource装配顺序
　　1. 如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常
　　2. 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常
　　3. 如果指定了type，则从上下文中找到类型匹配的唯一bean进行装配，找不到或者找到多个，都会抛出异常
　　4. 如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配； 
 b。@Autowired默认是按照类型装配注入的，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它required属性为false。如果我们想使用按名称装配，可以结合@Qualifier注解一起使用。如下： 
    @Autowired  @Qualifier("personDaoBean") 

    private PersonDao  personDao; 
 c。@Resource注解是又J2EE提供，而@Autowired是由Spring提供，故减少系统对spring的依赖建议使用   @Resource的方式；
d。 @Resource和@Autowired都可以书写标注在字段或者该字段的setter方法之上


左上角outlook界面 - 文件 - 选项 - 邮件 找到如图所示位置，点掉这个对勾就行了

 在中欧收获 lifelong friendship. 这些年长期漂泊在外，并且频繁变换工作地点，很难和人建立长期的联系。

 我也期望借着回归校园，转换生活状态。幸运的是，中欧通过卓有成效的选拔机制，已经帮我找到了志同道合者 —— 有相同价值观，却迥异又有趣的背景，并且都已是各自领域翘楚的中欧同学们。



 套用罗大佑的话描绘这幅场景再合适不过 —— 是这些年职场中不顾一切的倔强，柔软了我们今日融入中欧的模样。

 苏格拉底说：“未经审视的人生，是不值得过的。” 申请MBA的过程，正是一个审视过去，畅想未来的好机会。对于正在准备申请文书的同学，我的建议其实很鸡汤：正如德尔斐神谕，“认识你自己”。申请文书的准备其实就是在回答经典的哲学三问 ——“我是谁？我从哪里来？我要到哪里去？”理清三个答案之间的逻辑，找到其中的gap，并且合理地解释“中欧如何帮我填补这些gap”，则能写出一份满意的申请文书。
中欧的申请，具体看看，看来只能做做土炮了，哈哈。。。。


常用的设定方式有以下三种：
通过实现 InitializingBean/DisposableBean 接口来定制初始化之后/销毁之前的操作方法；
通过 <bean> 元素的 init-method/destroy-method属性指定初始化之后 /销毁之前调用的操作方法；
在指定方法上加上@PostConstruct 或@PreDestroy注解来制定该方法是在初始化之后还是销毁之前调用。 


https://www.cnblogs.com/fanguangdexiaoyuer/p/5944861.html



