# ApplicationContext是如何被注入的

## 前言
ApplicationContext是Spring中的重要组件,它不是bean,因此无法通过getBean获取它,但是可以注入,这是如何实现的?
```java
//ERROR No qualifying bean of type 'org.springframework.context.ApplicationContext' available
applicationContext.getBean(ApplicationContext.class);

//SUCCESS
@Component
public class SimpleBean3 {
    @Autowired
    private ApplicationContext applicationContext;
}
```

本文将简述Application及类似的BeanFactory在未被容器管理的情况下是如何实现注入的逻辑的.



## 正文

Application不是Bean但是能够注入,说明注入的逻辑中必定对其进行了特殊处理

