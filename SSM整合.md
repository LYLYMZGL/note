# SSM整合
Spring、SpringMVC、MyBatis
 
SSM整合所有jar包：<br/>
```java
asm-3.3.1.jar
cglib-2.2.2.jar
com.springsource.org.aopalliance-1.0.0.jar
com.springsource.org.aspectj.weaver-1.6.8.RELEASE.jar
commons-dbcp-1.4.jar
commons-logging-1.1.1.jar
commons-pool-1.5.6.jar
javassist-3.17.1-GA.jar
jstl.jar
mybatis-3.2.7.jar
mybatis-spring-1.2.2.jar
mysql-connector-java-5.1.44-bin.jar
spring-aop-4.2.4.RELEASE.jar
spring-aspects-4.2.4.RELEASE.jar
spring-beans-4.2.4.RELEASE.jar
spring-context-4.2.4.RELEASE.jar
spring-core-4.2.4.RELEASE.jar
spring-expression-4.2.4.RELEASE.jar
spring-jdbc-4.2.4.RELEASE.jar
spring-tx-4.2.4.RELEASE.jar
spring-web-4.1.3.RELEASE.jar
spring-webmvc-4.1.3.RELEASE.jar
standard.jar
``` 
1、Spring和MyBatis：需要整合。将MyBatis的SqlSessionFactory交给Spring<br/>
思路：<br/>
SqlSessionFactory -> SqlSession -> StudentMapper -> CRUD<br/>
可以发现，MyBatis最终是通过SqlSessionFactory来操作数据库，Spring整合MyBatis其实就是将MyBatis的SqlSessionFactory交给Spring。<br/>
SM整合步骤：<br/>
1)、导入jar包<br/>
2)、类-表<br/>
在数据库中创建数据表，并在项目中书写相应的实体类。<br/>
Student类——student表<br/>
student表如下：<br/>

![](https://user-gold-cdn.xitu.io/2019/7/6/16bc4c5f03197705?w=554&h=254&f=png&s=65726)
Student类：<br/>
具体代码：<br/>
```java
package org.pdsu.entity;
 
public class Student {
       private int stuNo;
       private String stuName;
       private int stuAge;
       public int getStuNo() {
              return stuNo;
       }
       public void setStuNo(int stuNo) {
              this.stuNo = stuNo;
       }
       public String getStuName() {
              return stuName;
       }
       public void setStuName(String stuName) {
              this.stuName = stuName;
       }
       public int getStuAge() {
              return stuAge;
       }
       public void setStuAge(int stuAge) {
              this.stuAge = stuAge;
       }
       @Override
       public String toString() {
              return "Student [stuNo=" + stuNo + ", stuName=" + stuName + ", stuAge=" + stuAge + "]";
       }
      
}
``` 
3)、MyBatis配置文件conf.xml(数据源、mapper.xml)——可省略，将该文件中的配置全部交由spring管理。<br/>
因此，现在用配置spring的配置文件applicationContext.xml来配置数据源和mapper.xml。<br/>
 
