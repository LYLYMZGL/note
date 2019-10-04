# 1、JDBC
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```
 application.yml配置文件：
```yml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://192.168.224.128:3307/jdbctest
    driver-class-name: com.mysql.jdbc.Driver
```
  效果：
  
默认是用org.apache.tomcat.jdbc.pool.DataSource作为数据源；

数据源的相关配置都在DataSourceProperties里面；

自动配置原理：

org.springframework.boot.autoconfigure.jdbc：

1、参考DataSourceConfiguration，根据配置创建数据源，默认使用Tomcat连接池；可以使用spring.datasource.type可以指定自定义的数据源类型；

2、SpringBoot默认可以支持：
```java
org.apache.tomcat.jdbc.pool.DataSource
com.zaxxer.hikari.HikariDataSource
org.apache.commons.dbcp.BasicDataSource
org.apache.commons.dbcp2.BasicDataSource
```
3、自定义数据源类型
```java
@ConditionalOnMissingBean({DataSource.class})
@ConditionalOnProperty(
    name = {"spring.datasource.type"}
)
static class Generic {
    Generic() {
    }

    @Bean
    public DataSource dataSource(DataSourceProperties properties) {
        //使用DataSourceBuilder创建数据源，利用反射创建响应type的数据源，并且绑定相关属性
        return properties.initializeDataSourceBuilder().build();
    }
}
```
4、**DataSourceInitializer是一个ApplicationListener：**

作用：

1）、runSchemaScripts();运行建表语句；

2）、runDataScripts();运行插入数据的sql语句；

默认只需要将文件命名为：
```java
schema-*.sql、data-*.sql
默认规则：schema.sql、schema-all.sql;
可以使用
schema:
  - classpath:department.sql
  指定位置
```
5、操作数据库：自动配置了JdbcTemplate操作数据

# 2、整合Druid数据源
导入Druid数据源

pom.xml:
```xml
<!--引入druid数据源-->
<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.18</version>
</dependency>
```
application.yml:
```yml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://192.168.224.128:3307/jdbctest
    driver-class-name: com.mysql.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource

      #   数据源其他配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    #   配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500

#    schema:
#      - classpath:department.sql
```
DruidConfig.class：
```java
package com.atguigu.springboot.config;

import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.support.http.StatViewServlet;
import com.alibaba.druid.support.http.WebStatFilter;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;

/**
 * @author 梁一亮
 * @date 2019/9/13 -  21:05
 */
@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid(){
        return new DruidDataSource();
    }

    //配置Druid的监控
    //1、配置一个管理后台的Servlet
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String,String> initParams=new HashMap<>();
        initParams.put("loginUsername","admin");
        initParams.put("loginPassword","123456");
        initParams.put("allow","");//默认就是允许所有访问
        initParams.put("deny","localhost");
        bean.setInitParameters(initParams);
        return bean;
    }
    //2、配置一个web监控的filter
    @Bean
    public FilterRegistrationBean  webStatFilter(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());

        Map<String,String> initParams=new HashMap<>();
        initParams.put("exclusions","*.js,*.css,/druid/*");
        bean.setInitParameters(initParams);

        bean.setUrlPatterns(Arrays.asList("/*"));
        return bean;
    }
}
```
# 3、整合Mybatis
```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.1</version>
</dependency>
```

![](https://user-gold-cdn.xitu.io/2019/10/4/16d95e6ebbd6e532?w=692&h=205&f=png&s=8111)

步骤：

1）、配置数据源相关属性（见上一章节Druid）

2）、给数据库建表

3）、创建JavaBean 

**4）、注解版**
```java
package com.atguigu.springboot.mapper;

import com.atguigu.springboot.bean.Department;
import org.apache.ibatis.annotations.*;

/**
 * @author 梁一亮
 * @date 2019/9/14 -  10:36
 */
//指定这是一个操作数据库的mapper
@Mapper
public interface DepartmentMapper {
    @Select("select * from department where id=#{id}")
    public Department getDeptById(Integer id);

