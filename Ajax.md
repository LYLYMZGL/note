# 1  Ajax原理及JS方式实现
1、Ajax： “Asynchronous Javascript And XML”（异步 JavaScript 和 XML）<br>
异步刷新：如果网页中某一个地方需要修改，异步刷新可以使：只刷新需要修改的部分，而页面中的其他地方保持不变。例如：百度搜索框、视频的刷新。<br>
实现方式：<br>
1）JS<br>
通过XMLHttpRequest对象实现Ajax。<br>
①XMLHttpRequest对象的方法：<br>
a、open(提交方式（get、post）,服务器地址,true)：与服务端建立连接<br>
第三个参数中可以有两个取值，分别为true和false。true代表使用异步刷新、false代表使用全局刷新。<br>
b、send()：发送数据<br>
get：send(null)   （请求参数的值以“?请求参数名=请求参数值”的方式放置在open方法的服务器地址的后面）<br>
post：send(请求参数值)<br>
c、setRequestHeader(header,value)：设置请求头信息<br>
get：不需要设置此方法<br>
post：需要设置<br>
如果请求元素中包含了文件上传：setRequestHeader(“Content-Type”,”multipart/form-data”);<br>
如果不包含文件上传：setRequestHeader(“Content-Type”,”application/x-www-form-urlencoded”);<br>
②XMLHttpRequest对象的属性：<br>
readyState：请求状态（只有状态为4，代表请求完毕）<br>

status：响应状态（只有200代表响应正常）<br>

onreadystatechange：回调函数<br>
responseText：响应格式为String<br>
responseXML：响应格式为XML<br>
 
代码实现：<br>
①index.jsp：<br>
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<script type="text/javascript">
       // post方式
       function register(){
              var mobile=document.getElementById("mobile").value;
              // 通过Ajax异步方式请求服务器
              // 若声明变量时，不适用var关键字，则表示该变量为全局变量，在整个页面内都有效（反之，则为局部变量）
              xmlHttpRequest=new XMLHttpRequest();
             
              // 设置xmlHttpRequest对象的回调函数(该地方是对方法的引用，故不需要书写())
              xmlHttpRequest.onreadystatechange=callback;
             
              // 建立一个连接（请求方式，服务器地址，是否异步刷新）
              xmlHttpRequest.open("post","MobileServlet",true);
              // POST方式需要设置请求头信息（没有文件上传）
              xmlHttpRequest.setRequestHeader("Content-Type","application/x-www-form-urlencoded");
              // 发送数据
              xmlHttpRequest.send("mobile="+mobile);// k=v
       }
      
       // get方式
       function registerGet(){
              var mobile=document.getElementById("mobile").value;
              // 通过Ajax异步方式请求服务器
              // 若声明变量时，不适用var关键字，则表示该变量为全局变量，在整个页面内都有效（反之，则为局部变量）
              xmlHttpRequest=new XMLHttpRequest();
             
              // 设置xmlHttpRequest对象的回调函数(该地方是对方法的引用，故不需要书写())
              xmlHttpRequest.onreadystatechange=callback;
             
              // 建立一个连接（请求方式，服务器地址，是否异步刷新）
              xmlHttpRequest.open("get","MobileServlet?mobile="+mobile,true);
              // get方式不需要设置请求头信息
              //xmlHttpRequest.setRequestHeader("Content-Type","application/x-www-form-urlencoded");
              // 发送数据
              xmlHttpRequest.send(null);// k=v
       }
      
       // 定义回调函数（接收服务端的返回值）
       function callback(){
              if(xmlHttpRequest.readyState==4 && xmlHttpRequest.status==200){
                     // 接收服务端返回的数据
                     var data=xmlHttpRequest.responseText;//服务端返回值为String格式
                     if(data=="true"){
                            alert("该号码已存在，请更换！");
                     }else{
                            alert("注册成功！");
                     }
              }
       }
