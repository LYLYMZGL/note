## 1.2 Spring的IOC的注解开发
### 1.2.1 Spring的IOC的注解开发的入门
#### 1.2.1.1 创建web项目，引入jar包
在Spring4的版本中，除了引入基本的开发包外，还需要引入aop的包。

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21dc025922953?w=425&h=166&f=jpeg&s=57275)
 
#### 1.2.1.2 引入Spring的配置文件
在src下创建applicationContext.xml<br>
引入约束：使用注解开发引入context约束。<br>
约束：spring-framework-4.2.4.RELEASE-dist\spring-framework-4.2.4.RELEASE\docs\spring-framework-reference\html\xsd-configuration.html<br>

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21dc82154ae11?w=553&h=110&f=jpeg&s=55111)
 
#### 1.2.1.3 创建接口和实现类
#### 1.2.1.4 开启Spring的组件扫描

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21dcc26e17be8?w=554&h=56&f=jpeg&s=32734)
 
#### 1.2.1.5 在类上添加注解

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21dd029e95677?w=554&h=131&f=jpeg&s=25576)
 
#### 1.2.1.6 编写测试类

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21dd554de89b7?w=553&h=81&f=jpeg&s=22915)
 
#### 1.2.1.7 注解方式设置属性的值
注解方式：使用注解方式，可以没有set方法的。<br>
属性如果有set方法，需要将属性注入的注解添加到set方法上。<br>
属性如果没有set方法，需要将属性注入的注解添加到属性上。<br>

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21dda4fd6d13e?w=554&h=227&f=jpeg&s=49501)
 
### 1.2.2 Spring的IOC的注解的详解
#### 1.2.2.1 @Component：组件
修饰一个类，将这个类交给Spring管理。<br>
这个注解有三个衍生注解（功能类似），修饰类。<br>
**@Controller：web层**<br>
**@Service：service层**<br>
**@Repository：dao层**<br>
#### 1.2.2.2 属性注入的注解
普通属性：<br>
**@Value：设置普通属性的值**<br>
对象类型属性：<br>
@Autowired：设置对象类型的属性的值，但是按照**类型**完成属性注入。<br>
我们习惯是按照名称完成属性注入；必须让@Autowired和@Qualifier一起使用完成按照名称属性注入。<br>
**@Resource：完成对象类型的属性的注入，按照名称完成属性注入。**<br>
#### 1.2.2.3 Bean的其他的注解
生命周期相关的注解（了解）<br>
@PostConstruct：初始化方法<br>
@PreDestroy：销毁方法<br>
Bean作用范围的注解<br>
**@Scope：作用范围**<br>
**singleton：默认单例**<br>
**prototype：多例**<br>
request<br>
session<br>
globalsession<br>
### 1.2.3 IOC的XML和注解开发比较
#### 1.2.3.1 XML和注解的比较
适用场景<br>
XML：可以适用任何场景。<br>
结构清晰，维护方便。<br>
注解：有些地方用不了，这个类不是自己提供的。<br>
开发方便。<br>
#### 1.2.3.2 XML和注解整合开发
**XML管理Bean，注解完成属性注入。**<br>
 
## 1.3 Spring的AOP的XML开发
### 1.3.1 AOP的概述
#### 1.3.1.1 什么是AOP

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e98e2a59c16?w=840&h=156&f=png&s=16695)
 
AOP：面向切面编程。AOP是OOP的扩展和延伸，解决OOP开发遇到问题。<br>
#### 1.3.1.2 Spring底层的AOP实现原理
动态代理<br>
JDK动态代理：只能对实现了接口的类产生代理。<br>
Cglib动态代理（类似于Javassist第三方代理技术）：对没有实现接口的类产生代理对象。生成子类对象。<br>
### 1.3.2 Spring的AOP底层实现（了解）
#### 1.3.2.1 JDK动态代理

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21df9f0d88c88?w=553&h=295&f=jpeg&s=63587)
 

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e00488f8099?w=554&h=137&f=jpeg&s=33856)
 
#### 1.3.2.2 Cglib动态代理
Cglib：第三方开源代码生成类库，动态添加类的属性和方法。<br>
 

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e0437115e10?w=554&h=370&f=jpeg&s=108104)
 

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e0905dd5de6?w=554&h=281&f=jpeg&s=88933)
 
