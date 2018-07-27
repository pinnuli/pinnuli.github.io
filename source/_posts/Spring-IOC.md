---
title: Spring IOC
date: 2018-07-27 18:56:18
categories: "Java Web"
tags:
    - Java Web
    - Spring
---

### Spring注入
1.设置注入
2.构造注入
### Bean
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
- 定义
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
- 使用
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

#### Bean的自动装配
#### Resource&ResourceLoader
