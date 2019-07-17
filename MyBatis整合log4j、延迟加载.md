# 11 MyBatis整合log4j、延迟加载
前置知识<br>
1、MyBatis整合log4j<br>
①导入jar包(在下载MyBatis.zip中的lib中包含此jar包)<br>
log4j.jar<br>
②开启日志，并指定使用的具体日志（若不指定，则MyBatis就会按照以下顺序寻找日志）<br>
SLF4J → Apache Commons Logging → Log4j → JDK logging<br>
在SqlMapConfig.xml配置文件中：
```xml
<settings>
             <!-- 开启日志，并指定使用的具体日志 -->
             <!-- 若不指定，则MyBatis会按照此顺序查找日志 -->
             <!-- SLF4J → Apache Commons Logging → Log4j → JDK logging -->
             <setting name="logImpl" value="LOG4J"/>
</settings>
```
③编写配置日志输出文件log4j.properties<br>
a、内容如下
```properties
log4j.rootLogger=DEBUG, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```
b、具体解释<br>
log4j.rootLogger=DEBUG, stdout<br>
指定日志在哪里输出和输出的级别；<br>
stdout指定日志在控制台输出；<br>
DEBUG指定日志的级别为调试。<br>
 
设置日志级别后，只会打印该级别及该级别以上的信息，该级别以下的信息会被忽略。
日志级别（从低到高）：<br>
DEBUG<INFO<WARN<ERROR（调试<提示<警告<错误）<br>
建议：在开发时设置为debug，在运行时设置为info及以上级别。<br>
 
log4j.appender.stdout=org.apache.log4j.ConsoleAppender<br>
指定显示方式为控制台普通方式
 
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout<br>
指定显示的平铺方式为平铺<br>
平铺方式：平铺、列式
 
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n<br>
指定日志的格式<br>
④观察控制台的日志信息<br>
可以通过日志信息，详细的阅读mybatis执行情况（观察mybatis实际执行的SQL语句，以及SQL中的参数和返回结果）<br>
2、延迟加载（懒加载）<br>
一对一、一对多、多对一、多对多<br>
一对多：<br>
如果不采用延迟加载，即就是立即加载。查询时会将“一”和“多”都查询，即班级和班级中的所有学生。<br>
如果想要暂时只查询“一”的一方，而“多”的一方先不查询，而是在需要的时候再去查询，就称之为延迟加载。<br>
 
使用延迟加载的步骤：（例如：先查询班级，按需查询学生）<br>
1、开启延迟加载（在SqlMapConfig.xml文件中配置settings）<br>
2、配置两个Mapper.xml文件<br>
班级Mapper.xml<br>
学生Mapper.xml<br>
注意：resultMap中的collection标签中的select属性的值为另外一个Mapper.xml文件的namespace+select标签的id值，column属性值指定数据库中的外键。<br>
## 11.1 查询全部学生信息，并延迟加载学生证信息（一对一）
### 11.1.1 在SqlMapConfig.xml文件中开启延迟加载，并加载Mapper.xml映射文件
```xml
<settings>
             <!-- 开启日志，并指定使用的具体日志 -->
             <!-- 若不指定，则MyBatis会按照此顺序查找日志 -->
             <!-- SLF4J → Apache Commons Logging → Log4j → JDK logging -->
             <setting name="logImpl" value="LOG4J"/>
             <!-- 开启延迟加载 -->
             <setting name="lazyLoadingEnabled" value="true"/>
             <!-- 关闭立即加载 -->
             <setting name="aggressiveLazyLoading" value="false"/>
</settings>
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
             <mapper resource="org/pdsu/mapper/studentCardMapper.xml"/>
</mappers>
```
### 11.1.2 在Mapper.xml文件中书写查询标签
①studentMapper.xml文件<br>
```xml
<!-- 通过resultMap实现一对一关联查询和延迟加载 -->
        <select id="queryStudentWithOOLazyLoad" resultMap="student_card_lazyload_map">
             <!-- 先查学生 -->
             select * from student
        </select>
        <resultMap type="Student" id="student_card_lazyload_map">
             <!-- 学生的信息 -->
             <id property="stuNo" column="stuno"/>
             <result property="stuName" column="stuname"/>
             <result property="stuAge" column="stuage"/>
             <!-- 一对一时，对象成员使用association映射,javaType指定该属性的类型 -->
             <!-- 此次采用延迟加载:在查询学生时，并不立即加载学生证信息 -->
             <!-- 学生证,通过select在需要的时候在查学生证 -->
             <!-- 立即加载书写在association标签内部，延迟加载在association标签中书写 -->
              <association property="card" javaType="StudentCard" select="org.pdsu.mapper.StudentCardMapper.queryCardById" column="cardid">
                     <!--
                     <id property="cardId" column="cardid"/>
                     <result property="cardInfo" column="cardinfo"/>
                     -->
              </association>         
        </resultMap>
```
②studentCardMapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace:该mapper.xml映射文件的唯一标识 -->
<mapper namespace="org.pdsu.mapper.StudentCardMapper">
       <!-- 查询学生证信息 -->
       <select id="queryCardById" parameterType="int" resultType="StudentCard">
              <!-- 查询学生对应的学生证 -->
              select * from studentcard where cardid=#{cardId}
       </select>
