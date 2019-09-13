# Docker
## 1、简介
Docker是一个开源的应用容器引擎；是一个轻量级容器技术；

Docker支持将软件编译成一个镜像；然后再将镜像中各种软件做好配置，将镜像发布出去，其他使用者可以直接使用这个镜像；

运行中的这个镜像称为容器，容器启动时非常快速的。

![](https://user-gold-cdn.xitu.io/2019/9/13/16d2ae901e9dfb14?w=187&h=173&f=png&s=20302)

![](https://user-gold-cdn.xitu.io/2019/9/13/16d2ae951ab55914?w=959&h=567&f=png&s=32867)

## 2、核心概念
docker主机(Host)：安装了Docker程序的机器（Docker直接安装在操作系统之上）；

docker客户端(Client)：连接Docker主机进行操作；

docker仓库(Registry)：用来保存各种打包好的软件镜像；

docker镜像(Images)：软件打包好的镜像；放在Docker仓库中；

docker容器(Container)：镜像启动后的实例称为一个容器；容器是独立运行的一个或一组应用；

![](https://user-gold-cdn.xitu.io/2019/9/13/16d2ae9eb456bd22?w=600&h=446&f=png&s=107893)
使用Docker的步骤：

1）、安装Docker；

2）、去Docker仓库找到这个软件对应的镜像；

3）、使用Docker运行这个镜像，这个镜像就会生成一个Docker容器；

4）、对容器的启动停止就是对软件的启动停止；

## 3、安装Docker
### 1)、安装Linux虚拟机
1）、VMWare（太重量级了）、VirtualBox（Oracle提供的比较小巧的虚拟机）；
安装VirtualBox；

2）、导入虚拟机文件centos7-atguigu.ova；

3）、双击启动Linux虚拟机；使用root/123456登录；

4）、使用Linux客户端连接Linux服务器进行命令操作；

5）、设置虚拟机网络；

桥接网络==选好网卡（wireless为无限网卡）===接入网线；

6）、设置好网络以后使用命令重启虚拟机的网络；(CentOS 7)
```shell
service network restart
```
7）、查看Linux的IP地址；
```shell
ip addr
```
8）、使用客户端连接；
### 2）、在虚拟机上安装Docker
步骤：
```shell
1、检查内核版本，必须是3.10及以上
uname -r
（若内核版本不超过3.10，则使用yum update更新，在重启虚拟机）
2、安装docker
sudo apt-get install -y docker.io
yum install docker
3、输入y确认安装
4、启动docker
[root@localhost ~]# systemctl start docker
[root@localhost ~]# docker -v
Docker version 1.13.1, build 7f2769b/1.13.1
5、开机启动docker
[root@localhost ~]# systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
6、停止docker
systemctl stop docker
```
## 4、Docker常用命令&操作
### 1）、镜像操作
|操作 |命令 |说明|
|-----|-----|----|
|检索 |docker search 关键字 eg：docker search redis| 我们经常去docker hub上检索镜像的详细信息，如镜像的TAG。|
|拉取 |docker pull 镜像名:tag| :tag是可选的，tag表示标签，多为软件的版本，默认是latest|
|列表 |docker images| 查看所有本地镜像|
|删除 |docker rmi image-id |删除指定的本地镜像|

https://hub.docker.com/
### 2）、容器操作
软件镜像（QQ安装程序）----运行镜像----产生一个容器（正在运行的软件，运行的QQ）；

步骤：
```shell
1、搜索镜像
[root@localhost ~]# sudo docker search tomcat
2、拉取镜像
[root@localhost ~]# docker pull tomcat
    若在拉取镜像时出现如下错误：
root@ubuntu:~# docker pull mysql:5.6
Error response from daemon: Get https://registry-1.docker.io/v2/library/mysql/manifests/5.6: Get https://auth.docker.io/token?scope=reposit
ory%3Alibrary%2Fmysql%3Apull&service=registry.docker.io: net/http: TLS handshake timeout
    解决方案：
   配置阿里云镜像加速器：
   1）、打开daemon.json文件并修改
   vim /etc/docker/daemon.json
   添加如下内容：
   {
  "registry-mirrors": ["https://vl8d92d5.mirror.aliyuncs.com"]
   } 
   保存并退出
   2）、重启docker服务
   systemctl restart docker
3、根据镜像启动容器
docker run --name mytomcat -d tomcat:latest
4、docker ps  
查看运行中的容器
5、 停止运行中的容器
docker stop  容器的id
6、查看所有的容器
docker ps -a
7、启动容器
docker start 容器id
8、删除一个容器
 docker rm 容器id
9、启动一个做了端口映射的tomcat
[root@localhost ~]# docker run -d -p 8888:8080 tomcat
-d：后台运行
-p: 将主机的端口映射到容器的一个端口    主机端口:容器内部的端口

10、为了演示简单关闭了linux的防火墙
service firewalld status ；查看防火墙状态
service firewalld stop：关闭防火墙
11、查看容器的日志
docker logs container-name/container-id

更多命令参看
https://docs.docker.com/engine/reference/commandline/docker/
可以参考每一个镜像的文档
```

### 3）、安装Mysql示例
```shell
docker pull mysql
```

错误的启动
```shell
[root@localhost ~]# docker run --name mysql01 -d mysql
42f09819908bb72dd99ae19e792e0a5d03c48638421fa64cce5f8ba0f40f5846

mysql退出了
[root@localhost ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                           PORTS               NAMES
42f09819908b        mysql               "docker-entrypoint.sh"   34 seconds ago      Exited (1) 33 seconds ago                            mysql01
538bde63e500        tomcat              "catalina.sh run"        About an hour ago   Exited (143) About an hour ago                       compassionate_
goldstine
c4f1ac60b3fc        tomcat              "catalina.sh run"        About an hour ago   Exited (143) About an hour ago                       lonely_fermi
81ec743a5271        tomcat              "catalina.sh run"        About an hour ago   Exited (143) About an hour ago                       sick_ramanujan


//错误日志
[root@localhost ~]# docker logs 42f09819908b
error: database is uninitialized and password option is not specified 
  You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD；
  这个三个参数必须指定一个
```

正确的启动
```shell
[root@localhost ~]# docker run --name mysql01 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
b874c56bec49fb43024b3805ab51e9097da779f2f572c22c695305dedd684c5f
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
b874c56bec49        mysql               "docker-entrypoint.sh"   4 seconds ago       Up 3 seconds        3306/tcp            mysql01
```

做了端口映射
```shell
[root@localhost ~]# docker run -p 3306:3306 --name mysql02 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
ad10e4bc5c6a0f61cbad43898de71d366117d120e39db651844c0e73863b9434
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
ad10e4bc5c6a        mysql               "docker-entrypoint.sh"   4 seconds ago       Up 2 seconds        0.0.0.0:3306->3306/tcp   mysql02
```

几个其他的高级操作
```shell
docker run --name mysql03 -v /conf/mysql:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
把主机的/conf/mysql文件夹挂载到 mysqldocker容器的/etc/mysql/conf.d文件夹里面
改mysql的配置文件就只需要把mysql配置文件放在自定义的文件夹下（/conf/mysql）
​
docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
指定mysql的一些配置参数
```

## 5、远程连接docker中Mysql遇到的问题（1251）
错误截图：

![](https://user-gold-cdn.xitu.io/2019/9/13/16d2aeef854b3bf0?w=707&h=127&f=png&s=6993)
原因：mysql 8.0 默认使用 caching_sha2_password 身份验证机制；客户端不支持新的加密方式。

**解决方案：**

1）、进入mysql容器内部
```shell
root@ubuntu:/home/liang# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                             
  NAMES
cd3eab2def17        mysql               "docker-entrypoint.s…"   6 hours ago         Up 14 minutes       0.0.0.0:3306->3306/tcp, 33060/tcp 
  mysql02
8334d084c645        tomcat              "catalina.sh run"        7 hours ago         Up 15 minutes       0.0.0.0:8888->8080/tcp            
  lucid_dewdney
root@ubuntu:/home/liang# docker exec -it mysql02 bash
```
2）、登录mysql
```shell
root@cd3eab2def17:/# mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 15
Server version: 8.0.17 MySQL Community Server - GPL

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```
3）、设置用户配置项

①查看用户信息
```shell
mysql> select host,user,plugin,authentication_string from mysql.user; 
+-----------+------------------+-----------------------+------------------------------------------------------------------------+
| host      | user             | plugin                | authentication_string                                                  |
+-----------+------------------+-----------------------+------------------------------------------------------------------------+
| %         | root             | caching_sha2_password | $A$005$xA%3?g@%lpQpcZC4Y4AYAQzKYg.IENOGsTzNYlRayqkYSYuAa/rOg/rGF0 |
| localhost | mysql.infoschema | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | mysql.session    | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | mysql.sys        | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | root             | caching_sha2_password | $A$005$Yc63%#Zg1NS3-Z]SUgTF0ZFe8KKZ7WhtJa6LJn3Hx2aOSfQp7JsL6Lg.fD7 |
+-----------+------------------+-----------------------+------------------------------------------------------------------------+
5 rows in set (0.00 sec)
```
备注：host为 % 表示不限制ip localhost表示本机使用 plugin非mysql_native_password 则需要修改密码

②修改加密方式
```shell
mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
Query OK, 0 rows affected (0.09 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.04 sec)
```
然后再查看用户信息
```shell
mysql> select host,user,plugin,authentication_string from mysql.user;
+-----------+------------------+-----------------------+------------------------------------------------------------------------+
| host      | user             | plugin                | authentication_string                                                  |
+-----------+------------------+-----------------------+------------------------------------------------------------------------+
| %         | root             | mysql_native_password | *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9                              |
| localhost | mysql.infoschema | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | mysql.session    | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | mysql.sys        | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | root             | caching_sha2_password | $A$005$Yc63%#Zg1NS3-Z]SUgTF0ZFe8KKZ7WhtJa6LJn3Hx2aOSfQp7JsL6Lg.fD7 |
+-----------+------------------+-----------------------+------------------------------------------------------------------------+
5 rows in set (0.00 sec)
```
4）、测试：连接成功

![](https://user-gold-cdn.xitu.io/2019/9/13/16d2af11e87d396d?w=481&h=548&f=png&s=16691)
