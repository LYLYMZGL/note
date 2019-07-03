# SpringMVC之原生Servlet API
在SpringMVC中使用原生态的ServletAPI:<br/>
在Controller中书写一个测试方法，在方法的参数部分声明原生态的Servlet API **（不用再使用new运算符实例化，这个操作SpringMVC已经默认自动的帮我们做好了）**，声明完后即可在测试方法中使用原生态的Servlet API。<br/>
①在index.jsp中书写一个超链接：<br/>
```java
<a href="SpringMVCHandler/testServletAPI">testServletAPI</a>
<br>
```
②在Controller中书写一个测试方法，并在方法的参数部分声明相对应的Servlet API：<br/>
```java
       //如果想在SpringMVC中使用原生态的ServletAPI，则在方法的参数中直接声明出该对象即可使用(实例化的过程SpringMVC会帮我们自动做好)
       @RequestMapping(value="testServletAPI")
       public String testServletAPI(HttpServletRequest request,HttpServletResponse response) {
              System.out.println("HttpServletRequest:"+request+" HttpServletResponse:"+response);
              return "success";
       }
 
```
