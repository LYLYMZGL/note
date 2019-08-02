#  第一章 java语言概述
软件开发：系统软件---应用软件

人机交互方式：①图形化界面 ②命令行方式

常用的DOS命令：
```java
dir 列出当前目录下的文件以及文件夹
md  创建目录
rd  删除目录
cd  进入指定目录
cd..退回到上一级目录
cd\ 退回到根目录
del 删除文件
eixt推出dos命令行
```
java技术体系平台<br>
Java SE标准版　　支持面向桌面级应用的Java平台<br>
Java EE企业版　　为开发企业环境下的应用程序提供的一套解决方案<br>
Java ME小型版　　支持Java程序运行在移动终端上的平台<br>
Java Card 　　　　支持一些Java小程序运行在小内存设备上的平台<br>
Java语言的特点<br>
面向对象<br>
健壮性<br>
跨平台性(Write once,Run Anywhere) Java虚拟机(JVM)<br>
6、JDK(Java Development Kit    Java开发工具包) <br>
      包含java开发工具，也包含了JRE<br>
   JRE(Java Runtime Environment     Java运行环境)<br>
	   包括Java虚拟机(JVM)<br>
   安装JDK并配置path环境变量<br>
命令行运行Java程序<br>
将Java代码编写到扩展名为.java的文件中<br>
通过javac命令对该java文件进行编译<br>
通过java命令对生成的class文件进行运行<br>
```java
public class hellojava{
public static void main(String[] args){
      System.out.println(“hello world!”);
}
}
```
源文件以.java结尾<br>
源文件中可以有多个class声明的类<br>
类中可以有主方法(即main()方法)，其格式是固定的：public static void main(String[] args){}<br>
main()方法是程序的入口，方法内是程序的执行部分。<br>
一个源文件中只能有一个声明为public的类，同时要求此类的类名与源文件名一致<br>
每个语句都以“；”结束<br>
执行程序：编译 javac.exe  编译完，生成诸多个.class字节码文件，运行 java.exe<br>
8、注释：
```java
①单行注释// 
②多行注释/* */ （多行注释不能够嵌套）
③文档注释/** */  解析：javadoc -d 文件目录名 -author -version 源文件名.java
```
9、JDK提供的关于旗下所有的包、类的文档API<br>
