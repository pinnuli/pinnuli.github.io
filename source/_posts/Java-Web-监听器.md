---
title: Java Web 监听器
date: 2018-07-25 19:50:55
categories: "JavaWeb"
tags:
    - JavaWeb
---

### 按监听的对象划分
#### 1.用于监听应用程序环境对象（`ServletContext`）的事件监听器，实现`ServletContextListener`接口
```java
public class MyServletContextListener implements ServletContextListener {

	public void contextInitialized(ServletContextEvent servletcontextevent) {
		String initParam = servletcontextevent.getServletContext().getInitParameter("initParam");
		System.out.println("contextInitialized : initParam = "+initParam);
	}

	public void contextDestroyed(ServletContextEvent servletcontextevent) {
		System.out.println("contextDestroyed");
	}

}
```
#### 2.用于监听用户会话对象（`HttpSession`）的事件监听器，实现`HttpSessionListener`接口
```java
public class MyHttpSessionListener implements HttpSessionListener {

	public void sessionCreated(HttpSessionEvent httpsessionevent) {
		System.out.println("sessionCreated");
	}

	public void sessionDestroyed(HttpSessionEvent httpsessionevent) {
		System.out.println("sessionDestroyed");
	}

}
```
#### 3.用于监听请求消息对象（`ServletRequest`）的事件监听器，实现`ServletRequestListener`接口
```java
public class MyServletRequestListener implements ServletRequestListener {

	public void requestDestroyed(ServletRequestEvent servletrequestevent) {
		System.out.println("requestDestroyed ");
	}

	public void requestInitialized(ServletRequestEvent servletrequestevent) {
		String name = servletrequestevent.getServletRequest().getParameter("name");
		System.out.println("requestInitialized name:"+name);
	}

}
```
----
### 按监听的事件划分
#### 1.监听域对象自身的创建和销毁的事件监听器
> 即按监听对象划分的那几种

#### 2.监听域对象的属性的增加和删除的事件监听器，实现`ServletContextAttributeListener`，`HttpSessionAttributeListener` 或 `ServletRequestAttributeListener`接口
```java
public class MyServletContextAttributeListener implements ServletContextAttributeListener {

	public void attributeAdded(ServletContextAttributeEvent servletcontextattributeevent) {
		System.out.println("ServletContext_attributeAdded:"+servletcontextattributeevent.getName());
	}

	public void attributeRemoved(ServletContextAttributeEvent servletcontextattributeevent) {
		System.out.println("ServletContext_attributeRemoved:"+servletcontextattributeevent.getName());

	}

	public void attributeReplaced(ServletContextAttributeEvent servletcontextattributeevent) {
		System.out.println("ServletContext_attributeReplaced:"+servletcontextattributeevent.getName());

	}

}
```
> `HttpSession` 和 `ServletRequest` 同理，只是方法参数类型不同

#### 3.监听绑定到`HttpSession`域中的某个对象的状态的事件监听器
> 这种情况不需要专门设计一个作为监听器的类，可以作为一个实体类，然后继承需要的接口：

```java
public class User implements HttpSessionBindingListener,HttpSessionActivationListener,Serializable {

	private String username;
	
	public void valueBound(HttpSessionBindingEvent httpsessionbindingevent) {
		System.out.println("valueBound Name:"+httpsessionbindingevent.getName());
	}

	public void valueUnbound(HttpSessionBindingEvent httpsessionbindingevent) {
		System.out.println("valueUnbound Name:"+httpsessionbindingevent.getName());
	}

	public String getUsername() {
		return username;
	}

	public void setUsername(String username) {
		this.username = username;
	}

	//钝化
	public void sessionWillPassivate(HttpSessionEvent httpsessionevent) {
		System.out.println("sessionWillPassivate "+httpsessionevent.getSource());
	}
	//活化
	public void sessionDidActivate(HttpSessionEvent httpsessionevent) {
		System.out.println("sessionDidActivate "+httpsessionevent.getSource());
	}
}
```
绑定和解除绑定：实现`HttpSessionBindingListener`接口
钝化和活化：实现`HttpSessionActivationListener`和`Serializable`接口
> 实现`Serializable`接口是因为钝化时需要将seesion序列化存储到文件或者数据库，活化时需要反序列化

> 参阅：
  [慕课网：JAVA Web开发技术应用——监听器](https://www.imooc.com/learn/271)