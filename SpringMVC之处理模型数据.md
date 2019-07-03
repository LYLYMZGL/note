# SpringMVC之处理模型数据
如果跳转时需要带上数据：View(视图)、Model(数据)，则可以使用以下方式：
`ModelAndView、ModelMap、Map、Model`-------数据放在了request作用域
`@SessionAttributes、@ModelAttribute`。<br/>
1）、ModelAndView<br/>
①在index.jsp页面中书写一个超链接：<br/>
```java
<a href="SpringMVCHandler/testModelAndView">testModelAndView</a>
<br>
```
②在Controller中书写测试方法：<br/>
```java
       //前面的测试方法中都只是返回一个视图，而该方法返回的既有数据又有视图
       @RequestMapping(value="testModelAndView")
       public ModelAndView testModelAndView() {//ModelAndView:既有数据又有视图
              //加视图
              ModelAndView mv=new ModelAndView("success");//在该方式下，视图解析器也会给其加上前缀和后缀(/views/success.jsp)
             
              //加数据
              Address address=new Address();
              address.setHomeAddress("DL");
              address.setSchoolAddress("pdsu");
              Student stu=new Student();
              stu.setId(12);
              stu.setName("zs");
              stu.setAddress(address);
             
              mv.addObject("student", stu);//自动将数据加入到request域里，相当于request.setAttribute("student", stu);
             
              return mv;
       }
```
③在success.jsp页面中使用EL表达式显示相应的数据：<br/>
```java
<h1>welcome to springmvc</h1><br>
${requestScope.student.id} - ${requestScope.student.name} -${requestScope.student.address}<br>
```
2）、ModelMap<br/>
①在index.jsp中书写超链接：<br/>
```java
<a href="SpringMVCHandler/testModelMap">testModelMap</a>
<br>
```
②在Controller中书写测试方法：<br/>
```java
       @RequestMapping(value="testModelMap")
       public String testModelMap(ModelMap mm) {
              //加数据
              Address address=new Address();
              address.setHomeAddress("DL");
              address.setSchoolAddress("pdsu");
              Student stu=new Student();
              stu.setId(12);
              stu.setName("zs");
              stu.setAddress(address);
             
              mm.put("student2", stu);//数据放置在request域里
             
              return "success";
       }
```
③在success.jsp页面中使用EL表达式显示相应的数据：<br/>
```java
${requestScope.student2.id } - ${requestScope.student2.name } -${requestScope.student2.address}<br>
```
3）、Map<br/>
①在index.jsp中书写超链接：<br/>
```java
<a href="SpringMVCHandler/testMap">testMap</a>
<br>
```
②在Controller中书写测试方法：<br/>
```java
       @RequestMapping(value="testMap")
       public String testMap(Map<String,Object> map) {
              //加数据
              Address address=new Address();
              address.setHomeAddress("DL");
              address.setSchoolAddress("pdsu");
              Student stu=new Student();
              stu.setId(12);
              stu.setName("zs");
              stu.setAddress(address);
             
              map.put("student3", stu);//数据放置在request域里
              return "success";
       }
```
③在success.jsp页面中使用EL表达式显示相应的数据：<br/>
```java
${requestScope.student3.id } - ${requestScope.student3.name } -${requestScope.student3.address}<br>
```
4）、Model<br/>
①在index.jsp中书写超链接：<br/>
```java
<a href="SpringMVCHandler/testModel">testModel</a>
<br>
```
②在Controller中书写测试方法：<br/>
```java
       @RequestMapping(value="testModel")
       public String testModel(Model model) {
              //加数据
              Address address=new Address();
              address.setHomeAddress("DL");
              address.setSchoolAddress("pdsu");
              Student stu=new Student();
              stu.setId(12);
              stu.setName("zs");
              stu.setAddress(address);
             
              model.addAttribute("student4", stu);//数据放置在request域里
              return "success";
       }
```
③在success.jsp页面中使用EL表达式显示相应的数据：<br/>
```java
${requestScope.student4.id } - ${requestScope.student4.name } -${requestScope.student4.address}<br>
```
5）、@SessionAttributes<br/>
若要将数据放入session中，则可以使用@SessionAttributes()注解<br/>
a、value属性：<br/>
①在Controller所在类的上方加入@SessionAttributes注解<br/>
```java
@SessionAttributes(value= {"student3","student4"})//若要在request中存放student4和student3对象，则同时将该对象加入session中
```
②在success.jsp中显示session中的信息<br/>
```java
${sessionScope.student.id } - ${sessionScope.student.name } -${sessionScope.student.address}<br>
      
${sessionScope.student2.id } - ${sessionScope.student2.name } -${sessionScope.student2.address}<br>
      
${sessionScope.student3.id } - ${sessionScope.student3.name } -${sessionScope.student3.address}<br>
      
${sessionScope.student4.id}-${sessionScope.student4.name} -${sessionScope.student4.address}<br>
```
b、types属性：<br/>
①在Controller所在类的上方加入@SessionAttributes注解<br/>
```java
@SessionAttributes(types=Student.class)//若要在request中存放student类型的对象，则同时将该类型的对象加入session中
```
②在success.jsp中显示session中的信息<br/>
```java
${sessionScope.student.id } - ${sessionScope.student.name } -${sessionScope.student.address}<br>
      
${sessionScope.student2.id } - ${sessionScope.student2.name } -${sessionScope.student2.address}<br>
      
${sessionScope.student3.id } - ${sessionScope.student3.name } -${sessionScope.student3.address}<br>
      
${sessionScope.student4.id}-${sessionScope.student4.name} -${sessionScope.student4.address}<br>
```
6）、@ModelAttribute：<br/>
应用场景：<br/>
I）.经常在更新的时候使用<br/>
II）.在不改变原有代码的基础上，插入一个新方法。<br/>
执行流程：在任何一次请求前都会先执行一次@ModelAttribute修饰的方法。且通过map.put()来向目标方法传递数据，默认情况下：map.put()方法的key就是方法参数类型的首字母小写；若不是，则应在目标方法的参数部分加上@ModelAttribute注解加以说明。
注：@ModelAttribute 会在该类的每个方法执行前均被执行一次，因此使用的时候需要注意。<br/>
 
操作：将ID为31的学生名字从zs修改为ls。执行修改操作时，应先查询后修改<br/>
①在index.jsp中书写表单请求：<br/>
```java
    <form action="SpringMVCHandler/testModelAttribute" method="post">
                  id:<input type="text" name="id">
                  name:<input type="text" name="name">
                  <input type="submit" value="修改">
    </form>
```
②在Controller中书写查询和修改的测试方法：<br/>
```java
        //查询
       @ModelAttribute//在任何一次请求前都会先执行一次@ModelAttribute修饰的方法
       public void queryStudentById(Map<String, Object> map) {
              //模拟三层查询数据库的操作
              Student stu=new Student();
              stu.setId(31);
              stu.setName("zs");
              stu.setAge(23);
              //通过以下语句自动的将数据传递给testModelAttribute方法的参数
//            map.put("student", stu);//约定：map的key就是方法参数类型的首字母小写（默认的情况）
              //若map中的key与方法参数类型的首字母小写不一致，则在testModelAttribute方法参数里加上@ModelAttribute注解加以说明
              map.put("stu", stu);
       }
      
       //修改 zs--ls
       @RequestMapping(value="testModelAttribute")
       public String testModelAttribute(@ModelAttribute("stu") Student stu) {
              stu.setName("ls");//将名字修改为ls
              System.out.println("id:"+stu.getId()+" name:"+stu.getName()+" age:"+stu.getAge());
              return "success";
       }
 
```
