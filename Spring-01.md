## 1.2 Spring4学习路线
 
Spring第一天：Spring的概述、SpringIOC入门（XML）、Spring的Bean管理、Spring属性注入<br>
Spring第二天：Spring的IOC的注解方式、Spring的AOP开发（XML）<br>
Spring第三天：Spring的AOP的注解开发、Spring的声明式事务、JdbcTemplate<br>

## 1.3 Spring的概述
### 1.3.1 Spring的概述
 
#### 1.3.1.1什么是Spring
![](https://user-gold-cdn.xitu.io/2019/7/21/16c13e9c23b4d8bb?w=553&h=97&f=jpeg&s=40414)
 
Spring叫做SE/EE开发的一站式框架。<br>
一站式框架：有EE开发的每一层解决方案。<br>
WEB层：SpringMVC<br>
Service层：Spring的Bean管理，Spring声明式事务<br>
DAO层：Spring的Jdbc模板，Spring的ORM模板。<br>
#### 　1.3.1.2 为什么学习Spring
![](https://user-gold-cdn.xitu.io/2019/7/21/16c13ea71663278d?w=554&h=374&f=jpeg&s=193271)
 
#### 1.3.1.3 Spring的版本
Spring3.x 和Spring4.x<br>
### 1.3.2 Spring的入门（IOC）
#### 1.3.2.1 什么是IOC
IOC：Inversion of Control（控制反转）<br>
控制反转：将对象的创建权反转给（交给）Spring。<br>
#### 1.3.2.2 下载Spring的开发包
[官网](http://spring.io/)
#### 1.3.2.3 解压Spring的开发包
![](https://user-gold-cdn.xitu.io/2019/7/21/16c13ebabe361a26?w=554&h=134&f=jpeg&s=36862)
docs：Spring的开发规范和API<br>
libs：Spring的开发的jar包和源码<br>
schema：Spring的配置文件的约束<br>
#### 1.3.2.4 创建web项目，引入jar包
![](https://user-gold-cdn.xitu.io/2019/7/21/16c13ec3e5ee5807?w=539&h=397&f=jpeg&s=81779)

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13ec90cf4b998?w=471&h=336&f=jpeg&s=89800)
#### 1.3.2.5 创建接口和类

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13ecf2f573799?w=295&h=153&f=jpeg&s=29259)
 

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13ed196c7f480?w=493&h=261&f=jpeg&s=60828) 

 
问题：<br>
如果底层的实现切换了，需要修改源代码，能不能不修改程序源代码对程序进行扩展？<br>

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13ed6c833b162?w=554&h=337&f=jpeg&s=76853)
 
#### 1.3.2.6 将实现类交给Spring管理
在spring的解压路径下spring-framework-4.2.4.RELEASE\docs\spring-framework-reference\html\xsd-configuration.html<br>

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13edeaa660f86?w=554&h=90&f=jpeg&s=34045)
 
#### 1.3.2.7 编写测试类

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13ee3c9f36f10?w=554&h=103&f=jpeg&s=26845)
 
#### 1.3.2.8 IOC和DI
IOC：控制反转，将对象的创建权反转给了Spring<br>
DI：依赖注入，前提必须有IOC的环境，Spring管理这个类的时候将这个类的依赖的属性注入（设置）进来。<br>
面向对象的时候<br>
依赖<br>
```java
Class A{
 
}
Class B{
Public void xxx(A a){
 
}
}
```
继承:is a<br>
```java
Class A{
 
}
Class B extends A{
 
}
```
聚合:has a<br>
## 1.4 Spring的工厂类
### 1.4.1 Spring的工厂类
#### 1.4.1.1 Spring工厂类的结构图

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13ef35c792004?w=482&h=397&f=jpeg&s=116871)
 
ApplicationContext继承BeanFactory。
#### 1.4.1.2 BeanFactory：老版本的工厂类
BeanFactory：调用getBean的时候，才会生成类的实例。
#### 1.4.1.3 ApplicationContext：新版本的工厂类
ApplicationContext：加载配置文件的时候，就会将Spring管理的类都实例化。<br>
ApplicationContext有两个实现类<br>
ClassPathXmlApplicationContext:加载类路径下的配置文件<br>
FileSystemXmlApplicationContext:加载文件系统下的配置文件<br>
 
## 1.5 Spring的配置
### 1.5.1 XML的提示配置
#### 1.5.1.1 Schema的配置

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13effebdbf55a?w=551&h=486&f=jpeg&s=99347)
 