</script>
<title>Insert title here</title>
</head>
<body>
       手机:<input type="text" id="mobile"><br>
       <input type="button" value="注册" onclick="registerGet()">
</body>
</html>
```
②MobileServlet.java：
```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
              // 设置post方式的编码（get方式的编码在server.xml中设置URIEncoding）
              request.setCharacterEncoding("UTF-8");
              response.setCharacterEncoding("UTF-8");
              response.setContentType("text/html;charset=UTF-8");
              String mobile = request.getParameter("mobile");
              // 假设此时数据库中只有一个号码（18888888888）
              // 使用此方式能够防止空指针异常（相对于mobile.equals("18888888888")此方式）
              PrintWriter out = response.getWriter();
              if("18888888888".equals(mobile)) {
                     out.write("true");
              }else {
                     out.write("false");
              }
       }
 
       protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
              doGet(request, response);
       }
``` 
2）JQuery（推荐）<br>
# 2  JQuery方式的Ajax-ajax、get、post
JQuery：推荐<br>
1、<br>
```jsp
$.ajax({
        请求方式:get/post,   （或type: get/post）
        url:服务器地址,
        data:请求数据,       
        success:function(result,testStatus){
              
        },
        error:function(xhr,errorMessage,e){
              
        }
});
```
方法中的参数都是可选的，若不需要可以不写。<br>
代码实现：<br>
①index2.jsp：<br>
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<script type="text/javascript" src="js/jquery.js"></script>
<script type="text/javascript">
       function register(){
              // 获取输入框的值
              var mobile=$("#mobile").val();
              $.ajax({
                     请求方式:"post",
                     url:"MobileServlet",
                     data:"mobile="+mobile,
                     success:function(result){
                            if(result=="true"){
                                   alert("该号码已存在，请更换！");
                            }else{
                                   alert("注册成功！");
                            }
                     },
                     error:function(errorMessage,e){
                            alert("系统错误！");
                     }
              });
       }
</script>
<title>Insert title here</title>
</head>
<body>
       手机:<input type="text" id="mobile">
       <input type="button" value="注册" onclick="register()">
</body>
</html>
```
②MobileServlet.java：
```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
              // 设置post方式的编码（get方式的编码在server.xml中设置URIEncoding）
              request.setCharacterEncoding("UTF-8");
              response.setCharacterEncoding("UTF-8");
              response.setContentType("text/html;charset=UTF-8");
              String mobile = request.getParameter("mobile");
              // 假设此时数据库中只有一个号码（18888888888）
              // 使用此方式能够防止空指针异常（相对于mobile.equals("18888888888")此方式）
              PrintWriter out = response.getWriter();
              if("18888888888".equals(mobile)) {
                     out.write("true");
              }else {
                     out.write("false");
              }
       }
 
       protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
              doGet(request, response);
       }
```
2、每个参数的顺序不能够打乱（因为没有key-value）<br>
```jsp
$.get(
       服务器地址,        
       请求数据,
       function(result){        （该方式只有success）
             
},
预期返回值类型（”xml”,”json”,”text”）（该值可以省略）
);
```
代码实现：<br>
①index2.jsp：<br>
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<script type="text/javascript" src="js/jquery.js"></script>
<script type="text/javascript">
       function register(){
              // 获取输入框的值
              var mobile=$("#mobile").val();
              $.get(
                            "MobileServlet",
                            "mobile="+mobile,
                            function(result){
                                   if(result=="true"){
                                          alert("该号码已存在，请更换！");
                                   }else{
                                          alert("注册成功！");
                                   }
                            },
                            "text"
                     );
       }
</script>
<title>Insert title here</title>
</head>
<body>
       手机:<input type="text" id="mobile">
       <input type="button" value="注册" onclick="register()">
