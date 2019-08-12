# 三、Spring Boot日志
## 1、日志框架
小张：开发了一个大型系统；<br>
1、System.out.println();将关键数据打印在控制台；去掉？写在一个文件？<br>
2、框架来记录系统的一些运行时信息；日志框架；zhanglogging.jar；<br>
3、高大上的几个功能？异步模式？自动归档；zhanglogging-good.jar？<br>
4、将以前的框架卸载下来？换上新的框架，重新修改之前相关的API；zhanglogging-perfect.jar;<br>
5、JDBC-----数据库驱动；<br>
写了一个统一的接口层；日志门面（日志的一个抽象层）；logging-abstract.jar；<br>
给项目中导入具体的日志实现就可以了；我们之前的日志框架都是实现的抽象层；<br>

**市面上的日志框架**<br>
JUL、JCL、jboss-logging、logback、log4j、log4j2、slf4j.......<br>
|日志门面（日志的抽象层） |日志实现|
|-------------------------|--------|
|JCL（Jakarta Commons Logging）、**SLF4J（Simple Logging Facade for Java）**、jboss-logging Log4j | JUL（java.util.logging）、Log4j2 、**Logback**|

左边选一个门面（抽象层）、右边来选一个实现；<br>
日志门面：SLF4j；<br>
日志实现：Logback；<br>

SpringBoot：底层是Spring框架，Spring框架默认是用JCL；<br>
**SpringBoot选用SLF4j和Logback；**
## 2、SLF4j使用
### 1）、如何在系统中使用SLF4j
以后开发的时候，日志记录方法的调用，不应该来直接调用日志的实现类，而是调用日志抽象层里面的方法；<br>
给系统里面导入 slf4j 的 jar和 logback 的实现 jar <br>
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

图示：

