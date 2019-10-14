# 一、JSR107
Java Caching定义了5个核心接口，分别是CachingProvider，CacheManager，Cache，Entry和Expiry。
* CachingProvider定义了创建、配置、获取、管理和控制多个CacheManager。一个应用可以在运行期间访问多个CachingProvider。
* CacheManager定义了创建、配置、获取、管理和控制多个唯一命名的Cache，这些Cache存在于CacheManager的上下文中。一个CacheManager仅被一个CachingProvider所拥有。
* Cache是一个类似于Map的数据结构并临时存储以Key为索引的值。一个Cache仅被一个CacheManager所拥有。
* Entry是一个存储在Cache中的Key-Value对。
* Expiry每一个存储在Cache中的条目有一个定义的有效期。一旦超过这个时间，条目为过期的状态。一旦过期，条目将不可访问、更新和删除。缓存有效期可以通过ExpiryPolicy设置。
  ![](https://user-gold-cdn.xitu.io/2019/9/15/16d348bda9d50431?w=625&h=418&f=jpeg&s=22181)

# 二、Spring缓存抽象
Spring从3.1开始定义了org.springframework.cache.Cache和org.springframework.cache.CacheManager接口来统一不同的缓存技术；并支持使用JCache（JSR-107）注解简化我们的开发；


|                | 几个重要概念&缓存注解                              |
| -------------- | ---------------------------------------- |
| Cache          | 缓存接口，定义缓存操作。实现有：RedisCache、EhCacheCache、ConcurrentMapCache等 |
| CacheManager   | 缓存管理器，管理各种缓存(Cache)组件                    |
| @Cacheable     | 主要针对方法配置，能够根据方法的请求参数对其结果进行缓存             |
| @CacheEvict    | 清空缓存                                     |
| @CachePut      | 保证方法被调用，又希望结果被缓存                         |
| @EnableCaching | 开启基于注解的缓存                                |
| KeyGenerator   | 缓存数据时Key生成策略                             |
| serialize      | 缓存数据时value序列化策略                          |

**SpEL表达式**
![](https://user-gold-cdn.xitu.io/2019/9/15/16d351cde7528fd6?w=880&h=420&f=jpeg&s=39458)


```java
/**
 * 一、搭建基本环境
 * 1、导入数据库文件  创建出department和employee表
 * 2、创建JavaBean封装数据
 * 3、整合MyBatis操作数据库
 *      1）、配置数据源信息
 *      2）、使用注解版的MyBatis;
 *          ①、@MapperScan指定需要扫描的mapper接口所在的包
 *  二、快速体验缓存
 *  步骤：
 *  1、开启基于注解的缓存  @EnableCaching
 *  2、标注缓存注解即可
 *      @Cacheable
 *      @CacheEvict
 *      @CachePut
 *  默认使用的是ConcurrentMapCacheManager==ConcurrentMapCache：将数据保存在ConcurrentMap< Object, Object>
 *  开发中使用缓存中间件：redis、memcache、ehcache；
 *  三、整合redis作为缓存
 *  Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。
 *  1、安装redis：使用docker进行安装；
 *  2、引入redis的starter
 *  3、配置redis
 *  4、测试缓存
 *      原理：CacheManager=====Cache 缓存组件来实际给缓存中存取数据
 *      1）、引入redis的starter，容器中保存的是RedisCacheManager;
 *      2）、RedisCacheManager帮我们创建RedisCache来作为缓存组件；RedisCache通过操作redis缓存数据 的
 *      3）、默认保存数据 k-v 都是Object；利用序列化保存；如何保存为JSON
 *          ①、引入了redis的starter，cacheManager变为RedisCacheManager；
 *          ②、默认创建的RedisCacheManager操作redis的时候使用的是RedisTemplate<Object,Object></>
 *          ③、RedisTemplate<Object,Object></>是默认使用jdk的序列化机制
 *      4）、自定义CacheManager
 *
 */
```

# 三、搭建基本环境

1、JavaBean

1)、Department类
```java
package com.atguigu.cache.bean;

import java.io.Serializable;

public class Department implements Serializable {
	
	private Integer id;
	private String departmentName;
	
	
	public Department() {
		super();
		// TODO Auto-generated constructor stub
	}
	public Department(Integer id, String departmentName) {
		super();
		this.id = id;
		this.departmentName = departmentName;
	}
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public String getDepartmentName() {
		return departmentName;
	}
	public void setDepartmentName(String departmentName) {
		this.departmentName = departmentName;
	}
	@Override
	public String toString() {
		return "Department [id=" + id + ", departmentName=" + departmentName + "]";
	}
}

```
2）、Employee类
```java
package com.atguigu.cache.bean;

import java.io.Serializable;

public class Employee implements Serializable {
	
	private Integer id;
	private String lastName;
	private String email;
	private Integer gender; //性别 1男  0女
	private Integer dId;
	
	
	public Employee() {
		super();
	}

	
	public Employee(Integer id, String lastName, String email, Integer gender, Integer dId) {
		super();
		this.id = id;
		this.lastName = lastName;
		this.email = email;
		this.gender = gender;
		this.dId = dId;
	}
	
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public String getLastName() {
		return lastName;
	}
	public void setLastName(String lastName) {
		this.lastName = lastName;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	public Integer getGender() {
		return gender;
	}
	public void setGender(Integer gender) {
		this.gender = gender;
	}
	public Integer getdId() {
		return dId;
	}
	public void setdId(Integer dId) {
		this.dId = dId;
	}
	@Override
	public String toString() {
		return "Employee [id=" + id + ", lastName=" + lastName + ", email=" + email + ", gender=" + gender + ", dId="
				+ dId + "]";
	}
}

```
2、整合MyBatis操作数据库

1）、配置数据源信息(application.properties)
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/spring_cache?useSSL=false
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
# 开启驼峰命名匹配规则
mybatis.configuration.map-underscore-to-camel-case=true
# 开启指定包的日志
logging.level.com.atguigu.cache.mapper=debug

# 在启动的时候开启自动配置报告
debug=true

```
2）、使用注解版MyBatis

@MapperScan("com.atguigu.cache.mapper")
```java
@MapperScan("com.atguigu.cache.mapper")
@SpringBootApplication
@EnableCaching
public class Springboot01CacheApplication {

    public static void main(String[] args) {
        SpringApplication.run(Springboot01CacheApplication.class, args);
    }

}

```
# 四、快速体验缓存
1、开启基于注解的缓存

@EnableCaching
```java
@MapperScan("com.atguigu.cache.mapper")
@SpringBootApplication
@EnableCaching
public class Springboot01CacheApplication {

    public static void main(String[] args) {
        SpringApplication.run(Springboot01CacheApplication.class, args);
    }

}
```
2、标注缓存注解

1）、EmployeeService
```java
package com.atguigu.cache.service;

import com.atguigu.cache.bean.Employee;
import com.atguigu.cache.mapper.EmployeeMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.*;
import org.springframework.stereotype.Service;

/**
 * @author 梁一亮
 * @date 2019/9/15 -  21:00
 */
@CacheConfig(cacheNames = "emp",cacheManager = "employeeCacheManager")   //抽取缓存的公共配置
@Service
public class EmployeeService {
    @Autowired
    EmployeeMapper employeeMapper;

    /**
     * 将方法的运行结果进行缓存；以后再要相同的数据，直接从缓存中获取，不用在调用方法；
     *
     * CacheManager管理多个Cache组件，对缓存的真正CRUD操作中，每一个缓存组件有自己唯一一个名字；
     * 几个属性：
     *      cacheNames/value：指定缓存组件的名字；将方法的返回结果放在哪个缓存中，是数组的方式，可以指定多个缓存；
     *      key：缓存数据使用的key；可以用它来指定。默认是使用方法参数的值   1-方法的返回值
     *          编写SpEL：  #id:参数id的值   #a0  #p0   #root.args[0]
     *          getEmp[id]  ----->,key = "#root.methodName+'['+#id+']'"
     *
     *      keyGenerator：Key的生成器；可以自己指定Key的生成器的组件id
     *          key/keyGenerator：二选一使用
     *
     *      cacheManager：指定缓存管理器；或者指定cacheResolver缓存解析器
     *
     *      condition：指定符合条件的情况下才缓存；
     *          condition = "#id>0"
     *          ,condition = "#a0>1"：第一个参数的值>1的时候才进行缓存
     *
     *      unless：否定缓存；当unless指定的条件为true，方法的返回值就不会被缓存；可以获取到结果进行判断
     *              unless = "#result == null "
     *              unless = ”#a0 == 2 “ ：如果第一个参数的值是2，结果不缓存
     *       sync：是否使用异步模式
     *          默认是使用同步；若使用异步模式，则不支持unless
     *
     *  原理：
     *      1、自动配置类；CacheAutoConfiguration
     *      2、缓存的配置类
     *      org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration
     *      org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration
     *      org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration
     *      org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration
     *      org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration
     *      org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration
     *      org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration
     *      org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration
     *      org.springframework.boot.autoconfigure.cache.GuavaCacheConfiguration
     *      org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration【默认】
     *      org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration
     *      3、哪个配置类默认生效：SimpleCacheConfiguration
     *      4、给容器中注册了一个CacheManager：ConcurrentMapCacheManager
     *      5、可以获取和创建ConcurrentMapCache类型的缓存组件；它的作用：将数据保存在ConcurrentMap中；
     *          运行流程：
     *          @Cacheable：
     *          1、方法运行之前，先去查询Cache（缓存组件），按照cacheNames指定的名字获取
     *              （CacheManager先获取相应的缓存），第一个获取缓存如果没有Cache组件会自动创建。
     *          2、去Cache中查找缓存的内容，使用一个Key，默认就是方法的参数；
     *              key是按照某种策略生成的；默认是使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key
     *                  SimpleKeyGenerator生成key的默认策略：
     *                      如果没有参数：key=new  SimpleKey();
     *                      如果有一个参数：key=参数的值
     *                      如果有多个参数：key=new SimpleKey(params);
     *          3、没有查到缓存就调用目标方法；
     *          4、将目标方法返回的结果，放进缓存中
     *
     *      @Cacheable 标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值作为key去查询缓存，
     *      如果没有就运行方法将结果放入缓存中。以后再来调用就可以直接使用缓存中的数据。
     *
     *      核心：
     *          1）、使用CacheManager【ConcurrentMapCacheManager】按照名字得到Cache【ConcurrentMapCache】组件；
     *          2）、key使用keyGenerator生成的，默认是SimpleKeyGenerator
     *
     * @param id
     * @return
     */
    @Cacheable(cacheNames ={ "emp"}/*,keyGenerator = "mykeyGenerator",condition = "#a0>1"*/)
    public Employee getEmp(Integer id){
        System.out.println("查询"+id+"号员工");
        Employee emp = employeeMapper.getEmpById(id);
        return emp;
    }

    /**
     * @CachePut：既调用方法，又更新缓存数据；
     *  修改了数据库的某个数据，同时更新缓存；
     *   运行时机：
     *   1、先调用目标方法
     *   2、将目标方法的结果缓存起来
     *
     *   测试步骤：
     *   1、查询一号员工；查到的结果会放在缓存中
     *          key=1  value=   gender=1
     *   2、以后的查询还是之前的结果
     *   3、更新一号员工：【lastName=zhangsan；gender=0】
     *          将方法的返回值也放进缓存了；
     *          Key：传入的Employee对象   值：返回的Employee对象；
     *   4、查询一号员工？
     *      应该是更新后的员工；
     *          将两者的key设置成一样的即可更新
     *          key = "#employee.id"：使用传入的参数的员工id；
     *          key = "#result.id"：使用返回后的id
     *              @Cacheable的key是不能用#result的
     *      为什么是没更新之前的数据？【一号员工没有在缓存中更新】
     */
    @CachePut(value = "emp",key = "#employee.id")
    public Employee updateEmp(Employee  employee){
        System.out.println("updateEmp："+employee);
        employeeMapper.updateEmp(employee);
        return employee;
    }

    /**
     * @CacheEvict：缓存清除
     *  key：指定要清除的数据
     *  allEntries = true：指定清除这个缓存中所有的数据
     *  beforeInvocation =false：缓存的清除是否在方法执行之前执行
     *      默认代表缓存清除操作是在方法执行之后执行；如果出现异常缓存就不会清除
     *
     *   beforeInvocation =true：
     *      代表清除缓存曹组是在方法运行之前执行，无论方法是否出现异常，缓存都清除
     */
    @CacheEvict(value = "emp"/*,key = "#id"*/,allEntries = true)
    public void deleteEmp(Integer id){
        System.out.println("deleteEmp："+id);
//        employeeMapper.deleteEmpById(id);
    }

    // @Caching定义复杂的缓存规则
    @Caching(
            cacheable = {
                    @Cacheable(/*value = "emp",*/key = "#lastName")
            },
            put = {
                    @CachePut(/*value = "emp",*/key = "#result.id"),
                    @CachePut(/*value = "emp",*/key = "#result.email")
            }
    )
    public Employee getEmpByLastName(String lastName){
        return employeeMapper.getEmpByLastName(lastName);
    }
}

```
2）、DeptService
```java
package com.atguigu.cache.service;

import com.atguigu.cache.bean.Department;
import com.atguigu.cache.mapper.DepartmentMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.cache.Cache;
import org.springframework.cache.annotation.CacheConfig;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.stereotype.Service;

/**
 * @author 梁一亮
 * @date 2019/9/26 -  8:37
 */
@CacheConfig(cacheManager = "departmentCacheManager")
@Service
public class DeptService {
    @Autowired
    private DepartmentMapper departmentMapper;

    @Qualifier("departmentCacheManager")
    @Autowired
    private RedisCacheManager departmentCacheManager;

    /**
     * 缓存的数据能够存入Redis;
     * 第二次从缓存中查询就不能反序列化回来；
     * 存的是dept的JSON数据；CacheManager默认使用的是RedisTemplate<Object,Employee></>操作Redis
     * @param id
     * @return
     */
    /*@Cacheable(cacheNames = "dept")
    public Department getDeptById(Integer id){
        System.out.println("查询部门"+id);
        Department department = departmentMapper.getDeptById(id);
        return department;
    }*/

    //使用缓存管理器得到缓存，进行API调用
    public  Department getDeptById(Integer id){
        System.out.println("查询部门"+id);
        Department department = departmentMapper.getDeptById(id);
        //获取某个缓存
        Cache dept = departmentCacheManager.getCache("dept");
        dept.put("dept:1",department);

        return department;
    }
}

```
# 五、整合Redis作为缓存
1、安装redis：使用docker进行安装
```shell
# 拉取redis镜像
docker pull redis 
# 运行redis镜像，生成容器
docker run -d -p 6379:6379 --name myredis redis
```
2、引入redis的starter(pom.xml)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.12.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.atguigu</groupId>
    <artifactId>springboot-01-cache</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-01-cache</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--引入redis的Starter-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.0</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```
3、配置redis(application.properties)
```properties
# 指定redis的主机和端口
spring.redis.host=192.168.224.130
spring.redis.port=6379
```
4、测试缓存
1)、配置CacheManager为RedisCacheManager，并使用JSON序列化机制（MyRedisConfig）
```java
package com.atguigu.cache.config;

import com.atguigu.cache.bean.Department;
import com.atguigu.cache.bean.Employee;
import org.springframework.boot.autoconfigure.cache.CacheManagerCustomizers;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;

/**
 * @author 梁一亮
 * @date 2019/9/25 -  16:58
 */
@Configuration
public class MyRedisConfig {

    @Bean
    public RedisTemplate<Object, Employee> employeeRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws  Exception{
        RedisTemplate<Object, Employee> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Employee> serializer = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
        template.setDefaultSerializer(serializer);
        return template;
    }

    //CacheManagerCustomizers可以来定制缓存的一些规则
    @Primary//将某个缓存管理器作为默认使用
    @Bean
    public RedisCacheManager  employeeCacheManager(RedisTemplate<Object,Employee> employeeRedisTemplate){
        RedisCacheManager cacheManager = new RedisCacheManager(employeeRedisTemplate);
        //key多了一个前缀

        //使用前缀；默认会将CacheName作为Key的前缀；
        cacheManager.setUsePrefix(true);
        return cacheManager;
    }

    @Bean
    public RedisTemplate<Object, Department> departmentRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws  Exception{
        RedisTemplate<Object, Department> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Department> serializer = new Jackson2JsonRedisSerializer<Department>(Department.class);
        template.setDefaultSerializer(serializer);
        return template;
    }

    @Bean
    public RedisCacheManager  departmentCacheManager(RedisTemplate<Object,Department> departmentRedisTemplate){
        RedisCacheManager cacheManager = new RedisCacheManager(departmentRedisTemplate);
        //key多了一个前缀

        //使用前缀；默认会将CacheName作为Key的前缀；
        cacheManager.setUsePrefix(true);
        return cacheManager;
    }
}

```
2）、测试