</body>
</html>
```
②MobileServlet.java：
```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
              // 设置post方式的编码（get方式的编码在server.xml中设置URIEncoding）
              request.setCharacterEncoding("UTF-8");
              response.setCharacterEncoding("UTF-8");
              response.setContentType("text/html;charset=UTF-8");
              String mobile = request.getParameter("mobile");
              // 假设此时数据库中只有一个号码（18888888888）
              // 使用此方式能够防止空指针异常（相对于mobile.equals("18888888888")此方式）
              PrintWriter out = response.getWriter();
              if("18888888888".equals(mobile)) {
                     out.write("true");
              }else {
                     out.write("false");
              }
       }
 
       protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
              doGet(request, response);
       }
``` 
3、每个参数的顺序不能够打乱（因为没有key-value）<br>
```jsp
$.post(
       服务器地址,        
       请求数据,
       function(result){        （该方式只有success）
             
        },
        预期返回值类型（”xml”,”json”,”text”） （该值可以省略）   
        );
```
代码实现：<br>
①index2.jsp：<br>
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<script type="text/javascript" src="js/jquery.js"></script>
<script type="text/javascript">
       function register(){
              // 获取输入框的值
              var mobile=$("#mobile").val();
              $.post(
                            "MobileServlet",
                            "mobile="+mobile,
                            function(result){
                                   if(result=="true"){
                                          alert("该号码已存在，请更换！");
                                   }else{
                                          alert("注册成功！");
                                   }
                            },
                            "text"
                     );
       }
</script>
<title>Insert title here</title>
</head>
<body>
       手机:<input type="text" id="mobile">
       <input type="button" value="注册" onclick="register()">
</body>
</html>
```
②MobileServlet.java：<br>
```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
              // 设置post方式的编码（get方式的编码在server.xml中设置URIEncoding）
              request.setCharacterEncoding("UTF-8");
              response.setCharacterEncoding("UTF-8");
              response.setContentType("text/html;charset=UTF-8");
              String mobile = request.getParameter("mobile");
              // 假设此时数据库中只有一个号码（18888888888）
              // 使用此方式能够防止空指针异常（相对于mobile.equals("18888888888")此方式）
              PrintWriter out = response.getWriter();
              if("18888888888".equals(mobile)) {
                     out.write("true");
              }else {
                     out.write("false");
              }
       }
 
       protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
              doGet(request, response);
       }
``` 
# 3  JQuery方式的Ajax-load和getJson
4、该方法直接将服务端的返回值，返回给了$(JQuery选择器)所选择的元素。<br>
```jsp
$(JQuery选择器).load(
       服务器地址,        
       请求数据,
       function(result){        （该方式只有success）（通常function可以省略）
             
},
);
```
代码实现：<br>
①index2.jsp：<br>
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<script type="text/javascript" src="js/jquery.js"></script>
<script type="text/javascript">
       function register(){
              // 获取输入框的值
              var mobile=$("#mobile").val();
              $("#tip").load(
                            "MobileServlet",
                            "mobile="+mobile
              );
       }
</script>
<title>Insert title here</title>
</head>
<body>
       手机:<input type="text" id="mobile">
       <input type="button" value="注册" onclick="register()">
       <span id="tip"></span>
</body>
</html>
```
②MobileServlet.java：<br>
```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
              // 设置post方式的编码（get方式的编码在server.xml中设置URIEncoding）
              request.setCharacterEncoding("UTF-8");
              response.setCharacterEncoding("UTF-8");
              response.setContentType("text/html;charset=UTF-8");
              String mobile = request.getParameter("mobile");
              // 假设此时数据库中只有一个号码（18888888888）
              // 使用此方式能够防止空指针异常（相对于mobile.equals("18888888888")此方式）
              PrintWriter out = response.getWriter();
              if("18888888888".equals(mobile)) {
                     //out.write("true");
                     out.write("此号码已存在！");
              }else {
                     //out.write("false");
                     out.write("注册成功！");
              }
       }
 
       protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
              doGet(request, response);
       }
