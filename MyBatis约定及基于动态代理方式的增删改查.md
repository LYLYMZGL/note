# 3 MyBatis约定及基于动态代理方式的增删改查
## 3.1 Mybatis约定
mapper动态代理方式的CRUD（MyBatis接口开发）：<br/>
mapper动态代理基于约定。<br/>
原则：约定优于配置<br/>
①硬编码方式：<br/>
例如：在abc.java文件中书写Configuration conf=new Configuration(); conf.setName(“myProject”); 代码来设置项目的名称。<br/>
②配置方式：<br/>
例如：在abc.xml配置文件中书写<name>myProject</name>标签来设置项目的名称。<br/>
③约定：约定默认值就是myProject<br/>
 
具体实现步骤：<br/>
*1、基础环境*<br/>
①导入jar包<br/>
```java
mybatis-3.4.6.jar
mysql-connector-java-5.1.7-bin.jar
```
②配置SqlMapConfig.xml配置文件<br/>
*2、配置Mapper.xml配置文件*<br/>
*3、书写接口和与Mapper.xml文件中的标签相对应的方法*<br/>
*4、测试*<br/>
与基础方式的不同之处：<br/>
约定的目标：省略掉statement（namespace.id），即根据约定直接可以定位出sql语句。<br/>
```java
        /* 要实现接口中的方法和Mapper.xml中的标签一一对应，要满足以下约定：
        * 1.方法名和mapper.xml文件中标签的id值相同
        * 2.方法的输入参数和mapper.xml文件中标签的parameterType类型一致
        * (若mapper.xml中没有parameterType,则说明方法没有输入参数)
        * 3.方法的返回值和mapper.xml文件中标签的resultType类型一致
        * (无论查询结果是一个还是多个(Student/List<Student>),mapper.xml标签中的resultType都只写一个(Student))
        * (若mapper.xml中没有resultType,则方法的返回值为void)
        * 4.namespace的值就是接口的全类名
        */
```        
匹配的过程：<br/>
1、根据接口名找到mapper.xml文件(通过namespace=接口全类名)<br/>
2、根据接口的方法名找到mapper.xml文件中的SQL标签（方法名=SQL标签的id值）<br/>
通过以上2点可以保证：<br/>
当我们调用接口中的方法时，程序能自动定位到某一个mapper.xml文件中的sql标签。<br/>
习惯：通常把mapper.xml映射文件和接口放在同一个包下。<br/>
## 3.2 基于动态代理方式的增删改查
### 3.2.1 查询指定学号的学生
#### 3.2.1.1 studentMapper.xml
```xml
<select id="queryStudentById" parameterType="int" resultType="org.pdsu.entity.Student">
             select * from student where stuno=#{stuno}
</select>
```
#### 3.2.1.2 StudentMapper接口
```java
      //查询单个学生
       public Student queryStudentById(int stuno);
```       
#### 3.2.1.3 测试
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
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student student = studentMapper.queryStudentById(1);
              //打印结果
              System.out.println(student.toString());
              //释放资源
              sqlSession.close();
       }
```       
### 3.2.2 查询所有学生
#### 3.2.2.1 studentMapper.xml
```xml
<select id="queryAllStudents" resultType="org.pdsu.entity.Student">
            select * from student
</select>
```
#### 3.2.2.2 StudentMapper接口
```java
//查询全部学生
       public List<Student> queryAllStudents();
```       
#### 3.2.2.3 测试
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
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              List<Student> list = studentMapper.queryAllStudents();
              //打印结果
              for(Student student:list) {
                     System.out.println(student.toString());
              }
              //释放资源
              sqlSession.close();
       }
```       
### 3.2.3 增加学生
#### 3.2.3.1 studentMapper.xml
```xml
<insert id="addStudent" parameterType="org.pdsu.entity.Student">
            insert into student(stuname,stuage,gradename) values(#{stuName},#{stuAge},#{gradeName})
</insert>
```
#### 3.2.3.2 StudentMapper接口
```java
//增加学生
       public void addStudent(Student student);
```
#### 3.2.3.3 测试
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
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student student=new Student("xn",27,"s2");
              studentMapper.addStudent(student);
              //提交事务
              sqlSession.commit();
              //释放连接
              sqlSession.close();
       }
```       
### 3.2.4 删除指定学号的学生
#### 3.2.4.1 studentMapper.xml
```xml
<delete id="deleteStudentByStuno" parameterType="int">
            delete from student where stuno=#{stuno}
         </delete>
```
#### 3.2.4.2 StudentMapper接口
```java
//删除指定学号的学生
       public void deleteStudentByStuno(int stuno);
```
#### 3.2.4.3 测试
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
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              studentMapper.deleteStudentByStuno(8);
              //提交事务
              sqlSession.commit();
              //释放资源
              sqlSession.close();
       }
```       
### 3.2.5 修改指定学号的学生
#### 3.2.5.1 studentMapper.xml
```xml
<update id="updateStudentByStuno" parameterType="org.pdsu.entity.Student">
            update student set stuname=#{stuName},stuage=#{stuAge},gradename=#{gradeName} where stuno=#{stuNo}
         </update>
```
#### 3.2.5.2 StudentMapper接口
```java
//修改指定学号的学生
       public void updateStudentByStuno(Student student);
```
#### 3.2.5.3 测试
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
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student student=new Student(4,"一笑奈何",23,"q5");
              studentMapper.updateStudentByStuno(student);
              //提交事务
              sqlSession.commit();
              //释放资源
              sqlSession.close();
       }
``` 
 
