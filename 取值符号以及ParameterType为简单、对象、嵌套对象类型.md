# 5 取值符号以及ParameterType为简单、对象、嵌套对象类型
前置知识<br/>
输入参数：parameterType<br/>
1、类型为简单类型（8个基本类型+String）<br/>
`#{ }、${ }`的区别：<br/>
①#{任意标识符}<br/>
`${value}`，其中的标识符只能是value<br/>
②#{ }自动给String类型加上’ ’ （自动类型转换）<br/>
`${ }`原样输出，但是适合于动态排序<br/>
③#{ }可以防止SQL注入<br/>
`${ }`不防止SQL注入<br/>
`#{ }、${ }`的相同之处：<br/>
①都可以获取对象的值<br/>
②获取嵌套类型对象<br/>
2、对象类型<br/>
`#{对象属性名}`<br/>
`${对象属性名}`
 
## 5.1 根据某一列名动态排序（${ }的应用）
### 5.1.1 在SqlMapConfig.xml文件中加载Mapper.xml，并设置别名
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
### 5.1.2 在Mapper.xml文件中书写查询标签
```xml
<!-- 根据某一列名进行动态排序 -->
        <select id="queryStudentsByColumn" parameterType="String" resultType="Student">
             select stuno,stuname,stuage,gradename from student order by ${value} desc
        </select>
```
### 5.1.3 在接口中书写相应方法
```java
//根据某一列名进行动态排序
       public List<Student> queryStudentsByColumn(String column);
```
### 5.1.4 测试
```java
//根据某一列名进行动态排序
       public static void queryStudentsByColumn() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              List<Student> list = studentMapper.queryStudentsByColumn("stuno");
              //打印结果
              for(Student stu:list) {
                     System.out.println(stu.toString());
              }
              //释放资源
              sqlSession.close();
       }
``` 
## 5.2 根据年龄或姓名查询学生（分别使用#{ }和${ }）
### 5.2.1 在SqlMapConfig.xml文件中加载Mapper.xml，并设置别名
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
### 5.2.2 在Mapper.xml文件中书写查询标签
```xml
<!-- 根据年龄或姓名查询学生 -->
        <!-- 分别使用#{}、${} 进行查询 -->
        <select id="queryStudentsStuageOrStuname" parameterType="Student" resultType="Student">
             <!-- select stuno,stuname,stuage,gradename from student where stuage=#{stuAge} or stuname like #{stuName} -->
             select stuno,stuname,stuage,gradename from student where stuage=${stuAge} or stuname like '%${stuName}%'
        </select>
```
### 5.2.3 在接口中书写相应方法
```java
//根据年龄或姓名进行查询
       public List<Student> queryStudentsStuageOrStuname(Student student);
```
### 5.2.4 测试
```java
//根据年龄或姓名进行查询
       public static void queryStudentsStuageOrStuname() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student student=new Student();
              student.setStuAge(23);
//            student.setStuName("%s%");
              student.setStuName("s");
              List<Student> list = studentMapper.queryStudentsStuageOrStuname(student);
              //打印结果
              for(Student stu:list) {
                     System.out.println(stu.toString());
              }
              //释放资源
              sqlSession.close();
       }
```
## 5.3 根据地址查询学生（嵌套类型对象）
### 5.3.1 在SqlMapConfig.xml文件中加载Mapper.xml，并设置别名
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
### 5.3.2 在Mapper.xml文件中书写查询标签
```xml
<resultMap type="Student3" id="student3Result">
             <id property="stuNo" column="stuno"/>
             <result property="stuName" column="stuname"/>
             <result property="stuAge" column="stuage"/>
             <result property="gradeName" column="gradename"/>
             <result property="address.homeAddress" column="homeaddress"/>
             <result property="address.schoolAddress" column="schooladdress"/>
</resultMap>
        <!-- 根据地址查询学生(嵌套类型对象) -->
        <select id="queryStudentByAddress1" parameterType="Address" resultMap="student3Result">
             select stuno,stuname,stuage,gradename,homeaddress,schooladdress from student where homeaddress=#{homeAddress} and schooladdress=#{schoolAddress}
        </select>
```
### 5.3.3 在接口中书写相应方法
```java
//根据地址查询学生
       public List<Student3> queryStudentByAddress1(Address address);
```
### 5.3.4 测试
```java
//根据地址查询学生
       public static void queryStudentByAddress1() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Address address=new Address();
              address.setHomeAddress("cq");
              address.setSchoolAddress("xm");
              List<Student3> list = studentMapper.queryStudentByAddress1(address);
              //打印结果
              for(Student3 stu:list) {
                     System.out.println(stu.toString());
//                   System.out.println(stu.getAddress());
              }
              //释放资源
              sqlSession.close();
       }
```
## 5.4 根据地址查询学生（输入参数为级联属性）
### 5.4.1 在SqlMapConfig.xml文件中加载Mapper.xml，并设置别名
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
### 5.4.2 在Mapper.xml文件中书写查询标签
```xml
<resultMap type="Student3" id="student3Result">
             <id property="stuNo" column="stuno"/>
             <result property="stuName" column="stuname"/>
             <result property="stuAge" column="stuage"/>
             <result property="gradeName" column="gradename"/>
             <result property="address.homeAddress" column="homeaddress"/>
             <result property="address.schoolAddress" column="schooladdress"/>
</resultMap>
       <!-- 根据地址查询学生(输入参数为级联属性) -->
        <select id="queryStudentByAddress" parameterType="Student3" resultMap="student3Result">
             select stuno,stuname,stuage,gradename,homeaddress,schooladdress from student where homeaddress=#{address.homeAddress} and schooladdress=#{address.schoolAddress}
        </select>
```
### 5.4.3 在接口中书写相应方法
```java
//根据地址查询学生(输入参数为级联属性)
       public List<Student3> queryStudentByAddress(Student3 student);
```
### 5.4.4 测试
```java
//根据地址查询学生(输入参数为级联属性)
       public static void queryStudentByAddress() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student3 student=new Student3();
              Address address=new Address();
              address.setHomeAddress("cq");
              address.setSchoolAddress("xm");
              student.setAddress(address);
              List<Student3> list = studentMapper.queryStudentByAddress(student);
              //打印结果
              for(Student3 stu:list) {
                     System.out.println(stu.toString());
//                   System.out.println(stu.getAddress());
              }
              //释放资源
              sqlSession.close();
       }
```       
