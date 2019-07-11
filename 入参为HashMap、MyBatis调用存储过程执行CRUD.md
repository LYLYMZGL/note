# 6 入参为HashMap、MyBatis调用存储过程执行CRUD
前置知识<br/>
1、若输入参数为多个，有两种解决方法：<br/>
①使用对象类型做输入参数（如：Student等）。<br/>
②使用HashMap作为输入参数。<br/>
原理：用map中的key匹配sql语句中的#{key}占位符，如果匹配成功，就用map中key对应的value值替换占位符。<br/>
 
2、MyBatis调用存储过程<br/>
MyBatis调用存储过程，实现查询(statementType="CALLABLE"设置SQL的执行方式是存储过程)<br/>
存储过程的输入参数，用HashMap来传递->  map.put("输入参数名",输入参数的值);<br/>
存储过程的输出参数，使用map.get("输出参数名")来获取<br/>
注：在MyBatis中调用存储过程的语法（仅限该存储过程）：<br/>
```xml
{
  CALL 存储过程名(#{输入参数名,jdbcType=输入参数的类型,mode=IN},#{输出参数名,jdbcType=输出参数的类型,mode=OUT })
}
```
只要在SqlMapConfig.xml中配置了`<transactionManager type="JDBC"/>` ，则增删改都需要手动提交事务（`sqlSession.commit();`）
 
## 6.1 根据年龄或姓名查询学生（输入参数为HashMap）
### 6.1.1 在SqlMapConfig.xml文件中加载Mapper.xml，并设置别名
```xml
<!-- 设置别名(别名不区分大小写) -->
        <typeAliases>
             <!-- 定义单个别名 -->
             <!-- <typeAlias type="org.pdsu.entity.Student" alias="Student"/> -->
             <!-- 批量定义别名(别名为该类的类名(不带包名，忽略大小写)) -->
             <package name="org.pdsu.entity"/>
        </typeAliases>
 
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
</mappers>
```
### 6.1.2 在Mapper.xml文件中书写查询标签
```xml
<!-- 使用HashMap -->
        <select id="queryStudentsStuageOrStunameWithHashMap" parameterType="HashMap" resultType="Student">
             select stuno,stuname,stuage,gradename from student where stuage=${stuAge} or stuname like '%${stuName}%'
        </select>
```        
### 6.1.3 在接口中书写相应方法
```java
//使用HashMap
public List<Student> queryStudentsStuageOrStunameWithHashMap(Map<String,Object> map);
```
### 6.1.4 测试
```java
//根据年龄和姓名查询学生（使用HashMap）
       public static void queryStudentsStuageOrStunameWithHashMap() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Map<String,Object> map=new HashMap<>();
              map.put("stuAge", 23);
              map.put("stuName", "zs");
              List<Student> list = studentMapper.queryStudentsStuageOrStunameWithHashMap(map);
              //打印结果
              for(Student stu:list) {
                     System.out.println(stu.toString());
              }
              //释放资源
              sqlSession.close();
       }
```
## 6.2 根据存储过程查询某个年级的学生总数
### 6.2.1 在数据库中创建存储过程
```java
create procedure queryCountByGradeWithProcedure(in gname varchar(20),out scount int)
begin
       select count(*) into scount from student where gradename=gname;
end;
```
### 6.2.2 在SqlMapConfig.xml文件中加载Mapper.xml，并设置别名
```xml
<!-- 设置别名(别名不区分大小写) -->
        <typeAliases>
             <!-- 定义单个别名 -->
             <!-- <typeAlias type="org.pdsu.entity.Student" alias="Student"/> -->
             <!-- 批量定义别名(别名为该类的类名(不带包名，忽略大小写)) -->
             <package name="org.pdsu.entity"/>
        </typeAliases>
 
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
</mappers>
```
### 6.2.3 在Mapper.xml文件中书写存储过程
```xml
<!--
             根据存储过程查询某个年级的学生总数
             MyBatis调用存储过程，实现查询(statementType="CALLABLE"设置SQL的执行方式是存储过程)
             存储过程的输入参数，用HashMap来传递->  map.put("输入参数名",输入参数的值);
             存储过程的输出参数，使用map.get("输出参数名")来获取
        -->
        <select id="queryCountByGradeWithProcedure" statementType="CALLABLE" parameterType="HashMap">
             {
                    CALL queryCountByGradeWithProcedure(#{gname,jdbcType=VARCHAR,mode=IN},#{scount,jdbcType=INTEGER,mode=OUT})
             }
        </select>
```
### 6.2.4 在接口中书写相应方法
```java
//根据存储过程查询某个年级的学生总数
       public void queryCountByGradeWithProcedure(Map<String,Object> map);
```
### 6.2.5 测试
```java
//根据存储过程查询某个年级的学生总数
       public static void queryCountByGradeWithProcedure() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              //通过map给存储过程指定参数
              Map<String,Object> map=new HashMap<>();
              map.put("gname", "g1");//指定存储过程的输入参数gname的值是g1
              //调用存储过程，并传入输入参数
              studentMapper.queryCountByGradeWithProcedure(map);
              //获取存储过程的输出参数
              int count = (int) map.get("scount");
              //打印结果
              System.out.println(count);
              //释放资源
              sqlSession.close();
       }
```
## 6.3 根据存储过程删除某个学号的学生
### 6.3.1 在数据库中创建存储过程
```java
create procedure deleteStudentBystunoWithProcedure(in sno int)
begin
       delete from student where stuno=sno;
end;
```
### 6.3.2 在SqlMapConfig.xml文件中加载Mapper.xml，并设置别名
```xml
<!-- 设置别名(别名不区分大小写) -->
        <typeAliases>
             <!-- 定义单个别名 -->
             <!-- <typeAlias type="org.pdsu.entity.Student" alias="Student"/> -->
             <!-- 批量定义别名(别名为该类的类名(不带包名，忽略大小写)) -->
             <package name="org.pdsu.entity"/>
        </typeAliases>
 
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
</mappers>
```
### 6.3.3 在Mapper.xml文件中书写存储过程
```xml
<!--
             根据存储过程删除某个学号的学生
-->
         <delete id="deleteStudentBystunoWithProcedure" statementType="CALLABLE" parameterType="HashMap">
            {
                   CALL deleteStudentBystunoWithProcedure(#{sno,jdbcType=INTEGER,mode=IN})
            }
         </delete>
```
### 6.3.4 在接口中书写相应方法
```java
//根据存储过程删除某个学号的学生
       public void deleteStudentBystunoWithProcedure(Map<String,Object> map);
```
### 6.3.5 测试
```java
//根据存储过程删除某个学号的学生
       public static void deleteStudentBystunoWithProcedure() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              //通过map给存储过程指定参数
              Map<String,Object> map=new HashMap<>();
              map.put("sno", 9);
              //调用存储过程，并传入输入参数
              studentMapper.deleteStudentBystunoWithProcedure(map);
              //提交事务
              sqlSession.commit();
              //释放资源
              sqlSession.close();
       }
``` 
