---
title: JSP学习笔记
date: 2018-05-03 21:17:47
categories: "JSP笔记" 
tags:
	- JSP
	- JavaWeb
---
### 一、jsp简介

#### 1、jsp三大指令

**page指令**:<%@page 属性="" %>,位于jsp页面顶端，可以有多个

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" import="java.text.*"%>

```
**taglib指令**：标签库

**include**
include指令:`<%@include file="date.jsp"%>`
include动作:`<jsp:include page="url" flush="true|false"/>`
> page:要包含的页面,
flush：被包含的页面是否从缓冲区读取

**include指令与include动作的区别：**
![diff_between_includeCommand_includeAction](https://i.imgur.com/YyHsnwD.png)

**forward动作**：`<jsp: forward page="url"/>`
等同于：`request.getRequestDispatcher("/url").forward(request,response);`

**param动作**：`<jsp:param name="参数名" value="参数值">`
常与<jsp:forward>一起使用，作为其的子标签:

``` jsp
<jsp:forward page="user.jsp">
	<%--用<jsp:param "></jsp:param>添加参数--%>
	<jsp:param name="email" value="11111111@163.com"></jsp:param>
</jsp:forward>
```

#### 2、jsp注释
- html的注释
	<!-- html注释 -->
- jsp的注释
 <%-- jsp注释 -->（客户端不可见）
- jsp脚本注释:
//单行
/* */ 多行
 
#### 3、jsp脚本
<% java代码 %>

#### 4、jsp声明变量或方法
<%! java代码 %>

#### 5、jsp表达式
<%=表达式 %>  ps:不可;分号结束

#### 6、jsp页面的生命周期

![jsp_life_cycle.png](https://i.imgur.com/U27ljB4.png) 


### 二、jsp内置对象
** 九大内置对象：**`out`,`request`,`response`,`session`,`application`,`Page`,`pageContext`,`exception`,`config`

**out**

```html

	<%
    	out.println("<h2>静夜思</h2>");
    	out.println("床前明月光<br>");
    	out.println("疑是地上霜<br>");
    	out.flush();
   		/*out.clear();会抛出异常*/
    	out.clearBuffer();//这里不会抛出异常
    	out.println("举头望明月<br>");
    	out.println("低头思故乡<br>");
    %>

    缓冲区大小：<%= out.getBufferSize()%>byte<br>
    缓冲区剩余大小：<%= out.getRemaining()%>byte<br>
    是否自动清空缓冲区：<%= out.isAutoFlush()%><br>
```

**request**
 
```html

	<%
        request.setCharacterEncoding("utf-8");//解决post中文乱码问题，但无法解决get，get解决需要直接Tomcat配置文件
        request.setAttribute("password","123456");//设置属性密码
    %>
    用户名：<%= request.getParameter("username")%><br>
    爱好：
    <%
        if(request.getParameterValues("favorite") != null){  //这里需要判断为不为空，jsp这里不能将String数组看为Boolean
            String[] favorites = request.getParameterValues("favorite");
            for (int i = 0; i < favorites.length; i++) {
                out.println(favorites[i] + "&nbsp;&nbsp;&nbsp;");
            }
        }
        String realPath = request.getRealPath("requset.jsp");%><br>

    密码：
    <%=request.getAttribute("password")%><br>

    请求体的MIME类型：
    <%=request.getContentType()%><br>

    协议类型和版本号：
    <%=request.getProtocol()%><br>

    服务器主机名：
    <%=request.getServerName()%><br>

    服务器端口号：
    <%=request.getServerPort()%><br>

    请求文件长度：
    <%=request.getContentLength()%><br>

    请求的客户端地址：
    <%=request.getRemoteAddr()%><<br>

    请求的真实路径：
    <%=request.getRealPath("requset.jsp")%><br>

    请求的上下文路径：
    <%=request.getContextPath()%>
```
	
**response**

```html

	<%
        response.setContentType("text/html;charset=utf-8");
        out.println("<h1>response内置对象</h1>");
        out.println("<hr>");
        //out.flush();
        /* 因为getWrite获得的输出流对象会先于内置对象out输出，
        所以要先清空缓冲区，使out强制输出，否则结果会是先输出outer
        再输出out*/

        PrintWriter outer = response.getWriter();
        outer.println("大家好，我是response生成的输出流outer");
	//  response.sendRedirect("login.jsp");//重定向
	//  response.sendRedirect("request.jsp");
        request.getRequestDispatcher("request.jsp").forward(request,response);//转发
    %>

