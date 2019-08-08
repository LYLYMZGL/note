# 一、配置文件
Spring Boot 使用一个全局的配置文件，配置文件名是固定的；
* application.properties
* application.yml

配置文件放在==src/main/resource==目录或者==类路径/config==下

配置文件的作用：修改SpringBoot自动配置的默认值；SpringBoot在底层将所有的东西都给我们自动配置好了；

YAML（YAML Ain't Markup Language）这是一种递归写法：（不管是还是不是都与标记语言脱不了关系）<br>
    YAML A Markup Language：是一个标记语言；<br>
    YAML isn't Markup Language：不是一个标记语言；<br>
标记语言：<br>
以前的配置文件：大多使用的是xxx.xml文件<br>
**.yml** 是YAML（YAML Ain't Markup Language）语言的文件，**以数据为中心**，比json、xml等更适合做配置文件
* http://www.yaml.org/ 参考语法规范

全局配置文件可以对一些默认配置值进行修改  <br>

YAML：配置实例
```yml
server:
  port: 8081
```
  XML：
```xml
<server>
    <port>8081</port>
</server>
```
# 二、YAML语法
## 1、YAML基本语法<br>
k:(空格)v：表示一对键值对（空格必须有）<br>
以**空格**的缩进来控制层级关系；只要是左对齐的一列数据，都是同一个层级的
```yml
server:
    port: 8081
    path: /hello
```
属性和值也是大小写敏感；<br>

使用缩进表示层级关系<br>
缩进时不允许使用Tab键，只允许使用空格<br>
缩进的空格数目不重要，只要相同层级的元素左侧对齐即可<br>
大小写敏感<br>
## 2、值的写法
**字面量;普通的值（数字、字符串、布尔）：** <br>
- k: v：字面量直接来写；
    -  字符串默认不用加上单引号或者双引号；
        - ""：双引号；不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思
            - name: "zhangsan  \n  lisi"；输出：zhangsan  换行  lisi
        - ''：单引号；会转义特殊字符，特殊字符最终只是一个普通的字符串数据
            - name: 'zhangsan  \n  lisi'；输出：zhangsan  \n  lisi

**对象、Map（属性和值）（键值对）：** <br>
-    k: v：在下一行来写对象的属性和值的关系；注意缩进
     - 对象还是k: v的方式
