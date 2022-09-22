---
layout: post
title: "SpringMVC（2）功能与数据处理"
date:   2022-9-19
tags: [SpringMVC]
comments: true
author: Bo
---

 ## 一、功能

### 1.1 重定向与转发

**通过SpringMVC来实现转发和重定向 - 无需视图解析器；**

测试前，将视图解析器注释掉

现在我们没有视图解析器了，再测一下之前的代码：

```java
@Controller
public class ModelTest {
    @RequestMapping("/m1")
    public String test1(Model model){
        model.addAttribute("msg","modelTest");
        return "test";
    }
```

我们这样已经404访问不到了，因为我们没有视图解析器，需要吧前缀和后缀都加上：

```java
return "/WEB-INF/jsp/test.jsp";
```

才能访问到，**因为第一个斜杠是表示在web目录下**

这样转发是不明显的

```java
//转发二
return "forward:/WEB-INF/jsp/test.jsp";
```

这样转发是比较好的

对应的重定向：

```java
//重定向
return "redirect:/test.jsp";
```

注意：**重定向访问不到WEB-INF下的资源**，这个test.jsp只能在web目录下

那么如果我们有视图解析器呢：

```java
  @RequestMapping("/rsm2/t1")
   public String test1(){
       //转发
       return "index";
   }
  @RequestMapping("/rsm2/t2")
   public String test2(){
       //重定向
       return "redirect:/index.jsp";
  }
```

转发可以不用写全路径，**但是重定向必须**

重定向 , 不需要视图解析器 , 本质就是重新请求一个新地方嘛 , 所以注意路径问题.

## 二、数据处理

### 2.1 处理提交数据

#### 2.1.1 提交的域名称和处理方法的参数名一致

提交数据 : http://localhost:8080/hello?name=bo

处理方法 :

```java
@RequestMapping("/hello")
public String hello(String name){
   System.out.println(name);
   return "hello";
}
```

直接传入String name 参数

 http://localhost:8080/hello URL不传参，后台打印null

 http://localhost:8080/hello?name=bo 后台打印bo



你也可以用restful风格来写

```java
@Controller
public class HelloController {
    @RequestMapping("/hello/{name}")
    public String hello( @PathVariable String name){
        System.out.println(name);
        return "hello";//返回视图，会被视图解析器处理
    }
}
```

这样，访问http://localhost:8080/Springmvc01_war_exploded/hello/bo，后台输出bo

#### 2.1.2 提交的域名称和处理方法的参数名不一致

提交数据 : http://localhost:8080/hello?username=bo

这时url上是username，方法的参数是name

处理方法 :

```java
//@RequestParam("username") : username提交的域的名称 .
@RequestMapping("/hello")
public String hello(@RequestParam("username") String name){
   System.out.println(name);
   return "hello";
}
```

后台输出 : bo

#### 2.1.3 提交的是一个对象

要求提交的表单域和对象的属性名一致  , 参数使用对象即可

1、实体类

```java
public class User {
   private int id;
   private String name;
   private int age;
   //构造
   //get/set
   //tostring()
}
```

2、url要提交的数据 : http://localhost:8080/Springmvc01_war_exploded/t2?name=bo&id=1&age=19

如何处理：

```java
    @GetMapping("/t2")
    public String test2(User user){
        System.out.println(user);
        return "test";
    }
}
```

- 前端传递的参数，如果参数名在方法上，则可以直接使用
- 前端传递的对象user，则匹配user中的字段名，都能匹配到则OK，若某个字段与user本身参数名不一致，则为null：

比如我用这个url请求http://localhost:8080/Springmvc01_war_exploded/t2?username=bo&id=1&age=19

我们看到这时用的是username，与user中属性name不一致，运行后，后台打印`User{id=1，name=null，age=19}`



### 2.2 数据显示到前端

#### 2.2.1 通过ModelAndView

我们最前面就是如此。

```java
public class ControllerTest1 implements Controller {

   public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
       //返回一个模型视图对象
       ModelAndView mv = new ModelAndView();
       mv.addObject("msg","ControllerTest1");
       mv.setViewName("test");
       return mv;
  }
}
```

#### 2.2.2 第二种 : 通过ModelMap

但我们一般用model

#### 2.2.3 Model

Model

```java
@RequestMapping("/t2/hello")
public String hello(@RequestParam("username") String name, Model model){
   //封装要显示到视图中的数据
   //相当于req.setAttribute("name",name);
   model.addAttribute("msg",name);
   System.out.println(name);
   return "test";
}
```

