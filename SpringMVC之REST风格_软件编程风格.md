# SpringMVC之REST风格_软件编程风格
1、介绍<br/>
请求方式：<br/>
GET：查<br/>
POST ：增<br/>
DELETE ：删<br/>
PUT：改<br/>
 
普通浏览器都只支持：GET和POST请求，而不支持DELETE和PUT请求。所以要通过过滤器来为普通浏览器增加DELETE和PUT请求方式。<br/>
SpringMVC中提供了一个过滤器：HiddenHttpMethodFilter过滤器<br/>
HiddenHttpMethodFilter过滤器工作过程：如果是一般的请求（如：GET、POST），则直接将其发送到服务端。而若是其他请求（如：DELETE、PUT），则过滤器将其过滤为DELETE或PUT请求，发给服务端。<br/>

2、过滤的条件：<br/>
①必须type=”hidden” name=”_method”，且value值为DELETE或PUT其中一个。<br/>
例如：
```java
<input type=”hidden” name=”_method” value=”DELETE/PUT”>
```
②请求方式必须为POST请求<br/>

3、SpringMVC中实现DELETE和PUT请求：<br/>
①增加过滤器<br/>
```java
 <!-- 配置过滤器 ，拦截一切请求（在filter过滤器中拦截所有请求为/*）-->
  <!-- 目的：给普通浏览器增加DELETE、PUT请求方式 -->
  <filter>
     <filter-name>HiddenHttpMethodFilter</filter-name>
     <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
  </filter>
  <filter-mapping>
     <filter-name>HiddenHttpMethodFilter</filter-name>
     <url-pattern>/*</url-pattern><!-- 拦截所有请求 -->
  </filter-mapping>
```
②在index.jsp页面中书写表单提交<br/>
```java
       <form action="SpringMVCHandler/testRest/1234" method="post">
              <input type="submit" value="增">
       </form>
      
       <form action="SpringMVCHandler/testRest/1234" method="post">
              <input type="hidden" name="_method" value="DELETE">
              <input type="submit" value="删">
       </form>
      
       <form action="SpringMVCHandler/testRest/1234" method="post">
              <input type="hidden" name="_method" value="PUT">
              <input type="submit" value="改">
       </form>
      
       <form action="SpringMVCHandler/testRest/1234" method="get">
              <input type="submit" value="查">
       </form>
 
```
注：<br/>
A.method必须是POST方式<br/>
B.通过隐藏域的value值，设置实际的请求方式DELETE、PUT<br/>
C.input 标签中的type必须为hidden、name必须为_method<br/>

③在Controller中书写测试方法
```java
       @RequestMapping(value="testRest/{id}",method=RequestMethod.POST)
       public String testPost(@PathVariable("id") Integer id) {
              System.out.println("POST:增"+id);
              //调用Service层实现真正的增
              //视图解析器会将返回的字符串中加入一个前缀和一个后缀，最终的页面为/views/success.jsp
              return "success";//默认使用了请求转发的跳转方式
       }
      
       @RequestMapping(value="testRest/{id}",method=RequestMethod.GET)
       public String testGet(@PathVariable("id") Integer id) {
              System.out.println("GET:查"+id);
              //调用Service层实现真正的增
              //视图解析器会将返回的字符串中加入一个前缀和一个后缀，最终的页面为/views/success.jsp
              return "success";//默认使用了请求转发的跳转方式
       }
      
       /**
        * 若当前项目是运行在Tomcat 8.0环境下，当过滤成DELETE和PUT请求，到达控制器后，返回时(forward)会报405的错误提示
        * 解决方案：
        * ①将Tomcat 8.0 改为Tomcat 7.0，在Tomcat 7.0下运行是正常的
        * ②在Controller里的方法上加上@ResponseBody()注解，返回一个字符串(无法返回原本的jsp页面)
        * ③将请求转发改为响应重定向(在显式写出转发或重定向时，需要加上文件所在的全部路径，视图解析器中的前缀和后缀对其无效)
        */
       @RequestMapping(value="testRest/{id}",method=RequestMethod.DELETE)
//     @ResponseBody()
       public String testDelete(@PathVariable("id") Integer id) {
              System.out.println("Delete:删"+id);
              //调用Service层实现真正的增
              //视图解析器会将返回的字符串中加入一个前缀和一个后缀，最终的页面为/views/success.jsp
              return "redirect:/views/success.jsp";
       }
      
       @RequestMapping(value="testRest/{id}",method=RequestMethod.PUT)
//     @ResponseBody()
       public String testPut(@PathVariable("id") Integer id) {
              System.out.println("Put:改"+id);
              //调用Service层实现真正的增
              //视图解析器会将返回的字符串中加入一个前缀和一个后缀，最终的页面为/views/success.jsp
              return "redirect:/views/success.jsp";
       }
```
注：<br/>
通过method=RequestMethod.DELETE 匹配具体的请求方式<br/>
此外，可以发现，当映射名相同时@RequestMapping(value="testRest/{id}")，可以通过method处理不同的请求。
