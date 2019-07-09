# 4 属性文件、全局参数、别名、类型转换器、resultMap
## 4.1 属性文件
属性文件的使用：<br/>
可以将数据库配置信息单独放入db.properties文件中，然后再SqlMapConfig.xml配置文件中的`<configuration>`标签里使用`<properties resource="db.properties"/>`标签引入该文件；接着在数据库连接池的属性值的位置使用${key}来获取db.properties文件中对应key的value值，即可完成数据库连接池的配置。<br/>
```properties 
db.properties：
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql:///mybatis?characterEncoding=utf-8
jdbc.username=root
jdbc.password=root
``` 
SqlMapConfig.xml：<br/>
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
 PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
        <!-- 引入db.properties文件 -->
        <properties resource="db.properties"/>
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
                             <property name="driver" value="${jdbc.driver}"/>
                             <property name="url" value="${jdbc.url}"/>
                             <property name="username" value="${jdbc.username}"/>
                             <property name="password" value="${jdbc.password}"/>
                      </dataSource>
               </environment>
        </environments>
        <mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
        </mappers>
</configuration>
``` 
## 4.2 全局参数（初学时通常不要设置）
全局参数的设置：在SqlMapConfig.xml配置文件中的`<configuration>`标签里使用`<settings>`标签设置。<br/>
```xml
<!-- 全局参数的设置 -->
        <settings>
             <setting name="cacheEnabled" value="false"/>
             <setting name="lazyLoadingEnabled" value="false"/>
        </settings>
``` 
## 4.3 别名
### 4.3.1 内置别名

### 4.3.2 自定义别名
```xml
<!-- 设置别名(别名不区分大小写) -->
        <typeAliases>
             <!-- 定义单个别名 -->
             <!-- <typeAlias type="org.pdsu.entity.Student" alias="Student"/> -->
             <!-- 批量定义别名(别名为该类的类名(不带包名，忽略大小写)) -->
             <package name="org.pdsu.entity"/>
        </typeAliases>
``` 
## 4.4 类型处理器（类型转换器）
### 4.4.1 内置常见的类型处理器
| 类型处理器 | Java类型  | JDBC类型 |
|------|------------|------------|
|BooleanTypeHandler | Boolean、boolean |  任何兼容的布尔值 |
|ByteTypeHandler |    Byte、byte     |    任何兼容的数字或字节类型|
|ShortTypeHandler  |     Short、short   |    任何兼容的数字或短整型|
|IntegerTypeHandler |    Integer、int   |    任何兼容的数字和整型|
|LongTypeHandler |    Long、long    |     任何兼容的数字或长整型|
|FloatTypeHandler |      Float、float |   任何兼容的数字或单精度浮点型|
|DoubleTypeHandler  |    Double、double  |   任何兼容的数字或双精度浮点型|
|BigDecimalTypeHandler | BigDecimal  |    任何兼容的数字或十进制小数类型|
|StringTypeHandler    |  String      |    char和varchar类型|
|ClobTypeHandler     | String      |    clob和longvarchar类型|
|NStringTypeHandler   |  String      |    nvarchar和nchar类型|
|NClobTypeHandler    | String      |       nclob类型|
|ByteArrayTypeHandler |  byte[ ]     |    任何兼容的字节流类型|

内置类型处理器有上述处理器等。<br/>
### 4.4.2 自定义类型处理器
具体步骤：<br/>
1、创建自定义类型处理器：需要实现TypeHandler接口（直接实现相对复杂）或继承BaseTypeHandler类。<br/>
2、配置<br/>
①在SqlMapConfig.xml配置文件中配置自定义类型处理器<br/>
②在Mapper.xml配置文件中书写相应的标签（select、insert、update、delete），并书写相应的resultMap标签。<br/>
3、在mapper接口中书写方法<br/>
4、测试<br/>
 
注：<br/>
1、如果类中属性和表中的字段能够合理识别(String-varchar),则可以使用resultType;否则使用resultMap<br/>
2、如果类中属性名和表中的字段名能够合理识别(stuNo-stuno),则可以使用resultType;否则使用resultMap<br/>
 
#### 4.4.2.1 根据学号查询学生
1、创建自定义类型处理器BooleanAndIntConverter类：<br/>
BooleanAndIntConverter类：<br/>
```java
package org.pdsu.converter;
 
import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
 
import org.apache.ibatis.type.BaseTypeHandler;
import org.apache.ibatis.type.JdbcType;
/**
 * 将Java类型中的Boolean类型转换成数据库中的int类型
 * @author admin
 */
//BaseTypeHandler<Boolean>:泛型中写的是要转化的Java类型
public class BooleanAndIntConverter extends BaseTypeHandler<Boolean>{
      
       //Java-->数据库
       /*
        * ps:PreparedStatement对象
        * i:PreparedStatement对象所操作的参数的位置
        * parameter:Java类型的值
        * jdbcType:JDBC操作的数据库类型
        *
        * (non-Javadoc)
        * @see org.apache.ibatis.type.BaseTypeHandler#setNonNullParameter(java.sql.PreparedStatement, int, java.lang.Object, org.apache.ibatis.type.JdbcType)
        */
       @Override
       public void setNonNullParameter(PreparedStatement ps, int i, Boolean parameter, JdbcType jdbcType)
                     throws SQLException {
              if(parameter==true) {
                     ps.setInt(i, 1);//true-->1
              }else {
                     ps.setInt(i, 0);//false-->0
              }
       }
 
