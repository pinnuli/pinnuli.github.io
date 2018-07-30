---
title: Spring Bean装配之Autowired注解
date: 2018-07-28 22:36:17
categories: "JavaWeb"
tags:
    - Spring
    - JavaWeb
---

### 1.可以将@Autowired注解为setter方法
```java
@Autowired
public void setInjectionDAO(InjectionDAO injectionDAO) {
    this.injectionDAO = injectionDAO;
}
```

----
### 2.可以用于构造器或成员变量

```java
@Autowired(required=false)
public void setInjectionDAO(InjectionDAO injectionDAO) {
    this.injectionDAO = injectionDAO;
}
```
默认情况下，如果因找不到合适的`bean`将会导致`autowiring`失败抛出异常，可以通过将其`required`设置为false表示并非必须，每个类只能有一个构造器标记为`required=true`,也就是只能有一个构造器为必须，这种情况下建议使用`@Required`注解：

```java
@Required
public void setInjectionDAO(InjectionDAO injectionDAO) {
    this.injectionDAO = injectionDAO;
}
```
 > `@Required`表示标记的`bean`属性在`bean`装配时必须被填充，通过在`bean`定义或者自动装配一个明确的属性值

----

### 3.用于注解众所周知的解析依赖性借口，如BeanFactory，ApplicationCon，Environment，ResourceLoader，ApplicationEventPublisher，MessageSource 等
```java
public class BeanSutowired{
    @Autowired
    private ApplicationContext context;
}
```
----
### 4.可通过添加注解给需要该类型的数组的字段或方法，提供ApplicationContext中的所有特定类型的bean
下面的示例中，`list`添加了`@Autowired`注解，那么所有的实现`BeanInterface`接口的`bean`，假如有`BeanimplOne`和`BeanimplTwo`都实现了`BeanInterface`接口，那么这时`list`中将包含有`BeanimplOne`和`BeanimplTwo`。

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
> 如果希望数组有序，可以使用`@Order`注解或者实现`org.springframework.core.Ordered`接口，但是对`map`无效

```java
@Order(1)
@Component
public class BeanimplTwo implements BeanInterface{
}
```
----
### 5.用于装配key为String的Map
下面的示例中，`map`添加了`@Autowired`注解，那么所有的实现`BeanInterface`接口的`bean`，加入有`BeanimplOne`和`BeanimplTwo`都实现了`BeanInterface`接口，那么这时map中将包含有键为`bean`名称和值为`bean`的两个元素。
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
----
### 6.@Qualifier
- 按类型自动装配可能多个bean实例的情况，可以使用Qualifier注解缩小范围或指定唯一
 
```java
@Autowired
@Qualifier("beanImplTwo")
private BeanInterface beanInterface;
```

- 用于指定单独的构造器参数或方法参数
- 用于注解集合类型变量

可以在bean的定义中使用`@Qualifier`注解给他限定一个范围，比如
```java
@Qualifier("beanImpl")
@Component
public class BeanImplTwo implements BeanInterface{
}
```
然后在注入时，使用`@Qualifier`限定，则下面的`list`将会匹配到所有`@Qualifier("beanImp")`的`bean`
```java
@Autowired
@Qualifier("beanImpl")
private List<BeanInterface> list;
```

> 参阅：
  [慕课网：Spring入门篇](https://www.imooc.com/learn/196)