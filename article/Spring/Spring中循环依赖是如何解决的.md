# Spring中循环依赖时如何解决的



前提 只针对setter方式

案例 两个简单bean

流程 

实例化   注入属性 init

注入属性前 暴露到singletonFactorys

入口getSingleton(String beanName, boolean allowEarlyReference)

1. 



```java
getSingleton(String beanName, boolean allowEarlyReference)
getSingleton(String beanName, ObjectFactory<?> objectFactory)    
```

DefaultSingletonBeanRegistry的四个属性

* singletonsCurrentlyInCreation 正在创建过程中的单例bean
* singletonObjects  完备的单例bean
* earlySingletonObjects 不完备单例bean
* singletonFactories 单例bean的工厂

四个阶段

* 标记开始创建
* 标记可以创建
* 标记创建但不完备
* 标记创建完成

singletonObjcts和earlySingletonObjects记录的都是引用,

```java
	
	/** Names of beans that are currently in creation. */
	private final Set<String> singletonsCurrentlyInCreation =
			Collections.newSetFromMap(new ConcurrentHashMap<>(16));
	/** Cache of singleton factories: bean name to ObjectFactory. */
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
	/** Cache of early singleton objects: bean name to bean instance. */
	private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
	/** Cache of singleton objects: bean name to bean instance. */
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
```



重点 getbean的递归





[高频面试题：Spring 如何解决循环依赖？](https://zhuanlan.zhihu.com/p/84267654)