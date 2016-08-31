---
title: struts2笔记
date: 2016-08-31 12:55
---

[TOC]

Struts2笔记
===


### 搭建顺序

#### 0x01 配置web.xml

```xml
<filter>
  <filter-name>struts2</filter-name>
  <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
</filter>

<filter-mapping>
  <filter-name>struts2</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

#### 0x02 新建struts.xml

> 在src目录下建立

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE struts PUBLIC
        "-//Apache Software Foundation//DTD Struts Configuration 2.3//EN"
        "http://struts.apache.org/dtds/struts-2.3.dtd">

<struts>
</struts>
```

#### 0x03 创建Action

```java
// 让该Action继承ActionSupport
public class HelloWorldAction extends ActionSupport {
    @Override
    public String execute() throws Exception {
        System.out.println("执行Action");
        return SUCCESS;
    }
}
```

#### 0x04 配置struts.xml

```xml
<package name="default" extends="struts-default" namespace="/">

    <action name="helloworld" class="com.demo.action.HelloWorldAction">
        
    </action>

</package>
```


### 原理以及文件结构

#### 原理

主要是基于serlvet来了一层过滤器

#### 配置文件

***web.xml***

	用于加载struts2框架，通过一个过滤器，`StrutsPrepareAndExecuteFilter`

***struts.xml***

	构建action映射，这个文件是`核心`

***struts-config.xml***

	链接Web客户端和模型组件

***struts.properties***

	它是一个全局属性文件，通过key-value保存

	可以把放在这个文件里的配置放在`struts.xml`通过`<constant>`元素配置

### 深入

#### 0x00 action搜索顺序

	http://localhost:8080/path1/path2/path3/hello.action

1. 若package存在：如path1/path2/path3
  判断action是否存在，不存在去默认namespace的package里搜索action
2. 若package不存在：如path1/path2/path3
  则去上一级路径的package

#### 0x01 动态调用

	针对一个action内有许多方法，但是我们不能因此写很多action

	方法有三种：`指定method属性`，`感叹号方式`，`通配符方式`


##### 方法二：感叹号方式

***修改struts.xml，添加属性***

```xml
<constant name="struts.enable.DynamicMethodInvocation" value="true"></constant>
```

***修改struts.xml，修改原action，添加`result`标签***

```xml
写在一个action内
<action name="helloaction" class="" >
  <result>/result.jsp</result>
  <result name="方法名A">/methoda.jsp</result>
  <result name="方法名B">/methodb.jsp</result>
</action>
```

***调用方法***

如果这个action的name是helloaction，像第二步那样。
helloaction.action 返回result.jsp
helloaction!方法名A.action 返回methoda.jsp
helloaction!方法名B.action 返回methodb.jsp

##### 方法三：通配符方式

```xml
<action name="helloaction_*" method="{1}" class="" >
  <result>/result.jsp</result>
  <result name="方法名A">/{1}.jsp</result>
  <result name="方法名B">/{1}.jsp</result>
</action>

<!--写在一个action内 -->
<action name="helloaction_*_*" method="{1}{2}" class="" >
  <result>/result.jsp</result>
  <result name="方法名A">/{1}{2}.jsp</result>
  <result name="方法名B">/{1}{2}.jsp</result>
</action>
```


	以此类推，下面有一个小技巧

```xml
<action name="*_*" method="{1}{2}" class="com.demo.{1}Action" >
  <result>/result.jsp</result>
  <result name="方法名A">/{2}.jsp</result>
  <result name="方法名B">/{2}.jsp</result>
</action>
```

	访问方式：localhost:8080/HellowAction_add.action

	解释一下：因为本身存在一个java文件叫HelloAction，所以可以使用这样的方式


#### 0x02 指定多个配置文件

	通过include标签引入多个配置文件，为不同模块创建模块文件

#### 0x03 默认action

	如果用户不小心输入了错误的地址，则会跳转到指定的一个页面

```xml
<default-action-ref name="error"></default-action-ref>

<action name="error">
    <result>/error.jsp</result>
</action>
```

#### 0x04 Struts后缀

```xml
<!--name可以改成你想要的，默认是action，多个可以用英文逗号隔开 --> 
<constant name="struts.action.extension" value="html"></constant>
```

#### 0x05 接受参数

- 方法一：使用action的属性接受参数

	这个方法只要你在指定的action的java文件中将参数作为成员变量声明，然后设置set和get方法就可以了。然后该成员变量就有值了。

- 方法二：使用domain model接受参数

	将表单内的字段抽象到一个对象中，放在model包中。并设置set和get方法

	在action的java文件中声明这个属性，比如 User user，然后通过get方法取值

	修改jsp中的表单中`<input name="">`中name的值，比如user.username
	
	这里注意，user是在变量名，是在action的java文件中的名字，然后后面的username这个名字对应着User类中的成员变量username

- 方法三：使用ModelDriven接受参数

	action的java类要实现`ModelDriven`接口，这是个范型，所以可以把User类放进去。

	在该action的java文件中实例化User类。然后实现`getModel`方法，返回已经实例化的User类

	然后jsp页面中的input标签内的name属性直接对应model类的成员变量名就可以了

#### 0x06 input返回

	表单提交错误则现实错误信息，要配合使用标签，重写valiate()方法

#### 0x07 获得Servlet API

- 方法一：ActionContext
- 方法二：实现***Aware接口
- 方法三：ServletActionContext

#### 0x08 注解

	通过注解可以解放struts.xml，直接在java文件里写注解就可以了。


### 拦截器

#### 使用拦截器方法

	在struts.xml中的相应action内添加标签`<interceptor-ref name=""/>`

#### 默认拦截器

	struts2自带了一些拦截器。

#### 自定义拦截器

	第一步、新建类继承`AbstractInterceptor`即可，然后实现主要方法`intercept`即可。还有初始化方法或者销毁方法

	第二步、在`struts.xml`中的package内，action外注册自定义拦截器。

```xml
<!-- 先注册这个拦截器 -->
<interceptors>
  <interceptor name="" class="" />
</interceptors>
<!-- 再使用这个拦截器 -->
<interceptor-ref name="myinterceptor" />
<!-- 还需要在自定义拦截器后面添加这个才能让自定义的拦截器成功，因为自定义拦截器会导致默认拦截器失效 -->
<interceptor-ref name="defaultStack" />
```

#### 堆叠拦截器

> 第一步 创建堆叠

```xml
<interceptor-stack name="basicStack">
   <interceptor-ref name="exception"/>
   <interceptor-ref name="servlet-config"/>
   <interceptor-ref name="prepare"/>
   <interceptor-ref name="checkbox"/>
   <interceptor-ref name="params"/>
   <interceptor-ref name="conversionError"/>
</interceptor-stack>
```

> 第二步 在action内使用这个堆叠拦截器

```xml
<interceptor-ref name="basicStack"/>
```
