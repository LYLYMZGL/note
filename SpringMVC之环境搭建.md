# SpringMVC之环境搭建
## 1、加入jar包
com.springsource.org.apache.commons.logging-1.1.1.jar<br/>
spring-aop-4.2.4.RELEASE.jar<br/>
spring-beans-4.2.4.RELEASE.jar<br/>
spring-context-4.2.4.RELEASE.jar<br/>
spring-core-4.2.4.RELEASE.jar<br/>
spring-expression-4.2.4.RELEASE.jar<br/>
spring-web-4.2.4.RELEASE.jar<br/>
spring-webmvc-4.2.4.RELEASE.jar<br/>
## 2、编写SpringMVC配置文件applicationContext.xml
①在applicationContext.xml配置文件中需要书写组件扫描<context:component-scan base-package="使用到注解的包"></context:component-scan>，来使SpringMVC能够解析指定包下的注解。<br/>
②在applicationContext.xml配置文件中需要书写视图解析器InternalResourceViewResolver，通过该视图解析器来为Controller层中返回的结果加上前缀和后缀，以此来生成需要跳转的页面的完整路径。
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation=
       "http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop   http://www.springframework.org/schema/aop/spring-aop.xsd
       http://www.springframework.org/schema/context        http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc   http://www.springframework.org/schema/mvc/spring-mvc.xsd ">
<!-- 有使用注解，必须要将使用注解的所在包放到扫描器里扫描进来，否则springmvc.xml配置文件无法识别 -->
       <!-- 扫描有注解的包 -->
       <context:component-scan base-package="org.liang.handler"></context:component-scan>
      
       <!-- 配置视图解析器（InternalResourceViewResolver） -->
       <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
              <!-- 通过视图解析器为控制器中返回的字符串加上前缀和后缀 -->
              <property name="prefix" value="/views/"></property>
              <property name="suffix" value=".jsp"></property>
       </bean>
</beans>
```
## 3、配置web.xml文件
①在web项目中通过在web.xml配置文件中书写DispatcherServlet来拦截所有请求，将其交给SpringMVC处理。通过这种方式在web项目中引入SpringMVC框架。<br/>
②在配置Servlet的时候，通过在`<init-param>`标签内部指定初始化参数contextConfigLocation的值，来说明SpringMVC的配置文件所在位置。<br/>
③设置让该Servlet在Tomcat服务器启动时，就创建并初始化该Servlet，这样就防止在第一次运行程序时，加载时间过长，进而影响用户体验。

```java
<!-- 配置DispatcherServlet拦截所有请求，并交给springmvc处理 -->
  <servlet>
     <servlet-name>springDispatcherServlet</servlet-name>
     <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
     <init-param>
            <!--contextConfigLocation属性告诉springmvc我的springmvc.xml配置文件放在哪里（配置路径）  -->
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
     </init-param>
     <!-- 启动项目时让下列配置自动生效（启动时以第一个的身份自动启动） -->
     <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
     <servlet-name>springDispatcherServlet</servlet-name>
     <url-pattern>/</url-pattern>
  </servlet-mapping>
```
若不想配置初始化参数，则需要将SpringMVC的配置文件放置到WEB-INF目录下，并更改配置文件名称为"`<servlet-name>`的值-servlet.xml"，使用默认的配置路径。_（并不推荐）_<br/>
**注：在该Servlet中，拦截所有请求使用的是“/”，而不是“/*”，这点需要特别注意。**<br/>
## 4、书写Controller类
①一个普通的类若想实现特定的功能，可以通过：实现接口、继承父类、注解、配置文件等，四种方式来实现。<br/>
一个类若被@Controller修饰时，表示当前类不是一个普通的类，而是一个控制器。<br/>
②下列代码的含义是：当访问一个路径为welcome的链接时，页面会跳转到/views/success.jsp页面。<br/>
```java
package org.liang.handler;
 
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
 
//有使用注解，必须要将使用注解的所在包放到扫描器里扫描进来，否则springmvc.xml配置文件无法识别
//声明了@Controller注解的了类就是一个控制器
@Controller
public class SpringMVCHandler {
      
       //springmvc拦截一个具体的请求使用的是@RequestMapping注解
       @RequestMapping("welcome")
       public String welcome(){
              //视图解析器会将返回的字符串中加入一个前缀和一个后缀，最终的页面为/views/success.jsp
              return "success"; //默认使用了请求转发的跳转方式
       }
}
```
## 5、书写index.jsp和success.jsp
通过在index.jsp页面点击Test Welcome的超链接，能够跳转到success.jsp页面。<br/>
index.jsp
```java
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
	<a href="welcome">Test Welcome</a>
</body>
</html>
```
success.jsp<br/>
```java
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
	<h1>欢迎</h1>
</body>
</html>
```