</mapper>
```
### 11.1.3 在接口中书写相应方法
```java
//一对一查询，使用延迟加载（查询所有学生的信息并延迟加载学生证信息）
       public List<Student> queryStudentWithOOLazyLoad();
```       
### 11.1.4 测试
```java
//一对一查询，使用延迟加载（查询所有学生的信息并延迟加载学生证信息）
       public static void queryStudentWithOOLazyLoad() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              List<Student> list = studentMapper.queryStudentWithOOLazyLoad();
              for(Student stu:list) {
                     System.out.println(stu.getStuNo()+", "+stu.getStuName()+", "+stu.getStuAge());
                     StudentCard card = stu.getCard();
                     System.out.println(card);
              }
              //释放资源
              sqlSession.close();
       }
``` 
通过debug可以发现，如果程序只需要学生信息，则只向数据库发送了查询学生的SQL;当我们后续需要用到学生证信息时，在第二次发送查询学生证的SQL。<br>
## 11.2 查询所有班级的信息，并延迟加载班级中的所有学生信息（一对多）
### 11.2.1 在SqlMapConfig.xml文件中开启延迟加载，并加载Mapper.xml映射文件
```xml
<settings>
             <!-- 开启日志，并指定使用的具体日志 -->
             <!-- 若不指定，则MyBatis会按照此顺序查找日志 -->
             <!-- SLF4J → Apache Commons Logging → Log4j → JDK logging -->
             <setting name="logImpl" value="LOG4J"/>
             <!-- 开启延迟加载 -->
             <setting name="lazyLoadingEnabled" value="true"/>
             <!-- 关闭立即加载 -->
             <setting name="aggressiveLazyLoading" value="false"/>
</settings>
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
             <mapper resource="org/pdsu/mapper/studentCardMapper.xml"/>
<mapper resource="org/pdsu/mapper/studentClassMapper.xml"/>
</mappers>
```
### 11.2.2 在Mapper.xml文件中书写查询标签
①studentClassMapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace:该mapper.xml映射文件的唯一标识 -->
<mapper namespace="org.pdsu.mapper.StudentClassMapper">
       <!-- 一对多关联查询,并使用延迟加载 -->
       <!-- 查询全部班级，并延迟加载其中的学生 -->
        <select id="queryClassAndStudentsLazyLoad" resultMap="class_student_lazyLoad_map">
             <!-- 先查询班级 -->
             select c.* from studentclass c
        </select>
        <!-- 类和表的对应关系 -->
        <resultMap type="StudentClass" id="class_student_lazyLoad_map">
             <!-- 因为type的主类是班级，因此先配置班级的信息 -->
             <id property="classId" column="classid"/>
             <result property="className" column="classname"/>
             <!--
                        配置成员属性:学生
                        若指定该属性的类型，则使用javaType;
                        若指定该属性的元素的类型，则使用ofType
              -->
              <!-- 在查询班级对应的学生 -->
              <collection property="students" ofType="Student" select="org.pdsu.mapper.StudentMapper.queryStudentByClassId" column="classid">
                   <!-- <id property="stuNo" column="stuno"/>
                   <result property="stuName" column="stuname"/>
                   <result property="stuAge" column="stuage"/> -->
              </collection>
        </resultMap>
      
</mapper>
```
②studentMapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace:该mapper.xml映射文件的唯一标识 -->
<mapper namespace="org.pdsu.mapper.StudentMapper">
<!-- 一对多关联查询，延迟加载需要的：查询班级中的所有学生 -->
        <select id="queryStudentByClassId" parameterType="int" resultType="Student">
             select * from student where classid=#{classId}
        </select>
</mapper>
```
### 11.2.3 在接口中书写相应方法
```java
package org.pdsu.mapper;
 
import java.util.List;
 
import org.pdsu.entity.StudentClass;
 
public interface StudentClassMapper {
       /* 要实现接口中的方法和Mapper.xml中的标签一一对应，要满足以下约定：
        * 1.方法名和mapper.xml文件中标签的id值相同
        * 2.方法的输入参数和mapper.xml文件中标签的parameterType类型一致
        * (若mapper.xml中没有parameterType,则说明方法没有输入参数)
        * 3.方法的返回值和mapper.xml文件中标签的resultType类型一致
        * (无论查询结果是一个还是多个(Student/List<Student>),mapper.xml标签中的resultType都只写一个(Student))
        * (若mapper.xml中没有resultType,则方法的返回值为void)
        * 4.namespace的值就是接口的全类名
        */
       //一对多关联查询，并使用延迟加载(查询全部班级并延迟加载班级中的所有学生)
       public List<StudentClass> queryClassAndStudentsLazyLoad();
}
```
### 11.2.4 测试
```java
//一对多关联查询，并使用延迟加载(查询全部班级并延迟加载班级中的所有学生)
       public static void queryClassAndStudentsLazyLoad() throws Exception{
              //加载MyBatis映射文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentClassMapper studentClassMapper=sqlSession.getMapper(StudentClassMapper.class);
              List<StudentClass> list=studentClassMapper.queryClassAndStudentsLazyLoad();
              for(StudentClass stuClass:list) {
                     System.out.println(stuClass.toString());
                     List<Student> stus=stuClass.getStudents();
                     for(Student student:stus) {
                            System.out.println(student.getStuNo()+", "+student.getStuName()+", "+student.getStuAge());
                     }
              }
              //释放资源
              sqlSession.close();
       }
```       
