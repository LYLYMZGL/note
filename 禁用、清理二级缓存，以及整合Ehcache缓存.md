# 13 禁用、清理二级缓存，以及整合Ehcache缓存
前置知识<br>
1、二级缓存：<br>
1）MyBatis自带的二级缓存：<br>
禁用：<br>
useCache属性的默认值为true。因此若要禁用某个select的二级缓存，则只需要将useCache的属性值设置为false即可。<br>
禁用二级缓存的步骤：<br>
Mapper.xml文件:
```xml
        <!-- 禁用该select的二级缓存 -->
        <select id="queryStuByStuno1" parameterType="int" resultType="Student3" useCache="false">
             select * from student where stuno=#{stuNo}
        </select>
``` 
命中率：<br>
第一次查询：二级缓存中没有，命中率0<br>
第二次查询：二级缓存中存在，命中率1/2<br>
第三次查询：二级缓存中存在，命中率2/3<br>
第四次查询：二级缓存中存在，命中率3/4<br>
![](https://user-gold-cdn.xitu.io/2019/7/19/16c08151ac8e677b?w=554&h=180&f=png&s=118850)

清理：<br>
1）、无论是一级缓存，还是二级缓存，只要执行了commit()操作，则缓存皆被清空。
（一般执行增删改操作时，都需要执行commit()操作，因此会清理掉缓存；这样设计的原因是为了防止脏数据）<br>
例如：执行查询操作时，会将查询到的数据放置到缓存中，在第二次查询同样的数据时，只需要读取缓存就可以查询到。但当执行增删改操作时，若更新的是之前查询到并放置在缓存中的数据，则必须要清空缓存，否则缓存和数据库的数据就将不一致，导致脏数据的出现。<br>
 
commit()会清理一级和二级缓存；但是清理二级缓存时，不能使用查询自身的SqlSession来commit()。<br>
2）、在select标签中，增加属性flushCache=”true”<br>
2、第三方提供的二级缓存：<br>
ehcache、memcache<br>
要想整合第三方提供的二级缓存（或自定义二级缓存），必须实现org.apache.ibatis.cache.Cache接口。该接口的默认实现类是PerpetualCache。<br>
1）、ehcache<br>
该厂商已经实现了Cache接口，因此只需要导入相应的jar包即可。<br>
a、整合ehcache：<br>
①导入jar包<br>
```java
ehcache-core.jar
mybatis-ehcache.jar
slf4j-api.jar
```
②编写ehcache配置文件：Ehcache.xml
```xml
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd">
    <!-- 当二级缓存的对象超出内存限制时(缓存对象的个数>maxElementsInMemory)，放入的硬盘文件路径 -->
       <diskStore path="e:\Ehcache"/>
       <!--
              maxElementsInMemory:设置在内存中存放的二级缓存对象的个数
              maxElementsOnDisk:设置在硬盘中存放的二级缓存对象的个数
              eternal:设置缓存是否永远不过期
              overflowToDisk:当内存中缓存的对象个数超过maxElementsInMemory值时，是否转移到硬盘中
              timeToIdleSeconds:当两次访问的间隔超过该值时（该值单位为s），将缓存对象失效
              timeToLiveSeconds:一个缓存对象最多存放的时间（生命周期）
              diskExpiryThreadIntervalSeconds:设置每隔多长时间，通过一个线程来清理硬盘中的缓存
              memoryStoreEvictionPolicy:当超过缓存对象的最大值时，处理的策略；使用LRU代表寻找最近最少使用的对象，然后将其删除，将新的对象放入该位置
              策略：
              LRU:最近最少使用
              FIFO:先进先出（队列）
              LFU:最不常使用
        -->
       <defaultCache
              maxElementsInMemory="1000"
              maxElementsOnDisk="1000000"
              eternal="false"
              overflowToDisk="false"
              timeToIdleSeconds="100"
              timeToLiveSeconds="100"
              diskExpiryThreadIntervalSeconds="120"
              memoryStoreEvictionPolicy="LRU">
       </defaultCache>
</ehcache>
```
③开启Ehcache二级缓存<br>
在Mapper.xml文件中开启：
```xml
<!-- 配置ehcache二级缓存 -->
        <!-- type属性指定哪个类实现了Cache接口 -->
        <cache type="org.mybatis.caches.ehcache.EhcacheCache">
             <!-- 通过设置属性可以覆盖Ehcache.xml中的值 -->
             <!-- <property name="maxElementsInMemory" value="2000"/> -->
        </cache>
```
## 13.1 验证关闭二级缓存，根据学号查询学生
### 13.1.1 在SqlMapConfig.xml文件中加载Mapper.xml映射文件
```xml
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
             <mapper resource="org/pdsu/mapper/studentCardMapper.xml"/>
             <mapper resource="org/pdsu/mapper/studentClassMapper.xml"/>
</mappers>
```
### 13.1.2 在Mapper.xml文件中书写查询标签并禁用二级缓存
```xml
<!-- 禁用该select的二级缓存 -->
        <select id="queryStuByStuno1" parameterType="int" resultType="Student3" useCache="false">
             select * from student where stuno=#{stuNo}
        </select>
```
### 13.1.3 在接口中书写相应方法
```java
//根据学号查询学生,禁用二级缓存
       public Student3 queryStuByStuno1(int stuNo);
```
### 13.1.4 测试
```java
//验证关闭二级缓存,根据学号查询学生
       public static void queryStuByStuno4() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student3 student3 = studentMapper.queryStuByStuno1(1);
              //释放资源
              sqlSession.close();//进行缓存的时刻
              //第二次查询
              SqlSession sqlSession2 = sqlSessionFactory.openSession();
              StudentMapper studentMapper2 = sqlSession2.getMapper(StudentMapper.class);
              Student3 student32 = studentMapper2.queryStuByStuno1(1);
              //打印结果
              System.out.println(student3);
              System.out.println(student32);
              //释放资源
              sqlSession2.close();
       }
```
### 13.1.5 结果
![](https://user-gold-cdn.xitu.io/2019/7/19/16c0815d981ac3ff?w=554&h=149&f=png&s=90835)

## 13.2 验证清理二级缓存（方法一），根据学号查询学生、修改执行学号的学生
### 13.2.1 在SqlMapConfig.xml文件中加载Mapper.xml映射文件
```xml
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
             <mapper resource="org/pdsu/mapper/studentCardMapper.xml"/>
             <mapper resource="org/pdsu/mapper/studentClassMapper.xml"/>
</mappers>
```
### 13.2.2 在Mapper.xml文件中书写查询标签和修改标签
```xml
<!--
             输出参数：实体对象类型
             根据学号查询学生
         -->
        <select id="queryStuByStuno" parameterType="int" resultType="Student3">
             select * from student where stuno=#{stuNo}
        </select>
 
<update id="updateStudentByStuno" parameterType="Student">
            update student set stuname=#{stuName},stuage=#{stuAge},gradename=#{gradeName} where stuno=#{stuNo}
         </update>
```
### 13.2.3 在接口中书写相应方法
```java
//根据学号查询学生
       public Student3 queryStuByStuno(int stuNo);
//修改指定学号的学生
       public void updateStudentByStuno(Student student);
```
### 13.2.4 测试
```java
//验证清理二级缓存,根据学号查询学生
       public static void queryStuByStuno5() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student3 student3 = studentMapper.queryStuByStuno(2);
              //释放资源
              sqlSession.close();//进行缓存的时刻
             
              //增删改
              SqlSession sqlSession3 = sqlSessionFactory.openSession();
              StudentMapper studentMapper3 = sqlSession3.getMapper(StudentMapper.class);
              Student stu=new Student();
              stu.setStuNo(2);
              stu.setStuName("lxs");
              stu.setStuAge(11);
              stu.setGradeName("ggggggg1");
              studentMapper3.updateStudentByStuno(stu);
              sqlSession3.commit();
             
              //第二次查询
              SqlSession sqlSession2 = sqlSessionFactory.openSession();
              StudentMapper studentMapper2 = sqlSession2.getMapper(StudentMapper.class);
              Student3 student32 = studentMapper2.queryStuByStuno(2);
              //释放资源
              sqlSession2.close();
              //打印结果
              System.out.println(student3);
              System.out.println(student32);
       }
```
### 13.2.5 结果
![](https://user-gold-cdn.xitu.io/2019/7/19/16c08165cbcad9f6?w=554&h=246&f=png&s=132136)

## 13.3 验证清理二级缓存（方法二），根据学号查询学生
### 13.3.1 在SqlMapConfig.xml文件中加载Mapper.xml映射文件
```xml
<mappers>
             <!-- 加载映射文件 -->
             <mapper resource="org/pdsu/mapper/studentMapper.xml"/>
             <mapper resource="org/pdsu/mapper/studentCardMapper.xml"/>
             <mapper resource="org/pdsu/mapper/studentClassMapper.xml"/>
</mappers>
```
### 13.3.2 在Mapper.xml文件中书写查询标签，并清理缓存
```xml
<!-- 清理该select的二级缓存 -->
        <select id="queryStuByStuno2" parameterType="int" resultType="Student3"  flushCache="true">
             select * from student where stuno=#{stuNo}
        </select>
```
### 13.3.3 在接口中书写相应方法
```java
//根据学号查询学生,清理二级缓存
       public Student3 queryStuByStuno2(int stuNo);
```
### 13.3.4 测试
```java
//验证清理二级缓存2,根据学号查询学生
       public static void queryStuByStuno6() throws Exception{
              //加载MyBatis配置文件
              Reader reader = Resources.getResourceAsReader("SqlMapConfig.xml");
              //创建SqlSessionFactory
              SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
              //创建SqlSession
              SqlSession sqlSession = sqlSessionFactory.openSession();
              StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
              Student3 student3 = studentMapper.queryStuByStuno2(2);
              //释放资源
              sqlSession.close();//进行缓存的时刻
             
              //第二次查询
              SqlSession sqlSession2 = sqlSessionFactory.openSession();
              StudentMapper studentMapper2 = sqlSession2.getMapper(StudentMapper.class);
              Student3 student32 = studentMapper2.queryStuByStuno2(2);
              //释放资源
              sqlSession2.close();
              //打印结果
              System.out.println(student3);
              System.out.println(student32);
       }
```
### 13.3.5 结果

![](https://user-gold-cdn.xitu.io/2019/7/19/16c08170fc8b73e9?w=554&h=199&f=png&s=105800)
