---
title: Spring Bean装配之各装配项 XML实现和注解实现
date: 2018-07-27 18:56:18
categories: "JavaWeb"
tags:
    - JavaWeb
    - Spring
---

### 一、Bean管理的XML配置实现
#### 1.Bean的配置项
- `Id`:Bean的唯一标识
- `Class`：对应实现的类
- `Scope`：范围
- `Constructor arguments`：构造器参数
- `Properties`：属性
- `Autowiring mode`：自动装配模式 
- `lazy-initialization mode`：懒加载模式
- `Initialization/destruction method`：初始化/销毁方法

#### 2.Bean的定义
这里以`InjectionImpl`中包含`InjectionDAO`成员变量为例，说明设置注入和构造注入,`InjectionImpl`类如下：

```java
public class InjectionServiceImpl implements InjectionService {
    private InjectionDAO injectionDAO;
}
```

##### 方式一：设置注入
`bean`的XML配置:

```XML
<bean id="injectionService" class="com.pinnuli.spring.ioc.injection.service.InjectionServiceImpl">
    <property name="injectionDAO" ref="injectionDAO"></property>
</bean>
<bean id="injectionDAO" class="com.pinnuli.spring.ioc.injection.dao.InjectionDAOImpl"></bean>
```

`InjectionIml`中setter方法:

```java
public void setInjectionDAO(InjectionDAO injectionDAO) {
    this.injectionDAO = injectionDAO;
}
```


##### 方式二：构造注入

bean的XML配置：

```XML
<bean id="injectionService" class="com.imooc.ioc.injection.service.InjectionServiceImpl">
    <constructor-arg name="injectionDAO" ref="injectionDAO"></constructor-arg>
</bean>
<bean id="injectionDAO" class="com.pinnuli.spring.ioc.injection.dao.InjectionDAOImpl"></bean>
```

`InjectionIml`中构造方法：

```java
public InjectionServiceImpl(InjectionDAOImpl injectionDAO) {
    this.injectionDAO = injectionDAO;
}
```


#### 2.Bean的作用域
- `singleton`:单例，一个Bean容器中指存在一份（默认情况下为singleton)
- `prototype`：每次使用（每次请求，即每次向IOC容器请求获取一个对象时）都创建新的实例，destroy方法不生效
- `request`：每次http请求创建一个实例且仅在当前request内有效
- `session`：同上，每次http请求创建，当前session有效
- `global session`：给予portel的web中有效，如果是在web中，则同session

XML文件中的配置
```XML
<bean id="beanScope" class="com.pinnuli.spring.ioc.bean.BeanScope" scope="singleton"></bean>
```

#### 3.Bean的生命周期
> 定义 &rArr; 初始化 &rArr;	使用 &rArr; 销毁

**初始化**
##### 方式一：实现org.springframework.beans.factory.InitializingBean借口，覆盖afterPropertiesSet方法
```java
public class ExampleInitializingBean implements InitialingBean{

    @Override
    public void afterPropertiesSet() throws Exception{
    }
}
```
##### 方式二：配置init-method
XML文件中的配置：
```XML
<bean id="exampleInitBean" class="example.Example" init-method="init"/>
```
对应实现类：
```Java
public class ExampleBean{
    public void init(){
    }
}
```

**销毁:**

##### 方式一：实现org.springframework.beans.factory.DisposableBean借口，覆盖destory方法
```java
public class ExampleInitializingBean implements DisposableBean{

    @Override
    public void destroy() throws Exception{
    }
}
```

##### 方式二：配置destory-method
XML文件中的配置：
```XML
<bean id="exampleInitBean" class="example.Example" init-method="destroy"/>
```

`destory`方法：
```Java
public class ExampleBean{
    public void destroy(){
    }
}
```

#### 配置全局默认初始化、销毁方法

```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd"
       default-init-method="init" default-destroy-mothod="destroy">
</beans>
```

>i.当三种方式同时配置时，实现接口和配置bean初始化/销毁方法会覆盖全局默认方法，全局默认方法会失效；
  ii.即使配置了全局方法，在具体实现中依然可以不定义对应的方法，不会有任何异常或报错；
  iii.一旦配置了bean初始化/销毁方法，则必须定义对应的初始化销毁方法。


#### 4.Bean的自动装配
- `No`：什么都不操作
- `byname`：即<bean>中的`id`，根据属性名自动装配，
- `byType`：即<bean>中的`class`

> i.如果容器中存在一个与制定属性类型相同的bean，将与该属性自动装配；
ii.如果存在多个该类型的bean，则抛出异常，并指出不能使用`byType`方式进行自动装配
iii.如果没有找到匹配的bean，则不进行任何操作
以上两种情况bean的XML配置如下：

