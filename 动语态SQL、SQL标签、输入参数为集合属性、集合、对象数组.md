# 8 动语态SQL、SQL标签、输入参数为集合属性、集合、对象数组
前置知识<br>
1、使用MyBatis实现动态SQL<br>
MyBatis提供了`<if>`、`<where>`、`<foreach>`等标签来实现SQL语句的动态拼接。<br>
注：`<where>`标签会自动处理本标签内部`<if>`里的and（若只有一个或多个条件，则会将第一个 [第一个指的是实际执行的sql语句的第一个`<if>`] `<if>`中的and去掉），但不会处理之后的and。<br>
 
例如：使用`<foreach>`来查询学号为1、2、3的学生的信息<br>
```java
select stuno,stuname from student where stuno in(1,2,3)
``` 
`<foreach>`迭代的类型：数组、对象数组、集合、属性（Grade类中有一个属性为：`List<Integer> ids`）<br>
①属性（Grade类中有一个属性为：`List<Integer> ids`）<br>
![](https://user-gold-cdn.xitu.io/2019/7/14/16bef6eccc354bbc?w=554&h=245&f=png&s=95151)

②数组<br>
无论在编写代码时，传递的是什么名称的数组，在Mapper.xml文件中，必须用array关键字代替该数组。<br>
![](https://user-gold-cdn.xitu.io/2019/7/14/16bef6f55258db7a?w=553&h=202&f=png&s=50651)

③集合<br>
无论在编写代码时，传递的是什么名称的集合，在Mapper.xml文件中，必须用list关键字代替该数组。<br>
![](https://user-gold-cdn.xitu.io/2019/7/14/16bef70035f9ccd5?w=553&h=210&f=png&s=50402)

④对象数组<br>
无论在编写代码时，传递的时什么名称的对象数组，在Mapper.xml文件中，输入参数parameterType必须为Object[]，且在查询标签内也同样使用array关键字来代替对象数组。
注：对象数组中的每一个元素为对象，而实际需要的是对象中的某一个属性，因此在`<foreach>`标签内使用时为#{item名.属性名}<br>
![](https://user-gold-cdn.xitu.io/2019/7/14/16bef7029e7656c1?w=554&h=216&f=png&s=71981)

2、SQL片段<br>
将相似的代码提取出来<br>
 
在Java中使用的是方法；在数据库中使用的是存储过程、存储函数；在MyBatis中使用的是SQL片段。<br>
 
步骤：<br>
a、提取相似代码<br>
```xml
<sql id=”值”>
<!--相似代码-->
</sql>
```
b、引用<br>
```xml
<include refid=”id的值”></include>
```
注：若sql片段所在的位置和引用处不在同一个文件中时，需要在 refid的值中加上sql片段所在位置的namespace，即refid="namespace.id"。<br>
![](https://user-gold-cdn.xitu.io/2019/7/14/16bef70a4ea92518?w=553&h=258&f=png&s=68571)

## 8.1 根据姓名或 姓名和年龄 查询学生（动态SQL）
### 8.1.1 在SqlMapConfig.xml文件中加载Mapper.xml映射文件
```xml
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
</mappers>
```
### 8.1.2 在Mapper.xml文件中书写查询标签
```xml
<!--
             使用sql标签实现动态sql
             根据姓名或 姓名和年龄 查询学生
         -->
        <select id="queryStuByStunameOrStuageWishSQLTag" parameterType="Student" resultType="Student">
             select stuno,stuname,stuage from student where
             <!-- test中放置条件（多个条件，只能使用and） -->
             <if test="stuName!=null and stuName!='' ">
                    stuname=#{stuName}
             </if>
             <if test="stuAge!=null and stuAge!=0">
                    and stuage=#{stuAge}
             </if>
        </select>
```
### 8.1.3 在接口中书写相应方法
```java
//根据姓名或 姓名和年龄 查询学生
       public Student queryStuByStunameOrStuageWishSQLTag(Student stu);
```
### 8.1.4 测试
```java
//根据姓名或 姓名和年龄 查询学生
       public static void queryStuByStunameOrStuageWishSQLTag() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student stu=new Student();
              stu.setStuName("ls");
              stu.setStuAge(25);
              Student student = studentMapper.queryStuByStunameOrStuageWishSQLTag(stu);
              //打印结果
              System.out.println(student);
              //释放资源
              sqlSession.close();
       }
```
## 8.2 根据 姓名或年龄 或者 姓名和年龄 或者无条件 查询学生（动态SQL）
### 8.2.1 在SqlMapConfig.xml文件中加载Mapper.xml映射文件
```xml
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
</mappers>
```
### 8.2.2 在Mapper.xml文件中书写查询标签
```xml
<!--
             使用sql标签实现动态sql
             根据 姓名或年龄 或者 姓名和年龄 或者无条件 查询学生
         -->
        <select id="queryStuByStunameOrStuageWishSQLTag1" parameterType="Student" resultType="Student">
             select stuno,stuname,stuage from student where 1=1
             <!-- test中放置条件（多个条件，只能使用and） -->
             <if test="stuName!=null and stuName!='' ">
                    and stuname=#{stuName}
             </if>
             <if test="stuAge!=null and stuAge!=0">
                    and stuage=#{stuAge}
             </if>
        </select>
```
### 8.2.3 在接口中书写相应方法
```java
//根据 姓名或年龄 或者 姓名和年龄 或者无条件 查询学生
       public List<Student> queryStuByStunameOrStuageWishSQLTag1(Student stu);
```
### 8.2.4 测试
```java
//根据 姓名或年龄 或者 姓名和年龄 或者无条件 查询学生
       public static void queryStuByStunameOrStuageWishSQLTag1() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student stu=new Student();
//            stu.setStuName("ls");
//            stu.setStuAge(25);
              List<Student> list = studentMapper.queryStuByStunameOrStuageWishSQLTag1(stu);
              //打印结果
              System.out.println(list);
              //释放资源
              sqlSession.close();
       }
```
## 8.3 根据 姓名或年龄 或者 姓名和年龄 或者无条件 查询学生（动态SQL）(改进，推荐使用)
### 8.3.1 在SqlMapConfig.xml文件中加载Mapper.xml映射文件
```xml
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
</mappers>
```
### 8.3.2 在Mapper.xml文件中书写查询标签
```xml
<!--
             使用sql标签实现动态sql
             根据 姓名或年龄 或者 姓名和年龄 或者无条件 查询学生（改进）
         -->
        <select id="queryStuByStunameOrStuageWishSQLTag2" parameterType="Student" resultType="Student">
             select stuno,stuname,stuage from student
             <where>
                    <!-- test中放置条件（多个条件，只能使用and） -->
                    <if test="stuName!=null and stuName!='' ">
                           and stuname=#{stuName}
                    </if>
                    <if test="stuAge!=null and stuAge!=0">
                           and stuage=#{stuAge}
                    </if>
             </where>
        </select>
```
### 8.3.3 在接口中书写相应方法
```java
//根据 姓名或年龄 或者 姓名和年龄 或者无条件 查询学生（改进）
       public List<Student> queryStuByStunameOrStuageWishSQLTag2(Student stu);
```
### 8.3.4 测试
```java
//根据 姓名或年龄 或者 姓名和年龄 或者无条件 查询学生（改进）
       public static void queryStuByStunameOrStuageWishSQLTag2() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student stu=new Student();
//            stu.setStuName("ls");
//            stu.setStuAge(25);
              List<Student> list = studentMapper.queryStuByStunameOrStuageWishSQLTag2(stu);
              //打印结果
              System.out.println(list);
              //释放资源
              sqlSession.close();
       }
```
## 8.4 根据学号集合查询学生（动态SQL、<foreach>、学号集合在属性中）
### 8.4.1 书写Grade类
```java
package org.pdsu.entity;
 
import java.util.List;
 
public class Grade {
       //该年级所包含的学生学号
       private List<Integer> stuNos;
 
       public List<Integer> getStuNos() {
              return stuNos;
       }
       public void setStuNos(List<Integer> stuNos) {
              this.stuNos = stuNos;
       }
       @Override
       public String toString() {
              return "Grade [stuNos=" + stuNos + "]";
       }
}
``` 
### 8.4.2 在SqlMapConfig.xml文件中加载Mapper.xml映射文件
```xml
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
</mappers>
```
### 8.4.3 在Mapper.xml文件中书写查询标签
```xml
<!--
             使用sql标签实现动态sql
             根据学号集合来查询学生（学号集合在Grade类的stuNos属性中）
             <foreach>
         -->
        <select id="queryStudentWithNosInGrade" parameterType="Grade" resultType="Student">
             select * from student
             <where>
                    <if test="stuNos!=null and stuNos.size>0">
                           <!--
                                  for(Integer stuNo:stuNos){
                                         #{stuNo}
                                  }

collection:表示要遍历的集合名称
open:代表需要遍历的集合的开始部分(该部分不会被遍历，只会在最后拼接的时候，加在遍历集合的前面)
close:代表遍历集合的结束部分(该部分同样不会被遍历，只会在最后拼接时，加载遍历集合的后面)
item:代表给遍历集合起的别名，在循环体中使用该名称来代表集合中的每一个数据<br>
separator:指定在集合中每一个元素与其他元素的分隔符
                                 
拼接后:select * from student and stuno in(#{stuNo},#{stuNo}...)
                            -->
                           <foreach collection="stuNos" open=" and stuno in(" close=")" item="stuNo" separator=",">
                                  #{stuNo}
                           </foreach>
                    </if>
             </where>
        </select>
```
### 8.4.4 在接口中书写相应方法
```java
//根据学号集合查询学生(学号集合放置在属性中)
       public List<Student> queryStudentWithNosInGrade(Grade grade);
```
### 8.4.5 测试
```java
//根据学号集合查询学生(学号集合放置在属性中)
       public static void queryStudentWithNosInGrade() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Grade grade=new Grade();
              List<Integer> stuNos=new ArrayList<>();
              stuNos.add(1);
              stuNos.add(2);
              stuNos.add(3);
              grade.setStuNos(stuNos);
              List<Student> list = studentMapper.queryStudentWithNosInGrade(grade);
              //打印结果
              System.out.println(list);
              //释放资源
              sqlSession.close();
       }
```
## 8.5 根据学号集合查询学生（动态SQL、<foreach>、学号集合在数组中）
### 8.5.1 在SqlMapConfig.xml文件中加载Mapper.xml映射文件
```xml
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
</mappers>
```
### 8.5.2 在Mapper.xml文件中书写查询标签
```xml
<!--
             使用sql标签实现动态sql
             根据学号集合来查询学生(学号集合在数组中)
             <foreach>
         -->
        <select id="queryStudentWithArray" parameterType="int[]" resultType="Student">
             select * from student
             <where>
                    <!--
                           在MyBatis中使用array关键字来代表简单数组
                     -->
                    <if test="array!=null and array.length>0">
                           <foreach collection="array" open=" and stuno in(" close=")" item="stuNo" separator=",">
                                  #{stuNo}
                           </foreach>
                    </if>
             </where>
        </select>
```
### 8.5.3 在接口中书写相应方法
```java
//根据学号集合查询学生(学号集合放置在数组中)
       public List<Student> queryStudentWithArray(int[] stuNos);
```
### 8.5.4 测试
```java
//根据学号集合查询学生(学号集合放置在数组中)
       public static void queryStudentWithArray() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              int[] stuNos= {1,2,3};
              List<Student> list = studentMapper.queryStudentWithArray(stuNos);
              //打印结果
              System.out.println(list);
              //释放资源
              sqlSession.close();
       }
```
## 8.6 根据学号集合查询学生（动态SQL、<foreach>、学号集合在集合中）
### 8.6.1 在SqlMapConfig.xml文件中加载Mapper.xml映射文件
```xml
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
</mappers>
```
### 8.6.2 在Mapper.xml文件中书写查询标签
```xml
<!--
             使用sql标签实现动态sql
             根据学号集合来查询学生(学号集合在集合中)
             <foreach>
         -->
        <select id="queryStudentWithList" parameterType="list" resultType="Student">
              select * from student
              <where>
                     <!--
                            在MyBatis中使用list关键字来代替集合
                      -->
                      <if test="list!=null and list.size>0">
                           <foreach collection="list" open=" and stuno in(" close=")" item="stuNo" separator=",">
                                  #{stuNo}
                           </foreach>
                      </if>
              </where>          
        </select>
```
### 8.6.3 在接口中书写相应方法
```java
//根据学号集合来查询学生(学号集合在集合中)
       public List<Student> queryStudentWithList(List<Integer> stuNos);
```
### 8.6.4 测试
```java
//根据学号集合来查询学生(学号集合在集合中)
       public static void queryStudentWithList() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              List<Integer> stuNos=new ArrayList<>();
              stuNos.add(1);
              stuNos.add(2);
              stuNos.add(3);
              List<Student> list = studentMapper.queryStudentWithList(stuNos);
              //打印结果
              System.out.println(list);
              //释放资源
              sqlSession.close();
       }
```
## 8.7 根据学号集合查询学生（动态SQL、<foreach>、学号集合在对象数组的每一个元素的学号属性中）
### 8.7.1 在SqlMapConfig.xml文件中加载Mapper.xml映射文件
```xml
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
</mappers>
```
### 8.7.2 在Mapper.xml文件中书写查询标签
```xml
<!--
             使用sql标签实现动态sql
             根据学号集合来查询学生(学号集合在对象数组的每个元素的学号属性中)
             <foreach>
             MyBatis在输入参数中使用Object[]来代表对象数组
         -->
        <select id="queryStudentWithObjectArray" parameterType="Object[]" resultType="Student">
             select * from student
             <where>
                    <!--
                           对象数组中依然使用array关键字来代表数组
                     -->
                     <if test="array!=null and array.length>0">
                          <foreach collection="array" open=" and stuno in (" close=")" item="student" separator=",">
                                 #{student.stuNo}
                          </foreach>
                     </if>
             </where>
        </select>
```
### 8.7.3 在接口中书写相应方法
```java
//根据学号集合来查询学生(学号集合在对象数组的每个元素的学号属性中)
       public List<Student> queryStudentWithObjectArray(Student[] stus);
```
### 8.7.4 测试
```java
//根据学号集合来查询学生(学号集合在对象数组的每个元素的学号属性中)
       public static void queryStudentWithObjectArray() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student stu1=new Student();
              stu1.setStuNo(1);
              Student stu2=new Student();
              stu2.setStuNo(2);
              Student stu3=new Student();
              stu3.setStuNo(3);
              Student[] stus=new Student[] {stu1,stu2,stu3};
              List<Student> list = studentMapper.queryStudentWithObjectArray(stus);
              //打印结果
              System.out.println(list);
              //释放资源
              sqlSession.close();
       }
```
## 8.8 根据学号集合查询学生（SQL片段、动态SQL、<foreach>、学号集合在对象数组的每一个元素的学号属性中）
### 8.8.1 在SqlMapConfig.xml文件中加载Mapper.xml映射文件
```xml
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
</mappers>
```
### 8.8.2 在Mapper.xml文件中书写查询标签和SQL片段
```xml
<!--
             SQL片段
         -->
        <!-- 1、提取相似代码 -->
        <sql id="ObjectArrayStunos">
             <where>
                    <!--
                           对象数组中依然使用array关键字来代表数组
                     -->
                    <if test="array!=null and array.length>0">
                     <foreach collection="array" open=" and stuno in (" close=")" item="student" separator=",">
                          #{student.stuNo}
                     </foreach>
                    </if>
             </where>
        </sql>
        <select id="queryStudentWithObjectArray1" parameterType="Object[]" resultType="Student">
             select * from student
             <!-- 2、引用 -->
             <!-- 若sql片段所在的位置和引用处不在同一个文件中时，需要在 refid的值中加上sql片段所在位置的namespace，
             即refid="namespace.id"-->
             <include refid="ObjectArrayStunos"></include>
        </select>
```
### 8.8.3 在接口中书写相应方法
```java
//SQL片段
       public List<Student> queryStudentWithObjectArray1(Student[] stus);
```
### 8.8.4 测试
```java
//SQL片段
       public static void queryStudentWithObjectArray1() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student stu1=new Student();
              stu1.setStuNo(1);
              Student stu2=new Student();
              stu2.setStuNo(2);
              Student stu3=new Student();
              stu3.setStuNo(3);
              Student[] stus=new Student[] {stu1,stu2,stu3};
              List<Student> list = studentMapper.queryStudentWithObjectArray1(stus);
              //打印结果
              System.out.println(list);
              //释放资源
              sqlSession.close();
       }
```       