![](https://user-gold-cdn.xitu.io/2019/8/12/16c8613784c39509?w=924&h=458&f=png&s=175797)
每一个日志的实现框架都有自己的配置文件。使用slf4j以后，**配置文件还是做成日志实现框架自己本身的配置文件**。
### 2）遗留问题
A系统（slf4j+logback）：Spring（commons-logging）、Hibernate（jboss-logging）、MyBatis、...<br>
统一日志记录，即使是别的框架也和我一起统一使用slf4j进行输出？<br>

![](https://user-gold-cdn.xitu.io/2019/8/12/16c86143a02ac1eb?w=827&h=581&f=jpeg&s=58187)
如何让系统中所有的日志都统一到slf4j：<br>
**1、将系统中其他日志框架先排除出去；** <br>
**2、用中间包来替换原有的日志框架；**<br>
**3、我们导入slf4j其他的实现**<br>
## 3、SpringBoot日志关系
启动器：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

SpringBoot使用它来做日志功能：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```

底层依赖关系：
（高版本）

![](https://user-gold-cdn.xitu.io/2019/8/12/16c861580f241c2a?w=863&h=235&f=png&s=15967)
（低版本：比较契合原理）

![](https://user-gold-cdn.xitu.io/2019/8/12/16c8615bd99e9fac?w=681&h=251&f=jpeg&s=21444)
总结：<br>
①SpringBoot底层也是使用slf4j+logback的方式进行日志记录<br>
②SpringBoot也把其他的日志都替换成了slf4j；<br>
③中间替换包？即log4j-over-slf4j.jar包中的实现类都换成了slf4j的实现类。<br>
```java
@SuppressWarnings("rawtypes")
public abstract class LogFactory {

    static String UNSUPPORTED_OPERATION_IN_JCL_OVER_SLF4J = "http://www.slf4j.org/codes.html#unsupported_operation_in_jcl_over_slf4j";

    static LogFactory logFactory = new SLF4JLogFactory();
```

![](https://user-gold-cdn.xitu.io/2019/8/12/16c861689091209e?w=335&h=168&f=png&s=55273)

4)、如果要引入其他框架，一定要把这个框架的默认日志依赖移除掉。<br>
Spring框架使用的是commons-logging;<br>
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <exclusions>
    <exclusion>
    	<groupId>commons-logging</groupId>
    	<artifactId>commons-logging</artifactId>
    </exclusion>
    </exclusions>
</dependency>
```
**SpringBoot能自动适配所有的日志，而且底层使用slf4j+logback的方式记录日志，引入其他框架的时候，只需要把这个框架依赖的日志框架都排除掉；**

## 4、日志使用
### 1)、默认配置
SpringBoot默认帮我们配置好了日志；<br>

测试类：<br>
```java
package com.atguigu.springboot;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBoot03LoggingApplicationTests {

    //记录器
    Logger logger = LoggerFactory.getLogger(getClass());

    @Test
    public void contextLoads() {
            /*
                日志的级别：
                由低到高： trace<debug<info<warn<error
                可以调整输出的日志级别，日志就只会在这个级别及以后的高级别生效
             */
            logger.trace("这是trace日志");
            logger.debug("这是debug日志");
            //SpringBoot默认给我们设置的日志级别是info级别的；没有指定级别的就用SpringBoot默认规定的级别:root级别=info
            logger.info("这是info日志");
            logger.warn("这是warn日志");
            logger.error("这是error日志");

    }

}
```

日志输出格式：
```xml
    %d 表示日期时间，
    %thread 表示线程名，
    %-5level：级别从左显示5个字符宽度
    %logger{50} 表示logger名字最长50个字符，否则按照句点分割。
    %msg：日志消息，
    %n 是换行符

例如：%d{yyyy-MM-dd   HH:mm:ss.SSS}   [%thread]   %-5level    %logger{50} -   %msg%n
```
SpringBoot修改日志的默认配置：<br>
application.properties：<br>
```properties
# 指定 具体包的日志级别：logging.level；
logging.level.com.atguigu=trace

# 不指定路径在当前项目下生成springboot.log日志
# 可以指定完整的路径：在指定的路径下生成指定文件
# logging.file=springboot.log

# logging.file和logging.path是冲突设置，若两个都设置，则只有logging.file生效
# 在当前磁盘的根目录下创建spring文件夹和里面的log文件夹，使用spring.log作为默认日志文件
logging.path=/spring/log

# 指定在控制台输出的日志的格式
# 格式通常为：%d{yyyy-MM-dd   HH:mm:ss.SSS}   [%thread]   %-5level    %logger{50} -   %msg%n
logging.pattern.console=%d{yyyy-MM-dd}   [%thread]   %-5level    %logger{50} -   %msg%n
# 指定在文件中日志输出的格式
logging.pattern.file=%d{yyyy-MM-dd } ===  [%thread]   %-5level    %logger{50} -   %msg%n
```

|logging.file| logging.path |Example |Description|
|------------|--------------|--------|-----------|
|(none) |(none) | |只在控制台输出|
|指定文件名 |(none)| my.log| 输出日志到my.log文件|
|(none) |指定目录 |/var/log |输出到指定目录的 spring.log 文件中|

### 2）指定配置
给类路径下放上每个日志框架自己的配置文件即可；SpringBoot就不使用他的默认配置了。<br>
放置配置文件的规则如下：<br>
|Logging System| Customization|
|--------------|--------------|
|Logback |logback-spring.xml, logback-spring.groovy, logback.xml or logback.groovy|
|Log4j2 |log4j2-spring.xml or log4j2.xml|
|JDK (Java Util Logging)| logging.properties|
logback.xml：直接就被日志框架识别了；<br>
logback-spring.xml：日志框架就不直接加载日志的配置项，而由SpringBoot解析日志配置。且可以使用SpringProfile高级Profile功能。<br>
```xml
<springProfile name="staging">
	<!-- configuration to be enabled when the "staging" profile is active -->
    可以指定某段配置只在某个环境下生效
</springProfile>
```
否则，如果使用logback.xml作为日志配置文件，还要使用proﬁle功能，会有以下错误：<br>
```xml
no applicable action for [springProfile]
```


如：<br>
```xml
<appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <!--
        日志输出格式：
			%d表示日期时间，
			%thread表示线程名，
			%-5level：级别从左显示5个字符宽度
			%logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
			%msg：日志消息，
			%n是换行符
        -->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <springProfile name="dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ----> [%thread] ---> %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
            <springProfile name="!dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ==== [%thread] ==== %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
        </layout>
    </appender>
```
## 5、切换日志框架<br>
可以按照slf4j的日志适配图，进行相关切换；<br>
slf4j+log4j的方式：<br>
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <exclusion>
      <artifactId>logback-classic</artifactId>
      <groupId>ch.qos.logback</groupId>
    </exclusion>
    <exclusion>
      <artifactId>log4j-over-slf4j</artifactId>
      <groupId>org.slf4j</groupId>
    </exclusion>
  </exclusions>
</dependency>

<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
</dependency>
```
切换为log4j2：<br>
```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>spring-boot-starter-logging</artifactId>
                    <groupId>org.springframework.boot</groupId>
                </exclusion>
            </exclusions>
   </dependency>

<dependency>
	  <groupId>org.springframework.boot</groupId>
	  <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```
