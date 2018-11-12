---
title: 'Spring Bean装配之基于Java的容器注解 '
date: 2018-07-29 17:13:43
categories: "Spring笔记"
tags:
    - JavaWeb
    - Spring
copyright: true
---

### @Bean
用于配置和初始化一个有SpringIOC容器管理的新对象的方法，类似于XML配置文件的<bean/>，通常和`@Configuration`配合使用

```java
@Configuration
public class StoreConfig {

    @Bean(name = "store",initMethod = "init",destroyMethod="destroy")
    public BeanStore beanStore(){
        return new BeanStore();
    }
}
```
相当于以下XML配置
```XML
<beans>

    <bean id="store" class="com.pinnuli.spring.ioc.beanannotation.BeanStore" init-method="init" destroy-method="destroy"></bean>
</beans>
```
> 如果`@Bean`没有指定名称，则默认为方法名，这里即是beanStore。</br>
如果需要指定范围，即XML配置时的属性`scope`，那么可以用`@Scop`e注解，且可以配置`@Scope`注解的`proxyMode`属性来配置代理方式，即XML配置时的`scope-proxy`属性

```java
@Scope(value="prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
```
----
### @ImportResource,@Value
> 通过`@ImportResource`加载资源文件,`@Value`获取属性值，比如有文件`config.properties`内容如下：
```
jdbc.username=root
password=root
url=127.0.0.1
```

则可以通过将此内容配置到XML文件，如`config.xml`：
```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd" >

    <context:property-placeholder location="classpath:/config.properties"/>

</beans>
```

然后在定义`bean`时，通过`@ImportResource`加载文件，通过`@Value`获取属性的值
```java
@Configuration
@ImportResource("classpath:config.xml")
public class StoreConfig {

    @Value("${url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${password}")
    private String password;

    @Bean
    public MyDriverManager myDriverManager() {
        return new MyDriverManager(url, username, password);
    }
}
```
> 注意：这里的用户名字段如果直接为`username`，那么得到的将是当前系统的用户名，而不是文件中的属性值，所以一般此类属性名称加前缀，如`jdbc.username`,`jdbc.password`等

----
### 基于泛型的自动装配
比如现有如下接口：
```java
public interface Store<T> {
}
```

`IntegerStore`和`StringStore`是他的两个实现类：
```java
public class IntegerStore implements Store<Integer> {
}
```
```java
public class StringStore implements Store<String> {
}
```
那么在定义`bean`时可以按照如下方法：
```java
@Autowired
private Store<String> s1;

@Autowired
private Store<Integer> s2;

@Bean
public StringStore stringStore() {
    return new StringStore();
}

@Bean
public IntegerStore integerStore() {
    return new IntegerStore();
}
```
> 那么`s1`将会自动装配到`StringStore`，`s2`将会是`IntegerStore`

----
### JSR支持
#### JSR250的支持
1.@Resource 
注解变量或方法，且有一个name属性值，默认`Spring`解释改值为被注入`bea`n的名称，若没有指定`name`,那么名称从方法名或者属性名得出
```java
@Service
public class JsrServie {
	
	@Resource
	private JsrDAO jsrDAO;
	
	@Resource
	public void setJsrDAO(@Named("jsrDAO") JsrDAO jsrDAO) {
		this.jsrDAO = jsrDAO;
	}
}
```
2.@PostConstruct和@PreDestroy
`@PostConstruct`,初始化，相当于`init-Method`属性
`@PreDestroy`，销毁，相当于`destroy-Method`属性
```java
@PostConstruct
public void init() {
}

@PreDestroy
public void destroy() {
}
```
#### JSR330的支持
> 需要依赖`javax.injet`包

1.@Inject
与`@Autowired`等效，可以使用于类，属性，方法，构造器
2.@Named
使用特定名称进行依赖注入，与`@Component`等效

```java
@Named
public class JsrServie {
	
	@Inject
	private JsrDAO jsrDAO;
	
	@Inject
	public void setJsrDAO(@Named("jsrDAO") JsrDAO jsrDAO) {
		this.jsrDAO = jsrDAO;
	}
}
```

> 参阅:
  [慕课网：Spring入门篇](https://www.imooc.com/learn/196)