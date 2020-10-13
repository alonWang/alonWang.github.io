# Spring AOP解析


### AspectJAwareAdvisorAutoProxyCreator实现代理的BeanPostProcessor

#### postProcessAfterInitialization阶段实现代理
* earlyProxyReferences包含的bean不做代理
* targetSourcedBeans包含的bean不做代理

##### 首次调用缓存Advisor
`org.springframework.aop.aspectj.annotation.BeanFactoryAspectJAdvisorsBuilder#buildAspectJAdvisors` 
初次调用会缓存所有的advisor

获取所有bean,找到带有@Aspect注解的,查找所有没有带@PointCut注解的方法,
对这些方法,获取其AspectAnnotation(@PointCut,@Around,@Before,@After等等) 
*请注意,由于这里的过滤,在这里肯定不会有@PointCut的,原因未知*
再将annotation封装为AspectJExpressionPointcut.
再看下字段上有没有Advisor,有的话也添加

#### 查找适用于当前bean的Advisor

会在advisors首部添加一个 ExposeInvocationInterceptor.ADVISOR ,它承载了exposeProxy的功能

#### 创建代理
配置ProxyFactory,用到前面的advisors,还有一些基础配置例如 exposeProxy,proxyTargetClass,beanClass等等
再将创建工作委托给DefaultAopProxyFactory
根据是否,实现接口,强制使用cglib等决定使用哪种代理方式,5.x默认使用 ObjenesisCglibAopProxy  即不依赖构造方法的cglib

最后将这些数据配置好,创建一个cglib形式的代理


### 调用
对被代理方法的调用会跑到 org.springframework.aop.framework.CglibAopProxy.DynamicAdvisedInterceptor#intercept
将当前调用封装为 CglibMethodInvocation,它继承自 ReflectiveMethodInvocation
而ReflectiveMethodInvocation承载了链式调用的功能

这里会从头到尾调用advisor,ReflectiveMethodInvocation.invoke(ReflectiveMethodInvocation)
例如
```
	public Object invoke(MethodInvocation mi) throws Throwable {
		try {
			return mi.proceed();
		}
		finally {
			invokeAdviceMethod(getJoinPointMatch(), null, null);
		}
	}
```
会按照它的逻辑顺序执行.
必定会调用到ReflectiveMethodInvocation.process()




