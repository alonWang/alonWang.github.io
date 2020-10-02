# ApplicationContext是如何被注入的

## 前言
ApplicationContext是Spring中的重要组件,它不是bean,因此无法通过getBean获取它,但是可以通过Autowired注入获得,这是如何实现的?其中必定有特殊的处理,本文将简述ApplicationContext及类似的BeanFactory是如何实现注入逻辑的.
```java
//ERROR No qualifying bean of type 'org.springframework.context.ApplicationContext' available
applicationContext.getBean(ApplicationContext.class);

//SUCCESS
@Component
public class SimpleBean3 {
    @Autowired
    private ApplicationContext applicationContext;  
    @Autowired
    private SimpleBean2 simpleBean2;
}
```

## 正文

### TLDR

* 普通Bean的元数据存放在DefaultListableBeanFactory的beanDefinitionNames和beanDefinitionMap,它们通过Spring提供的机制**自动注册添加**,是Spring提供的功能
```java
	/** List of bean definition names, in registration order. */
	private volatile List<String> beanDefinitionNames = new ArrayList<>(256);
	/** Map of bean definition objects, keyed by bean name. */
	private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
```


* 而ApplicationContext和BeanFactory存储在DefaultListableBeanFactory的resolvableDependencies,它们需要**手动注册添加**,是Spring的框架内部逻辑
```java
	/** Map from dependency type to corresponding autowired value. */
	private final Map<Class<?>, Object> resolvableDependencies = new ConcurrentHashMap<>(16);
```

在**查找依赖时**,会同时搜寻这两个地方,因此ApplicationContext也能被查找到.

而**getBean时**只会查找上面的BeanDefinitionMap,因此找不到ApplicationContext.

### 注入流程

#### 手动注册ApplicationContext

在AbstractApplicationContext.prepareBeanFactory()中,ApplicationContext被注册到resolvableDependencies中

```java
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		//...忽略部分代码
    
		// BeanFactory interface not registered as resolvable type in a plain factory.
		// MessageSource registered (and found for autowiring) as a bean.
		beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
		beanFactory.registerResolvableDependency(ResourceLoader.class, this);
		beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
		beanFactory.registerResolvableDependency(ApplicationContext.class, this);
        //...忽略部分代码
	}
```

#### 生成Bean时查找依赖

在 [SpringBoot中@Autowired是如何生效的](https://alonwang.github.io/#/./article/Spring/SpringBoot中@Autowired是如何生效的?id=springboot中autowired是如何生效的)那篇文章中,讲到了带有@Autowired字段的在AutowiredAnnotationPostProcessor.postProcessProperties()中完成注入,查找依赖的入口就在`metadata.inject(bean, beanName, pvs)`

```java
	public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
		InjectionMetadata metadata = findAutowiringMetadata(beanName, bean.getClass(), pvs);
		try {
            //### 注入 ###
			metadata.inject(bean, beanName, pvs);
		}
		catch (BeanCreationException ex) {
			throw ex;
		}
		catch (Throwable ex) {
			throw new BeanCreationException(beanName, "Injection of autowired dependencies failed", ex);
		}
		return pvs;
	}
```

大致的流程如下

> AutowiredAnnotationBeanPostProcesspr.postProcessProperties()
>
> =>InjectionMetadata.inject()
>
> ==>AutowiredFieldElement.inject()
>
> ===>DefaultListableBeanFactory.resolveDependency()
>
> ====>DefaultListableBeanFactory.doResolveDependency()
>
> =====>DefaultListableBeanFactory.findAutowireCandidates()

![image-20201002121515238](C:\Users\wangw\IdeaProjects\alonwang.github.io\article\Spring\img\SpringBoot-inject-stack.png)

我们直接跳到DefaultListableBeanFactory.findAutowireCandidates(),可以看到是同时从BeanDefinitionNames和resolvableDependencies两个地方查找符合条件的依赖.

```java
	protected Map<String, Object> findAutowireCandidates(
			@Nullable String beanName, Class<?> requiredType, DependencyDescriptor descriptor) {
		String[] candidateNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
				this, requiredType, true, descriptor.isEager());
		Map<String, Object> result = new LinkedHashMap<>(candidateNames.length);
        //从resolvableDependencies中寻找
		for (Map.Entry<Class<?>, Object> classObjectEntry : this.resolvableDependencies.entrySet()) {
			Class<?> autowiringType = classObjectEntry.getKey();
			if (autowiringType.isAssignableFrom(requiredType)) {
				Object autowiringValue = classObjectEntry.getValue();
				autowiringValue = AutowireUtils.resolveAutowiringValue(autowiringValue, requiredType);
				if (requiredType.isInstance(autowiringValue)) {
					result.put(ObjectUtils.identityToString(autowiringValue), autowiringValue);
					break;
				}
			}
		}
        //从BeanDefinitionNames中寻找
		for (String candidate : candidateNames) {
			if (!isSelfReference(beanName, candidate) && isAutowireCandidate(candidate, descriptor)) {
				addCandidateEntry(result, candidate, descriptor, requiredType);
			}
		}
		//...忽略部分代码
		return result;
	}
```

至此 ApplicationContext的注入逻辑完毕.