```XML
<beans default-autowire="byType"/"byName">
    <bean id="autoWiringService" class="com.pinnuli.spring.ioc.autowiring.AutoWiringService"></bean>

    <bean id="autoWiringDAO" class="com.pinnuli.spring.ioc.autowiring.AutoWiringDAO"></bean>
</beans>
```

- `Constructor`: 应用于构造器参数，与`byType`类似，如果容器没有找到与构造器参数类型一致的bean，则抛出异常

> 对应的类中的构造方法和setter方法与设置注入或构造注入一致

----
### 二、Bean管理的注解实现
#### 1.用注解实现时，需要配置以下XML文件扫描有Bean注解的类：
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
- `<context:annotation-config/>`
仅会查找同一个`applicat context`中的bean注解,即扫描完成注册后的bean中方法和成员变量的注解
通过在基于XML的Spring配置如下标签


- `<context:component-scan>`会扫描所有有bean注解的类，并注册到IOC容器，包含了`<context:annotation-config>`的全部功能，因而通常只需要使用前者，而不用后者

```XML
<context:component-scan base-package="org.example>
```
> base-package表示扫描包下的所有类

#### 2.使用过滤器进行自定义扫描
默认情况下，类被自动发现并注册bean的条件是：使用了`@Component`，`@Repository`，`@Service`，`@Controller`注解，或者使用@`Component`的自定义注解，可以通过过滤器修改上述的行为

```XML
<beans>
    <context:component-scan base-package="org.example">
        <context:include-filter type="regex" expression=".*Stub.*Respository"/>
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
    <context:component-scan/>    
</beans>
```
> 还可以使用`use-default-filters="false"`禁用自动发现与注册


#### Bean的定义
Bean名称是由`BeannameGenerator`生成的，默认情况下为类名的首字母变为小写

> `@Component`，`@Repository`，`@Service`,`@Controller`都有一个那么属性用于显示设置Bean的名称,如
```java
@Component("beanName")
public class BeanAnnotation {
}
```

也可自定义命名策略,实现`BeanNameGenerator`接口，并一定要包含一个无参构造器
```XML
<beans>
    <context:component-scan base-package="org.example" name-generator="org.example.MyNameGenerator"/>
</beans>
```

#### Bean的作用域
通常情况下启动查找的Spring组件，其scope是`singleton`，可用`@Scope`表示scope,
```java
@Scope("prototype")
```

也可自定义scope策略，实现实现`ScopeMetadataResolver`接口，并一定要包含一个无参构造器
```XML
<beans>
    <context:component-scan base-package="org.example" name-generator="org.example.MyScopeResolver"/>
</beans>
```

> 对于自动装配注解，参见[Spring Bean装配之Autowired注解](/2018/07/28/Spring-Bean装配之Autowired注解/#more)

#### 代理方式
有三个值可选：`no`,`interfaces`,`targetClass`，默认情况下为no

可以配置`@Scope`注解的`proxyMode`属性来配置代理方式，即XML配置时的`scope-proxy`属性
```java
@Scope(value="prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
```

可以在XML文件中使用`scope-proxy`属性指定代理
```XML
<beans>
    <context:component-scan base-package="org.example" scope-proxy="interfaces"/>
</beans>
```
----
### 三、Resource&ResourceLoader
#### Resource
针对资源文件的统一入口，用于Spring加载资源文件
- UrlResource:URL对应的资源，根据一个URL地址即可构建
- ClassPathResource：获取类路径下的资源文件
- FileSystemResource：获取文件系统里面的资源
- ServletContextResource：ServletContext封装的资源，用于访问ServletContext环境下的资源
- InputStreamResource：针对于输入流封装的资源
- ByArrayResource：针对于字节数组封装的资源


#### ResourceLoader
> i.所有的application contexts都实现了`ResourceLoader`接口，即可以通过`ApplicationContext`获得Resource实例
ii.使用参数的前缀说明获取资源的类型

##### 1.类路径下的资源文件
```java
Resource resource = applicationContext.getResource("classpath:config.txt");
```
##### 2.文件系统中的资源
```java
Resource resource = applicationContext.getResource("file:/var/SpringDemo/src/main/resources/config.txt");
```
##### 3.URL对应的资源
```java
Resource resource = applicationContext.getResource("url:httpS://www.pinnuli.com/index.html");
```
> 没有前缀时，取决于`ApplicationContext`的路径（之后再添加解释）

```java
Resource resource = applicationContext.getResource("config.txt");
```

> 参阅：
  [慕课网：Spring入门篇](https://www.imooc.com/learn/196)