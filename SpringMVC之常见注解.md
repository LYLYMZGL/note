# SpringMVC之常见注解
## 1、@PathVariable AND @RequestParam
1)、REST风格传值使用@PathVariable<br/>
①在index.jsp中书写表单请求：<br/>
```java
<form action="SpringMVCHandler/testRest/1234" method="post">
              <input type="hidden" name="_method" value="PUT">
              <input type="submit" value="改">
</form>
```
②在Controller中书写相应方法：<br/>
```java
       @RequestMapping(value="testRest/{id}",method=RequestMethod.PUT)
//     @ResponseBody()
       public String testPut(@PathVariable("id") Integer id) {
              System.out.println("Put:改"+id);
              //调用Service层实现真正的增
              //视图解析器会将返回的字符串中加入一个前缀和一个后缀，最终的页面为/views/success.jsp
              return "redirect:/views/success.jsp";//默认使用了请求转发的跳转方式
       }
```
2)、普通风格传值使用@RequestParam<br/>
①在index.jsp中书写表单请求：<br/>
```java
<form action="SpringMVCHandler/testParam" method="get">
              <input type="text" name="uname">
              <input type="submit" value="查">
</form>
```
②在Controller中书写相应方法：<br/>
```java
       @RequestMapping(value="testParam")
       public String testParam(@RequestParam("uname") String name) {
//            方法参数里面的注解就等价于在Servlet中书写下面的代码:
//            String name=request.getParameter("uname");
              System.out.println("uname:"+name);
              return "success";
       }
```
2、分析
```java
       @RequestMapping(value="testParam")
       public String testParam(@RequestParam("uname") String name,@RequestParam(value="uage",required=false,defaultValue="23") Integer age) {
//            方法参数里面的注解就等价于在Servlet中书写下面的代码:
//            String name=request.getParameter("uname");
              System.out.println("uname:"+name);
              System.out.println("uage:"+age);
              return "success";
       }
```
`@RequestParam("uname") String name`：接收前台传递的值，等价于`request.getParameter(“uname”);`<br/>
`required=false` ：该属性不是必须的<br/>
`defaultValue="23"` ：默认值为23<br/>
3、获取请求头信息：@RequestHeader<br/>
①在index.jsp中书写超链接：
```java
<a href="SpringMVCHandler/testRequestHeader">testRequestHeader</a>
```
②在Controller中书写相应方法：<br/>
```java
    @RequestMapping(value="testRequestHeader")
    public String testRequestHeader(@RequestHeader("Accept-Language") String al)
    {
           System.out.println("Accept-Language："+al);
           return "success";
    }
```
通过`@RequestHeader("Accept-Language") String al`获取请求头中的Accept-Language值，并将值保存在al中。<br/>
4、获取Cookie的值：@CookieValue<br/>
前置知识：服务端在第一次接受客户端的请求时，会给该客户端分配一个session（该session包含一个sessionID），并且服务端在第一次响应客户端时，将该sessionID赋值给JSESSIONID并传递给客户端的Cookie中。<br/>
①在index.jsp中书写超链接：
```java
<a href="SpringMVCHandler/testCookieValue">testCookieValue</a>
```
②在Controller中书写相应方法：<br/>
```java
    @RequestMapping(value="testCookieValue")
           public String testCookieValue(@CookieValue("JSESSIONID") String jsessionId) {
                  System.out.println("JSESSIONID="+jsessionId);
                  return "success";
    }
```
小结：
SpringMVC处理各种参数的流程/逻辑：<br/>
请求：前端发送请求a -> @RequestMapping(“a”)<br/>
处理请求中的参数xyz:<br/>
@RequestMapping(“a”)<br/>
public String aa(@Xxx注解(“xyz”) 参数类型  xyz){<br/>
 <br/>
}<br/>
5、使用对象（实体类Student接受请求参数）：<br/>
对比：之前在Servlet中接收表单数据时，需要使用很多的request.getParameter()方法来获取表单中的数据，且还要实例化一个Address和Student对象，并对他们使用setXxx()方法进行赋值，然后才可以打印出相关的数据。<br/>
而SpringMVC中则可以使用对象（实体类来接收请求参数），极大的简化了这个流程。<br/>
1）、先编写两个实体类<br/>
①Student类<br/>
![](https://user-gold-cdn.xitu.io/2019/3/5/1694d99ad3e184ab?w=553&h=425&f=png&s=87186)
②Address类<br/>
![](https://user-gold-cdn.xitu.io/2019/3/5/1694d9afd872a77e?w=553&h=306&f=png&s=82465)
2）、在index.jsp中书写一个表单请求<br/>
```java
    <form action="SpringMVCHandler/testObjectProperties" method="post">
                  id:<input type="text" name="id">
                  name:<input type="text" name="name">
                  homeAddress:<input type="text" name="address.homeAddress">
                  schoolAddress:<input type="text" name="address.schoolAddress">
                  <input type="submit" value="提交">
    </form>
```
3）、在Controller中书写相应方法：<br/>
```java
    @RequestMapping(value = "testObjectProperties")
           public String testObjectProperties(Student student) {
                  System.out.println("id:" + student.getId() + " name:" + student.getName() + " homeAddress:"
                                + student.getAddress().getHomeAddress() + " schoolAddress:" + student.getAddress().getSchoolAddress());
                  return "success";
           }
```


