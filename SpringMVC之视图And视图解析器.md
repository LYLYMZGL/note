# SpringMVC之视图And视图解析器
视图的顶级接口：View<br/>
视图解析器顶级接口：ViewResolver
 
常见的视图和视图解析器：<br/>
InternalResourceView：该类的子类JstlView可以解析JSTL、实现国际化操作。<br/>
结论：SpringMVC解析jsp时，会默认使用InternalResourceView，如果发现jsp中包含JSTL语言，则自动转为JstlView。
 
什么是国际化？<br/>
针对不同的地区、不同国家，进行不同的显示，让其能够表达相同的意思。
 
具体实现国际化的步骤：<br/>
A．创建资源文件<br/>
格式：<br/>
①基名_语言_地区.properties<br/>
②基名_语言.properties<br/>
通常基名命名为base或i18n<br/>
i18n :代表的是英文单词<br/> internationalization的首末字符i和n，18为中间的字符数，是“国际化”的简称。
 
在i18n.properties文件中书写国际化的格式为Key-Value对，即welcome=欢迎
 
i18n.properties：如果在其他资源文件中没有设置一些属性的值，则在该文件中查找。
在i18n.properties资源文件中加入#XXX内容，其中的XXX不会显示出中文，而是显示中文对应的ASCII码。（转换的原理：是通过jdk的bin目录中的可执行文件native2ascii.exe实现转换的，效果如下图）<br/>
![](https://user-gold-cdn.xitu.io/2019/3/9/1696131674aa7408?w=564&h=241&f=png&s=6164)
例如：<br/>
base_zh_CN.properties<br/>
base_zh.properties<br/>
B． 配置SpringMVC.xml配置文件，加载资源文件<br/>
```java
       <!-- index.jsp中的链接将请求发送到Controller中的测试方法中，Controller响应到success.jsp页面，ResourceBundleMessageSource在响应时介入程序 -->
       <!--
              加载国际化资源文件（当你加上了该配置，SpringMVC就会把基名为i18n的配置文件自动加入进去 ）
              1.将ResourceBundleMessageSource在程序加载时，加入SpringMVC：
                     SpringMVC在启动时，会自动查找一个id="messageSource"的bean，如果有，则自动加载
              2.如果配置了ResourceBundleMessageSource，则该类会在程序响应时介入
       -->
       <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
              <!-- basename代表基名 -->
              <property name="basename" value="i18n"></property>
       </bean>
```
ResourceBundleMessageSource会在SpringMVC响应程序时介入(解析国际化资源文件)，即实现国际化。<br/>
C． 通过JSTL使用国际化<br/>
① 加入jar包<br/>
![](https://user-gold-cdn.xitu.io/2019/3/9/1696136e35c56b58?w=397&h=221&f=png&s=9001)
② 在index.jsp页面中书写链接：<br/>
```java
<a href="SpringMVCHandler/testI18N">testI18N</a>
<br>
```
③ 在Controller中书写测试方法：<br/>
```java
       @RequestMapping(value="testI18N")
       public String testI18N() {
              return "success";
       }
```
④ 在success.jsp中引入JSTL标签库提供的国际化操作，并使用<br/>
```java
<!-- JSTL标签库提供的国际化操作 -->
<%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt" %>
 
<fmt:message key="resource.welcome"></fmt:message>
<fmt:message key="resource.exist"></fmt:message>
<br>
```
InternalResourceViewResolver：通过在springmvc.xml配置文件中加上InternalResourceViewResolver视图解析器，并在其内部加上前缀和后缀的修饰，就可以将控制器Controller中返回的字符串渲染成jsp页面。
