# 12 MyBatis一级缓存、二级缓存
前置知识<br>
查询缓存：一级缓存、二级缓存<br>
1、一级缓存：同一个SqlSession对象共享一级缓存（只要SqlSession对象commit()后，就会清空缓存）。<br>
MyBatis默认开启一级缓存，如果使用同样的SqlSession对象查询相同的数据，则只会在第一次查询时，向数据库发送SQL语句，并将查询的结果放入到SqlSession中（作为缓存存在）；后续再次使用该SqlSession对象查询同样的数据时，直接从缓存中查询该对象即可（即省略了数据库的访问）。<br>
2、二级缓存：<br>
MyBatis默认情况没有开启二级缓存，需要手工打开。<br>
（MyBatis默认整个Mapper.xml文件的二级缓存是关闭的，然而默认开启Mapper.xml文件中的每一个select标签的二级缓存。因此在使用二级缓存时，只需要手动开启整个Mapper.xml文件的二级缓存即可）<br>
开启二级缓存的步骤：<br>
①SqlMapConfig.xml
```xml
        <settings>
             <!-- 开启日志，并指定使用的具体日志 -->
             <!-- 若不指定，则MyBatis会按照此顺序查找日志 -->
             <!-- SLF4J → Apache Commons Logging → Log4j → JDK logging -->
             <setting name="logImpl" value="LOG4J"/>
             <!-- 开启二级缓存 -->
             <setting name="cacheEnabled" value="true"/>
        </settings>
```        
②在具体的mapper.xml中声明开启
```xml
<mapper namespace="org.pdsu.mapper.StudentMapper">
        <!-- 声明此namespace开启二级缓存 -->
        <cache/>
</mapper>
``` 
根据异常提示：NotSerializableException（无序列化异常）可知，MyBatis的二级缓存是将对象放入硬盘文件中。<br>
序列化：内存->硬盘<br>
反序列化：硬盘->内存<br>
**准备缓存的对象所对应的类，以及该类的所有级联属性所对应的类，都必须实现了序列化接口。若不实现序列化接口，则该类会在序列化的过程中丢失。**<br>
 
**写入二级缓存的时机：数据在一开始并没有立即放置在硬盘（二级缓存）中，而是先存放到一级缓存中。因为I/O操作也是非常耗费性能的。当SqlSession对象被关闭的时候（SqlSession.close()）才会将一级缓存中的数据存放到二级缓存（硬盘）中。**<br>
1）MyBatis自带二级缓存：**同一个namespace**生成的mapper对象。<br>
回顾：namespace的值就是接口的全类名（包名.类名），通过接口可以产生代理对象（StudentMapper对象），即namespace决定了studentMapper对象的产生。<br>
**结论：只要产生的Mapper对象来自于同一个namespace，则这些对象共享二级缓存。**<br>
范围：同一个namespace。<br>
如果多个Mapper.xml文件的namespace的值相同，则通过这些Mapper.xml文件产生的Mapper对象，仍然共享二级缓存。<br>
 
如果是同一个SqlSession对象进行多次相同的查询，则直接进入一级缓存查询；如果不是同一个SqlSession对象进行多次相同的查询（但是均来自同一个namespace），则进入二级缓存查询。<br>
 
