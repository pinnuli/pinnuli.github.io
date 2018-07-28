---
title: Spring IOC
date: 2018-07-27 18:56:18
categories: "Java Web"
tags:
    - Java Web
    - Spring
---

### 一、Spring注入
> 这里以`InjectionImpl`中包含`InjectionDAO`成员变量为例，说明设置注入和构造注入,`InjectionImpl`类如下：

```java
public class InjectionServiceImpl implements InjectionService {
    private InjectionDAO injectionDAO;
}
```

#### 1.设置注入
> bean的XML配置

```XML
<bean id="injectionService" class="com.pinnuli.spring.ioc.injection.service.InjectionServiceImpl">
    <property name="injectionDAO" ref="injectionDAO"></property>
</bean>
<bean id="injectionDAO" class="com.pinnuli.spring.ioc.injection.dao.InjectionDAOImpl"></bean>
```

> InjectionIml中setter方法

```java
public void setInjectionDAO(InjectionDAO injectionDAO) {
    this.injectionDAO = injectionDAO;
}
```

#### 2.构造注入

> bean的XML配置

```XML
<bean id="injectionService" class="com.imooc.ioc.injection.service.InjectionServiceImpl">
    <constructor-arg name="injectionDAO" ref="injectionDAO"></constructor-arg>
</bean>
<bean id="injectionDAO" class="com.pinnuli.spring.ioc.injection.dao.InjectionDAOImpl"></bean>
```

> InjectionIml中构造方法

```java
public InjectionServiceImpl(InjectionDAOImpl injectionDAO) {
    this.injectionDAO = injectionDAO;
}
```

### Bean装配
#### Bean的配置项
- Id:Bean的唯一标识
- Class：对应实现的类
- Scope：范围
- Constructor arguments：构造器参数
- Properties：属性
- Autowiring mode：自动装配模式 
- lazy-initialization mode：懒加载模式
- Initialization/destruction method：初始化/销毁方法

#### Bean的作用域
- singleton:单例，一个Bean容器中指存在一份（默认情况下为singleton)
- prototype：每次使用（每次请求，即每次向IOC容器请求获取一个对象时）都创建新的实例，destory方法不生效
- request：每次http请求创建一个实例且仅在当前request内有效
- session：同上，每次http请求创建，当前session有效
- global session：给予portel的web中有效，如果是在web中，则同session

#### Bean的生命周期
> 定义 初始化 使用 销毁

- 初始化，两种方式
    1.实现org.springframework.beans.factory.InitializingBean借口，覆盖afterPropertiesSet方法
    ```java
    public class ExampleInitializingBean implements InitialingBean{

        @Override
        public void afterPropertiesSet() throws Exception{
        }
    }
    ```
    2.配置init-method
    bean配置：
    ```XML
    <bean id="exampleInitBean" class="example.Example" init-method="init"/>
    ```
    ```Java
    public class ExampleBean{
        public void init(){
        }
    }
    ```

- 销毁
1.实现org.springframework.beans.factory.DisposableBean借口，覆盖destory方法
    ```java
    public class ExampleInitializingBean implements DisposableBean{

        @Override
        public void destory() throws Exception{
        }
    }
    ```
    2.配置destory-method
    bean配置：
    ```XML
    <bean id="exampleInitBean" class="example.Example" init-method="destory"/>
    ```
    ```Java
    public class ExampleBean{
        public void destory(){
        }
    }
    ```
配置全局默认初始化、销毁方法：
```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd"
       default-init-method="init" default-destory-mothod="destory">
</beans>
``` 
> 当三种方式同时配置时，实现接口和配置bean初始化/销毁方法会覆盖全局默认方法，全局默认方法会失效；
  即使配置了全局方法，在具体实现中依然可以不定义对应的方法，不会有任何异常或报错；
  一旦配置了bean初始化/销毁方法，则必须定义对应的初始化/销毁方法。

