---
title: Java Web 监听器
date: 2018-07-25 19:50:55
tags:
---
### 监听器的启动顺序
优先级： 监听器 > 过滤器 > Servlet

### 监听器的分类
#### 按监听的对象划分
1.用于监听应用程序环境对象（ServletContext）的事件监听器，实现`ServletContextListener`接口
2.用于监听用户会话对象（HttpSession）的事件监听器，实现`HttpSessionListener`接口
3.用于监听请求消息对象（ServletRequest）的事件监听器，实现`ServletRequestListener`接口
#### 按监听的事件划分
1.监听域对象自身的创建和销毁的事件监听器
2.监听域对象的属性的增加和删除的事件监听器，实现`ServletContextAttributeListener`，`HttpSessionAttributeListener` 或 `ServletRequestAttributeListener`接口
3.监听绑定到HttpSession域中的某个对象的状态的事件监听器
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

 