### 1.5.2 Bean的相关的配置
#### 1.5.2.1 `<bean>`标签的id和name的配置
id ：使用了约束中的唯一约束。里面不能出现特殊字符的。<br>
name ：没有使用约束中的唯一约束（理论上可以出现重复的，但是实际开发不能出现的）。里面可以出现特殊字符。<br>
Spring和Struct1框架整合的时候，才会体现出id和name的区别，否则通常情况下，两者相同<br>
`<bean name=”/user” class=””/>`
注：在<bean>标签中，使用id或者name指定一个名字，都可以在ApplicationContext的getBean()方法中的参数值指定为id或者name的值，并生成相应的接口对应的实现类。<br>
### 1.5.2.2 Bean的生命周期的配置（了解）
init-method：Bean被初始化的时候执行的方法<br>
destroy-method：Bean被销毁的时候执行的方法（Bean是单例创建，工厂关闭）<br>
注：当要执行销毁方法的时候，必须使用ApplicationContext实现类创建的对象才有close方法。<br>
### 1.5.2.3 Bean的作用范围的配置（重点）
scope：Bean的作用范围<br>
singleton：默认的，Spring会采用单例模式创建这个对象。<br>
prototype：多例模式。（Structs2和Spring整合一定会用到）<br>
request：应用在web项目中，Spring创建这个类以后，将这个类存入到request范围中。<br>
session：应用在web项目中，Spring创建这个类以后，将这个类存入到session范围中。<br>
globalsession：应用在web项目中，必须在porlet环境下使用。但是如果没有这种环境，相当于session。<br>
## 1.6 Spring的Bean管理（XML方式）
### 1.6.1 Spring的Bean的实例化方式（了解）
Bean已经都交给Spring管理，Spring创建这些类的时候，有几种方式：<br>
#### 1.6.1.1无参构造方法的方式（默认）
l 编写类

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13f1c642e511c?w=576&h=249&f=jpeg&s=43520)
 
l 编写配置

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13f2139ea4103?w=577&h=42&f=jpeg&s=24784)
 
#### 1.6.1.2 静态工厂实例化的方式
l 编写Bean2的静态工厂

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13f288d8bea69?w=577&h=232&f=jpeg&s=49810)
 
l 配置

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13f2bb27a1f6e?w=577&h=29&f=jpeg&s=16300)
 
#### 1.6.1.3 实例工厂实例化的方式
l Bean3的实例工厂

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13f436d7b05ac?w=576&h=257&f=jpeg&s=52844)
 
l 配置

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13f460e4eb5df?w=577&h=43&f=jpeg&s=27946)
 
### 1.6.2 Spring的属性注入
#### 1.6.2.1构造方法的方式的属性注入
构造方法的属性注入

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13f50225f324a?w=529&h=104&f=jpeg&s=47152)
 
#### 1.6.2.2 Set方法的方式的属性注入
 
Set方法的属性注入

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13f54613335f3?w=553&h=101&f=jpeg&s=51015)
 
Set方法设置对象类型的属性

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13f5ca4a57910?w=554&h=110&f=jpeg&s=44352)
 
#### 1.6.2.3 P名称空间的属性注入（Spring2.5以后）
通过引入p名称空间完成属性的注入：<br>
写法：<br>
普通属性    p:属性名=”值”<br>
对象属性    p:属性名-ref=”值”<br>
p名称空间的引入<br>

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13f635324569c?w=554&h=100&f=jpeg&s=57381)
 
使用p名称空间

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13f6756041d01?w=554&h=57&f=jpeg&s=29131)
 
#### 1.6.2.4 SpEL的属性注入（Spring3.0以后）
SpEL：Spring Expression Language，Spring的表达式语言。<br>
语法：<br>
`
#{SpEL}
`

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13f6f8902eb3f?w=553&h=185&f=jpeg&s=78819) 
### 1.6.3 集合类型属性注入
#### 1.6.3.1 配置
```xml
<!-- Spring的集合属性的注入 -->
<!-- 注入数组类型 -->
<bean id="collectionBean" class="com.itheima.spring.demo5.CollectionBean">
<!-- 数组类型 -->
<property name="arrs">
<list>
<value>王东</value>
<value>赵洪</value>
<value>李冠希</value>
</list>
</property>
<!-- 注入list集合 -->
<property name="list">
<list>
<value>李兵</value>
<value>赵汝河</value>
<value>邓艾</value>
</list>
</property>
<!-- 注入Set集合 -->
<property name="set">
<set>
<value>aaa</value>
<value>bbb</value>
<value>ccc</value>
</set>
</property>
<!-- 注入Map集合 -->
<property name="map">
<map>
<entry key="aaa" value="111"/>
<entry key="bbb" value="222"/>
<entry key="ccc" value="333"/>
</map>
</property>
</bean>
```
## 1.7 Spring的分模块开发的配置
### 1.7.1 分模块配置
#### 1.7.1.1 在加载配置文件的时候，加载多个

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13f7b24704104?w=554&h=30&f=jpeg&s=21584)
 
#### 1.7.1.2 在一个配置文件中引入多个配置文件

![](https://user-gold-cdn.xitu.io/2019/7/21/16c13f7e48644024?w=523&h=49&f=jpeg&s=13524)
 