2）第三方提供的二级缓存：<br>
 
 
## 12.1 根据学号查询学生，验证一级缓存
### 12.1.1 在SqlMapConfig.xml文件中加载Mapper.xml映射文件
```xml
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
             <mapper resource="org/pdsu/mapper/studentCardMapper.xml"/>
             <mapper resource="org/pdsu/mapper/studentClassMapper.xml"/>
</mappers>
```
### 12.1.2 在Mapper.xml文件中书写查询标签
```xml
        <!--
             输出参数：实体对象类型
             根据学号查询学生
         -->
        <select id="queryStuByStuno" parameterType="int" resultType="Student3">
             select * from student where stuno=#{stuNo}
        </select>
```
### 12.1.3 在接口中书写相应方法
```java
//根据学号查询学生
       public Student3 queryStuByStuno(int stuNo);
```
### 12.1.4 测试
```java
       //验证一级缓存,根据学号查询学生
       public static void queryStuByStuno2() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              //执行sql语句
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student3 student3 = studentMapper.queryStuByStuno(1);
              /*
               * 当SqlSession对象被提交后,会清空一级缓存。
               */
//            sqlSession.commit();
              Student3 student32 = studentMapper.queryStuByStuno(1);
              //打印结果
              System.out.println(student3);
              System.out.println(student32);
              //释放资源
              sqlSession.close();
       }
```
### 12.1.5 结果
![](https://user-gold-cdn.xitu.io/2019/7/18/16c029bc12173ee3?w=554&h=115&f=png&s=76965)

## 12.2 根据学号查询学生，验证二级缓存
### 12.2.1 在SqlMapConfig.xml文件中加载Mapper.xml映射文件，并开启二级缓存
```xml
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
             <mapper resource="org/pdsu/mapper/studentCardMapper.xml"/>
             <mapper resource="org/pdsu/mapper/studentClassMapper.xml"/>
</mappers>
<settings>
             <!-- 开启日志，并指定使用的具体日志 -->
             <!-- 若不指定，则MyBatis会按照此顺序查找日志 -->
             <!-- SLF4J → Apache Commons Logging → Log4j → JDK logging -->
             <setting name="logImpl" value="LOG4J"/>
             <!-- 开启延迟加载 -->
             <setting name="lazyLoadingEnabled" value="true"/>
             <!-- 关闭立即加载 -->
             <setting name="aggressiveLazyLoading" value="false"/>
             <!-- 开启二级缓存 -->
             <setting name="cacheEnabled" value="true"/>
        </settings>
```
### 12.2.2 在Mapper.xml文件中书写查询标签，并声明此namespace开启二级缓存
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace:该mapper.xml映射文件的唯一标识 -->
<mapper namespace="org.pdsu.mapper.StudentMapper">
        <!-- 声明此namespace开启二级缓存 -->
        <cache/>
<!--
             输出参数：实体对象类型
             根据学号查询学生
         -->
        <select id="queryStuByStuno" parameterType="int" resultType="Student3">
             select * from student where stuno=#{stuNo}
        </select>
</mapper>
```
### 12.2.3 在接口中书写相应方法
```java
//根据学号查询学生
       public Student3 queryStuByStuno(int stuNo);
```
### 12.2.4 为即将被缓存的对象对应的类及级联属性实现序列化接口
```java
package org.pdsu.entity;
 
import java.io.Serializable;
 
public class Student3 implements Serializable{
       /**
        *
        */
       private static final long serialVersionUID = 1L;
       private int stuNo;
       private String stuName;
       private int stuAge;
       private String gradeName;
       private boolean stuSex;
       private Address address;
      
       public Student3() {
              super();
       }
       public Student3(String stuName, int stuAge, String gradeName) {
              super();
              this.stuName = stuName;
              this.stuAge = stuAge;
              this.gradeName = gradeName;
       }
       public Student3(int stuNo, String stuName, int stuAge, String gradeName) {
              super();
              this.stuNo = stuNo;
              this.stuName = stuName;
              this.stuAge = stuAge;
              this.gradeName = gradeName;
       }
       public Student3(int stuNo, String stuName, int stuAge, String gradeName, boolean stuSex, Address address) {
              super();
              this.stuNo = stuNo;
              this.stuName = stuName;
              this.stuAge = stuAge;
              this.gradeName = gradeName;
              this.stuSex = stuSex;
              this.address = address;
       }
       public Address getAddress() {
              return address;
       }
       public void setAddress(Address address) {
              this.address = address;
       }
       public boolean isStuSex() {
              return stuSex;
       }
       public void setStuSex(boolean stuSex) {
              this.stuSex = stuSex;
       }
       public int getStuNo() {
              return stuNo;
       }
       public void setStuNo(int stuNo) {
              this.stuNo = stuNo;
       }
       public String getStuName() {
              return stuName;
       }
       public void setStuName(String stuName) {
              this.stuName = stuName;
       }
       public int getStuAge() {
              return stuAge;
       }
       public void setStuAge(int stuAge) {
              this.stuAge = stuAge;
       }
       public String getGradeName() {
              return gradeName;
       }
       public void setGradeName(String gradeName) {
              this.gradeName = gradeName;
       }
       @Override
       public String toString() {
              return "Student3 [stuNo=" + stuNo + ", stuName=" + stuName + ", stuAge=" + stuAge + ", gradeName=" + gradeName
                            + ", address=" + address + "]";
       }
}
```
### 12.2.5 测试
```java
//验证二级缓存,根据学号查询学生
       public static void queryStuByStuno3() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student3 student3 = studentMapper.queryStuByStuno(1);
              //释放资源
              sqlSession.close();//进行缓存的时刻
              //第二次查询
              SqlSession sqlSession2 = sqlSessionFactory.openSession();
              StudentMapper studentMapper2 = sqlSession2.getMapper(StudentMapper.class);
              Student3 student32 = studentMapper2.queryStuByStuno(1);
              //打印结果
              System.out.println(student3);
              System.out.println(student32);
              //释放资源
              sqlSession2.close();
       }
```
### 12.2.6 结果
![](https://user-gold-cdn.xitu.io/2019/7/18/16c029da4dfaa578?w=553&h=139&f=png&s=91220)
