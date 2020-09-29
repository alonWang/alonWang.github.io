# ApplicationContext是如何被注入的

## 前言
ApplicationContext是Spring中的重要组件,它不是bean,因此无法通过getBean获取它,但是可以通过Autowired注入,也可以通过ApplicationContextAware接口注入，这是如何实现的?
```java
//ERROR No qualifying bean of type 'org.springframework.context.ApplicationContext' available
applicationContext.getBean(ApplicationContext.class);

//SUCCESS
@Component
public class SimpleBean3 implements ApplicationContextAware{
    @Autowired
    private ApplicationContext applicationContext;
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
     //...
    }    
}
```

本文将简述Application及类似的BeanFactory在未被容器管理的情况下是如何实现注入的逻辑的.



## 正文

### Aware接口

```java
public interface Aware {

}
public interface ApplicationContextAware extends Aware {
    void setApplicationContext(ApplicationContext var1) throws BeansException;
}
```

> A marker superinterface indicating that a bean is eligible to be notified by the Spring container of a particular framework object through a callback-style method. 

Aware接口给用户一个获取ApplicationContext，BeanFactory等的方式。