### 1.3.3 Spring的AOP的开发（AspectJ的XML的方式）
#### 1.3.3.1 Spring的AOP的简介
AOP思想最早是由AOP联盟组织提出的。Spring使用这种思想最好的框架。<br>
Spring的AOP有自己实现的方式（非常繁琐）。AspectJ是一个AOP的框架。Spring引入AspectJ作为自身AOP的开发。<br>
Spring两套AOP开发方式<br>
Spring传统方式（弃用）<br>
**Spring基于AspectJ的AOP的开发（使用）。**<br>
#### 1.3.3.2 AOP开发中相关术语：

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e126c40c5b9?w=554&h=251&f=jpeg&s=78568)
 
### 1.3.4 Spring的AOP的入门（AspectJ的XML的方式）
#### 1.3.4.1 创建web项目，引入jar包
引入基本开发包<br>

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e180a60b7b3?w=554&h=118&f=jpeg&s=44946)
 
引入AOP开发的相关jar包<br>

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e1d62823e2b?w=554&h=93&f=jpeg&s=43385)
 
#### 1.3.4.2 引入Spring的配置文件
引入AOP的约束<br>

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e224b65b788?w=554&h=125&f=jpeg&s=45099)
 
#### 1.3.4.3 编写目标类并完成配置

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e27755af081?w=554&h=48&f=jpeg&s=21284)
 
#### 1.3.4.4 编写测试类：
Spring整合Junit单元测试<br>

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e2b3d3a30a2?w=554&h=347&f=jpeg&s=81723)
 
#### 1.3.4.5 编写一个切面类
编写切面<br>

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e2fe05315e3?w=531&h=200&f=jpeg&s=39950)
 
将切面类交给Spring<br>

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e32e5a3819e?w=554&h=37&f=jpeg&s=18185)
 
#### 1.3.4.6 通过AOP的配置实现

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e36ef4fbe52?w=553&h=96&f=jpeg&s=29167)
 
### 1.3.5 Spring中通知类型
#### 1.3.5.1 前置通知：在目标方法执行之前进行操作
前置通知：获得切入点信息<br>

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e3c8cd581b8?w=553&h=64&f=jpeg&s=37272)
 

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e4178e153f6?w=554&h=77&f=jpeg&s=23573)
 
#### 1.3.5.2 后置通知：在目标方法执行之后进行操作
后置通知：获得方法的返回值<br>

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e44cdbe170e?w=553&h=66&f=jpeg&s=37220)
 

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e487ccf3844?w=553&h=87&f=jpeg&s=31133)
 
#### 1.3.5.3 环绕通知：在目标方法执行之前和之后进行操作
环绕通知可以阻止目标方法的执行<br>

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e4de79009f3?w=554&h=64&f=jpeg&s=38941)
 

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e513c93673b?w=554&h=108&f=jpeg&s=38917)
 
#### 1.3.5.4 异常抛出通知：在程序出现异常的时候，进行的操作

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e55677899a2?w=553&h=68&f=jpeg&s=39679)
 

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e5a27fae321?w=554&h=128&f=jpeg&s=60920)
 
#### 1.3.5.5 最终通知：无论代码是否有异常，总是会执行

![](https://user-gold-cdn.xitu.io/2019/7/24/16c21e5d93856393?w=554&h=155&f=jpeg&s=63290)
 
#### 1.3.5.6 引介通知（不用会）
### 1.3.6 Spring的切入点表达式写法：
#### 1.3.6.1 切入点表达式语法：
基于execution的函数完成的<br>
语法<br>
[访问修饰符] 方法返回值 包名.类名.方法名(参数)<br>
注：“访问修饰符”可写可不写<br>
```java
public  void  com.itheima.spring.CustomerDao.save(..)
```
注：返回空类型的，指定包下的指定方法<br>
```java
*  *.*.*.*Dao.save(..)
```
注：返回任意类型的，任意包下**类名为某某Dao**的save方法<br>
```java
*  com.itheima.spring.CustomerDao+.save(..)
```
注：返回任意类型的，指定包下的**类及其子类**中的save方法<br>
```java
*  com.itheima.spring..*.*(..)
```
注：返回任意类型的，com.itheima包下的**spring包及其子包的所有类的所有方法**<br>
