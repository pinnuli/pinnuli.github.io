---
title: Bean装配之Autowired注解
date: 2018-07-28 22:36:17
tags:
---

#### Autowired注解说明
- 可以将@Autowired注解为setter方法
```java
@Autowired
public void setInjectionDAO(InjectionDAO injectionDAO) {
    this.injectionDAO = injectionDAO;
}
```
- 可以用于构造器或成员变量
> 默认情况下，如果因找不到合适的bean将会导致autowiring失败抛出异常，可以通过将其required设置为false表示并非必须
```java
@Autowired(required=false)
public void setInjectionDAO(InjectionDAO injectionDAO) {
    this.injectionDAO = injectionDAO;
}
```

> 每个类只能有一个构造器标记为required=true,也就是只能有一个构造器为必须，这种情况下建议使用@Required注解
```java
@Required
public void setInjectionDAO(InjectionDAO injectionDAO) {
    this.injectionDAO = injectionDAO;
}
```
 > @Required表示标记的bean属性在bean装配时必须被填充，通过在bean定义或者自动装配一个明确的属性值

- 用于注解众所周知的解析依赖性借口，如BeanFactory，ApplicationCon，Environment，ResourceLoader，ApplicationEventPublisher，MessageSource 等
```java
public class BeanSutowired{
    @Autowired
    private ApplicationContext context;
}
```

- 可通过添加注解给需要该类型的数组的字段或方法，提供ApplicationContext中的所有特定类型的bean
> 比如下面的示例中，list添加了@Autowired注解，那么所有的实现BeanInterface接口的bean，假如有BeanimplOne和BeanimplTwo都实现了BeanInterface接口，那么这时list中将包含有BeanimplOne和BeanimplTwo。

```java
@Component
public class BeanInvoker implements BeanInterface{

    @Autowired
    private List<BeanInterface> list;

    public void say(){
        System.out.println("list...");
        for (BeanInterface bean : list) {
            System.out.println(bean.getClass().getName());
        }
}
```
> 如果希望数组有序，可以使用@Order注解或者实现org.springframework.core.Ordered接口
```java
@Order(1)
@Component
public class BeanimplTwo implements BeanInterface{
}
```
- 用于装配key为String的Map
> 比如下面的示例中，map添加了@Autowired注解，那么所有的实现BeanInterface接口的bean，加入有BeanimplOne和BeanimplTwo都实现了BeanInterface接口，那么这时map中将包含有键为bean名称和值为bean的两个元素。
```java
@Component
public class BeanInvoker implements BeanInterface{

    @Autowired
    private Map<String, BeanInterface> map;

    public void say(){
        System.out.println("map...");
        for (Map.Entry<String, BeanInterface> entry : map.entrySet()) {
            System.out.println(entry.getKey() + "      " + entry.getValue().getClass().getName());
        }
}
```