       //数据库-->Java
       @Override
       public Boolean getNullableResult(ResultSet rs, String columnName) throws SQLException {
              int sexNum=rs.getInt(columnName);
//            if(sexNum==1) {
//                   return true;
//            }else {
//                   return false;
//            }
              return sexNum==1?true:false;
       }
 
       //数据库-->Java
       @Override
       public Boolean getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
              int sexNum=rs.getInt(columnIndex);
              return sexNum==1?true:false;
       }
 
       //数据库-->Java
       //CallableStatement-->存储过程
       @Override
       public Boolean getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
              int sexNum=cs.getInt(columnIndex);
              return sexNum==1?true:false;
       }
      
}
``` 
2、配置<br/>
①在SqlMapConfig.xml文件中配置类型处理器<br/>
```xml
<!-- 自定义类型处理器 -->
        <typeHandlers>
             <!-- jdbcType的值只能为大写(INTEGER) -->
             <typeHandler handler="org.pdsu.converter.BooleanAndIntConverter" javaType="Boolean" jdbcType="INTEGER"/>
        </typeHandlers>
```        
②在Mapper.xml文件中书写查询标签和resultMap<br/>
```xml
<resultMap type="Student" id="studentResult">
             <!-- 分为主键id和非主键result -->
             <id property="stuNo" column="stuno"/>
             <result property="stuName" column="stuname"/>
             <result property="stuAge" column="stuage"/>
             <result property="gradeName" column="gradename"/>
             <!-- jdbcType的值只能为大写(INTEGER) -->
             <result property="stuSex" column="stusex" javaType="Boolean" jdbcType="INTEGER"/>
        </resultMap>
        <!--
             查询单个学生:使用了类型处理器
             1、如果类中属性和表中的字段能够合理识别(String-varchar),则可以使用resultType;否则使用resultMap
             2、如果类中属性名和表中的字段名能够合理识别(stuNo-stuno),则可以使用resultType;否则使用resultMap
         -->
        <select id="queryStudentByIdWithConverter" parameterType="int"  resultMap="studentResult">
             select * from student where stuno=#{stuno}
        </select>
```
3、在mapper接口中书写方法<br/>
```java
//查询单个学生:使用了类型处理器
       public Student queryStudentByIdWithConverter(int stuno);
```
4、测试<br/>
```java
//查询单个学生:使用了类型处理器
       public static void queryStudentByIdWithConverter() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student student = studentMapper.queryStudentByIdWithConverter(1);
              //打印结果
              System.out.println(student.toString());
              //释放资源
              sqlSession.close();
       }
``` 
#### 4.4.2.2 增加学生：使用了类型处理器
1、创建自定义类型处理器BooleanAndIntConverter类：同上<br/>
2、配置<br/>
①在SqlMapConfig.xml文件中配置类型处理器：同上v
②在Mapper.xml文件中书写插入标签：<br/>
```xml
<insert id="addStudentWithConverter" parameterType="Student">
             insert into student(stuname,stuage,gradename,stusex) values(#{stuName},#{stuAge},#{gradeName},#{stuSex,javaType=Boolean,jdbcType=INTEGER})
        </insert>
```        
3、在mapper接口中书写方法<br/>
```java
//增加学生:使用了类型处理器
       public void addStudentWithConverter(Student student);
```
4、测试<br/>
```java
//增加学生:使用了类型处理器
       public static void addStudentWithConverter() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行Sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student student=new Student("hh",23,"g1");
              student.setStuSex(true);
              studentMapper.addStudentWithConverter(student);
              //提交事务
              sqlSession.commit();
              //释放资源
              sqlSession.close();
       }
``` 
#### 4.4.2.3 根据学号查询学生（属性名和数据库字段名不一致）
1、创建自定义类型处理器BooleanAndIntConverter类：同上<br/>
2、配置<br/>
①在SqlMapConfig.xml文件中配置类型处理器：同上<br/>
②在Mapper.xml文件中书写查询标签和resultMap：(Student2只和Student第一个属性的名称不一致，Student为stuno、Student2为id)<br/>
```xml
<!-- 类中属性名和表中的字段名不一致 -->
        <resultMap type="Student2" id="studentResult1">
             <!-- 分为主键id和非主键result -->
             <id property="id" column="stuno"/>
             <result property="stuName" column="stuname"/>
             <result property="stuAge" column="stuage"/>
             <result property="gradeName" column="gradename"/>
             <!-- jdbcType的值只能为大写(INTEGER) -->
             <result property="stuSex" column="stusex" javaType="Boolean" jdbcType="INTEGER"/>
        </resultMap>
        <select id="queryStudentWithId" parameterType="int" resultMap="studentResult1">
             select * from student where stuno=#{id}
        </select>
```
3、在mapper接口中书写方法<br/>
```java
//查询单个学生:属性名和数据库字段不一致
       public Student2 queryStudentWithId(int id);
```
4、测试<br/>
```java
//查询单个学生:属性名和数据库字段不一致
       public static void queryStudentWithId() throws Exception {
              //加载MyBatis配置文件(为了访问数据库)
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //sqlSession -- connection
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student2 student = studentMapper.queryStudentWithId(1);
              //打印结果
              System.out.println(student.toString());
              //释放资源
              sqlSession.close();
       }
 ```