``` 
5、如果客户端是getJSON,则需要以JSON格式返回数据<br>
```jsp
$.getJSON(
       服务器地址,
       JSON格式的请求数据,  （多个参数使用 {“name”:”zs”,”age”:23} ）
       function(result){
      
},
“json”    （该值可以省略）
);
```
代码实现：<br>
①index2.jsp：<br>
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<script type="text/javascript" src="js/jquery.js"></script>
<script type="text/javascript">
       function register(){
              // 获取输入框的值
              var mobile=$("#mobile").val();
/*
              // JSON对象（key:value，key都是字符串，value为任意）
              var student={"name":"zs","age":23};
              //    打印JSON对象（通过对象 .key获取value值）
              //alert(student.name+"--"+student.age);
             
              // JSON数组
              var students=[
                     {"name":"zs","age":23},
                     {"name":"ls","age":25},
                     {"name":"ww","age":27}
              ];
           //    打印JSON数组
              alert(students[1].name+"--"+students[1].age);
           */
           $.getJSON(
                 "MobileServlet",
                 {"mobile":mobile},
                 function(result){
                        if(result.msg=="true"){
                               alert("已存在，注册失败! ");
                        }else{
                               alert("注册成功！");
                        }
                 }
           );
       }
</script>
<title>Insert title here</title>
</head>
<body>
       手机:<input type="text" id="mobile">
       <input type="button" value="注册" onclick="register()">
       <span id="tip"></span>
</body>
</html>
```
②MobileServlet.java：<br>
```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
              // 设置post方式的编码（get方式的编码在server.xml中设置URIEncoding）
              request.setCharacterEncoding("UTF-8");
              response.setCharacterEncoding("UTF-8");
              response.setContentType("text/html;charset=UTF-8");
              String mobile = request.getParameter("mobile");
              // 假设此时数据库中只有一个号码（18888888888）
              // 使用此方式能够防止空指针异常（相对于mobile.equals("18888888888")此方式）
              PrintWriter out = response.getWriter();
              if("18888888888".equals(mobile)) {
                     //out.write("true");
                     //out.write("此号码已存在！");
                     //     如果客户端是getJSON,则需要以JSON格式返回数据
                     out.write("{\"msg\":\"true\"}");// "{\"msg\":\"true\"}"
              }else {
                     //out.write("false");
                     //out.write("注册成功！");
//                   如果客户端是getJSON,则需要以JSON格式返回数据
                     out.write("{\"msg\":\"false\"}");// "{\"msg\":\"false\"}"
              }
       }
 
       protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
              doGet(request, response);
       }
```
# 4  Ajax处理JSON对象<br>
使用JSON对象必须导入jar包<br>
1、导入jar包<br>
```java
com.springsource.org.apache.commons.beanutils-1.7.0.jar
com.springsource.org.apache.commons.collections-3.2.1.jar
com.springsource.org.apache.commons.lang-2.1.0.jar
com.springsource.org.apache.commons.logging-1.1.1.jar
ezmorph-1.0.6.jar
json-lib-2.4-jdk15.jar
```
2、使用<br>
1）、JSON中只有一个对象<br>
①index2.jsp：<br>
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<script type="text/javascript" src="js/jquery.js"></script>
<script type="text/javascript">
       function register(){
              // 获取输入框的值
              var mobile=$("#mobile").val();
           $.getJSON(
                 "MobileServlet",
                 {"mobile":mobile},
                 function(result){
                        if(result.msg=="true"){
                               alert("已存在，注册失败! ");
                        }else{
                               alert("注册成功！");
                        }
                 }
           );
       }
      
       function testJson(){
              $.getJSON(
                     "JsonServlet",
                     {"name":"zs","age":23},
                     function(result){
                            //JavaScript需要通过eval()函数，将返回值转为一个JavaScript能够识别的JSON对象
                            var jsonStudent=eval(result.stu1);//json对象 .key
                            alert(jsonStudent.name+"--"+jsonStudent.age);
                     }
              );
       }