    @Delete("delete from department where id=#{id}")
    public int deleteDeptById(Integer id);
    
    //使用主键，且主键是Department类的id属性（插入完成后，会将主键也一同注入进Department对象中）
    @Options(useGeneratedKeys = true,keyProperty = "id")
    @Insert("insert into department(departmentName) values(#{departmentName})")
    public int insertDept(Department department);

    @Update("update department set departmentName=#{departmentName} where id=#{id}")
    public int updateDept(Department department);
}
```
问题：

当数据库表中列名使用驼峰命名法，但是实体类却并没有使用驼峰命名法，会导致属性无法成功映射。

解决方法：自定义Mybatis的配置规则，即给容器中添加一个ConfigurationCustomizer即可。
```java
package com.atguigu.springboot.config;

import org.mybatis.spring.boot.autoconfigure.ConfigurationCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author 梁一亮
 * @date 2019/9/14 -  11:06
 */
@Configuration
public class MyBatisConfig {
    @Bean
    public ConfigurationCustomizer configurationCustomizer(){
        return new ConfigurationCustomizer(){

            @Override
            public void customize(org.apache.ibatis.session.Configuration configuration) {
                //开启驼峰命名法
                configuration.setMapUnderscoreToCamelCase(true);
            }
        };
    }
}
```
```java
使用MapperScan批量扫描所有的Mapper接口
@MapperScan(value = "com.atguigu.springboot.mapper")
@SpringBootApplication
public class SpringBoot06DataMybatisApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBoot06DataMybatisApplication.class, args);
    }

}
```
5)、配置文件版
```yml
mybatis:
  config-location: classpath:mybatis/mybatis-config.xml 指定全局配置文件的位置
  mapper-locations: classpath:mybatis/mapper/*.xml 指定sql映射文件的位置
```
只要指定了如上配置，剩下的用法与以前的用法一致

更多使用参照
http://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/

# 4、整合SpringData JPA
1）、SpringData简介

![](https://user-gold-cdn.xitu.io/2019/10/4/16d95e867e6fbe50?w=1007&h=546&f=png&s=59902)
2）、整合SpringData JPA

JPA：ORM（Object Relational Mapping）；

①、编写一个实体类（bean）和数据表进行映射，并配置好映射关系；
```java
package com.atguigu.springboot.entity;

import javax.persistence.*;

/**
 * @author 梁一亮
 * @date 2019/9/14 -  17:01
 */
//使用JPA注解配置映射关系
@Entity   //告诉JPA这是一个实体类（和数据表映射的类）
@Table(name = "tb1_user")  //@Table来指定和哪个数据表对应；如果省略默认表名就是user；
public class User {
    //使用快捷键Alt+Insert可以选择生成Getter、Setter等
    @Id  //这是一个主键
    @GeneratedValue(strategy = GenerationType.IDENTITY) //自增主键
    private Integer id;

    @Column(name = "last_name",length = 50)  //这是和数据表对应的一个列
    private String lastName;
    @Column  //省略默认列名就是属性名
    private String email;

    public Integer getId() {
        return id;
    }

    public String getLastName() {
        return lastName;
    }

    public String getEmail() {
        return email;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```
2）、编写一个Dao接口来操作实体类对应的数据表（Repository）
```java
package com.atguigu.springboot.repository;

import com.atguigu.springboot.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

/**
 * @author 梁一亮
 * @date 2019/9/14 -  17:14
 */
//继承JpaRepository<操作的数据类型,主键的类型>来完成对数据库的操作
public interface UserRepository  extends JpaRepository<User,Integer> {

}
```
3）、基本的配置（参照JpaProperties）

application.yml；
```yml
spring:
  datasource:
    url: jdbc:mysql://192.168.224.128:3307/jpatest
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
  jpa:
    hibernate:
#      更新或创建数据表结构（当数据表不存在时创建，当实体类修改后数据表会自动修改）
      ddl-auto: update
#      控制台显示SQL
    show-sql: true
```
