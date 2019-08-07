# 一、Spring Boot简介
**Spring Boot：简化Spring应用开发的一个框架；整个Spring技术栈的一个大整合；J2EE开发的一站式解决方案。**

Spring Boot来简化Spring应用开发，约定大于配置，去繁从简，just run 就能创建一个独立的，产品级别的应用。<br>
背景：<br>
J2EE笨重的开发、繁多的配置、低下的开发效率、复杂的部署流程、第三方技术集成难度大。<br>
解决：<br>
“Spring全家桶”时代<br>
Spring Boot →J2EE一站式解决方案<br>
Spring Cloud→分布式整体解决方案<br>
优点：<br>
* 快速创建独立运行的Spring项目以及主流框架集成
* 使用嵌入式的Servlet容器，应用无需打包成war包
* starters自动依赖与版本控制
* 大量的自动配置，简化开发，也可修改默认值
* 无需配置xml，五代码生成，开箱即用
* 准生产环境的运行时应用监控
* 与云计算的天然集成

# 二、微服务
2014 ，martin fowler<br>
微服务：架构风格（服务微化）<br>
一个应用应用是一组小型服务；可以通过HTTP的方式进行互通；<br>
每一个功能元素最终都是一个可独立替换和独立升级的软件单元；<br>

# 三、环境约束
jdk1.8：Spring Boot1.7及以上<br>
maven3.x：maven3.3 以上版本<br>
IntelliJIDEA2019<br>
SpringBoot 1.5.10.RELEASE<br>

①MAVEN设置：<br>
给maven的settings.xml配置文件的profiles标签添加<br>
```xml
    <profile>
        <id>jdk-1.8</id>
        <activation>
            <activeByDefault>true</activeByDefault>
            <jdk>1.8</jdk>
        </activation>
        <properties>
            <maven.compiler.source>1.8</maven.compiler.source>
            <maven.compiler.target>1.8</maven.compiler.target>
            <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
        </properties>
    </profile>
```
给maven配置本地仓库<br>
```xml
<localRepository>D:/apache-maven-3.0.5-bin/apache-maven-3.0.5/repository</localRepository>
```
②IDEA设置<br>
在File中点击Settings，并在其中设置：<br>

