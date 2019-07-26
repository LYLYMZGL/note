## 1.2 Spring的AOP的基于AspectJ的注解开发
### 1.2.1 Spring的基于AspectJ的注解的AOP开发
#### 1.2.1.1 创建项目，引入jar包

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d863209d5f7d?w=348&h=217&f=jpeg&s=78102)
 
#### 1.2.1.2 引入配置文件

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d8684d5c5e1b?w=554&h=227&f=jpeg&s=119654)
 
#### 1.2.1.3 编写目标类并配置

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d86d22702a68?w=554&h=71&f=jpeg&s=22596)
 
#### 1.2.1.4 编写切面类并配置

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d870d235de05?w=547&h=203&f=jpeg&s=45467)
 

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d873f23cc265?w=554&h=71&f=jpeg&s=22769)
 
#### 1.2.1.5 使用注解的AOP对象目标类进行增强
在配置文件中打开注解的AOP开发

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d87786f429e0?w=439&h=54&f=jpeg&s=21824)
 
在切面类上使用注解

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d87b04930a58?w=554&h=205&f=jpeg&s=48173)
 
#### 1.2.1.6 编写测试类

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d87e953cb64b)
 
### 1.2.2 Spring的注解的AOP的通知类型
#### 1.2.2.1 @Before：前置通知

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d883538b624e?w=553&h=81&f=jpeg&s=28472)
 
#### 1.2.2.2 @AfterReturning：后置通知

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d8865c5dbec4?w=554&h=59&f=jpeg&s=21303)
 
#### 1.2.2.3 @Around：环绕通知

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d8920959a303?w=554&h=123&f=jpeg&s=48066)
 
#### 1.2.2.4 @AfterThrowing：异常抛出通知

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d895ed00a07e?w=553&h=77&f=jpeg&s=24921)
 
#### 1.2.2.5 @After：最终通知

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d898e7e19712?w=554&h=94&f=jpeg&s=25082)
 
### 1.2.3 Spring的注解的AOP的切入点的配置
#### 1.2.3.1 Spring的AOP的注解切入点的配置

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d89d20e84855?w=553&h=120&f=jpeg&s=51969)
 
 
## 1.3 Spring的JDBC模板的使用
### 1.3.1 Spring的JDBC的模板
Spring是EE开发的一站式的框架，有EE开发的每层的解决方案。Spring对持久层也提供了解决方案：ORM模块和JDBC的模板。<br>
Spring提供了很多的模板用户简化开发：<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d8ad6c22320e?w=554&h=218&f=jpeg&s=81010)
 
#### 1.3.1.1 JDBC模板使用的入门
创建项目，引入jar包<br>
引入基本开发包<br>
数据库驱动<br>
Spring的JDBC模板的jar包<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d8b0ec75350c?w=397&h=202&f=jpeg&s=66293)
 
#### 1.3.1.2 创建数据库和表
```sql
create database spring4_day03;
use spring4_day03;
create table account(
id int primary key auto_increment,
name varchar(20),
money double
);
```
#### 1.3.1.3 使用JDBC的模板：保存数据

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d8b494d9da25?w=554&h=311&f=jpeg&s=95860)
 
### 1.3.2 将连接池和模板交给Spring管理
#### 1.3.2.1 引入Spring的配置文件

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d8b864a51cc2?w=554&h=150&f=jpeg&s=66519)
 
#### 1.3.2.2 使用JDBC的模板
引入Spring_aop的jar包<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d8c3108c9100?w=554&h=267&f=jpeg&s=97716)
 
### 1.3.3 使用开源的数据库连接池：
#### 1.3.3.1 DBCP的使用
引入jar包<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d8c679c813b0?w=554&h=68&f=jpeg&s=26587)
 
配置DBCP连接池<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d8c9e0d62c89?w=554&h=141&f=jpeg&s=65046)
 
#### 1.3.3.2 C3P0的使用
引入C3P0连接池jar包<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d8ce98eab9be?w=554&h=24&f=jpeg&s=13164)
 
