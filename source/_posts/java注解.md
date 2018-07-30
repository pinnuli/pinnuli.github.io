---
title: java注解
date: 2018-07-23 12:39:36
categories: "javase笔记" 
tags:
    - JavaSE
---
### 一、注解分类
- 源码注解（SOURCE）：注解只在源码中存在，编译成.class文件就不存在
- 编译时注解（CLASS）：注解在源码和.class文件都存在
- 运行时注解（RUNTIME）：在运行阶段还起作用，甚至会影响运行逻辑的注解
- 元注解：注解的注解

----
### 二、自定义注解
#### 定义：
```java
//元注解
@Target({ElementType.METHOD,ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented

//定义注解和成员变量
public @interface Description{
    String desc();
    String author();
    int age() default 20;
}
```
- `@Target({ElementType.METHOD,ElementType.TYPE})`

> 作用域,可以包括`CONSTRUCTOR`(构造方法)、`FIELD`(字段)、`LOCAL_VARIABLE`(局部变量)、`METHOD`(方法)、`PACKAGE`(包)、`PARAMETER`(参数)、`TYPE`(类和接口)声明中，这里作用域为方法、类和接口

- `@Retention(RetentionPolicy.RUNTIME)`

> 生命周期，可以设置为Source，CLASS，RUNTIME,这里生命周期是运行时

- `@Inherited`

> 标识性注解，允许子类继承，只适用于类的继承，对接口的继承无效，而且只会继承类级别的注解，不会继承超类的方法和成员变量的注解

- `@Documented`

> 生成javadoc包含注解信息

- `@interface`

>1.使用关键字`@interface`定义注解，
>2.成员以无参无异常方式声明，可以用default指定一个默认值
>3.成员类型是受限的，合法的类型包括基本类型，String，Class，Annotation，Enumeration
>4.如果注解只有一个成员，则成员名必须为value()，在使用时可以忽略成员名和赋值号,即`Description("test")`
>5.注解可以没有成员名，叫标识注解

#### 使用：
@<注解名>(<成员名1>=<成员值1>,<成员名2>=<成员值2>,...)
```java
Description(desc="I am pinnuli",author="pinnuli",age=20)
public String test(){
    return "test";
}
```
----
### 三、解析注解
> 通过反射获取类、函数或成员上的运行时追截信息，从而实现动态控制程序运行的逻辑

1.使用类加载器加载类
```java
Class c = Class.forname("com.test.Student");
```
2.找到类上的注解,拿到注解实例
```java
if(c.isAnnotationPresent(Description.class)){
    Description d = (Description)c.getAnnotation(Description.class);
}
```
3.找到方法上的注解
- 方法一

```java
Method[] ms = c.getMethods();
for(Method m:ms){
    if(m.isAnnotationPresent(Description.class)){
        Description d = (Description)m.getAnnotation(Description.class);
    }
}
```
- 方法二

```java
Method[] ms = c.getMethods();
for(Method m:ms){
    for(Annotation a:as){
        if(a instanceof Description){
            Description d = (Description)a;
        }
    }
}
```

> 参阅：
  [慕课网：全面解析Java注解](https://www.imooc.com/learn/456)