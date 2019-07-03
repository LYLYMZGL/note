# SpringMVC之RequestMapping映射
1、当运行SpringMVC程序时，在地址栏输入的地址是去映射@RequestMapping注解。且可以与类名与方法名不一致。<br/>
2、@RequestMapping注解通过value属性指定映射路径。若@RequestMapping注解中只有一个value属性，则可以省略value，直接在@RequestMapping注解中书写映射地址。例如：`@RequestMapping("main")`。<br/>
注：`@RequestMapping("main")`和`@RequestMapping("/main")`两种方式的含义是一样的。<br/>
3、@RequestMapping注解通过method属性指定请求方式。请求方式有四种，分别是get、post、delete、put。例如：`@RequestMapping(value="welcome",method=RequestMethod.POST)//映射`<br/>
4、@RequestMapping注解通过params属性指定请求参数，即设置“key=value”或“key!=value”的方式。<br/>
例如：`params={"username=zs","age!=23","!height"}`<br/>
解析：<br/>
1)、"username=zs"：代表必须有一个请求参数的name值为username，且在输入的时候，请求参数的值为zs。<br/>
2)、"age!=23"：有以下两种情况<br/>
①如果有一个参数的name="age"，则参数的值不能为23。<br/>
②没有一个参数的name="age"的请求参数。<br/>
3)、"!height"：代表不能有一个参数的name="height"。<br/>
5、@RequestMapping注解通过headers属性指定请求头。<br/>
例如：<br/>
```java
@RequestMapping(value="welcome2",headers={
"Accept=text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8",
"Accept-Encoding=gzip, deflate, br"})//映射
```
6、ant风格的请求路径（即支持使用通配符）<br/>
\? ：单个字符<br/>
\* ：任意个字符（0个或多个）<br/>
\*\* ：任意目录<br/>
例如：在Controller中：<br/>
```java
       @RequestMapping(value="welcome3/*/test")//映射
       public String welcome3(){
              //视图解析器会将返回的字符串中加入一个前缀和一个后缀，最终的页面为/views/success.jsp
              return "success";//默认使用了请求转发的跳转方式
       }
      
       @RequestMapping(value="welcome4/**/test")//映射
       public String welcome4(){
              //视图解析器会将返回的字符串中加入一个前缀和一个后缀，最终的页面为/views/success.jsp
              return "success";//默认使用了请求转发的跳转方式
       }
      
       @RequestMapping(value="welcome5/a?c/test")//映射
       public String welcome5(){
              //视图解析器会将返回的字符串中加入一个前缀和一个后缀，最终的页面为/views/success.jsp
              return "success";//默认使用了请求转发的跳转方式
       }
```
在index.jsp中：
```java
<a href="SpringMVCHandler/welcome3/asdf/test">get Test welcome3 *</a><br>
<a href="SpringMVCHandler/welcome4/asd/f/test">get Test welcome4 **</a><br>
<a href="SpringMVCHandler/welcome5/axc/test">get Test welcome5 ?</a><br>
```
7、@PathVariable注解<br/>
①在index.jsp界面中传值name2=zs<br/>
`
<a href="SpringMVCHandler/welcome6/zs">get Test welcome6 pathvariable</a><br>
`<br/>
②在Controller中先查询welcome6，在找{name2}，并查看是否匹配；@PathVariable("name2") ，若匹配则将值放入到参数name1中，并在控制台输出。<br/>
```java
@RequestMapping(value="welcome6/{name2}")//映射
public String welcome6(@PathVariable("name2") String name1){
       System.out.println(name1);
       //视图解析器会将返回的字符串中加入一个前缀和一个后缀，最终的页面为/views/success.jsp
       return "success";//默认使用了请求转发的跳转方式
}
```