配置C3P0连接池<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d8d18db31cd0?w=554&h=52&f=jpeg&s=27263)
 
### 1.3.4 抽取配置到属性文件
#### 1.3.4.1 定义一个属性文件

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d8d65f6a1311?w=554&h=134&f=jpeg&s=51807)
 
#### 1.3.4.2 在Spring的配置文件中引入属性文件
第一种：<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d8ec302d8ace?w=930&h=86&f=png&s=8211)
 
第二种：<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d8dac74b9111?w=554&h=45&f=jpeg&s=22662)
 
#### 1.3.4.3 引入属性文件的值

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d8f0fcb34998?w=554&h=110&f=jpeg&s=52579)
 
#### 1.3.4.4 测试

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d8f448869122?w=554&h=72&f=jpeg&s=19206)
 
### 1.3.5 使用JDBC模板完成CRUD的操作
#### 1.3.5.1 保存操作

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d8fcc36f699a?w=835&h=105&f=png&s=5302)
 
#### 1.3.5.2 修改操作

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d8fdc1baca85?w=948&h=127&f=png&s=6011)
 
#### 1.3.5.3 删除操作

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d901371978fb?w=554&h=88&f=jpeg&s=24471)
 
#### 1.3.5.4 查询操作
查询某个属性<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d90f55ed9813?w=554&h=146&f=jpeg&s=34366)
 
查询返回对象或集合<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d915b9e66f3c?w=554&h=148&f=jpeg&s=37702)
 
数据封装<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d91926fb6d93?w=554&h=179&f=jpeg&s=46694)
 
## 1.4 Spring的事务管理
### 1.4.1 事务的回顾
#### 1.4.1.1 什么是事务
事务：逻辑上的一组操作，组成这组操作的各个单元，要么全都成功，要么全都失败。<br>
#### 1.4.1.2 事务的特性
原子性：事务不可分割<br>
一致性：事务执行前后数据完整性保持一致<br>
隔离性：一个事务的执行不应该受到其他事务的干扰<br>
持久性：一旦事务结束，数据就持久化到数据库<br>
#### 1.4.1.3 如果不考虑隔离性引发安全性问题
读问题<br>
脏读：一个事务读到另一个事务未提交的数据<br>
不可重复读：一个事务读到另一个事务已经提交的update的数据，导致一个事务中多次查询结果不一致<br>
虚读、幻读：一个事务读到另一个事务已经提交的insert的数据，导致一个事务中多次查询结果不一致。<br>
写问题<br>
丢失更新<br>
#### 1.4.1.4 解决读问题
设置事务的隔离级别<br>
Read uncommitted：未提交读，任何读问题都解决不了。<br>
Read committed：已提交读，解决脏读，但是不可重复读和虚读有可能发生。（oracle数据库使用该方式）<br>
Repeatable read：重复读，解决脏读和不可重复读，但是虚读有可能发生。（MySQL数据库使用该方式）<br>
Serializable：解决所有读问题。（虽然能解决所有问题，但是效率太低）<br>
### 1.4.2 Spring的事务管理的API
#### 1.4.2.1 PlatformTransactionManager：平台事务管理器
平台事务管理器：接口，是Spring用于管理事务的真正的对象。<br>
DataSourceTransactionManager：底层使用JDBC管理事务<br>
HibernateTransactionManager：底层使用Hibernate管理事务<br>
#### 1.4.2.2 TransactionDefinition：事务定义信息
事务定义：用于定义事务的相关的信息，隔离级别、超时信息、传播行为、是否只读。<br>
#### 1.4.2.3 TransactionStatus：事务的状态
事务状态：用于记录在事务管理过程中，事务的状态的对象。<br>
#### 1.4.2.4 事务管理的API的关系：
Spring进行事务管理的时候，首先平台事务管理器根据事务定义信息进行事务的管理，在事务管理的过程中，产生各种状态，将这些状态的信息记录到事务状态的对象中。<br>
### 1.4.3 Spring的事务的传播行为
#### 1.4.3.1 Spring的传播行为
Spring中提供了七种事务的传播行为：<br>
保证多个操作在同一个事务中<br>
PROPAGATION_REQUIRED：默认值。如果A中有事务，使用A中的事务。如果A没有，创建一个新的事务，将操作包含进来。<br>
PROPAGATION_SUPPORTS：支持事务。如果A中有事务，使用A中的事务。如果A没有事务，不使用事务。<br>
PROPAGATION_MANDATORY：如果A中有事务，使用A中的事务。如果A没有事务，抛出异常。
保证多个操作不在同一个事务中<br>
PROPAGATION_REQUIRES_NEW：如果A中有事务，将A的事务挂起（暂停），创建新事务，只包含自身操作。如果A中没有事务，创建一个新事务，包含自身操作。<br>
PROPAGATION_NOT_SUPPORTED：如果A中有事务，将A的事务挂起。不使用事务管理。<br>
PROPAGATION_NEVER：如果A中有事务，抛出异常。<br>
嵌套式事务<br>
PROPAGATION_NESTED：嵌套事务。如果A中有事务，按照A的事务执行，执行完成后，设置一个保存点。执行B中的操作，如果没有异常，执行通过。如果有异常，可以选择回滚到最初始位置，也可以回滚到保存点。<br>
 