```

**请求转发和请求重定向的区别：**

![redirectAndTransmit.PNG](https://i.imgur.com/wWWpmzd.png)

**session**
HttpSession的实例，周期：在第一个jsp页面被加载时自动创建，即浏览器连接到服务器开始，关闭浏览器离开这个服务器结束，在服务器的几个页面之间切换，服务器应当知道这是一个客户，就可以用session对象

```html

	<%
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy年mm月dd日 HH:mm:ss");
        Date d = new Date(session.getCreationTime());
        session.setAttribute("username","pinnuli");
        session.setAttribute("password","123456");
        session.setAttribute("age",20);
	//  session.setMaxInactiveInterval(10);

	//设置session最大生成期限，单位秒,也可在web.xml中设置session-timeout

    %>
    Session创建时间：
    <%=sdf.format(d)%><br>

    Session的ID：
    <%=session.getId()%><br>

    Session中获取属性值：
    <%=session.getAttribute("username")%><br>

    Session保存的属性数组：
    <%
        String[] names = session.getValueNames();
        for(int i=0; i<names.length; i++){
            out.println(names[i] + "&nbsp;&nbsp;");
        }
	//        session.invalidate();//销毁当前会话,每次刷新一次页面就会新建一个session
    %><br>
    <%--测试不同页面是否同一个session--%>
    <a href="session_page2.jsp">跳转到session_page2</a>
```

**application**
实现用户间数据的共享，可存放全局边变量，相当于java的静态变量

```html

	<%
        application.setAttribute("city","广州");
        application.setAttribute("postcode","510000");
        application.setAttribute("email","guangzhou@163.com");
    %>
    所在城市：<%=application.getAttribute("city")%><br>
    所有属性：
    <%
        Enumeration attributes = application.getAttributeNames();
        while (attributes.hasMoreElements()){
            out.println(attributes.nextElement() + "&nbsp;&nbsp;");
        }
    %><br>

    jsp(serviet)引擎名和版本号：<%=application.getServerInfo()%><br>
```
	
**page、pageContext**

```html
	<h3>page:</h3>当前page页面的字符串描述：<%=page.toString()%><br><br>

   <h3>pageContext:</h3>用户名：从session中获取属性-<%=pageContext.getSession().getAttribute("username")%><br>

    <%--跳转到其他页面--%>
   <%--<%
       pageContext.forward("out.jsp");
   %>--%>
    include方法，包含其他页面:
        <%
            pageContext.include("out.jsp");
        %>
```

**exception**

```html

	异常消息：<%=exception.getMessage()%><br>
    异常的字符串描述：<%=exception.toString()%>
```


### 三、jsp使用Javabean

#### 1. Javabean的设计原则

- 必须是公有类
- 必须包含无参构造方法
- 属性私有
- 用getter()和setter()进行封装

例如：

```java

public class Students{
	private String name;
	private int age;

	public  Students(){
	
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}

}

```
#### 2. 存取Javabean有关的jsp动作元素  
**在jsp页面中使用Javabeans：**

方法一：像使用普通java类一样，创建Javabean实例
方法二：在jsp页面中通常使用jsp动作标签使用javabean,常用的动作标签：userBeans、setProperty、getProperty

- `<jsp:useBeans>`

在jsp页面中实例化或者在指定范围内使用Javabean：

`<jsp:useBeans id="标示符" class="java类名" scope="作用范围"/>`
> scope属性：指定Javabean的作用范围
> page：当前页面,重定向和转发都无效
> request：可通过HttpRequest.getAttribute()取得Javabean对象，重定向无效，转发有效
> session：可通过HttpSession.getAttribute()取得Javabean对象，同个会话有效
> application:可通过application.getAttribute()取得Javabean对象，不同会话都有效

例如：
```html
	<jsp:useBean id="myUsers" class="com.po.Users" scope="application"></jsp:useBean>
	用户名：<jsp:getProperty name="myUsers" property="username"></jsp:getProperty>
	密码：<jsp:getProperty name="myUsers" property="password"></jsp:getProperty>
```

也可使用内置对象获取：
```html
	用户名：<%=((Users)application.getAttribute("myUsers")).getUsername()%>
	密码： <%=((Users) application.getAttribute("myUsers")).getPassword()%>


```

- `<jsp:setProperty>`
	
```html	
	根据表单自动匹配所有属性:
	<jsp:setProperty name="myUsers" property="username"></jsp:setProperty>

	根据表单匹配部分属性:
	<jsp:setProperty name="myUsers" property="username"></jsp:setProperty>

	与表单无关，通过手工赋值给属性:
	<jsp:setProperty name="myUsers" property="password" value="hahahaha"></jsp:setProperty>

	通过url传参数给属性赋值:
	<jsp:setProperty name="myUsers" property="password" param="testparam"></jsp:setProperty>

```

- `<jsp:getProperty>`

```html
	使用getProperty获取属性值:
	<jsp:getProperty name="myUsers" property="username"></jsp:getProperty>

```
	
> 参阅：
  [慕课网：JAVA遇见HTML——JSP篇](https://www.imooc.com/learn/166)
	
