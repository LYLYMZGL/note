# 7 输出参数为简单类型、对象类型、HashMap及resultMap使用
前置知识<br>
输出参数<br>
1、输出参数为简单类型或实体对象类型：<br>
①输出参数为简单类型<br>
在Mapper.xml配置文件中，使用resultType指定输出的参数类型。<br>
②输出参数为实体对象类型<br>
在Mapper.xml配置文件中，使用resultType指定输出的参数类型。<br>
③输出参数为实体对象类型的集合<br>
在Mapper.xml配置文件中，使用resultType指定输出的参数类型。<br>
resultType中只需要写集合中元素的类型即可，但是在接口中仍然要写集合类型。<br>
2、输出参数为HashMap类型：<br>
在sql语句中，为所需要返回的每一列起别名，作为HashMap中的key。<br>
 
HashMap本身是一个集合，可以存放多个元素。因此，HashMap对应一个学生的多个属性。<br>
3、使用resultMap指定输出类型及映射关系<br>
1）、当实体类属性类型和数据库字段类型不一致时，使用resultMap。<br>
2）、当实体类属性名和数据库字段名不一致时：<br>
①使用resultMap。<br>
②使用resultType+HashMap。<br>
语法格式：<br>
select 表字段名 “类属性名” from 表名<br>
## 7.1 查询学生总数（简单类型）
### 7.1.1 在SqlMapConfig.xml文件中加载Mapper.xml映射文件
```xml
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
</mappers>
```
### 7.1.2 在Mapper.xml文件中书写查询标签
```xml
<!--
             输出参数：简单类型
             查询学生总数
         -->
        <select id="queryStudentCount" resultType="int">
             select count(*) from student
        </select>
```
### 7.1.3 在接口中书写相应方法
```java
//查询学生总数(输出参数)
       public int queryStudentCount();
```
### 7.1.4 测试
```java
//查询学生总数
       public static void queryStudentCount() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              int count = studentMapper.queryStudentCount();
              //打印结果
              System.out.println(count);
              //释放资源
              sqlSession.close();
       }
```
## 7.2 根据学号查询学生（实体对象类型）
### 7.2.1 在SqlMapConfig.xml文件中加载Mapper.xml映射文件
```xml
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
</mappers>
```
### 7.2.2 在Mapper.xml文件中书写查询标签
```xml
<!--
             输出参数：实体对象类型
             根据学号查询学生
         -->
        <select id="queryStuByStuno" parameterType="int" resultType="Student3">
             select * from student where stuno=#{stuNo}
        </select>
```
### 7.2.3 在接口中书写相应方法
```java
//根据学号查询学生
       public Student3 queryStuByStuno(int stuNo);
```
### 7.2.4 测试
```java
//根据学号查询学生
       public static void queryStuByStuno() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student3 student3 = studentMapper.queryStuByStuno(1);
              //打印结果
              System.out.println(student3);
              //释放资源
              sqlSession.close();
       }
```
## 7.3 查询某一个学生的学号和姓名（输出参数使用HashMap）
### 7.3.1 在SqlMapConfig.xml文件中加载Mapper.xml映射文件
```xml
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
</mappers>
```
### 7.3.2 在Mapper.xml文件中书写查询标签
```xml
<!--
             输出参数：HashMap类型
             查询某一个学生的学号和姓名
             通过对列起别名作为HashMap的key
         -->
         <select id="queryStudentOutByHashMap" resultType="HashMap">
            select stuno "no",stuname "name" from student where stuno=1
         </select>
```
### 7.3.3 在接口中书写相应方法
```java
//查询某一个学生的学号和姓名
       public HashMap<String,Object> queryStudentOutByHashMap();
```
### 7.3.4 测试
```java
//查询某一个学生的学号和姓名
       public static void queryStudentOutByHashMap() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              HashMap<String,Object> map = studentMapper.queryStudentOutByHashMap();
              //打印结果
              System.out.println(map);
              //释放资源
              sqlSession.close();
       }
```
## 7.4 查询所有学生的学号和姓名（HashMap类型）
### 7.4.1 在SqlMapConfig.xml文件中加载Mapper.xml映射文件
```xml
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
</mappers>
```
### 7.4.2 在Mapper.xml文件中书写查询标签
```xml
<!--
             输出参数：HashMap类型
             查询所有学生的学号和姓名
             通过对列起别名作为HashMap的key
         -->
         <select id="queryAllStudentOutByHashMap" resultType="HashMap">
            select stuno "no",stuname "name" from student
         </select>
```
### 7.4.3 在接口中书写相应方法
```java
//查询所有学生的学号和姓名
       public List<HashMap<String,Object>> queryAllStudentOutByHashMap();
```
### 7.4.4 测试
```java
//查询所有学生的学号和姓名
       public static void queryAllStudentOutByHashMap() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              List<HashMap<String,Object>> list = studentMapper.queryAllStudentOutByHashMap();
              //打印结果
              System.out.println(list);
              //释放资源
              sqlSession.close();
       }
```
## 7.5 根据学号查询学生的id和name（resultMap）
### 7.5.1 在SqlMapConfig.xml文件中加载Mapper.xml映射文件
```xml
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
</mappers>
```
### 7.5.2 在Mapper.xml文件中书写查询标签和resultMap
```xml
<!--
            实体类属性名和数据库字段名称不一致
            ①使用resultMap
            查询指定学号的学生的id和name
          -->
         <select id="queryStudentById1" parameterType="int" resultMap="queryStudentById1Map">
            select id,name from student_1 where id=#{id}
         </select>
         <resultMap type="Student" id="queryStudentById1Map">
            <!-- 指定类中的属性和表中的字段的对应关系 -->
            <id property="stuNo" column="id"/>
            <result property="stuName" column="name"/>
         </resultMap>
```
### 7.5.3 在接口中书写相应方法
```java
//根据指定序号查询学生的id和name
       public Student queryStudentById1(int id);
```
### 7.5.4 测试
```java
//根据指定序号查询学生的id和name
       public static void queryStudentById1() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student student = studentMapper.queryStudentById1(1);
              //打印结果
              System.out.println(student);
              //释放资源
              sqlSession.close();
       }
```
## 7.6 据学号查询学生的id和name（resultType+HashMap）
若查询出来的值为0或null等默认值，则代表别名书写错误，无法和类中属性对应。<br>
## 7.6.1 在SqlMapConfig.xml文件中加载Mapper.xml映射文件
```xml
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
</mappers>
```
### 7.6.2 在Mapper.xml文件中书写查询标签
```xml
<!--
            实体类属性名和数据库字段名称不一致
            ②使用resultType+HashMap
            通过给列加别名来实现数据库字段名和实体类属性名对应
            查询指定学号的学生的id和name
          -->
          <select id="queryStudentById1WithHashMap" parameterType="int" resultType="Student">
           select id "stuNo",name "stuName" from student_1 where id=#{id}
          </select>
```
### 7.6.3 在接口中书写相应方法
```java
//根据指定序号查询学生的id和name(通过HashMap+resultType实现)
       public Student queryStudentById1WithHashMap(int id);
```
### 7.6.4 测试
```java
//根据指定序号查询学生的id和name(通过HashMap+resultType实现)
       public static void queryStudentById1WithHashMap() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student student = studentMapper.queryStudentById1WithHashMap(1);
              //打印结果
              System.out.println(student);
              //释放资源
              sqlSession.close();
       }
```