</script>
<title>Insert title here</title>
</head>
<body>
       手机:<input type="text" id="mobile">
       <input type="button" value="注册" onclick="register()">
       <span id="tip"></span>
       <input type="button" value="测试JSON" onclick="testJson()">
</body>
</html>
```
②JsonServlet.java：<br>
```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
              request.setCharacterEncoding("UTF-8");
              response.setCharacterEncoding("UTF-8");
              response.setContentType("text/html;charset=UTF-8");
              PrintWriter out = response.getWriter();
             
              Student stu1=new Student();
              stu1.setName("zs");
              stu1.setAge(23);
             
              JSONObject json=new JSONObject();
              json.put("stu1", stu1);
             
              //  print可以返回的类型更多
              out.print(json);// 返回JSON对象 ，key="stu1",value=stu1对象
              out.close();
       }
 
       protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
              doGet(request, response);
       }
```
2）、JSON中有多个对象：<br>
①index2.jsp：<br>
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<script type="text/javascript" src="js/jquery.js"></script>
<script type="text/javascript">
       function register(){
              // 获取输入框的值
              var mobile=$("#mobile").val();
           $.getJSON(
                 "MobileServlet",
                 {"mobile":mobile},
                 function(result){
                        if(result.msg=="true"){
                               alert("已存在，注册失败! ");
                        }else{
                               alert("注册成功！");
                        }
                 }
           );
       }
      
       //     JSON中只有一个对象的情况
       /*function testJson(){
              $.getJSON(
                     "JsonServlet",
                     {"name":"zs","age":23},
                     function(result){
                            //JavaScript需要通过eval()函数，将返回值转为一个JavaScript能够识别的JSON对象
                            var jsonStudent=eval(result.stu1);//json对象 .key
                            alert(jsonStudent.name+"--"+jsonStudent.age);
                     }
              );
       }*/
      
       //     JSON中有多个对象
       function testJson(){
              $.getJSON(
                     "JsonServlet",
                     {"name":"zs","age":23},
                     function(result){
                            //JavaScript需要通过eval()函数，将返回值转为一个JavaScript能够识别的JSON对象
                            //result={"stu1":stu1,"stu2":stu2,"stu3":stu3}
                            var json=eval(result);
                            //$.each(遍历的对象，遍历的函数(下标索引，当前对象) )     ；当前对象可以用element或this
                            $.each(json,function(i,element){
                                   alert(this.name+"---"+this.age);
                            });
                     }
              );
       }
      
</script>
<title>Insert title here</title>
</head>
<body>
       手机:<input type="text" id="mobile">
       <input type="button" value="注册" onclick="register()">
       <span id="tip"></span>
       <input type="button" value="测试JSON" onclick="testJson()">
</body>
</html>
```
②JsonServlet.java：<br>
```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
              request.setCharacterEncoding("UTF-8");
              response.setCharacterEncoding("UTF-8");
              response.setContentType("text/html;charset=UTF-8");
              PrintWriter out = response.getWriter();
             
              String name = request.getParameter("name");
              String age = request.getParameter("age");
              System.out.println(name+"---"+age);
             
              Student stu1=new Student();
              stu1.setName("zs");
              stu1.setAge(25);
             
              Student stu2=new Student();
              stu2.setName("ls");
              stu2.setAge(33);
             
              Student stu3=new Student();
              stu3.setName("ww");
              stu3.setAge(44);
             
              JSONObject json=new JSONObject();
              json.put("stu1", stu1);
              json.put("stu2", stu2);
              json.put("stu3", stu3);
             
              //  print可以返回的类型更多
              //out.print(json);// 返回JSON对象，key="stu1",value=stu1对象
              out.print(json);// 返回JSON对象      {"stu1":stu1,"stu2":stu2,"stu3":stu3}
              out.close();
       }
 
       protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
              doGet(request, response);
       }
 ```
