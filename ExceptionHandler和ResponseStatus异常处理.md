# ExceptionHandler和ResponseStatus异常处理
异常处理：<br/>
SpringMVC要想实现异常处理，必须实现HandlerExceptionResolver接口。<br/>
 
该接口的每一个实现类，都是异常的一种处理方式：<br/>
ExceptionHandlerExceptionResolver 类：主要提供了@ExceptionHandler注解，并通过该注解处理异常。<br/>
 
A、<br/>
@ExceptionHandler标识的方法的参数，必须为异常类型(Throwable或其子类)，不能包含其他类型的参数<br/>
具体步骤：<br/>
在控制器中书写相应的代码：<br/>
1)、将错误信息打印到控制台<br/>
```java
       @ExceptionHandler({ArithmeticException.class})
       public String handlerException1(Exception e) {
              System.out.println(e);
              return "error";
       }
```       
2)、将错误信息同时输出到控制台和页面<br/>
```java
/*
        * 相当于catch
        * @ExceptionHandler标识的方法的参数，必须为异常类型(Throwable或其子类)，
        * 不能包含其他类型的参数
        */
       //该方法可以捕获本类中抛出的ArithmeticException异常和ArrayIndexOutOfBoundsException异常
       @ExceptionHandler({ArithmeticException.class,ArrayIndexOutOfBoundsException.class})
       public ModelAndView handlerException(Exception e) {
              ModelAndView mv=new ModelAndView("error");
              System.out.println(e);
              mv.addObject("e", e);
              return mv;
       }
``` 
异常处理路径：最短优先<br/>
如果有方法抛出了ArithmeticException，而该类中有两个对应的异常处理方法（ArithmeticException和Exception）。则该异常会被ArithmeticException所在的异常处理方法所处理，而不会被Exception所在的异常处理方法处理。即优先级为最短优先。
<br/> 
@ExceptionHandler默认只能捕获当前类中的异常方法。<br/>
如果发生异常的方法和处理异常的方法不再同一个类中，则需要在该异常处理方法所在的类加上@ControllerAdvice注解。<br/>
 
异常处理类如下：<br/>
```java
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.servlet.ModelAndView;
 
/*
 * 不是控制器，仅仅是用于处理异常的类
 */
@ControllerAdvice//加上该注解表示该类就是异常处理类
public class MyExceptionHandler {
       @ExceptionHandler({Exception.class})
       public ModelAndView testExceptionHandler(Exception e) {
              ModelAndView mv=new ModelAndView("error");
              System.out.println("该类可以处理所有异常，异常信息如下：");
              System.out.println(e);
              mv.addObject("e", e);//将异常信息放置到request域中
              return mv;
       }
}
```
总结：<br/>
①如果一个方法用于处理异常，并且只处理当前类中的异常。则需要在该方法前加@ExceptionHandler注解。<br/>
②如果一个方法用于处理异常，并且处理所有类中的异常。则需要在该类前加@ControllerAdvice注解，在异常处理方法前加@ExceptionHandler注解。<br/>
 
B、<br/>
ResponseStatusExceptionResolver类：提供了@ResponseStatus注解，并通过该注解来自定义异常显示页面。<br/>
 
具体代码：<br/>
①自定义异常显示页面（注解在类上）：<br/>
```java
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;
/*
 * value就是相当于404，reason相当于Message
 * value必须使用枚举实现HttpStatus.NOT_FOUND
 */
@ResponseStatus(value=HttpStatus.FORBIDDEN,reason="数组越界")
public class MyArrayIndexOutofBoundsException extends Exception{//自定义异常
      
}
②自定义异常显示页面（注解在方法上）：
@ResponseStatus(value=HttpStatus.CONFLICT,reason="测试")
       @RequestMapping("testResponseStatus")
       public void testResponseStatus() {
             
       }
``` 
C、<br/>
DefaultHandlerExceptionResolver类：SpringMVC在一些常见异常的基础上（300、404、500），新增加了一些异常，例如：<br/>
```java
* @see #handleNoSuchRequestHandlingMethod
 * @see #handleHttpRequestMethodNotSupported
 * @see #handleHttpMediaTypeNotSupported
 * @see #handleMissingServletRequestParameter
 * @see #handleServletRequestBindingException
 * @see #handleTypeMismatch
 * @see #handleHttpMessageNotReadable
 * @see #handleHttpMessageNotWritable
 * @see #handleMethodArgumentNotValidException
 * @see #handleMissingServletRequestParameter
 * @see #handleMissingServletRequestPartException
 * @see #handleBindException
``` 
handleHttpRequestMethodNotSupported：如果SpringMVC的处理方法限制为post方式。但实际请求为get，则会触发此异常显示的界面。<br/>
 
D、<br/>
SimpleMappingExceptionResolver类：通过配置来实现异常的处理。<br/>
具体代码：<br/>
在springmvc.xml配置文件中书写相应代码：<br/>
```xml
<!-- SimpleMappingExceptionResolver:以配置的方式处理异常 -->
       <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
              <!-- 如果发生异常，异常对象会被保存在exceptionAttribute的value值中，且该参数会被放置到request域中 -->
              <!-- 若该属性不写，则默认为<property name="exceptionAttribute" value="exception"></property> -->
              <!-- <property name="exceptionAttribute" value="e"></property> -->
              <property name="exceptionMappings">
                     <props>
                            <!-- 相当于catch(ArithmeticException e){跳转：error} -->
                            <prop key="java.lang.ArithmeticException">
                                   error
                            </prop>
                            <prop key="java.lang.NullPointerException">
                                   error
                            </prop>
                     </props>
              </property>
       </bean>
```       
