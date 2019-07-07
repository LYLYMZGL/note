# 2 MyBatis-Statement方式的增删改查
MyBatis约定：输入参数parameterType和输出参数resultType在形式上都只能有一个。<br/>
注意：<br/>
①若使用的transactionManager的type值为JDBC，则需要手动提交事务，即sqlSession.commit()。<br/>
②所有的标签`<select>`、`<update>`等，都必须要有sql语句，但是sql参数值是可选的；因此，在测试方法中有两个重载的方法：sqlSession.selectOne(namespace.id,参数值)、sqlSession.selectOne(namespace.id)，来分别匹配有参数值的sql和没有参数值的sql。
 
## 2.1 在SqlMapConfig.xml配置文件中加载Mapper.xml映射文件
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
             <mapper resource="org/pdsu/entity/studentMapper.xml"/>
        </mappers>
</configuration>
```
## 2.2 查询指定学号的学生
### 2.2.1 在Mapper.xml映射文件中书写sql语句
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace:该mapper.xml映射文件的唯一标识 -->
<mapper namespace="org.pdsu.entity.studentMapper">
        <!--
             id:该标签的唯一标识
             parameterType:输入参数的类型
             resultType:返回值类型
         -->
        <select id="queryStudentById" parameterType="int" resultType="org.pdsu.entity.Student">
             select * from student where stuno=#{stuno}
        </select>
</mapper>
```
### 2.2.2 测试
```java
//查询单个学生
       public static void queryStudentByStuno() throws Exception {
              //加载MyBatis配置文件(为了访问数据库)
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //sqlSession -- connection
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              String statement="org.pdsu.entity.studentMapper.queryStudentById";
              //selectOne(namespace.id,sql的参数值)
              Student student = sqlSession.selectOne(statement, 1);
              //打印结果
              System.out.println(student.toString());
              //释放资源
              sqlSession.close();
       }
```
## 2.3 查询所有学生
### 2.3.1 在Mapper.xml映射文件中书写sql语句
```xml
<!--
             若输入参数为基本类型或String,则#{}中可以写任何名称
             若输入参数为pojo类型,则#{}中只能写对象的属性名
            
             若输入参数为多种类型，且多个输入参数皆为pojo里的属性，则输入参数parameterType只需要写一个pojo类型即可
         -->
         <select id="queryAllStudents" resultType="org.pdsu.entity.Student">
            select * from student
         </select>
```
### 2.3.2 测试
```java
//查询全部学生
       public static void queryAllStudents() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              String statement="org.pdsu.entity.studentMapper.queryAllStudents";
              List<Student> list=sqlSession.selectList(statement);
              //打印结果
              for(Student student:list) {
                     System.out.println(student.toString());
              }
              //释放资源
              sqlSession.close();
       }
```
## 2.4 增加学生
### 2.4.1 在Mapper.xml映射文件中书写sql语句
```xml
<!--
             若输入参数为基本类型或String,则#{}中可以写任何名称
             若输入参数为pojo类型,则#{}中只能写对象的属性名
            
             若输入参数为多种类型，且多个输入参数皆为pojo里的属性，则输入参数parameterType只需要写一个pojo类型即可
         -->
         <insert id="addStudent" parameterType="org.pdsu.entity.Student">
            insert into student(stuname,stuage,gradename) values(#{stuName},#{stuAge},#{gradeName})
         </insert>
```
### 2.4.2 测试
```java
//增加学生
       public static void addStudent() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              String statement="org.pdsu.entity.studentMapper.addStudent";
              Student student=new Student("xn",27,"s2");
              sqlSession.insert(statement, student);
              //提交事务
              sqlSession.commit();
              //释放连接
              sqlSession.close();
       }
```
## 2.5 删除指定学号的学生
### 2.5.1 在Mapper.xml映射文件中书写sql语句
```xml
<delete id="deleteStudentByStuno" parameterType="int">
            delete from student where stuno=#{stuno}
         </delete>
``` 
### 2.5.2 测试
```java
//删除指定学号的学生
       public static void deleteStudentByStuno() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              String statement="org.pdsu.entity.studentMapper.deleteStudentByStuno";
              sqlSession.delete(statement, 6);
              //提交事务
              sqlSession.commit();
              //释放资源
              sqlSession.close();
       }
```       
## 2.6 修改指定学号的学生
### 2.6.1 在Mapper.xml映射文件中书写sql语句
```xml
<update id="updateStudentByStuno" parameterType="org.pdsu.entity.Student">
            update student set stuname=#{stuName},stuage=#{stuAge},gradename=#{gradeName} where stuno=#{stuNo}
         </update>
```         
### 2.6.2 测试
```java
//修改指定学号的学生
       public static void updateStudentByStuno() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              String statement="org.pdsu.entity.studentMapper.updateStudentByStuno";
              Student student=new Student(4,"奈何",23,"q5");
              sqlSession.update(statement, student);
              //提交事务
              sqlSession.commit();
              //释放资源
              sqlSession.close();
       }
``` 