Mapper:

①EmployeeMapper
```java
package com.atguigu.cache.mapper;

import com.atguigu.cache.bean.Employee;
import org.apache.ibatis.annotations.*;

/**
 * @author 梁一亮
 * @date 2019/9/15 -  20:40
 */
@Mapper
public interface EmployeeMapper {
    @Select("select * from employee where id=#{id}")
    public Employee getEmpById(Integer id);

    @Update("update  employee set  lastName=#{lastName},email=#{email},gender=#{gender},d_id=#{dId}  where id=#{id}")
    public void updateEmp(Employee  employee);

    @Delete("delete from employee where id=#{id}")
    public void deleteEmpById(Integer id);

    @Insert("insert into employee(lastName,email,gender,d_id) values(#{lastName},#{email},#{gender},#{dId})")
    public void insertEmp(Employee  employee);

    @Select("select * from employee where lastName=#{lastName}")
    public Employee getEmpByLastName(String lastName);
}

```
②DepartmentMapper
```java
package com.atguigu.cache.mapper;

import com.atguigu.cache.bean.Department;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;

/**
 * @author 梁一亮
 * @date 2019/9/15 -  20:40
 */
@Mapper
public interface DepartmentMapper {
    @Select("select * from department where id=#{id}")
    public Department getDeptById(Integer id);
}

```

