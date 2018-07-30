---
title: java反射
date: 2018-07-22 13:30:00
categories: "javase笔记" 
tags:
    - JavaSE
---
### 一、Class类的使用
类是对象，任何一个类都是`java.lang.class`类的实例对象，这个类，这个实例对象可以有三种表达方式，比如`Student`类：
- 任何一个类都有一个隐含的静态成员变量class

```java
Class c1 = Student.class;
```
- 通过getClass方法获得

```java
Class c2 = Student.getClass();
```
- forName

```java
Class c3 = Class.forName("com.Student");
```
> c1,c2,c3表示了Student类的类类型（class type),可以通过类的类类型创建该类的对象实例---->通过c1 or c2 or c3创建Student的实例对象
```java
Student stu = c1.newInstance();
```
----
### 二、动态加载类
- 静态加载类：编译时加载的类，通过new创建对象是静态加载类
- 动态加载类：运行时加载的类，编译时不用管类是否存在或者是否错误等问题

----
### 三、获取方法信息和成员变量，构造函数信息
#### 方法信息
- 所有的public的函数，包括父类继承而来的

```java
Method[] ms = c.getMethods();
```
- 所有该类自己声明的方法，不问访问权限

```java
Method[] ms = c.getDeclaredMethods();
```
- 返回值类型的类类型

```java
Class returnType = ms[i].getReturnType();
```
- 参数列表的类型的类类型

```java
Class[] paramTypes = ms[i].getParameterTypes();
```
- 方法的名称

```java
String methodName = ms[i].getName();
```
#### 成员变量
- 所有的public的成员变量的信息

```java
Field[] fs = c.getFields();
```
- 该类自己声明的成员变量的信息

```java
Field[] fs = c.getDeclaredFields();
```
- 成员变量的类型的类类型

```java
Class fieldType = field.getType();
```
- 成员变量的名称

```java
String fieldName = field.getName();
```
- 构造函数
- 所有的public构造函数

```java
Constructor[] cs = c.getConstructors();
```
- 所有的构造函数

```java
Constructor[] cs = c.getDeclaredConstructors();
```

- 构造函数的参数列表，得到的是参数列表的类类型

```java
Class fieldType = field.getType();
```

> 更多方法看API文档

----
### 四、方法反射的基本操作 

现有A类如下：
```java
class A{
    public void print(){
        System.out.println("helloworld");
    }
    public void print(int a,int b){
        System.out.println(a+b);
    }
        public void print(String a,String b){
    System.out.println(a.toUpperCase()+","+b.toLowerCase());
    }
}
```

1.要获取一个方法就是获取类的信息，获取类的信息首先要获取类的类类型
```java
A a1 = new A();
Class c = a1.getClass();
```
2.通过名称和参数列表来获取方法
```java
Method m = c.getMethod("print", int.class,int.class);
```
3.方法的反射操作
```java
Object o = m.invoke(a1, 10,20);
```
> 这个操作的效果等同于`a1.print(10,20)`

> 通过方法的反射，我们可以绕过编译

> 参阅：
  [慕课网：反射——Java高级开发必须懂的](https://www.imooc.com/learn/199)