```yml
friends: 
    lastName: zhangsan
    age: 20
```
行内写法
```yml
friends: {lastName: zhangsan,age: 18}
```
**数组（List、Set）：** <br>
用 **- 值**表示数组中的一个元素
```yml
pets: 
 - cat
 - dog
 - pig 
```
行内写法：
```yml
pets: [cat,dog,pig]
```
# 三、配置文件值注入
配置文件（application.yml）
```yml
person:
  lastName: zhangsan
  age: 20
  boss: false
  birth: 2019/5/20
  maps: {k1: v1,k2: v2}
  lists:
    - lisi
    - wangwu
    - zhaoliu
  dog:
    name: 小狗
    age: 2
```
JavaBean
```java
package com.atguigu.springboot.bean;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.Date;
import java.util.List;
import java.util.Map;

/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中
 *   @ConfigurationProperties： 告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
 *      prefix="person"  ：配置文件中哪个下面的所有属性进行一一映射
 *
 *      只有这个组件是容器中的组件，才能使用容器提供的功能@ConfigurationProperties；
 */
/* 若不使用@Component注解，只有@ConfigurationProperties注解时，会报下面的错误:
Not registered via @EnableConfigurationProperties or marked as Spring component
 */
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date  birth;

    private Map<String,Object> maps;
    private List<Object>  lists;
    private Dog dog;

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Boolean getBoss() {
        return boss;
    }

    public void setBoss(Boolean boss) {
        this.boss = boss;
    }

    public Date getBirth() {
        return birth;
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }

    public Map<String, Object> getMaps() {
        return maps;
    }

    public void setMaps(Map<String, Object> maps) {
        this.maps = maps;
    }

    public List<Object> getLists() {
        return lists;
    }

    public void setLists(List<Object> lists) {
        this.lists = lists;
    }

    public Dog getDog() {
        return dog;
    }

    public void setDog(Dog dog) {
        this.dog = dog;
    }

    @Override
    public String toString() {
        return "Person{" +
                "lastName='" + lastName + '\'' +
                ", age=" + age +
                ", boss=" + boss +
                ", birth=" + birth +
                ", maps=" + maps +
                ", lists=" + lists +
                ", dog=" + dog +
                '}';
    }
}
```
可以在pom.xml中导入配置文件处理器，以后编写配置就有提示了：
```xml
<!--导入配置文件处理器。配置文件进行绑定就会有提示-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```
## 1、properties配置文件在idea中默认UTF-8可能会乱码
在设置中将下列选项勾选上<br>
![设置properties文件编码](https://user-gold-cdn.xitu.io/2019/8/8/16c71608cd0dd388?w=1028&h=719&f=jpeg&s=60797)
若勾选过后运行还是中文乱码，则需要将properties文件中的中文使用在线编码转换，将中文转换成ASCII码，在properties文件中使用ASCII码才能使其运行时正常，不会乱码。
## 2、@Value获取值和@ConfigurationProperties获取值比较

 空 | @ConfigurationProperties | @Value
---|---|---
功能 | 批量注入配置文件中的属性 | 一个一个指定
松散绑定（松散语法）（例：last-name 与 lastName相同） | 支持 | 不支持
SpEL | 不支持 | 支持
JSR303数据校验 | 支持 | 不支持
复杂类型封装 | 支持 | 不支持

配置文件yml还是properties它们都能获取到值；<br>
如果说，我们只是在某个业务逻辑中需要获取以下配置文件中的某项值，就使用@Value；<br>
如果说，我们专门编写了一个JavaBean来和配置文件进行映射，就使用@ConfigurationProperties；<br>
 
## 3、配置文件注入值数据校验
```java
package com.atguigu.springboot.bean;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;
import org.springframework.validation.annotation.Validated;

import javax.validation.constraints.Email;
import java.util.Date;
import java.util.List;
import java.util.Map;


/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中
 *   @ ConfigurationProperties： 告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
 *      prefix="person"  ：配置文件中哪个下面的所有属性进行一一映射
 *
 *      只有这个组件是容器中的组件，才能使用容器提供的功能@ConfigurationProperties；
 */
/* 若不使用@Component注解，只有@ConfigurationProperties注解时，会报下面的错误:
Not registered via @EnableConfigurationProperties or marked as Spring component
 */
@Component
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {
    /**
     *  <bean class="Person">
     *         <properties name="lastName"   value="字面量/ ${key}从环境变量、配置文件中获取值/#{SpEL} "></properties>
     *  </bean>
     */

    //lastName必须是邮箱格式
    @Email
//    @Value("${person.last-name}")
    private String lastName;
//    @Value("#{11*2}")
    private Integer age;
//    @Value("true")
    private Boolean boss;
    private Date  birth;

    private Map<String,Object> maps;
    private List<Object>  lists;
    private Dog dog;

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Boolean getBoss() {
        return boss;
    }

    public void setBoss(Boolean boss) {
        this.boss = boss;
    }

    public Date getBirth() {
        return birth;
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }

    public Map<String, Object> getMaps() {
        return maps;
    }

    public void setMaps(Map<String, Object> maps) {
        this.maps = maps;
    }

    public List<Object> getLists() {
        return lists;
    }

    public void setLists(List<Object> lists) {
        this.lists = lists;
    }

    public Dog getDog() {
        return dog;
    }

    public void setDog(Dog dog) {
        this.dog = dog;
    }

    @Override
    public String toString() {
        return "Person{" +
                "lastName='" + lastName + '\'' +
                ", age=" + age +
                ", boss=" + boss +
                ", birth=" + birth +
                ", maps=" + maps +
                ", lists=" + lists +
                ", dog=" + dog +
                '}';
    }
}
```
## 4、@PropertySource & @ImportResource
@PropertySource 加载指定的配置文件；
```java
package com.atguigu.springboot.bean;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;
import org.springframework.validation.annotation.Validated;

import javax.validation.constraints.Email;
import java.util.Date;
import java.util.List;
import java.util.Map;


/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中
 *   @ ConfigurationProperties： 告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
 *      prefix="person"  ：配置文件中哪个下面的所有属性进行一一映射
 *
 *      只有这个组件是容器中的组件，才能使用容器提供的功能@ConfigurationProperties；
 *      @ ConfigurationProperties(prefix = "person") 默认从全局配置文件中获取值;
 *      使用@ PropertySource(value = {"classpath:person.properties"})注解，可以加载自定义的properties文件
 */
/* 若不使用@Component注解，只有@ConfigurationProperties注解时，会报下面的错误:
Not registered via @EnableConfigurationProperties or marked as Spring component
 */
@PropertySource(value = {"classpath:person.properties"})
@Component
@ConfigurationProperties(prefix = "person")
//@Validated
public class Person {
    /**
     *  <bean class="Person">
     *         <properties name="lastName"   value="字面量/ ${key}从环境变量、配置文件中获取值/#{SpEL} "></properties>
     *  </bean>
     */

    //lastName必须是邮箱格式
//    @Email
//    @Value("${person.last-name}")
    private String lastName;
//    @Value("#{11*2}")
    private Integer age;
//    @Value("true")
    private Boolean boss;
    private Date  birth;

    private Map<String,Object> maps;
    private List<Object>  lists;
    private Dog dog;

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Boolean getBoss() {
        return boss;
    }

    public void setBoss(Boolean boss) {
        this.boss = boss;
    }

    public Date getBirth() {
        return birth;
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }

    public Map<String, Object> getMaps() {
        return maps;
    }

    public void setMaps(Map<String, Object> maps) {
        this.maps = maps;
    }

    public List<Object> getLists() {
        return lists;
    }

    public void setLists(List<Object> lists) {
        this.lists = lists;
    }

    public Dog getDog() {
        return dog;
    }

    public void setDog(Dog dog) {
        this.dog = dog;
    }

    @Override
    public String toString() {
        return "Person{" +
                "lastName='" + lastName + '\'' +
                ", age=" + age +
                ", boss=" + boss +
                ", birth=" + birth +
                ", maps=" + maps +
                ", lists=" + lists +
                ", dog=" + dog +
                '}';
    }
}
```
@ImportResource：导入Spring的配置文件，让配置文件中的内容生效；<br>
Spring Boot里面没有Spring的配置文件，我们自己编写的配置文件，也不能自动识别；<br>
想让Spring的配置文件生效，加载进来；@ImportResource标注在一个配置类上；<br>
```java
package com.atguigu.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ImportResource;

/**
 *  @ImportResource(locations = {"classpath:beans.xml"}) :导入Spring的配置文件，让其生效
 */
@ImportResource(locations = {"classpath:beans.xml"})
@SpringBootApplication
public class SpringBoot02ConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBoot02ConfigApplication.class, args);
    }

}
```
不在使用编写Spring配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="helloService" class="com.atguigu.springboot.service.HelloService"></bean>
</beans>
```

SpringBoot推荐给容器中添加组件的方式；推荐使用全注解方式<br>
1、配置类    =======   Spring配置文件<br>
2、使用@Bean给容器中添加组件
```java
package com.atguigu.springboot.config;

import com.atguigu.springboot.service.HelloService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;


/**
 *  @Configuration：指明当前类是一个配置类；就是来替代之前的Spring配置文件
 *  在配置文件中用 <bean></bean>标签添加组件
 *
 */
@Configuration
public class MyAppConfig {

    //将方法的返回值添加到容器中；容器中这个组件默认的id就是方法名
    @Bean
    public HelloService helloService(){
        System.out.println("配置类@Bean给容器中添加组件了");
        return new HelloService();
    }
}
```
# 四、配置文件占位符
## 1、随机数
```java
${random.value}  
${random.int}
${random.long}
${random.int(10)}
${random.int[1024,65536]}
```
## 2、占位符获取之前配置的值，如果没有可以使用:指定默认值
```properties
person.last-name=zhangsan${random.uuid}
person.age=${random.int}
person.birth=2019/3/21
person.boss=false
person.maps.k1=v1
person.maps.k2=20
person.lists=a,b,c,d
person.dog.name=${person.hello:hello}_dog
person.dog.age=2
```
# 五、Profile
## 1、多Profile文件
我们在主配置文件编写的时候，文件名可以是 application-{profile}.properties/yml<br>
默认使用application.properties的配置；
## 2、yml支持多文档块方式
```yml
server:
  port: 8081
spring:
  profiles:
    active: prod
---
server:
  port: 8083
spring:
  profiles: dev
---
server:
  port: 8084
spring:
  profiles: prod    # 指定属于哪种环境
```
## 3、激活指定profile
- 在默认配置文件（application.properties）中指定 spring.profiles.active=dev 
- 命令行：
    - --spring.profiles.active=dev  (不打包的方式)

![](https://user-gold-cdn.xitu.io/2019/8/8/16c7163dc1f9c5ec?w=465&h=137&f=jpeg&s=14615)

![](https://user-gold-cdn.xitu.io/2019/8/8/16c7164326c7148c?w=1092&h=693&f=jpeg&s=59708)
    - java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev  （打包的方式）
- 虚拟机参数：
    - -Dspring.profiles.active=dev

![](https://user-gold-cdn.xitu.io/2019/8/8/16c7163f65291bf1?w=465&h=137&f=jpeg&s=14615)

![](https://user-gold-cdn.xitu.io/2019/8/8/16c716456e068a45?w=1092&h=693&f=jpeg&s=61311)

# 六、配置文件加载位置
- spring boot启动会扫描以下位置的application.properties或者application.yml文件作为spring boot的默认配置文件；
    - file：./config/
    - file：./
    - classpath：/config
    - classpath：/
    - （file目录表示的是Project目录；classpath目录只有是source root和resource root两种文件夹下的的文件才能被称为classpath）
    - 以上是按照==优先级从高到低==的顺序，所有的文件都会被加载，**高优先级配置**内容会**覆盖低优先级配置**内容。（可以互补配置）
   
    
**注意**：要配置项目的访问路径时，1.x和2.x版本的配置方法不一致<br>
1.x版配置如下：
```properties
# 配置项目的访问路径
server.context-path=/boot02
```
2.x版配置如下：
```properties
# 配置项目的访问路径
server.servlet.context-path=/boot02
```
- ==我们也可以通过配置spring.config.location来改变默认配置== <br>
- 在2.x版本中，若要保证配置的项目访问路径也生效，需要使用
```properties 
spring.config.additional-location=配置文件路径
```

**项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置；指定配置文件和默认加载的这些配置文件共同起作用形成互补配置；**<br>

代码如下：
```java
java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --spring.config.additional-location=配置文件路径
```

# 七、外部配置加载顺序
Spring Boot 支持多种外部配置方式<br>
这些方式的优先级如下：
[Spring 2.1.5 官方文档的外部配置内容](https://docs.spring.io/spring-boot/docs/2.1.5.RELEASE/reference/htmlsingle/#boot-features-external-config)

==**SpringBoot也可以从以下位置加载配置；优先级从高到低；高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置：**==

1. **命令行参数**
```java
java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --server.port=8087 --server.servlet.context-path=/abc
//多个配置用空格分开；--配置项=值
```
2. 来自java:comp/env的JNDI属性
3. Java系统属性（System.getProperties()）
4. 操作系统环境变量
5. RandomValuePropertySource配置的random.*属性值 <br>
<br>
==**由jar包外向jar包内进行寻找；<br>
优先加载带profile**==
6. **jar包外部的application-{profile}.properties或application.yml（带spring.profile）配置文件**
7. **jar包内部的application-{profile}.properties或application.yml（带spring.profile）配置文件**<br>
<br>
==**再来加载不带profile的**==
8. **jar包外部的application.properties或application.yml（不带spring.profile）配置文件**
9. **jar包内部的application.properties或application.yml（不带spring.profile）配置文件**<br>
<br>
10. @Configuration注解类上的@PropertySource
11. 通过SpringApplication.setDefaultProperties指定的默认属性<br>
所有支持的配置加载来源参考：[官方文档](https://docs.spring.io/spring-boot/docs/2.1.5.RELEASE/reference/htmlsingle/#boot-features-external-config)

# 八、自动配置原理
配置文件到底能写什么？怎么写？自动配置原理；<br>
[配置文件能配置的属性参照](https://docs.spring.io/spring-boot/docs/2.1.5.RELEASE/reference/htmlsingle/#common-application-properties)

## 1、自动配置原理：<br>
1）、SpringBoot启动的时候加载主配置类，开启了自动配置功能 ==@EnableAutoConfiguration== <br>
2）、@EnableAutoConfiguration 作用：<br>
- 利用AutoConfigurationImportSelector给容器中导入一些组件？
- 可以查看selectImports()方法的内容；
```java
//进入getAutoConfigurationEntry()方法内部
AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(
				autoConfigurationMetadata, annotationMetadata);
//内部
List<String> configurations = getCandidateConfigurations(annotationMetadata,
				attributes);//获取候选的配置
```

```java
//getCandidateConfigurations()方法内部
SpringFactoriesLoader.loadFactoryNames()
扫描所有jar包路径下 META-INF/spring.factories
把扫描到的这些文件的内容包装成properties对象
从properties中获取到EnableAutoConfiguration.class类（类名）对应的值，然后把他们添加在容器中
```
==将类路径下的 META-INF/spring.factories 里面配置的所有EnableAutoConfiguration的值加入到了容器中；==

```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.cloud.CloudServiceConnectorsAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jdbc.JdbcRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
org.springframework.boot.autoconfigure.elasticsearch.jest.JestAutoConfiguration,\
org.springframework.boot.autoconfigure.elasticsearch.rest.RestClientAutoConfiguration,\
org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration,\
org.springframework.boot.autoconfigure.http.codec.CodecsAutoConfiguration,\
org.springframework.boot.autoconfigure.influx.InfluxDbAutoConfiguration,\
org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration,\
org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
org.springframework.boot.autoconfigure.quartz.QuartzAutoConfiguration,\
org.springframework.boot.autoconfigure.reactor.core.ReactorCoreAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityRequestMatcherProviderAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.servlet.OAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.reactive.ReactiveOAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.resource.servlet.OAuth2ResourceServerAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.resource.reactive.ReactiveOAuth2ResourceServerAutoConfiguration,\
org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration,\
org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.function.client.ClientHttpConnectorAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplateAutoConfiguration
```
每一个这样的xxxAutoConfiguration类都是容器中的一个组件，都加入到容器中；用他们来做自动配置；<br>
3）、每一个自动配置类进行自动配置功能；<br>
4）、以**HttpEncodingAutoConfiguration（http编码自动配置）**为例解释自动配置原理
```java
@Configuration //表示这是一个配置类，和以前编写的配置文件一样，也可以给容器中添加组件
@EnableConfigurationProperties(HttpProperties.class) //启用指定类的ConfigurationProperties功能；将配置文件中对应的值和HttpProperties绑定起来；并把HttpProperties加入到IOC容器中
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET) //Spring底层的@Conditional注解，根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效；  判断当前应用是否是web应用，如果是，当前配置类生效
@ConditionalOnClass(CharacterEncodingFilter.class) //判断当前项目有没有CharacterEncodingFilter（SpringMVC中进行乱码解决的过滤器）这个类
@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled",
		matchIfMissing = true) //判断配置文件中是否存在某个配置spring.http.encoding.enabled;如果不存在，判断也是成立的
		//即使我们配置文件中不配置spring.http.encoding.enabled=true，也是默认生效的
