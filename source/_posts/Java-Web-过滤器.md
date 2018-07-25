---
title: Java Web 过滤器
date: 2018-07-25 14:36:41
categories: "Java Web" 
tags:
    - Java Web
---
### 过滤器的工作原理
![filetr_work_principle](/images/filter_work_principle.png)
----
### 过滤器的生命周期
![ filetr_lifecycle](/images/filter_lifecycle.png)
----
### 过滤器链
> Web项目中多个过滤器实现，多个过滤器对应同一个路近执行顺序如何？

过滤器链：
![filter_chain.png](/images/filter_chain.png)
过滤器链执行过程:
![filter_chain_process.png](/images/filter_chain_process.png)
 
### 过滤器分类
![filter_classify](/images/filter_classify.png)

使用`@WebFilter`注解声明过滤器，该注解会在部署时被容器处理，并根据其具体属性配置将其相应的类部署为过滤器
