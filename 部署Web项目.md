# 1      项目配置说明
## 1.1 将项目部署到Tomcat服务器
**1、在Eclipse中右键单击项目，然后Export选择WAR file，生成项目的WAR文件。步骤如图所示：**

![](https://user-gold-cdn.xitu.io/2019/6/29/16ba3c4e2a0fb4d8?w=510&h=444&f=jpeg&s=27317)
**2、将打包好的文件放置在Tomcat的webapp目录下，并在server.xml配置文件中配置相关标签。如图所示：**

![](https://user-gold-cdn.xitu.io/2019/6/29/16ba3c55bd75a84c?w=1139&h=637&f=jpeg&s=72664)
在server.xml文件的Host标签内部，添加以下代码：
```xml
<Context docBase="D:\apache-tomcat-8.5.37-windows-x64\apache-tomcat-8.5.37\webapps\BookAppointment.war" path="/BookAppointment" reloadable="true"/>
```
 docBase:指向项目的根目录所在的路径,由于我将项目打成了war包,所以直接指向这个war包就可以了(我的项目名为:BookAppointment). <br/>
path:是一个虚拟目录,这里设置成了"BookAppointment",则启动Tomcat后,你将通过http://localhost:8080/BookAppointment/*.jsp   来访问项目的相关页面. <br/>
reloadable:如果设置为"true",则表示当你修改jsp文件后,不需要重启服务器就可以实现页面显示的同步. <br/>
可以这样理解:将docBase实际目录下的项目,映射到${tomcat.home}\webapps目录下的虚拟项目path(这里的配置指的是BookAppointment项目).<br/>
**3、启动tomcat。双击tomcat解压文件里bin目录下的startup.bat。**<br/>
**4、在浏览器中输入相应地址即可。如图所示：**

![](https://user-gold-cdn.xitu.io/2019/6/29/16ba3c611d3e1566?w=1354&h=728&f=jpeg&s=26969)