4)、通过mapper.xml将类、表建立映射关系<br/>
具体代码：<br/>
StudentMapper.xml配置文件：<br/>
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace:该mapper.xml映射文件的唯一标识 -->
<mapper namespace="org.pdsu.mapper.StudentMapper">
       <select id="queryStudentByStuno" parameterType="int" resultType="org.pdsu.entity.Student">
              select * from student where stuNo=#{stuNo}
       </select>
      
       <!-- <insert id="addStudent" parameterType="org.pdsu.entity.Student">
              insert into student(stuno,stuname,stuage) values(#{stuNo},#{stuName},#{stuAge})
       </insert> -->
</mapper>
```
StudentMapper.java接口：<br/>
```java
package org.pdsu.mapper;
 
import org.pdsu.entity.Student;
 
public interface StudentMapper {
       public Student queryStudentByStuno(int stuno);
       public void addStudent(Student student);
}
``` 
5)、之前使用MyBatis：conf.xml -> SqlSessionFactory<br/>
现在整合的时候，需要通过Spring管理SqlSessionFactory，因此产生SqlSessionFactory所需要的数据库信息。<br/>
 
在web.xml配置文件中配置，将Spring纳入web项目<br/>
```xml
<!-- Web项目中，引入Spring -->
  <context-param>
     <param-name>contextConfigLocation</param-name>
     <param-value>classpath:applicationContext.xml</param-value>
  </context-param>
  <listener>
     <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
```  
6)、使用Spring整合 MyBatis：将MyBatis的SqlSessionFactory交给Spring<br/>
具体代码：<br/>
db.properties文件：<br/>
```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/college?characterEncoding=utf-8
jdbc.username=root
jdbc.password=root
```
applicationContext.xml配置文件：<br/>
```xml
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
      
      
       <!-- 数据源 -->
       <!-- 加载db.properties文件 -->
       <!-- <bean id="config" class="org.springframework.beans.factory.config.PreferencesPlaceholderConfigurer">
              <property name="locations">
                     <array>
                            <value>classpath:db.properties</value>
                     </array>
              </property>
       </bean> -->
       <!-- 加载db.properties文件 -->
       <context:property-placeholder location="classpath:db.properties"/>          
             
       <!-- 配置数据库信息(替代mybatis的配置文件conf.xml) -->
       <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
              <property name="driverClassName" value="${jdbc.driver}"></property>
              <property name="url" value="${jdbc.url}"></property>
              <property name="username" value="${jdbc.username}"></property>
              <property name="password" value="${jdbc.password}"></property>
       </bean>
      
       <!-- conf.xml:数据源、mapper.xml -->
       <!-- 配置MyBatis需要的核心类:SqlSessionFactory -->
       <!-- 在SpringIOC容器中创建MyBatis的核心类SqlSessionFactory -->
       <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
              <property name="dataSource" ref="dataSource"></property>
              <!-- 加载mapper.xml路径 -->
              <property name="mapperLocations" value="classpath:org/pdsu/mapper/*.xml"></property>
       </bean>
      
       <!-- 将MyBatis的SqlSessionFactory交给Spring -->
       <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
              <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
              <!-- 将org.pdsu.mapper包中所有的接口，产生与之对应的动态代理对象(对象名就是首字母小写的接口名) -->
              <property name="basePackage" value="org.pdsu.mapper"></property>
       </bean>
      
       <!-- 依赖注入：给service注入dao -->
       <bean id="studentService" class="org.pdsu.service.impl.StudentServiceImpl">
              <property name="studentMapper" ref="studentMapper"></property>
       </bean>
      
      
</beans>
```
7)(可选)log4j.properties文件：<br/>
```properties
# Global logging configuration
#\u5728\u5f00\u53d1\u73af\u5883\u4e0b\u65e5\u5fd7\u7ea7\u522b\u8981\u8bbe\u7f6e\u6210DEBUG\uff0c\u751f\u4ea7\u73af\u5883\u8bbe\u7f6e\u6210info\u6216error
log4j.rootLogger=DEBUG, stdout
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
``` 
2、Spring和SpringMVC：就是将Spring、SpringMVC各自配置一遍<br/>
整合SpringMVC：将SpringMVC加入项目即可。<br/>
1)、导入SpringMVC需要的jar包<br/>
2)、将SpringMVC加入项目(给项目加入SpringMVC支持):<br/>
在web.xml文件中配置SpringMVC：<br/>
```xml
<!-- 整合SpringMVC -->
   <servlet>
     <servlet-name>springDispatcherServlet</servlet-name>
     <!-- 拦截所有请求交给SpringMVC(该servlet就是SpringMVC的入口) -->
     <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
     <!-- 配置初始化参数，告诉SpringMVC，配置文件(springmvc.xml)在哪里 -->
     <!-- 若不是默认位置，则需要加上该配置 -->
     <init-param>
            <!-- 通过这两个配置，告诉SpringMVC的配置文件在哪里 -->
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:applicationContext-controller.xml</param-value>
     </init-param>
     <!-- 配置初始化参数，让该servlet的配置在启动Tomcat时自动生效 -->
     <!-- 启动时，以第一个的身份自动启动 -->
     <load-on-startup>1</load-on-startup>
  </servlet>
 
  <servlet-mapping>
     <servlet-name>springDispatcherServlet</servlet-name>
     <!-- 拦截所有请求 -->
     <url-pattern>/</url-pattern>
  </servlet-mapping>
```  
3)、编写SpringMVC配置文件：<br/>
applicationContext-controller.xml配置文件：<br/>
```xml
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
      
       <!-- 若在控制器中使用了注解，则该部分代码可以省略 -->
       <!-- 给Controller注入service -->
       <!-- <bean id="studentController" class="org.pdsu.controller.StudentController">
              <property name="studentService" ref="studentService"></property>
       </bean> -->
      
       <!-- 配置视图解析器 -->
       <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
              <property name="prefix" value="/views/"></property>
              <property name="suffix" value=".jsp"></property>
       </bean>
      
       <!-- SpringMVC的基础配置、标配 -->
       <mvc:annotation-driven></mvc:annotation-driven>
      
       <!-- 将控制器所在包加入IOC容器中 -->
       <context:component-scan base-package="org.pdsu.controller"></context:component-scan>
</beans>
```
3、实例：<br/>
前提：<br/>
①书写service层的接口StudentService.java：<br/>
```java
package org.pdsu.service;
 
import org.pdsu.entity.Student;
 
public interface StudentService {
       public Student queryStudentByStuno(int stuno);
}
```
②书写StudentService接口的实现类StudentServiceImpl：<br/>
```java
package org.pdsu.service.impl;
 
import org.pdsu.entity.Student;
import org.pdsu.mapper.StudentMapper;
import org.pdsu.service.StudentService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
 
public class StudentServiceImpl implements StudentService {
       //service依赖于dao(mapper)
       private StudentMapper studentMapper;
      
       public void setStudentMapper(StudentMapper studentMapper) {
              this.studentMapper = studentMapper;
       }
 
       @Override
       public Student queryStudentByStuno(int stuno) {
              Student s=studentMapper.queryStudentByStuno(stuno);
              return s;
       }
 
}
```
③在index.jsp页面中书写超链接：<br/>
```jsp
<a href="Controller/queryStudentByStuno/1">查询1号学生</a>
<br>
```
④在控制器StudentController.java文件中书写相应代码：<br/>
```java
package org.pdsu.controller;
 
import java.util.Map;
 
import org.pdsu.entity.Student;
import org.pdsu.service.StudentService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
 
@Controller
@RequestMapping("Controller")
public class StudentController {
       /*
        * 控制器依赖于Service
        * 使用注解实现依赖注入
        */
       @Autowired//也可以在applicationContext-controller.xml配置文件中依赖注入
//     @Qualifier("studentService")
       private StudentService studentService;
      
       public void setStudentService(StudentService studentService) {
              this.studentService = studentService;
       }
 
       @RequestMapping("queryStudentByStuno/{stuno}")
       public String queryStudentByStuno(@PathVariable("stuno") Integer stuNo,Map<String,Object> map) {
              Student s=studentService.queryStudentByStuno(stuNo);
              map.put("stu", s);
              return "result";
       }
}
```
⑤在result.jsp页面中书写相应代码：<br/>
```jsp
${requestScope.stu }
``` 
整合期间遇到的问题：<br/>
①在Spring的配置文件中加载mapper.xml文件，运行时居然找不到该文件。<br/>
解决方案：在mapperLocations属性的值前加上classpath:得以解决。<br/>
②在为Controller层中进行属性注入时，由于Spring配置文件中已经将Controller类文件配到了SpringIOC容器中，然而通过注解@Controller又将Controller类配到了SpringIOC容器中，导致报错。<br/>
解决方案：在Controller中通过@Controller注解将Controller配置到SpringIOC容器中，通过@Autowired或@Qualifier("studentService")来实现属性注入。<br/>
③在运行整个项目时，一直报出无法获取JDBC连接的错误。<br/>
解决方案：
在db.properties文件中给每一个key的前面都加上jdbc. ，并且在applicationContext.xml文件中配置数据库信息时，在每一个属性的value值中的${}里的每一个字符串前也都相应的加上jdbc. 即可。<br/>
④在运行整个项目时，一直报出Error creating document instance.的错误。<br/>
解决方案：在StudentMapper.xml文件的头部加上
```xml
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
``` 
约束即可。<br/>
 
