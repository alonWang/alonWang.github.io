# SpringBoot启动流程

## 前言



SpringBoot的启动过程涵盖了Spring Framework的方方面面,对于想研究源码的新手,很容易陷入无尽的函数调用栈,

因此本文将通过下图梳理启动流程,力求读者对此有一个全面的认识.

![](img/SpringBoot-startup.png)

## 正文

以下将按照逻辑顺序对图中的流程分类归纳

### StopWatch

StopWatch记录了启动总时长和运行的任务信息,支持以用户友好的格式打印出来.StopWatch在启动时**最先启动**,在refreshContext后结束.

###SpringApplicationRunListeners

在到达启动的不同阶段时,SpringApplicationRunListeners会对所有的SpringApplicationRUnListener进行通知(请注意两者差了一个**s**),以SpringApplicationRunListeners.starting()为例

```java
	public void starting() {
		for (SpringApplicationRunListener listener : this.listeners) {
			listener.starting();
		}
	}
```

*listeners通过*Spring Factory Loader*模式加载并实例化*

###  prepareEnvironment()

Environment主要包含两个关键概念: **profiles和properties**,这里根据对Environment进行初始化和配置.

### prepareContext()

配置ApplicationContext,例如设置Environment,注册默认的BeanNameGenerator,注册springBootBanner,应用ApplicationContextInitializer等等

###  refreshContext()

refreshContext()最终调用到ApplicationContext.refresh(),即**ApplicationContext的启动流程**,这是整个启动过程中的**核心流程**,包含了**Beans,IOC,AOP**的实现

### callRunners()

调用执行所有注册的ApplicationRunner和CommandLineRunner.