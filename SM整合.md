# Spring和MyBatis整合
## 1、思路：<br>
SqlSessionFactory->SqlSession->Mapper对象->CRUD<br>
可以发现：MyBatis最终是通过SqlSessionFactory来操作数据库。<br>
因此，Spring整合MyBatis其实就是将MyBatis的SqlSessionFactory交给Spring管理。<br>
## 2、步骤：<br>
### 1）、导入jar包<br>
```java
mybatis-spring.jar
spring-tx.jar
spring-jdbc.jar
spring-expression.jar
spring-context-support.jar
spring-core.jar
spring-context.jar
spring-beans.jar
spring-aop.jar
spring-web.jar
commons-logging.jar
commons-dbcp.jar
mysql-connector-java-5.1.7-bin.jar
mybatis.jar
log4j.jar
commons-pool.jar
```
### 2）、创建 类---表
### 3）、书写MyBatis配置文件SqlMapConfig.xml（该步骤可以省略）
### 4）、通过Mapper.xml将类、表建立映射关系
### 5）、配置Spring配置文件（applicationContext.xml）
之前使用MyBatis时，通过SqlMapConfig.xml à(生成) SqlSessionFactory。
现在整合的时候，需要通过Spring管理SqlSessionFactory。因此，产生SqlSessionFactory所需要的数据库信息不在放入SqlMapConfig.xml，而是放入Spring的配置文件中。<br>
### 6）、使用Spring-MyBatis整合产物开发程序
目标：通过Spring产生mybatis最终操作需要的动态mapper对象（StudentMapper对象）<br>
Spring产生动态mapper对象有三种方法：<br>
a、DAO层实现类继承SqlSessionDaoSupport类<br>
SqlSessionDaoSupport类提供了一个属性SqlSession，可以通过SqlSession产生mapper对象，进而实现增删改查。<br>
缺点：需要自己手动书写DAO层实现类<br>
b、省略掉第一种方式的DAO层实现类<br>
直接使用MyBatis提供的Mapper实现类：org.mybatis.spring.mapper.MapperFactoryBean<br>
缺点：每个mapper都需要配置一次<br>
c、批量产生mapper对象<br>
直接使用org.mybatis.spring.mapper.MapperScannerConfigurer类批量产生mapper对象<br>
## 1 增加学生（DAO层实现类继承SqlSessionDaoSupport类）
### 1.1 在mapper.xml文件中书写插入标签
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
 <!--
       namespace的值为StudentMapper接口的全类名，
         且接口应和mapper.xml文件放置在同一个包下，且同名。
  -->
 <mapper namespace="org.pdsu.mapper.StudentMapper">
      <insert id="addStudent" parameterType="org.pdsu.entity.Student">
             insert into student(stuno,stuname,stuage) values(#{stuNo},#{stuName},#{stuAge})
      </insert>
 </mapper>
```
### 1.2 在StudentMapper接口中书写相应方法
```java
package org.pdsu.mapper;
 
import org.pdsu.entity.Student;
 
public interface StudentMapper {
       public void addStudent(Student student);
}
```
### 1.3 在StudentMapper实现类中书写相应代码
```java
package org.pdsu.dao.impl;
 
import org.apache.ibatis.session.SqlSession;
import org.mybatis.spring.support.SqlSessionDaoSupport;
import org.pdsu.entity.Student;
import org.pdsu.mapper.StudentMapper;
 
public class StudentDaoImpl extends SqlSessionDaoSupport implements StudentMapper{
 
       @Override
       public void addStudent(Student student) {
              SqlSession sqlSession = super.getSqlSession();
              StudentMapper studentDao = sqlSession.getMapper(StudentMapper.class);
              studentDao.addStudent(student);
              /*
               * 之前MyBatis增删改需要commit(),是因为事务的提交方式为JDBC方式。
               * 而现在是将事务托管给Spring,Spring会帮我们自动提交,因此无需手动提交。
               * 具体是在DBCP数据库连接池时被自动提交。
               */
       }
      
}
```
### 1.4 在StudentService接口书写相应方法
```java
package org.pdsu.service;
 
import org.pdsu.entity.Student;
 
public interface StudentService {
       public void addStudent(Student student);
}
```
### 1.5 在StudentService接口的实现类中书写相应代码
```java
package org.pdsu.service.impl;
 
import org.pdsu.entity.Student;
import org.pdsu.mapper.StudentMapper;
import org.pdsu.service.StudentService;
 
public class StudentServiceImpl implements StudentService{
       private StudentMapper studentMapper;
      
       public void setStudentMapper(StudentMapper studentMapper) {
              this.studentMapper = studentMapper;
       }
 
       @Override
       public void addStudent(Student student) {
              //调用dao层
              studentMapper.addStudent(student);
       }
 
}
```
### 1.6 书写db.properties文件
```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/spring-mybatis?characterEncoding=utf-8
username=root
password=root
maxIdle=1000
maxActive=500
```
### 1.7 在applicationContext.xml文件中书写相应的代码
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
       <!-- 加载db.properties文件 -->
       <bean id="config" class="org.springframework.beans.factory.config.PreferencesPlaceholderConfigurer">
              <!-- 该属性为数组类型 -->
              <property name="locations">
                     <array>
                            <value>classpath:db.properties</value>
                     </array>
              </property>
       </bean>
       <!-- 配置数据库信息(替代mybatis的配置文件SqlMapConfig.xml) -->
       <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
              <property name="driverClassName" value="${driver}"></property>
              <property name="url" value="${url}"></property>
              <property name="username" value="${username}"></property>
              <property name="password" value="${password}"></property>
              <property name="maxIdle" value="${maxIdle}"></property>
              <property name="maxActive" value="${maxActive}"></property>
       </bean>
      
       <bean id="studentMapper" class="org.pdsu.dao.impl.StudentDaoImpl">
              <!-- dao实现类继承的SqlSessionDaoSupport类中有setSqlSessionFactory()方法 -->
              <!-- 将Spring配置的sqlSessionFactory对象交给mapper(dao) -->
              <property name="sqlSessionFactory" ref="sqlSessionFactory"></property>
       </bean>
      
       <bean id="studentService" class="org.pdsu.service.impl.StudentServiceImpl">
              <property name="studentMapper" ref="studentMapper"></property>
       </bean>
      
       <!-- 在SpringIOC容器中创建MyBatis的核心类SqlSessionFactory -->
       <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
              <property name="dataSource" ref="dataSource"></property>
              <!-- 加载MyBatis配置文件 -->
              <!-- <property name="configLocation" value="classpath:SqlMapConfig.xml"></property> -->
              <!-- 加载mapper.xml路径 -->
              <property name="mapperLocations" value="org/pdsu/mapper/*.xml"></property>
       </bean>
</beans>
```
### 1.8 测试
```java
package org.pdsu.test;
 
import org.pdsu.entity.Student;
import org.pdsu.service.StudentService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
 
public class Test {
       public static void main(String[] args) {
              ApplicationContext applicationContext=new ClassPathXmlApplicationContext("applicationContext.xml");
              StudentService studentService = (StudentService) applicationContext.getBean("studentService");
              Student student=new Student();
              student.setStuNo(1);
              student.setStuName("zs");
              student.setStuAge(23);
              studentService.addStudent(student);
       }
}
```
## 2 增加学生（使用MapperFactoryBean类来替代DAO层实现类）
### 2.1 在mapper.xml文件中书写插入标签
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
 <!--
       namespace的值为StudentMapper接口的全类名，
         且接口应和mapper.xml文件放置在同一个包下，且同名。
  -->
 <mapper namespace="org.pdsu.mapper.StudentMapper">
      <insert id="addStudent" parameterType="org.pdsu.entity.Student">
             insert into student(stuno,stuname,stuage) values(#{stuNo},#{stuName},#{stuAge})
      </insert>
 </mapper>
```
### 2.2 在StudentMapper接口中书写相应方法
```java
package org.pdsu.mapper;
 
import org.pdsu.entity.Student;
 
public interface StudentMapper {
       public void addStudent(Student student);
}
```
### 2.3 在StudentService接口书写相应方法
```java
package org.pdsu.service;
 
import org.pdsu.entity.Student;
 
public interface StudentService {
       public void addStudent(Student student);
}
```
### 2.4 在StudentService接口的实现类中书写相应代码
```java
package org.pdsu.service.impl;
 
import org.pdsu.entity.Student;
import org.pdsu.mapper.StudentMapper;
import org.pdsu.service.StudentService;
 
public class StudentServiceImpl implements StudentService{
       private StudentMapper studentMapper;
      
       public void setStudentMapper(StudentMapper studentMapper) {
              this.studentMapper = studentMapper;
       }
 
       @Override
       public void addStudent(Student student) {
              //调用dao层
              studentMapper.addStudent(student);
       }
 
}
```
### 2.5 书写db.properties文件
```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/spring-mybatis?characterEncoding=utf-8
username=root
password=root
maxIdle=1000
maxActive=500
```
### 2.6 在applicationContext.xml文件中书写相应的代码
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
       <!-- 加载db.properties文件 -->
       <bean id="config" class="org.springframework.beans.factory.config.PreferencesPlaceholderConfigurer">
              <!-- 该属性为数组类型 -->
              <property name="locations">
                     <array>
                            <value>classpath:db.properties</value>
                     </array>
              </property>
       </bean>
       <!-- 配置数据库信息(替代mybatis的配置文件SqlMapConfig.xml) -->
       <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
              <property name="driverClassName" value="${driver}"></property>
              <property name="url" value="${url}"></property>
              <property name="username" value="${username}"></property>
              <property name="password" value="${password}"></property>
              <property name="maxIdle" value="${maxIdle}"></property>
              <property name="maxActive" value="${maxActive}"></property>
       </bean>
      
       <!-- <bean id="studentMapper" class="org.pdsu.dao.impl.StudentDaoImpl">
              dao实现类继承的SqlSessionDaoSupport类中有setSqlSessionFactory()方法
              将Spring配置的sqlSessionFactory对象交给mapper(dao)
              <property name="sqlSessionFactory" ref="sqlSessionFactory"></property>
       </bean> -->
      
       <!--
              第二种方式生成mapper对象
              使用别人已经写好的类（省略掉DAO实现类实际上是因为使用别人写好的类）
        -->
       <bean id="studentMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
              <!-- 指定该类实现的接口 -->
              <property name="mapperInterface" value="org.pdsu.mapper.StudentMapper"></property>
              <!-- dao实现类MapperFactoryBean类中有setSqlSessionFactory()方法
              将Spring配置的sqlSessionFactory对象交给mapper(dao) -->
              <property name="sqlSessionFactory" ref="sqlSessionFactory"></property>
       </bean>
      
       <bean id="studentService" class="org.pdsu.service.impl.StudentServiceImpl">
              <property name="studentMapper" ref="studentMapper"></property>
       </bean>
      
       <!-- 在SpringIOC容器中创建MyBatis的核心类SqlSessionFactory -->
       <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
              <property name="dataSource" ref="dataSource"></property>
              <!-- 加载MyBatis配置文件 -->
              <!-- <property name="configLocation" value="classpath:SqlMapConfig.xml"></property> -->
              <!-- 加载mapper.xml路径 -->
              <property name="mapperLocations" value="org/pdsu/mapper/*.xml"></property>
       </bean>
</beans>
```
### 2.7 测试
```java
package org.pdsu.test;
 
import org.pdsu.entity.Student;
import org.pdsu.service.StudentService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
 
public class Test {
       public static void main(String[] args) {
              ApplicationContext applicationContext=new ClassPathXmlApplicationContext("applicationContext.xml");
              StudentService studentService = (StudentService) applicationContext.getBean("studentService");
              Student student=new Student();
              student.setStuNo(3);
              student.setStuName("ww");
              student.setStuAge(26);
              studentService.addStudent(student);
       }
}
```
## 3 增加学生（使用MapperScannerConfigurer类批量产生mapper对象）
### 3.1 在mapper.xml文件中书写插入标签
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
 <!--
       namespace的值为StudentMapper接口的全类名，
         且接口应和mapper.xml文件放置在同一个包下，且同名。
  -->
 <mapper namespace="org.pdsu.mapper.StudentMapper">
      <insert id="addStudent" parameterType="org.pdsu.entity.Student">
             insert into student(stuno,stuname,stuage) values(#{stuNo},#{stuName},#{stuAge})
      </insert>
 </mapper>
```
### 3.2 在StudentMapper接口中书写相应方法
```java
package org.pdsu.mapper;
 
import org.pdsu.entity.Student;
 
public interface StudentMapper {
       public void addStudent(Student student);
}
```
### 3.3 在StudentService接口书写相应方法
```java
package org.pdsu.service;
 
import org.pdsu.entity.Student;
 
public interface StudentService {
       public void addStudent(Student student);
}
```
### 3.4 在StudentService接口的实现类中书写相应代码
```java
package org.pdsu.service.impl;
 
import org.pdsu.entity.Student;
import org.pdsu.mapper.StudentMapper;
import org.pdsu.service.StudentService;
 
public class StudentServiceImpl implements StudentService{
       private StudentMapper studentMapper;
      
       public void setStudentMapper(StudentMapper studentMapper) {
              this.studentMapper = studentMapper;
       }
 
       @Override
       public void addStudent(Student student) {
              //调用dao层
              studentMapper.addStudent(student);
       }
 
}
```
### 3.5 书写db.properties文件
```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/spring-mybatis?characterEncoding=utf-8
username=root
password=root
```
### 3.6 在applicationContext.xml文件中书写相应的代码
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
       <!-- 加载db.properties文件 -->
       <bean id="config" class="org.springframework.beans.factory.config.PreferencesPlaceholderConfigurer">
              <!-- 该属性为数组类型 -->
              <property name="locations">
                     <array>
                            <value>classpath:db.properties</value>
                     </array>
              </property>
       </bean>
       <!-- 配置数据库信息(替代mybatis的配置文件SqlMapConfig.xml) -->
       <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
              <property name="driverClassName" value="${driver}"></property>
              <property name="url" value="${url}"></property>
              <property name="username" value="${username}"></property>
              <property name="password" value="${password}"></property>
       </bean>
      
      
      
       <!--
              第三种方式生成mapper对象(批量产生mapper对象)
              批量产生mapper对象在SpringIOC中的id值默认就是首字母小写的接口名(首字母小写的接口名=id值)
        -->
        <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
             <!-- dao实现类MapperScannerConfigurer类中有setSqlSessionFactoryBeanName()方法
             （此处属性必须使用sqlSessionFactoryBeanName）
             且sqlSessionFactoryBeanName属性为String类型，因此必须使用value
              将Spring配置的sqlSessionFactory对象交给mapper(dao) -->
              <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
              <!-- 指定批量产生哪个包的mapper对象 -->
              <property name="basePackage" value="org.pdsu.mapper"></property>
        </bean>
      
       <bean id="studentService" class="org.pdsu.service.impl.StudentServiceImpl">
              <!-- ref的值默认与首字母小写的接口名一致 -->
              <property name="studentMapper" ref="studentMapper"></property>
       </bean>
      
       <!-- 在SpringIOC容器中创建MyBatis的核心类SqlSessionFactory -->
       <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
              <property name="dataSource" ref="dataSource"></property>
              <!-- 加载MyBatis配置文件 -->
              <!-- <property name="configLocation" value="classpath:SqlMapConfig.xml"></property> -->
              <!-- 加载mapper.xml路径 -->
              <property name="mapperLocations" value="org/pdsu/mapper/*.xml"></property>
       </bean>
</beans>
```
### 3.7 测试
```java
package org.pdsu.test;
 
import org.pdsu.entity.Student;
import org.pdsu.service.StudentService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
 
public class Test {
       public static void main(String[] args) {
              ApplicationContext applicationContext=new ClassPathXmlApplicationContext("applicationContext.xml");
              StudentService studentService = (StudentService) applicationContext.getBean("studentService");
              Student student=new Student();
              student.setStuNo(4);
              student.setStuName("zl");
              student.setStuAge(27);
              studentService.addStudent(student);
       }
}
``` 
