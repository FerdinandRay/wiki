---
title: jsp总结
date: 2016-07-28 22:07
---

[TOC]

## 容器
> ***容器的功能***：

> - 通信支持

> - 声明周期管理

> - 多线程支持

> - 声明方式实现安全

> - JSP支持

`Tomcat`就是我们说的容器，可以说它是Apache的加强版，能处理jsp

## JSP

### 0x01 jsp元素
    1. scriptlet： <% %>
    2. 表达式： <%= %> 不需要分号
    3. 指令： <%@ 指令名 %>
        page, taglib, include
    4. 声明： <%! 声明变量表达式 %> 这里面声明的变量在servlet方法之外
    5. EL表达式： 更好的调用java方法返回一些值
    
### 0x02 隐式对象对应
	JspWriter --> out
	HttpServletRequest --> request
	HttpServletResponse --> response
	HttpSession --> session
	ServletContext --> application
	ServletConfig --> config
	Throwable --> exception
	PageContext --> pageContext
	Object --> page

### 0x03 四大作用域
	Page
	Request
	Session
	Application

### 0x04 EL表达式

#### EL隐式表达式

- pageScope
- requestScope
- sessionScope
- applicationScope
- param
- paramValues
- header
- headerValues
- cookie
- initParam
- pageContext(it's a javabean)

```java
// 使用EL表达式调用函数
// 1. 编写.java文件，里面写你需要调用的函数，要static的才行
public class ElUse {
    public static String say(){
        return "Hello World";
    }
}
// 2. 编写tld文件
<uri>SayFunction</uri>
<function>
    <name>myel</name>
    <function-class>com.fang.el.ElUse</function-class>
    <function-signature>java.lang.String say()</function-signature>
</function>
// 3. 在jsp文件写指令
<%@ taglib prefix="test" uri="SayFunction" %>
// 4. 使用
${test:say()}
```

### 0x05 表达式
```java
<%=Class.getInt()%> 不需要加分号
// 上一行代码会转换为
out.print(CLass.getInt());
```

### 0x06 JSP-->Servlet

容器会把JSP页面内的每一行html语言放在out.print()中，而java语句则不变

jsp其实也是一个servlet，容器会帮你做一些转变

## 疑难点

- `getServerPort()`&`getLocalPort()`&`getRemotePort()`

```java
getRemotePort()：得到远程客户的端口，就是发出请求的客户端口
getServerPort()：请求原来发送到哪个端口
getLocalPort()：请求最后发送到哪个端口

尽管请求要发送到端口，但是服务器会为每一个线程开一个不同的本地端口，这样一个用户就能同时处理多个用户

PS：所以个人判断getServerPort()得到的是访问服务器的端口，而getLocalPort()则是该请求在服务器本地开的端口号
```
- 幂等不非幂等

```java
幂等：多次运行不会对服务器产生副作用，如GET
非幂等：请求若多次发送，能较好处理该问题，如POST

买书问题，因为POST是非幂等，若是网页没有反应，则用户可能多次提交表单造成订单的重复
```

- `sendRedirect()`和`getRequestDispatcher()`

```java
sendRedirect()直接将请求转移到另一个页面，那么这个servlet获得这个请求之后不要对resp或者req做更改（标记，我也不知道这么说对不对）。就算有更改在另一个页面也无法获得更改。

不能对请求做响应的操作，一旦响应，则请求结束

通过getRequestDispatcher().forward(req, resp)，这样能够让req或者resp做些更改再转移到另一个页面
```

- ServeletConfig问题

```java
ServletConfig针对servlet
<servlet>
        <servlet-name>login</servlet-name>
        <servlet-class>LoginServlet</servlet-class>
        <init-param>
            <param-name>name</param-name>
            <param-value>test</param-value>
        </init-param>
</servlet>
这里配置的参数只对LoginServlet有用，其他servlet或者jsp要先把键值对设置在请求中，然后转发请求

ServletConfig是作为参数在容器初始化servlet之后调用init(ServletConfig config)的时候才给的
所以最好不要在init里面调用getServeletConfig()方法

容器在部署的时候只初始化一次，所以只读一次web.xml配置文件，然后将参数给ServletConfig和ServletContext
然后就不会再读了。所以修改配置文件之后要重新部署
```

- ServletContext问题

```java
ServletContext针对整个Web应用，在ServeletConfig之前创建，个人猜测，然后读取web.xml中的键值对
<context-param></context-param>在这里配置的参数，这个是<web-app>的下面一层，范围大。所以所有servlet和jsp都可以获得

serlvet和jsp都通过getServletContext()即可获得web.xml中的参数了
```

- ServletContextListener问题

```java
// 如果我想在应用初始化的时候传入一个对象该怎么办？我希望这个对象能够在应用初始化的时候做一些事情，销毁的时候也做一些事情。

// 1. 创建一个对象，这个对象是你需要在应用初始化的时候传入的一个对象
// 2. 定义一个context的初始化参数dbType，用来获取指定的数据库类型
// 3. 创建一个监听者类，实现接口ServletContextListener即可
@Override
public void contextInitialized(ServletContextEvent sce) {
    // 初始化函数
    // 这里简单的写一个，我们会从web.xml中获取一个String参数，假设传入一个工厂类，然后返回需要的数据库连接
    ServletContext sc = sce.getServletContext();
    String dbType = sc.getInitParameter("dbType");
    DataBaseMySql db = DbFactory.get(dbType);
    sc.setAttribute("db", db);
}

@Override
public void contextDestroyed(ServletContextEvent sce) {
    // 销毁函数
}
// 4. 在web.xml中添加listener
<listener>
    <listener-class>MyListener</listener-class>
</listener>

// 5. 然后写个继承自servlet的测试类测试能否获取即可
```

- HttpSessionBindingListener

```java
// HttpSessionBindingListener和HttpSessionAttributeListener的区别
// HttpSessionAttributeListener希望会话中增加、删除或替换某种类型的属性能够知道
// HttpSessionBindingListener让属性本身在增加到一个会话或从会话中被删除时得到通知
```

- 属性与参数的区别

```java
// 由于请求是并发的，所以要注意你的上下文属性，可能会被修改
// 上下文属性的安全应该通过同步上下文才对。不能同步doGet()等方法，因为同步这些方法意味着你的应用一次只能接待一个用户，而且如果有另一个servlet也访问这个属性，就没有办法了。
// 所以只要同步上下文即可
synchronized(getSerlvetConext()){
    // 在这里对属性进行操作
}
```

- 安全问题

```java
// 上下文需要安全，session需要安全，
// 但是我们无法对请求方法进行同步，这样只能阻止一个servlet的访问，不能阻止零另一个servlet，所以我们要在方法内部对这些接口或者对象的实例化成员同步

// 但是同步代码会增加开销，所以如果代码并不访问受保护的内容，则不需要同步
```

- 会话迁移

```java
// 如果在会话迁移的时候需要保存和恢复属性。这个属性是个对象，则这个对象需要实现HttpSessionActivationListener
```