#### 2.* 遇到的问题

我在前面测试的时候输入中文测试，浏览器显示的数据乱码了

该怎么办？

第一种方法自己写一个过滤器解决，但这是极端情况使用的方法

我们直接用SpringMVC给我们提供了一个过滤器 , 可以在web.xml中配置 .

```xml
<filter>
   <filter-name>encoding</filter-name>
   <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
   <init-param>
       <param-name>encoding</param-name>
       <param-value>utf-8</param-value>
   </init-param>
</filter>
<filter-mapping>
   <filter-name>encoding</filter-name>
   <url-pattern>/*</url-pattern>
</filter-mapping>
```



### 三、Json交互处理

### 3.1 Json

> 什么是JSON？

- JSON(JavaScript Object Notation, JS 对象标记) 是一种轻量级的数据交换格式，目前使用特别广泛。采用完全独立于编程语言的**文本格式**来存储和表示数据。

在 JavaScript 语言中，一切都是对象。因此，任何JavaScript 支持的类型都可以通过 JSON 来表示，例如字符串、数字、对象、数组等。看看他的要求和语法格式：

- 对象表示为键值对，数据由逗号分隔
- 花括号保存对象
- 方括号保存数组



**JSON 键值对**是用来保存 JavaScript 对象的一种方式，和 JavaScript 对象的写法也大同小异，键/值对组合中的键名写在前面并用双引号 "" 包裹，使用冒号 : 分隔，然后紧接着值：

```
{"name": "bo"}
{"age": "19"}
{"sex": "男"}
```



先写一个JavaScript说明：

```html
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <script type="text/javascript">
        //编写一个JavaScript对象
        const user = {
            name: "bo",
            age: 19,
            gender: "男"
        };
        //将js对象转换为json对象
        const json = JSON.stringify(user);
        console.log(json);

        //将json对象转换为JavaScript对象
        const obj = JSON.parse(json);
        console.log(obj);

        console.log(user);
    </script>
</head>
```

打开前端控制台查看：

![image-20220905121618745](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202209051216758.png)

看到分别是json对象和js对象



> Controller返回JSON数据

Jackson应该是目前比较好的json解析工具了

我们先新建一个module

我们这里使用Jackson，使用它需要导入它的jar包；

```xml
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-core -->
<dependency>
   <groupId>com.fasterxml.jackson.core</groupId>
   <artifactId>jackson-databind</artifactId>
   <version>2.13.4</version>
</dependency>
```

**导入包完成后记得去project structure去添加lib目录，把包全加进去。**

同样的，我们还要配置SpringMVC需要的配置

web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
        version="4.0">

   <!--1.注册servlet-->
   <servlet>
       <servlet-name>SpringMVC</servlet-name>
       <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
       <!--通过初始化参数指定SpringMVC配置文件的位置，进行关联-->
       <init-param>
           <param-name>contextConfigLocation</param-name>
           <param-value>classpath:springmvc-servlet.xml</param-value>
       </init-param>
       <!-- 启动顺序，数字越小，启动越早 -->
       <load-on-startup>1</load-on-startup>
   </servlet>

   <!--所有请求都会被springmvc拦截 -->
   <servlet-mapping>
       <servlet-name>SpringMVC</servlet-name>
       <url-pattern>/</url-pattern>
   </servlet-mapping>

   <filter>
       <filter-name>encoding</filter-name>
       <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
       <init-param>
           <param-name>encoding</param-name>
           <param-value>utf-8</param-value>
       </init-param>
   </filter>
   <filter-mapping>
       <filter-name>encoding</filter-name>
       <url-pattern>/</url-pattern>
   </filter-mapping>

</web-app>
```

springmvc-servlet.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:context="http://www.springframework.org/schema/context"
      xmlns:mvc="http://www.springframework.org/schema/mvc"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       https://www.springframework.org/schema/mvc/spring-mvc.xsd">

   <!-- 自动扫描指定的包，下面所有注解类交给IOC容器管理 -->
   <context:component-scan base-package="com.kuang.controller"/>

   <!-- 视图解析器 -->
   <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"
         id="internalResourceViewResolver">
       <!-- 前缀 -->
       <property name="prefix" value="/WEB-INF/jsp/" />
       <!-- 后缀 -->
       <property name="suffix" value=".jsp" />
   </bean>

</beans>
```

我们编写一个User的实体类，然后我们去编写我们的测试Controller；

```java
@Controller
public class UserController {
    @RequestMapping("/j1")
    @ResponseBody//它就不会走视图解析器会直接返回一个字符串
    public String json1(){
        User user = new User("bo",19,"男");
        return user.toString();
    }
```

