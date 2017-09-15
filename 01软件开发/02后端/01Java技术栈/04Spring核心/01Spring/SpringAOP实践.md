Springboot中使用AOP特性非常简单，使用`@AspectJ`注解，然后再配置中开启`AspectJ`即可。在日常的应用，有时可以将日志记录和异常处理在一个拦截器中统一处理，但有时在项目中无法通过一个拦截器解决所有切面问题，此时，就需要将日志、异常处理等拦截器功能拆分开来，但有一点是相同的，就是在需要时增加一个抽象层次用于拦截。

[spring aop之对象内部方法间的嵌套失效]
http://blog.csdn.net/doctor_who2004/article/details/51814476

[Spring 嵌套AOP 的顺序问题 指定Aspect优先级]
http://www.cnblogs.com/daxin/archive/2013/06/01/3112049.html


负载均衡
[{"routeId":"default-route-rule","serverGroups":[{"groupId":"default-group-key","weight":5,"servers":[{"serverId":"platform.lipin.lipinpayws.v1.lipinpayws","metadata":{"url":"http://ws.account.giftcard.uat.qa.nt.ctripcorp.com/GCPayWS","isManualInstance":"true","healthCheckUrl":"http://ws.account.giftcard.uat.qa.nt.ctripcorp.com/GCPayWS/checkhealth.json"},"alive":true}],"metadata":{}}]}]


[cat-client]: Router service response: {"kvs":{"enableCatFilter":"true","startTransactionTypes":"Cache.;Redis;Memcached","bizLogEnable":"true","logServers":"10.2.25.122:2280;10.2.25.138:2280;","globalClockConfig":"10.2.25.138:8080;true;false;2;true;false;2","enable":"true","matchTransactionTypes":"SQL;Message.Produce.Tried;Message.Consume.Poll.Tried","catLogEnable":"true","routers":"10.2.25.122:2280;10.2.25.123:2280;","logSample":"1.0","sample":"1.0"}}

[SpringAOP嵌套调用的解决办法]
http://fyting.iteye.com/blog/109236


# 基础知识 #
这部分的细节主要是注解的使用，可以参看之后示例。
参考资料
http://www.cnblogs.com/best/p/5736422.html

# 实践 #
AOP配置


	@Configuration
	@EnableAspectJAutoProxy
	public class AOPConfig {
	}



			ReflectionUtils.makeAccessible(method);
			return method.invoke(target, args);


Log（AOP）实例


	@Aspect
	@Component
	public final class LogInterceptor {
		private final static int DEFAULT_MAX_LOG_LENGTH = 8192;
	
		@Pointcut("execution(*  com.bjork.ws.XXXWSImplForSpring.*(..))")
		public void serviceMethodPointcut() {
		}
	
		@Pointcut("execution( * com.bjork.ws.agent..*.*(..)) && @annotation(com.bjork.ws.core.AgentOriginalMethod)")
		public void agentOriginalMethodPointcut() {
		}
	
		@Around("serviceMethodPointcut() || agentOriginalMethodPointcut()")
		// @Around("agentOriginalMethodPointcut()")
		public Object Interceptor(ProceedingJoinPoint pjp) throws Throwable {
			// 获取aop相关信息
			Signature signature = pjp.getSignature();
			MethodSignature methodSignature = (MethodSignature) signature;
			Method targetMethod = methodSignature.getMethod();
			Class<?> returnType = targetMethod.getReturnType();
			Object result = null;
			Logger logger = LoggerFactory.getLogger(pjp.getTarget().getClass());
	
			try {
				// 设置默认返回值
				result = returnType.newInstance();
	
				logger.info(String.format("%s方法调用开始！", targetMethod.getName()));
				// 约定只有一个参数
				Object uniqueParameter = pjp.getArgs()[0];
				logRequest(JsonHelper.serialize(uniqueParameter), logger);
				result = pjp.proceed();
				logResponse(JsonHelper.serialize(result), logger);
			} finally {
				logger.info(String.format("%s方法调用结束！", targetMethod.getName()));
			}
			return result;
		}
	
		protected void logRequest(String requestString, Logger logger) {
			if (requestString.length() <= DEFAULT_MAX_LOG_LENGTH)
				logger.info(String.format("请求体为：%s", requestString));
		}
	
		protected void logResponse(String responseString, Logger logger) {
			if (responseString.length() <= DEFAULT_MAX_LOG_LENGTH)
				logger.info(String.format("响应体为：%s", responseString));
		}
	}



Exception （AOP）实例


	@Aspect
	@Component
	public final class ServiceInterceptor {
		@Pointcut("execution(* com.bjork.ws.service..*(..)) "
			+ "&& @annotation(com.bjork.ws.core.ServiceOpenMethod)")
		public void serviceMethodPointcut() {
		}
	
		@Around("serviceMethodPointcut()")
		public GenericResult Interceptor(ProceedingJoinPoint pjp) {
			// 获取aop相关信息
			Signature signature = pjp.getSignature();
			MethodSignature methodSignature = (MethodSignature) signature;
			Method targetMethod = methodSignature.getMethod();
			// Class<?> returnType = targetMethod.getReturnType();
			GenericResult result = new GenericResult();
			Logger logger = LoggerFactory.getLogger(pjp.getTarget().getClass());
	
			try {
				// 设置默认返回值
				// result = returnType.newInstance();
	
				// 约定只有一个参数
				Object uniqueParameter = pjp.getArgs()[0];
				result = (GenericResult) pjp.proceed();
			} catch (ValidException vex) {
				result.getResultInfo().setIsSuccessful(false);
				result.getResultInfo().setCode(vex.getErrorCode());
				result.getResultInfo().setMessage(vex.getMessage());
				logger.info(vex.getMessage());
			} catch (BizException bex) {
				result.getResultInfo().setIsSuccessful(false);
				result.getResultInfo().setCode(bex.getErrorCode());
				result.getResultInfo().setMessage(bex.getMessage());
				logger.warn(bex.getMessage());
			} catch (ExternalCallException ecex) {
				result.getResultInfo().setIsSuccessful(false);
				result.getResultInfo().setCode(ecex.getErrorCode());
				result.getResultInfo().setMessage(ecex.getMessage());
				logger.warn(ecex.getMessage());
			} catch (Throwable ex) {
				result.getResultInfo().setIsSuccessful(false);
				result.getResultInfo().setCode(ExceptionInfo.SYSTEM_EXCEPTION_CODE);
				result.getResultInfo().setMessage(ExceptionInfo.SYSTEM_EXCEPTION_MESSAGE);
				logger.error(ex.getMessage(), ex);
			} finally {
			}
			return result;
		}
	}

AutoConfiguration配置


	@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })
	//@SpringBootApplication
	@ComponentScan("com.xxx.ws")
	public class ServiceInitializer extends SpringBootServletInitializer {
	
		@Override
		protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
			return application.sources(ServiceInitializer.class);
		}
	
		@Bean
		public LogAspect logAspect() {
			return new LogAspect();
		}
	
		@Bean
		public ServiceExceptionAspect serviceExceptionAspect() {
			return new ServiceExceptionAspect();
		}
	}