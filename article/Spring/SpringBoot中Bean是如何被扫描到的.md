# SpringBoot中Bean是如何被扫描到的?

## 前言

在SpringBoot中,只需要一个简单的启动类,就能自动完成很多复杂的工作, 包括bean的扫描.本文将探究SpringBoot如何扫描添加Bean.

```java
@SpringBootApplication
public class SpringLifecycleApplication {

    public static void main(String[] args) {
        ApplicationContext applicationContext = SpringApplication.run(SpringLifecycleApplication.class, args);
    }

}
```

## 正文

![](img/SpringBoot-bean.png)

### 1. createApplicationContext阶段: 注册ConfigurationClassPostProcessor
SpringBoot默认创建`AnnotationConfigApplicationContext`,它在构造时会创建`AnnotatedBeanDefinitionReader`,后者构造时会调用`AnnotationConfigUtils.registerAnnotationConfigProcessors()`注册`ConfigurationClassPostProcessor`的BeanDefinition.
![image-20201003221748289](img/SpringBoot-beanDefinition.png)
### 2. prepareContext阶段: 注册启动类SpringLifeCycleApplication的BeanDefinition

![image-20201003233304672](img/SpringBoot-bean-1.png)

### 3. refreshContext阶段的invokeBeanFactoryPostProcessor(): 调用ConfigurationClassPostProcessor.postProcessBeanDefinitionRegistry()

```java
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// ...
			try {
				// ...
				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);
				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);
				// ...
				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);
				// Last step: publish corresponding event.
				finishRefresh();
			}
			// ...
		}
	}
```

ConfigurationClassPostProcessor对应的逻辑是: 查找现有的BeanDefinition(包含SpringLifecycleApplication的BeanDefinition),对带有@Configuration注解的,使用`ConfigurationClassParser`进行解析处理

判断是否为满足@Configuration注解要求的逻辑如下

```java
	public static boolean isFullConfigurationCandidate(AnnotationMetadata metadata) {
		return metadata.isAnnotated(Configuration.class.getName());
	}
	//StandardAnnotationMetadata
	public boolean isAnnotated(String annotationName) {
		return (this.annotations.length > 0 &&
				AnnotatedElementUtils.isAnnotated(getIntrospectedClass(), annotationName));
	}
	//这个方法会递归寻找注解. 如SpringLifeCycleApplication只有注解@SpringBootApplication,没找到@Configuration
	//就会去寻找@SpringBootApplication这个注解包含的注解,按此规律递归寻找直至找到或者递归终止,对于@SpringBootApplication  
    //它拥有的注解@SpringBootConfiguration拥有@Configuration注解,因此最终是符合条件的
	public static boolean isAnnotated(AnnotatedElement element, String annotationName) {
		return Boolean.TRUE.equals(searchWithGetSemantics(element, null, annotationName, alwaysTrueAnnotationProcessor));
	}
```

### 4. ConfigurationClassParser

AnnotationApplicationContext,ConfigurationClassPostProcessor,ConfigurationClassParser

递归搜寻Annotation ComponentScanAnnotationParser ClassPathBeanDefinitionScanner 



BeanFactoryPostProcessor初始化时还没有 BeanpostProcessor,而@Autowired又是依赖AutowiredAnnotationBeanPostProcessor进行注入,这就导致BeanFactoryPostProcessor中的依赖不会被注入

对BeanPostProcessor,由于用户注册的BeanPostProcessor一定在AutowiredAnnotationBeanPostProcessor之后,依赖是可以注入的

