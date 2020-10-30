# Spring中循环依赖时如何解决的



背景 只针对setter和field注入这两种方式,

```

@Autowired
private int SimpleBean simpleBean;

```



案例 两个简单bean

```java
@Component
public class CycleReferenceDemo {
    @Component
    static class A {
        @Autowired
        private B b;
    }

    @Component
    static class B {
        @Autowired
        private A a;
    }

}
```
背景知识

bean的三个关键流程  实例化(构造)   注入属性 初始化(init())
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
2. 从singletonObjects和singletonsCurrentlyInCreation都没发现A,说明A还不存在
3. 标记A开始创建,记录到singletonsCurrentlyInCreation
4. 实例化A.
5. singletonFactories添加A
6. 开始注入属性,发现需要B,请求获取B
7. 从singletonObjects和singletonsCurrentlyInCreation都没发现B,说明B还不存在,
8. 标记B开始创建,记录到singletonsCurrentlyInCreation
9. 实例化B
10. singletonFactories添加B
11. 开始注入属性,发现需要A,请求获取A
12. 从singletonObjects没找到A但是singletonsCurrentlyInCreation中有A,就从singletonFactories找成功找到
13. 调用A的ObjectFactory获取到A的引用,生成bean, synchronize[移除singletonFactories,添加earlySingletonObjects],返回A给步骤10
14. B注入属性完成,初始化完成.移除singletonsCurrentlyInCreation
15. synchronize[添加B到singletonObjects,从earlySingletonObjects移除B]
16. 给步骤6 **返回B**
17. A注入属性完成,初始化完成.移除singletonsCurrentlyInCreation
18. synchronize[添加A到singletonObjects,从earlySingletonObjects移除B]
19. **返回A**

虽然B中的A是从earlySingletonObjects中获取的,但是他和最终存储在singletonObjects中的A是相同的引用.因此是相同的.





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




重点 getbean的递归





[高频面试题：Spring 如何解决循环依赖？](https://zhuanlan.zhihu.com/p/84267654)
[【Spring】浅谈spring为什么推荐使用构造器注入](https://www.cnblogs.com/joemsu/p/7688307.html)