### Aware
> Spring中提供了一些以Aware结尾的借口，实现了Aware借口的bean在被初始化之后，可以获取相应资源，并对相应的资源进行操作
#### Bean的自动装配
No：什么都不操作
byname：即<bean>中的id，根据属性名自动装配，
byType：即<bean>中的class
1.如果容器中攒在一个与制定属性类型相同的bean，将与该属性自动装配；
2.如果存在多个该类型的bean，则抛出异常，并指出不能使用byType方式进行自动装配
3.如果没有找到匹配的bean，则不进行任何操作

Constructor: 应用与构造器参数，与byType类似，如果容器没有找到与构造器参数类型一致的bean，则抛出异常
> 以上两种情况bean的XML配置如下：
```XML
<beans default-autowire="byType"/"buName">
    <bean id="autoWiringService" class="com.pinnuli.spring.ioc.autowiring.AutoWiringService"></bean>

    <bean id="autoWiringDAO" class="com.pinnuli.spring.ioc.autowiring.AutoWiringDAO"></bean>
</beans>
```
> 对应的类中的构造方法和setter方法与设置注入或构造注入一致
#### Resource&ResourceLoader
Resource
> 针对资源文件的统一入口，用于Spring加载资源文件
![resource](/image/resource.png)

ResourceLoader
> 素有的application contexts都实现了ResourceLoader接口，即可以通过ApplicationContext获得REsource实例
使用参数的前缀说明获取资源的类型
> 类路径下的资源文件
```java
Resource resource = applicationContext.getResource("classpath:config.txt");
```
> 文件系统中的资源
```java
Resource resource = applicationContext.getResource("file:/var/SpringDemo/src/main/resources/config.txt");
```
> URL对应的资源
```java
Resource resource = applicationContext.getResource("url:httpS://www.pinnuli.com/index.html");
```
> 没有前缀时，取决于ApplicationContext的路径（之后再添加解释）
```java
Resource resource = applicationContext.getResource("config.txt");
```
### Bean管理的注解实现
#### Bean的定义及作用域
<context:annotation-config/>
仅会查找同一个applicat context中的bean注解,即扫描完成注册后的bean中方法和成员变量的注解
通过在基于XML的Spring配置如下标签
```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd" >
        
    <context:annotation-config/>

</beans>
```
<context:component-scan>会扫描所有有bean注解的类，并注册到IOC容器，包含了<context:annotation-config>的全部功能，因而通常只需要使用前者，而不用后者
```XML
<context:component-scan base-package="org.example>
```
> base-package表示扫描包下的所有类

#### 使用过滤器进行自定义扫描
> 默认情况下，类被自动发现并注册bean的条件是：使用了@Controller，@Repository，@Service，@Controller注解，或者使用@Component的自定义注解
> 可以通过过滤器修改上面的行为
```XML
<beans>
    <context:component-scan base-package="org.example">
        <context:include-filter type="regex" expression=".*Stub.*Respository"/>
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
    <context:component-scan/>    
</beans>
```
> 还可以使用use-default-filters="false"禁用自动发现与注册

#### 定义Bean
Bean名称是由BeannameGenerator生成的，默认情况下为类名的首字母变为小写
> @Component，@Repository，@Service,@Controller都有一个那么属性用于显示设置Bean的名称,如@Component("beanName")

也可自定义命名策略,实现BeanNameGenerator借口，并一定要包含一个无参构造器
```XML
<beans>
    <context:component-scan base-package="org.example" name-generator="org.example.MyNameGenerator"/>
</beans>
```
#### 作用域
> 通常情况下启动查找的Spring组件，其scope是singleton，可用@Scope表示scope


也可自定义scope策略，实现实现ScopeMetadataResolver借口，并一定要包含一个无参构造器
```XML
<beans>
    <context:component-scan base-package="org.example" name-generator="org.example.MyScopeResolver"/>
</beans>
```

#### 代理方式
> 使用sceped-proxy属性指定代理，有三个值可选：no,interfaces,targetClass，默认情况下为no
```XML
<beans>
    <context:component-scan base-package="org.example" scope-proxy="interfaces"/>
</beans>
```


