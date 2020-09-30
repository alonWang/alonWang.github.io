# ApplicationContext是如何被注入的

## 前言
ApplicationContext是Spring中的重要组件,它不是bean,因此无法通过getBean获取它,但是它可以通过Autowired注入获得,这是如何实现的?其中必定有特殊的处理,本文将简述ApplicationContext及类似的BeanFactory在未被容器管理的情况下是如何实现注入逻辑的.
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

## 正文
```java
@Component
public class SimpleBean3 {
    @Autowired
    private ApplicationContext applicationContext;
    @Autowired
    private SimpleBean2 simpleBean2;
}
```
### 被容器管理的Bean是如何注入的

### ApplicationContext是如何注入的


