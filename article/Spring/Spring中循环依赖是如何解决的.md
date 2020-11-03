# Spring中循环依赖时如何解决的



背景 只针对setter注入和field注入这两种方式,

field注入实质是通过Field反射注入,下面统称setter注入

案例 两个简单bean

```java
@Component
public class A {
    @Autowired
    private B b;
}

    @Component
public class B {
    @Autowired
    private A a;
}
}
```
背景知识

setter注入时bean的创建流程:  

1. 实例化. 调用bean的无参构建函数生成一个的实例.
2. 注入属性. 从容器中获取该bean待注入字段的实例(如果没有,进入待注入字段对应bean的创建流程),进行注入.
3.  初始化. 如果bean实现了InitializingBean或@PostConstruct形式的初始化方法,进行调用



DefaultSingletonBeanRegistry的四个属性

```java
/** Names of beans that are currently in creation. */
private final Set<String> singletonsCurrentlyInCreation 
/** Cache of singleton factories: bean name to ObjectFactory. */
private final Map<String, ObjectFactory<?>> singletonFactories 
/** Cache of early singleton objects: bean name to bean instance. */
private final Map<String, Object> earlySingletonObjects 
/** Cache of singleton objects: bean name to bean instance. */
private final Map<String, Object> singletonObjects 
```
主要方法
```java
getSingleton(String beanName, boolean allowEarlyReference)
getSingleton(String beanName, ObjectFactory<?> objectFactory)    
```
流程 




1. **请求获取A**
2. 从`singletonObjects`和`singletonsCurrentlyInCreation`都没发现A,说明A还不存在
3. 标记A开始创建,记录到`singletonsCurrentlyInCreation`
4. 实例化A.`singletonFactories`添加A
6. 开始注入属性,发现需要B,请求获取B
7. 从`singletonObjects`和`singletonsCurrentlyInCreation`都没发现B,说明B还不存在,
8. 标记B开始创建,记录到`singletonsCurrentlyInCreation`
9. 实例化B,`singletonFactories`添加B
11. B开始注入属性,发现需要A,请求获取A
12. 从`singletonObjects`没找到A但是`singletonsCurrentlyInCreation`中有A,`earlySingletonObjects`也没有A,最后从`singletonFactories`成功找到
13. 调用A的ObjectFactory.getObject()获取到A的引用, synchronize[移除A的`singletonFactories`,添加A的`earlySingletonObjects`],返回A的引用给步骤9
14. B注入属性完成,初始化完成.移除`singletonsCurrentlyInCreation`
15. synchronize[添加B到`singletonObjects`,从`earlySingletonObjects`移除B]
16. 给步骤5 **返回B**
17. A注入属性完成,初始化完成.移除`singletonsCurrentlyInCreation`
18. synchronize[添加A到`singletonObjects`,从`earlySingletonObjects`移除A,从`singletonFactory`移除A]
19. **返回A**

虽然B中的A是从earlySingletonObjects中获取的,但是他和最终存储在singletonObjects中的A是相同的引用.因此是相同的.





* singletonsCurrentlyInCreation 正在创建过程中的单例bean
* singletonObjects  完备的单例bean
* earlySingletonObjects 不完备单例bean
* singletonFactories 单例bean的工厂

四个阶段

* 标记开始创建
* 记录已经实例化
* 标记创建但不完备
* 标记创建完成

singletonObjcts和earlySingletonObjects记录的都是引用,




重点 getbean的递归





[高频面试题：Spring 如何解决循环依赖？](https://zhuanlan.zhihu.com/p/84267654)
[【Spring】浅谈spring为什么推荐使用构造器注入](https://www.cnblogs.com/joemsu/p/7688307.html)