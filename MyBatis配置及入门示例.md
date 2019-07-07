# 1 MyBatis配置及入门示例
## 1.1 MyBatis的介绍
MyBatis 的前身是Apache的ibatis，在2010年迁移到了google code，并更名为MyBatis。<br/>
 
MyBatis可以简化JDBC操作，实现数据的持久化。v
MyBatis是ORM的一个实现（Hibernate也是ORM的一个实现），ORM全称为Object Relational Mapping，即对象关系映射。<br/>
例如：有一个Person类和Person表，能够将Person类的对象中属性与Person表中的字段一一对应，即为ORM。<br/>
ORM可以使得开发人员像操作对象一样操作数据库中的表。<br/>
## 1.2 开发MyBatis程序步骤：
### 1.2.1 创建工程，导入jar包
```java
mybatis-3.4.6.jar
mysql-connector-java-5.1.7-bin.jar
```
### 1.2.2 配置MyBatis
①SqlMapConfig.xml：配置数据库信息和需要加载的映射文件<br/>
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
 PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
        <!-- 通过environments的default值和environment的id来指定MyBatis运行时的数据库环境 -->
        <environments default="development">
              <!-- 开发环境 -->
               <environment id="development">
                     <!--
                            事务提交方式:
                            JDBC:利用JDBC方式处理事务，需要手动操作(commit/rollback/close)
                            MANAGED:将事务交由其他组件去托管(spring,jobss),默认会关闭连接     
                            若不想关闭，可以配置。代码如下：
                            <transactionManager type="MANAGED"/>      
                            <property name="closeConnection" value="false"/>
                      -->
                      <transactionManager type="JDBC"/>
                      <!-- 配置数据源 -->
                      <!--
                           数据源类型：
                           UNPOOLED:传统的JDBC模式(每次访问数据库均需要开启、释放连接,不推荐使用)
                           POOLED:使用数据库连接池
                           JNDI:从Tomcat中获取一个内置的数据库连接池
                       -->
                      <dataSource type="POOLED">
                            <!-- 配置数据库信息 -->
                             <property name="driver" value="com.mysql.jdbc.Driver"/>
                             <property name="url" value="jdbc:mysql:///mybatis?characterEncoding=utf-8"/>
                             <property name="username" value="root"/>
                             <property name="password" value="root"/>
                      </dataSource>
               </environment>
        </environments>
        <mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/entity/personMapper.xml"/>
        </mappers>
</configuration>
```
②创建person表和Person类v
person表：<br/>
![](https://user-gold-cdn.xitu.io/2019/7/7/16bcb939babc6c1c?w=554&h=227&f=png&s=43636)
Person类：<br/>
```java
package org.pdsu.entity;
 
public class Person {
       private int id;
       private String name;
       private int age;
       public Person() {
              super();
       }
       public Person(int id, String name, int age) {
              super();
              this.id = id;
              this.name = name;
              this.age = age;
       }
       public int getId() {
              return id;
       }
       public void setId(int id) {
              this.id = id;
       }
       public String getName() {
              return name;
       }
       public void setName(String name) {
              this.name = name;
       }
       public int getAge() {
              return age;
       }
       public void setAge(int age) {
              this.age = age;
       }
       @Override
       public String toString() {
              return "Person [id=" + id + ", name=" + name + ", age=" + age + "]";
       }
      
}
``` 
③personMapper.xml映射文件：书写增删改查标签，并在标签内部书写sql语句<br/>
```xxml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace:该mapper.xml映射文件的唯一标识 -->
<mapper namespace="org.pdsu.entity.personMapper">
        <!--
             id:该标签的唯一标识
             parameterType:输入参数的类型
             resultType:返回值类型
         -->
        <select id="queryPersonById" parameterType="int" resultType="org.pdsu.entity.Person">
             select * from person where id=#{id}
        </select>
</mapper>
```
④测试类<br/>
```java
package org.pdsu.entity;
 
import java.io.IOException;
import java.io.Reader;
 
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
 
public class TestMyBatis {
       public static void main(String[] args) throws Exception {
              //加载MyBatis配置文件(为了访问数据库)
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //sqlSession -- connection
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              String statement="org.pdsu.entity.personMapper.queryPersonById";
              //selectOne(namespace.id,sql的参数值)
              Person person = sqlSession.selectOne(statement, 1);
              //打印结果
              System.out.println(person.toString());
              //释放资源
              sqlSession.close();
       }
}
``` 