public class HttpEncodingAutoConfiguration {
    
    //它已经和SpringBoot的配置文件映射了
    private final HttpProperties.Encoding properties;

    //只有一个有参构造器的情况下，参数的值就会从容器中拿
    public HttpEncodingAutoConfiguration(HttpProperties properties) {
		this.properties = properties.getEncoding();
	}

    @Bean //给容器中添加一个组件，这个组件的某些值需要从properties中获取
	@ConditionalOnMissingBean
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}
```
根据当前不同的条件判断，决定这个配置类是否生效？<br>

一旦这个配置类生效；这个配置类就会给容器中添加各种组件；这些组件的属性是从对应的properties类中获取的，这些类里面的每一个属性又是和配置文件绑定的<br>
5）、所有在配置文件中能配置的属性都是在xxxProperties类中封装着；
```java
@ConfigurationProperties(prefix = "spring.http") //从配置文件中获取指定的值和bean的属性进行绑定
public class HttpProperties {
    public static class Encoding {

		public static final Charset DEFAULT_CHARSET = StandardCharsets.UTF_8;
```

精髓：<br>
1）、SpringBoot启动会加载大量的自动配置类<br>
2）、我们看我们需要的功能有没有SpringBoot默认写好的自动配置类；<br>
3）、我们再来看这个自动配置类中到底配置了哪些组件；（只要我们要用的组件有，我们就不需要再来配置了）<br>
4）、给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们就可以在配置文件中指定这些属性的值；

## 2、细节
1、@Conditional派生注解（Spring注解版原生的@Conditional作用）<br>
作用：必须是@Conditional指定的条件成立，才给容器中添加组件，配置类里面的所有内容才生效；

@Conditional扩展注解 | 作用（判断是否满足当前指定条件）
---|---
@ConditionalOnJava | 系统的Java版本是否符合要求
@ConditionalOnBean | 容器中存在指定的Bean
@ConditionalOnMissingBean | 容器中不存在指定的Bean
@ConditionalOnExpression  | 满足SpEL表达式指定
@ConditionalOnClass | 系统中有指定的类
@ConditionalOnMissingClass | 系统中没有指定的类
@ConditionalOnSingleCandidate | 容器中只有一个指定的Bean，或者这个Bean是首选Bean
@ConditionalOnProperty | 系统中指定的属性是否有指定的值
@ConditionalOnResource | 类路径下是否存在指定资源文件
@ConditionalOnWebApplication | 当前是Web环境
@ConditionalOnNotWebApplication | 当前不是Web环境
@ConditionalOnJndi | JNDI存在指定项

**自动配置类必须在一定条件下才能生效**<br>
我们怎么知道哪些自动配置类生效；<br>
我们可以通过启用debug=true属性；来让控制台打印自动配置报告，这样我们就可以很方便的知道哪些自动配置类生效；
```java
============================
CONDITIONS EVALUATION REPORT
============================


Positive matches:(自动配置类中启用的)
-----------------

   CodecsAutoConfiguration matched:
      - @ConditionalOnClass found required class 'org.springframework.http.codec.CodecConfigurer' (OnClassCondition)
      
   CodecsAutoConfiguration.JacksonCodecConfiguration matched:
      - @ConditionalOnClass found required class 'com.fasterxml.jackson.databind.ObjectMapper' (OnClassCondition)
   
   .......
   
Negative matches:(没有启用、没有匹配成功的自动配置类)
-----------------

   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'javax.jms.ConnectionFactory' (OnClassCondition)

   AopAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'org.aspectj.lang.annotation.Aspect' (OnClassCondition)

```