Controller：

①EmployeeController
```java
package com.atguigu.cache.controller;

import com.atguigu.cache.bean.Employee;
import com.atguigu.cache.service.EmployeeService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author 梁一亮
 * @date 2019/9/15 -  21:03
 */
@RestController
public class EmployeeController {

    @Autowired
    EmployeeService employeeService;

    @GetMapping("/emp/{id}")
    public Employee getEmployee(@PathVariable("id")  Integer id){
        Employee employee = employeeService.getEmp(id);
        return employee;
    }


    @GetMapping("/emp")
    public Employee updateEmployee(Employee employee){
        Employee employee1 = employeeService.updateEmp(employee);
        return employee1;
    }

    @GetMapping("/deleteEmp/{id}")
    public String deleteEmp(@PathVariable("id")  Integer id){
        employeeService.deleteEmp(id);
        return "success";
    }

    @GetMapping("/emp/lastname/{lastName}")
    public Employee getEmpByLastName(@PathVariable("lastName")  String lastName){
        return employeeService.getEmpByLastName(lastName);
    }
}

```
②DeptController

```java
package com.atguigu.cache.controller;

import com.atguigu.cache.bean.Department;
import com.atguigu.cache.service.DeptService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author 梁一亮
 * @date 2019/9/26 -  8:53
 */
@RestController
public class DeptController {
    @Autowired
    private DeptService deptService;

    @GetMapping("/dept/{id}")
    public Department getDept(@PathVariable("id") Integer id){
        return deptService.getDeptById(id);
    }
}

```