### 1.4.4 Spring的事务管理
#### 1.4.4.1 搭建Spring的事务管理的环境
创建Service的接口和实现类<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d92340dd710a?w=553&h=270&f=jpeg&s=48599)
 
创建DAO的接口和实现类<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d9264cae5dc5?w=554&h=210&f=jpeg&s=50837)
 
配置Service和DAO：交给Spring管理<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d9297fc9c6c7?w=554&h=116&f=jpeg&s=40665)
 
在DAO中编写扣钱和加钱方法：<br>
配置连接池和JDBC的模板（由于dao实现类继承了JdbcDaoSupport类，所以dao实现类本身就具有了注入连接池获得jdbc模板的方法）<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d92e7b3fabfc?w=554&h=154&f=jpeg&s=72217)
 
在DAO注入Jdbc的模板<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d93211254612?w=554&h=77&f=jpeg&s=32035)
 
测试<br>
 
### 1.4.5 Spring的事务管理：一类：编程式事务管理（需要手动编写代码）----了解
#### 1.4.5.1 第一步：配置平台事务管理器

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d93623e91993?w=554&h=46&f=jpeg&s=17004)
 
#### 1.4.5.2 第二步：Spring提供了事务管理的模板类

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d938fb423075?w=554&h=47&f=jpeg&s=22303)
 
#### 1.4.5.3 第三步：在业务层注入事务管理的模板

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d93c34b95ab5?w=554&h=88&f=jpeg&s=42329)
 
#### 1.4.5.4 编写事务管理的代码

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d93f9bd1577c?w=554&h=186&f=jpeg&s=43377)
 
#### 1.4.5.5 测试
### 1.4.6 Spring的事务管理：二类：声明式事务管理（通过配置实现）----AOP
#### 1.4.6.1 XML方式的声明式事务管理
第一步：引入AOP的开发包<br>
第二步：恢复转账环境<br>
第三步：配置事务管理器<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d942d2f53939?w=553&h=43&f=jpeg&s=18314)
 
第四步：配置增强<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d945dadacecf?w=554&h=143&f=jpeg&s=50334)
 
第五步：AOP的配置<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d948b25909a3?w=554&h=53&f=jpeg&s=18090)
 
测试<br>
#### 1.4.6.2 注解方式的声明式事务管理
第一步：引入AOP的开发包<br>
第二步：恢复转账环境<br>
第三步：配置事务管理器<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d94bcec83e1c?w=554&h=53&f=jpeg&s=18858)
 
第四步：开启注解事务<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d94e6b7a5bd3?w=554&h=43&f=jpeg&s=20454)
 
第五步：在业务层添加注解<br>

![](https://user-gold-cdn.xitu.io/2019/7/26/16c2d9517ec96059?w=554&h=54&f=jpeg&s=20121)
 
 
