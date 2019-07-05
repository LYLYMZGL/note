# SpringMVC实现文件上传
SpringMVC实现文件上传和Servlet方式实现的本质一样，都是通过`commons-io.jar`和`commons-fileupload.jar`来实现。<br/>
SpringMVC可以简化文件上传的代码，但是必须满足条件：实现MultipartResolver接口；而该接口的实现类SpringMVC也已经提供了CommonsMultipartResolver<br/>
 
具体步骤：（直接使用CommonsMultipartResolver实现上传）<br/>
①   导入jar包<br/>
```java
commons-io.jar
commons-fileupload.jar
```
②   在springmvc.xml文件中配置CommonsMultipartResolver，将其加入到SpringIOC容器中<br/>
```xml
       <!--
       配置CommonsMultipartResolver，将其加入SpringIOC容器，用于实现文件上传
       SpringIOC容器在初始化时，会自动寻找一个id="multipartResolver"的bean。若有，则将其自动加入到IOC容器中。
       因此该标签的id值是固定的
       -->
       <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
              <!-- 设置上传文件的默认编码 -->
              <property name="defaultEncoding" value="UTF-8"></property>
              <!--
              设置最大可以上传的单个文件大小，单位为字节(Byte)(1024字节=1KB)
              若value=-1，则代表无限制
              -->
              <property name="maxUploadSize" value="104857600"></property>
       </bean>
```
③在index.jsp文件中书写相应的测试代码：<br/>
```jsp
<!-- 文件上传中method="post"和enctype="multipart/form-data"值为固定值 -->
<form action="SpringMVCHandler/testUpload" method="post" enctype="multipart/form-data">
              <input type="file" name="file">
              描述:<input type="text" name="desc">
              <input type="submit" value="上传">
       </form>
       <br>
```
④在Controller中书写文件上传的处理方法：<br/>
```java
/*
        * 文件上传
        * 通过@RequestParam("desc")注解来获取文件的描述，
        * 以及通过@RequestParam("file") MultipartFile file来将前端指定的文件传递到file中保存
        */
       @RequestMapping(value="testUpload")
       public String testUpload(@RequestParam("desc") String desc,@RequestParam("file") MultipartFile file) {
              System.out.println("文件描述："+desc);
              //jsp中上传的文件：file
              InputStream input=null;
              OutputStream out=null;
              try {
                     input=file.getInputStream();//获取输入流
                     String fileName = file.getOriginalFilename();//获取文件上传时的初始名称
                     out=new FileOutputStream("e:/"+fileName);//通过输出流，指定文件上传后保存的路径
                     byte[] bs=new byte[1024];
                     int len;
                     /*
                      * 边读取文件中信息，边写入到上传位置所对应的文件中
                      */
                     while((len=input.read(bs))!=-1) {
                            out.write(bs, 0, len);
                     }
                     System.out.println("上传成功！");
              } catch (IOException e) {
                     e.printStackTrace();
              } finally {
                     if(input!=null) {
                            try {
                                   input.close();
                            } catch (IOException e) {
                                   e.printStackTrace();
                            }
                     }
                     if(out!=null) {
                            try {
                                   out.close();
                            } catch (IOException e) {
                                   e.printStackTrace();
                            }
                     }
              }
              return "success";
       }
```