![](https://user-gold-cdn.xitu.io/2019/8/7/16c6c51889e85c67?w=1015&h=716&f=jpeg&s=60353)
# 四、Spring Boot HelloWorld
一个功能：<br>
浏览器发送hello请求，服务器接收请求并处理，响应Hello World字符串；<br>
1、创建一个maven工程（jar）<br>
2、导入依赖spring boot相关的依赖<br>
在pom.xml文件中添加如下代码：<br>
```xml
<!-- Inherit defaults from Spring Boot -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.9.RELEASE</version>
</parent>

<!-- Add typical dependencies for a web application -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```
3、编写一个主程序，启动Spring Boot 应用<br>
```java
package com.atguigu;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;


/*
 * @SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot应用
 */
@SpringBootApplication
public class HelloWorldMainApplication {
    public static void main(String[] args) {
        //Spring 应用启动起来
        SpringApplication.run(HelloWorldMainApplication.class,args);
    }
}
```
4、编写相关的Controller、Service<br>
```java
package com.atguigu.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HelloController {

    @ResponseBody
    @RequestMapping("/hello")
    public String  hello(){
        return "hello world!";
    }
}
```
5、直接运行HelloWorldMainApplication主程序，然后使用浏览器访问localhost:8080/hello即可看到结果。<br>

![](https://user-gold-cdn.xitu.io/2019/8/7/16c6c52b2c974ac4?w=517&h=199&f=jpeg&s=13784)
6、简化部署<br>
在pom.xml文件中添加如下代码：<br>
```xml
<!--这个插件，可以将应用打包成一个可执行的jar包-->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```
将这个应用打包成jar包，在命令行下直接使用java -jar 命令执行jar文件。<br>
![](https://user-gold-cdn.xitu.io/2019/8/7/16c6c532b9a98c94?w=993&h=519&f=jpeg&s=106189)

然后再浏览器中输入localhost:8080/hello即可直接访问。<br>
# 五、Hello World探究
1、POM文件<br>
①父项目<br>
```xml
<!-- Inherit defaults from Spring Boot -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.9.RELEASE</version>
</parent>

//spring-boot-starter-parent它的父项目是：
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.0.9.RELEASE</version>
    <relativePath>../../spring-boot-dependencies</relativePath>
</parent>

//spring-boot-dependencies它来真正管理Spring Boot应用里面的所有依赖版本
```
Spring Boot的版本仲裁中心；<br>
以后我们导入依赖默认是不需要写版本；（没有在dependencies里面管理的依赖自然需要声明版本号）<br>
②启动器<br>
```xml
<!-- Add typical dependencies for a web application -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```
spring-boot-starter-web：<br>
spring-boot-starter：spring boot场景启动器；帮我们导入了web模块正常运行所依赖的组件；<br>

Spring Boot 将所有的功能场景都抽取出来，做成一个个的starters（启动器），只需要在项目里面引入这些starter ，相关场景的所有依赖都会导入进来。要用什么功能就导入什么场景的启动器。<br>
2、主程序类，主入口类<br>
```java
package com.atguigu;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/*
 * @SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot应用
 */
@SpringBootApplication
public class HelloWorldMainApplication {
    public static void main(String[] args) {
        //Spring 应用启动起来
        SpringApplication.run(HelloWorldMainApplication.class,args);
    }
}
```
**@SpringBootApplication**：（Spring Boot应用）使用该注解标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法来启动SpringBoot应用。<br>

@SpringBootApplication注解的内部如下：<br>
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
```
* **@SpringBootConfiguration**：Spring Boot 的配置类<br>
标注在某个类上，表示这是一个Spring Boot的配置类；<br>
@SpringBootConfiguration的内部：<br>
@Configuration：配置类上标注这个注解；<br>
配置类--------配置文件；（@Configuration注解内部有@Component注解）配置类也是容器中的一个组件：@Component<br>

* **@EnableAutoConfiguration**：开启自动配置功能
以前我们需要配置的东西，Spring Boot帮我们自动配置；**@EnableAutoConfiguration**告诉Spring Boot开启自动配置功能；这样自动配置才能生效；<br>

@EnableAutoConfiguration注解内部 部分代码 如下：<br>
```java
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
```
* **@AutoConfigurationPackage**：自动配置包
**@Import**({Registrar.class})：
Spring的底层注解@Import，给容器中导入一个组件；导入的组件由Registrar.class（AutoConfigurationPackages.Registrar）指定。<br>
**将主配置类（@SpringBootApplication标注的类）的所在包及其子包里面的所有组件扫描到Spring容器中。**<br>
* **@Import({AutoConfigurationImportSelector.class})**	
给容器中导入组件？<br>
AutoConfigurationImportSelector：导入哪些组件的选择器；<br>
将所有需要导入的组件以全类名的方式返回；这些组件就会被添加到容器中；<br>
会给容器中导入非常多的自动配置类（xxxAutoConfiguration）；就是给容器中导入这个场景需要的所有组件，并配置好这些组件；<br>

![](https://user-gold-cdn.xitu.io/2019/8/7/16c6c588acc136c4?w=737&h=236&f=jpeg&s=39681)
有了自动配置类，免去了我们手动编写配置注入功能组件等的工作；<br>
`SpringFactoriesLoader.loadFactoryNames(EnableAutoConfiguration.class,classLoader);`

**Spring Boot在启动的时候从类路径下META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置工作。 **     以前我们需要自己配置的东西，自动配置类都帮我们做了；<br>
J2EE的整体整合解决方案和自动配置都在spring-boot-autoconfigure-2.0.9.RELEASE.jar<br>

# 六、使用Spring Initializer快速创建Spring Boot 项目（STS则是Spring Starter Project）
IDE都支持使用Spring的项目创建向导快速创建一个Spring Boot项目；<br>
选择我们需要的模块；向导会联网创建Spring Boot项目；<br>

默认生成的Spring Boot项目：<br>
* 主程序已经生成好了，我们只需要书写我们自己的逻辑
* resource文件夹中目录结构
    * static：保存所有的静态资源：js、css、images（相当于web项目中的WebContent）
    * templates：保存所有的模板页面；（Spring Boot 默认jar包使用嵌入式的Tomcat，默认不支持JSP页面）；可以使用模板引擎（freemarker、thymeleaf）;
    * application.properties：Spring Boot应用的配置文件；可以修改一些默认设置；