然后配置Tomcat测试访问：

![image-20220905172232959](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202209051722197.png)

直接返回了对象，没有走视图解析器。**这个"?"是乱码问题。**

那么Jackson怎么解析呢？jackson有一个ObjectMapper对象，我们看下具体的用法

```java
@Controller
public class UserController {

   @RequestMapping("/json1")
   @ResponseBody
   public String json1() throws JsonProcessingException {
       //创建一个jackson的对象映射器，用来解析数据
       ObjectMapper mapper = new ObjectMapper();
       
       User user = new User("bo", 19, "男");
       //将我们的对象解析成为json格式
       String str = mapper.writeValueAsString(user);
       //由于@ResponseBody注解，这里会将str转成json格式返回；十分方便
       return str;
  }
}
```

再测试：

{"name":"bo","age":19,"gender":"?"}

发现返回的是json键值对

### 3.2 代码优化

先解决一下刚刚的乱码问题：

```java
@RequestMapping(value = "/j1",produces = "application/json;charset=utf-8")
```

这样就可以解决，但springmvc有更好的解决方法：

**乱码统一解决**

我们可以在springmvc的配置文件上添加一段消息StringHttpMessageConverter转换配置！

```xml
<mvc:annotation-driven>
   <mvc:message-converters register-defaults="true">
       <bean class="org.springframework.http.converter.StringHttpMessageConverter">
           <constructor-arg value="UTF-8"/>
       </bean>
       <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
           <property name="objectMapper">
               <bean class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
                   <property name="failOnEmptyBeans" value="false"/>
               </bean>
           </property>
       </bean>
   </mvc:message-converters>
</mvc:annotation-driven>
```

**返回json字符串统一解决**

在类上直接使用 **@RestController** ，这样子，里面所有的方法都只会返回 json 字符串了，**不用再每一个都添加@ResponseBody ！我们在前后端分离开发中，一般都使用 @RestController** ，十分便捷！

```java
@RestController
public class UserController {

   //produces:指定响应体返回类型和编码
   @RequestMapping(value = "/json1")
   public String json1() throws JsonProcessingException {
       //创建一个jackson的对象映射器，用来解析数据
       ObjectMapper mapper = new ObjectMapper();
       //创建一个对象
       User user = new User();
       //将我们的对象解析成为json格式
       String str = mapper.writeValueAsString(user);
       //由于@ResponseBody注解，这里会将str转成json格式返回；十分方便
       return str;
  }

}
```

启动tomcat测试，结果都正常输出！

mapper.writeValueAsString(user),括号里不仅可以是一个对象，也可以是集合

### 3.3 输出时间工具类

> 输出时间对象

增加一个新的方法

```java
@RequestMapping("/j3")
public String json3() throws JsonProcessingException {

   ObjectMapper mapper = new ObjectMapper();

   //创建时间一个对象，java.util.Date
   Date date = new Date();
   //将我们的对象解析成为json格式
   String str = mapper.writeValueAsString(date);
   return str;
}
```

运行结果 是一个时间戳，Jackson时间解析后默认的格式为timestamp，时间戳，是1970年1月1日到当前日期的毫秒数

我们自定义日期格式

```java
//不使用时间戳的方式
mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
//自定义日期格式对象
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
//指定日期格式
mapper.setDateFormat(sdf);

Date date = new Date();
String str = mapper.writeValueAsString(date);
```

**如果要经常使用的话，这样是比较麻烦的，我们可以将这些代码封装到一个工具类中；**

```java
package com.bo.utils;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;

import java.text.SimpleDateFormat;

public class JsonUtils {
   
   public static String getJson(Object object) {
       return getJson(object,"yyyy-MM-dd HH:mm:ss");
  }

   public static String getJson(Object object,String dateFormat) {
       ObjectMapper mapper = new ObjectMapper();
       //不使用时间差的方式
       mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
       //自定义日期格式对象
       SimpleDateFormat sdf = new SimpleDateFormat(dateFormat);
       //指定日期格式
       mapper.setDateFormat(sdf);
       try {
           return mapper.writeValueAsString(object);
      } catch (JsonProcessingException e) {
           e.printStackTrace();
      }
       return null;
  }
}
```

我们使用工具类，代码就更加简洁了！

```java
@RequestMapping("/j4")
public String json5() throws JsonProcessingException {
   Date date = new Date();
   String json = JsonUtils.getJson(date);
   return json;
}
```

