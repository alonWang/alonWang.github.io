# Spring是如何支持setter注入方式的循环依赖

## 前言

循环依赖是指两个bean相互依赖,如下面的A和B: A依赖于B,B又依赖于A.如果未加处理这会导致无限递归程序崩溃,然而在实例项目中这种情况循环依赖的情况并不少见.为此Spring做了一些努力,**解决了setter注入方式的循环依赖,对于构造器注入方式的循环只能检测并提前崩溃**.

```java
//setter注入方式的循环依赖
@Component
public class A {
    private B b;
  
    @Autowired
  	public void setB(B b){
      this.b=b;
    }
}

@Component
public class B {
    private A a;
  
  	@Autowired
  	public void setA(A a){
      this.a=a;
    }
}

//构造器注入方式的循环依赖
@Component
public class A {
    private B b;
  
   	public A(B b){
      this.b=b;
    }
}

@Component
public class B {
    private A a;
  
  	public B(B b){
      this.b=b;
    }
}
```
本文将简述Spring是如何支持setter注入方式的循环依赖的,并解释为何对构造器注入方式的循环依赖无能为力

## 正文

### 两种方式下bean的创建流程

![](https://raw.githubusercontent.com/alonWang/alonwang.github.io/master/article/Spring/img/spring-bean-create.svg)

在setter注入方式下的详细流程为

1. 实例化. 调用bean的无参构建函数生成实例.
2. 注入依赖属性. 从容器中获取该bean依赖属性的实例(如果没有,进入依赖属性对应bean的创建流程),进行注入.
3. 初始化. 如果bean实现了InitializingBean或@PostConstruct形式的初始化方法,进行调用

在构造器注入方式下的详细流程为

1. 获取依赖属性&实例化. 在容器中找到有参构造器中声明的参数的实例((如果没有,进入依赖属性对应bean的创建流程)),并用这些这些参数调用这个构造函数生成实例

2. 初始化. 同setter注入

### 构造器注入无法解决循环依赖的原因

**构造器注入必须先获取依赖属性才能完成实例化**,这是其无法解决循环依赖的根本原因.用上面的例子说明

1. 开始创建A
2. 获取A的依赖属性b对应的实例B,发现还没有,开始创建B
3. 获取B的依赖属性a对应的实例A,发现它正在创建中,Spring检测到这一点立刻报错,提示发生无法解决的循环依赖.



### setter注入解决循环依赖的方式

setter注入下实例化和依赖属性注入是分开的,这是其可以解决循环依赖的根本原因,还用上面的例子说明

1. 开始创建A
2. 调用A的无参构造函数实例化A,**把A存放在某个地方X,标识它是一个尚不完备但是可获取的bean**
3. 开始注入A的属性,获取A的依赖属性发现b对应的实例B还没创建,开始创建B
4. 与2类似,调用B的无参构造函数实例化B,把B存放在某个地方X,标识它是一个尚不完备但是可获取的bean
5. 开始注入B的属性,获取B的依赖属性**发现b对应的实例B还没在容器但是在X已经有了,就从X中获取到a**,B的注入完成
6. 完成B的初始化,放到容器中.
7. 返回B,给到步骤3,A的注入完成
8. 完成A的初始化,放到容器中.

流程如下图

![](https://raw.githubusercontent.com/alonWang/alonwang.github.io/master/article/Spring/img/Spring-cycle-reference.svg)



setter方式解决循环依赖的核心就是**提前将仅完成实例化的bean暴露出来,提供给其他bean**,这个暴露的地方就是图中的**地方X**,

这个地方X,在Spring代码中,对应的是`DefaultSingletonBeanRegistry`的两个属性

```java
/** Cache of singleton factories: bean name to ObjectFactory. */
private final Map<String, ObjectFactory<?>> singletonFactories 
/** Cache of early singleton objects: bean name to bean instance. */
private final Map<String, Object> earlySingletonObjects 
```
singletonFactories存储的是生成bean的工厂,工厂签名如下及添加逻辑如下

```java
public interface ObjectFactory<T> {
	T getObject() throws BeansException;
}
//实例化之后添加到singletonFactory
//getEarlyBeanReference会对bean做修改,例如代理或mock,因此返回的对象和传入的bean可能是不同的
addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
```

earlySingletonObjects存储的则是从这个工厂生成的bean.相关逻辑如下

```java
//DefaultSingletonBeanRegistry	
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			synchronized (this.singletonObjects) {
				singletonObject = this.earlySingletonObjects.get(beanName);
				if (singletonObject == null && allowEarlyReference) {
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
					if (singletonFactory != null) {
            //**对象转移**只在循环依赖时才会发生
						singletonObject = singletonFactory.getObject();
						this.earlySingletonObjects.put(beanName, singletonObject);
						this.singletonFactories.remove(beanName);
					}
				}
			}
      return singletonObject;
	}
```



为什么要在两个对象,而非直接用earlySingletonObjects存储getEarlyBeanReference生成的对象呢?

**节省资源**,getEarlyBeanReference是一个相对耗时的操作(生成代理,mock都不是简单操作),而大部分bean不会有循环依赖存在,也就不会发生对象转移. 不会调用到getEarlyBeanReference,进而节省资源.



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
![图片](https://github.com/alonWang/alonwang.github.io/blob/master/article/Spring/img/Spring-cycle-reference.svg)

[高频面试题：Spring 如何解决循环依赖？](https://zhuanlan.zhihu.com/p/84267654)
[【Spring】浅谈spring为什么推荐使用构造器注入](https://www.cnblogs.com/joemsu/p/7688307.html)