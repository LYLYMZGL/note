# Ajax请求SpringMVC，并且返回JSON格式的数据
1．  导入jar包<br/>
```java
jackson-annotations.jar
jackson-core.jar
jackson-databind.jar
```
2、在前端和服务端书写上相应代码<br/>
@ResponseBody修饰的方法，会将该方法的返回值，以一个Json数组的形式返回给前台。<br/>
服务端：<br/>
```java
@ResponseBody //告诉SpringMVC，此时返回的不是一个View页面，而是一个Ajax调用的返回值(Json数组)
       @RequestMapping(value="testJson")
       public List<Student> testJson() {
              //controller-service-dao
              //StudentService studentService=new StudentServiceImpl();
              //List<Student> students=studentService.queryAllStudent();
              //模拟调用service的查询操作
              Student stu1=new Student(1,"zs",23);
              Student stu2=new Student(2,"ls",45);
              Student stu3=new Student(3,"ww",56);
              List<Student> students=new ArrayList<>();
              students.add(stu1);
              students.add(stu2);
              students.add(stu3);
              return students;
       }
```
前端：服务端将返回值结果，以json数组的 形式传给了result<br/>
```js
<!-- 若下一行引入静态资源出错，则需要在springmvc.xml配置文件中加上<mvc:default-servlet-handler/>标签即可 -->
<script type="text/javascript" src="js/jquery.js"></script>
       <script type="text/javascript">
              $(document).ready(function(){
                     $("#testJson").click(function(){
                            //通过Ajax请求SpringMVC
                            $.post(
                                   //请求服务端
                                   "SpringMVCHandler/testJson",//服务器地址
                                   //{"name":"zs","age":23},
                                   //服务端处理完毕后的回调函数
                                   function(result){//result就相当于集合对象stduents，在加上@ResponseBody后，students实质上是一个json数组的格式
                                          for(var i=0;i<result.length;i++){ //遍历数组
                                                 alert(result[i].id+"-"+result[i].name+"-"+result[i].age);
                                          }
                                   }
                            );
                     });
              });
       </script>
``` 
测试按钮：<br/>
```jsp
<input type="button" value="testJson" id="testJson">
```
 
总结：前端通过Ajax请求服务端的Controller，Controller中对应的方法上使用了@ResponseBody注解修饰，使得返回的List集合中的数据变成一个Json数组，返回给前端界面，前端界面通过回调函数中的参数result来接收服务端传递过来的Json数组，并遍历Json数组中的元素，让其在前端页面通过提示框的形式依次显示出来。
