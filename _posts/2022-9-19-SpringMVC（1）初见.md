---
layout: post
title: "SpringMVC（1）初见"
date:   2022-9-19
tags: [SpringMVC]
comments: true
author: Bo
---

## 一、一些基础配置与认识

### 1.1 MVC架构

MVC：model模型（dao，service）  view视图（前端）  controller控制器（servlet）

是一种架构模式

### 1.2 第一个springmvc程序

首先创建一个maven项目，导入依赖，new module，加入web支持，配置web.xml,**注册DispatcherServlet**:

```xml
<!--1.注册DispatcherServlet-->
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--关联一个springmvc的配置文件:springmvc-servlet.xml-->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc-servlet.xml</param-value>
    </init-param>
    <!--启动级别-1-->
    <load-on-startup>1</load-on-startup>
</servlet>

<!--/ 匹配所有的请求；（不包括.jsp）-->
<!--/* 匹配所有的请求；（包括.jsp）-->
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

然后我们会发现classpath:springmvc-servlet.xml这一行爆红，因为我们还没有写这个配置文件,在resources下新建**springmvc-servlet.xml**：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```

然后我们要配置一些bean，这些bean都是spring写好的，我们复制过来就行：

1.添加 处理映射器

```xml
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
```

2.添加 处理器适配器

```xml
<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
```

3.添加 视图解析器

```xml
<!--视图解析器:DispatcherServlet给他的ModelAndView-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="InternalResourceViewResolver">
   <!--前缀-->
   <property name="prefix" value="/WEB-INF/jsp/"/>
   <!--后缀-->
   <property name="suffix" value=".jsp"/>
</bean>
```

视图解析器可以方便我们写文件路径，比如重定向和转发时，我们就不需要写具体路径了

DispatcherServlet绑定了一个配置文件，在我们访问资源时会关联这个配置文件

#### 1.2.1 示例

我们现在写一个controller的示例(implements接口)：

```java
public class HelloController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        ModelAndView mv = new ModelAndView();
        //业务代码
        String result = "HelloSpringMVC";
        mv.addObject("msg",result);
        //视图跳转,设置一个视图的名字即可
        mv.setViewName("test");
        return mv;
    }
}
```

写要跳转的jsp页面，显示ModelandView存放的数据，以及我们的正常页面；test.jsp:

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
${msg}
</body>
</html>
```

只要访问了这个HelloController资源，就会跳转到test.jsp,那么如何访问到这个资源呢？

我们需要在绑定的配置文件注册bean：

```xml
 <bean id="/hello" class="com.bo.controller.HelloController"/>
```

#### 1.2.2 错误

测试，**遇到的问题：访问出现404，排查步骤：**

1. 查看控制台输出，看一下是不是缺少了什么jar包。
2. **如果jar包存在，显示无法输出，就在IDEA的项目发布中，添加lib依赖！**
3. 重启Tomcat 即可解决！

我们的jar包是存在的，打开project structure发现这个项目没有lib目录

![image-20220901114503373](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202209011145275.png)

我们在web-inf下手动创建一个lib包，**点加号**把项目的所有jar包添加进去，重启tomcat即可

![image-20220901114803863](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202209011148843.png)

成功访问到msg中的内容

现在，我们发现不用再写servlet也可以实现页面的跳转，只是绑定了一个配置文件。



## 二、使用注解开发Springmvc

### 2.1 使用注解的一些配置

1、新建一个Moudle，springmvc-03-hello-annotation 。添加web支持！

2、由于Maven可能存在资源过滤的问题，我们将配置完善

3、配置web.xml，和刚刚一样

4、**添加Spring MVC配置文件**

和以前用注解开发一样，使用注解开发必须导入实现注解的依赖（驱动），所以我们需要加入一些配置:

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

   <!-- 自动扫描包，让指定包下的注解生效,由IOC容器统一管理 -->
   <context:component-scan base-package="com.bo.controller"/>
   <!-- 让Spring MVC不处理静态资源 -->
   <mvc:default-servlet-handler />
   <!--
   支持mvc注解驱动
       在spring中一般采用@RequestMapping注解来完成映射关系
       要想使@RequestMapping注解生效
       必须向上下文中注册DefaultAnnotationHandlerMapping
       和一个AnnotationMethodHandlerAdapter实例
       这两个实例分别在类级别和方法级别处理。
       而annotation-driven配置帮助我们自动完成上述两个实例的注入。
    -->
   <mvc:annotation-driven />

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

### 2.2 controller示例

编写一个Java控制类：com.kuang.controller.HelloController , 注意编码规范

```java
@Controller
@RequestMapping("/HelloController")
public class HelloController {

   //真实访问地址 : 项目名/HelloController/hello
   //相当于<bean name="/hello" class="com.bo.controller.HelloController"/>
   @RequestMapping("/hello")
   public String sayHello(Model model){
       //向模型中添加属性msg与值，可以在JSP页面中取出并渲染
       model.addAttribute("msg","hello,SpringMVC");
       
       //web-inf/jsp/hello.jsp
       return "hello";//可以被视图解析器处理
  }
}
```

- @Controller是为了让Spring IOC容器初始化时自动扫描到；
- @RequestMapping是为了映射请求路径，这里因为类与方法上都有映射所以访问时应该是/HelloController/hello；
- 方法中声明Model类型的参数是为了把Action中的数据带到视图中，**加上这个参数，Spring MVC会自动实例化一个Model对象用于向视图中传值；**
- 方法返回的结果是视图的名称hello，加上配置文件中的前后缀变成WEB-INF/jsp/**hello**.jsp。

视图层hello.jsp：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
${msg}
</body>
</html>
```

配置好tomcat运行即可

方法＋注解==servlet

### 2.3 小结

1. 新建一个web项目
2. 导入相关jar包
3. 编写web.xml , 注册DispatcherServlet
4. 编写springmvc配置文件
5. 接下来就是去创建对应的控制类 , controller
6. 最后完善前端视图和controller之间的对应
7. 测试运行调试.

通常，我们只需要**手动配置视图解析器**，而**处理器映射器**和**处理器适配器**只需要开启**注解驱动**即可，而省去了大段的xml配置。



## 三、Controller和Restful风格

### 3.1 控制器Controller：

- 控制器负责提供访问应用程序的行为，通常通过接口定义或注解定义两种方法实现。
- 控制器负责解析用户的请求并将其转换为一个模型。
- 在Spring MVC中一个控制器类可以包含多个方法
- 在Spring MVC中，对于Controller的配置方式有很多种

Controller是一个接口，在org.springframework.web.servlet.mvc包下，接口中只有一个方法；

```java
//实现该接口的类获得控制器功能
public interface Controller {
   //处理请求且返回一个模型与视图对象
   ModelAndView handleRequest(HttpServletRequest var1, HttpServletResponse var2) throws Exception;
}
```

根据我们第一个springmvc程序，这种方法返回一个模型，可以进行视图转换。

我们写好一个实现类，还需要去Spring配置文件中注册请求的bean

这样开发是很麻烦的，所以我们使用注解。

### 3.2 关于@RequestMapping

- @RequestMapping注解用于映射url到控制器类或一个特定的处理程序方法。可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。
- 为了测试结论更加准确，我们可以加上一个项目名测试 myweb
- 只注解在方法上面：

```java
@Controller
public class TestController {
   @RequestMapping("/h1")
   public String test(){
       return "test";
  }
}
```

访问路径：http://localhost:8080 / 项目名 / h1

- 同时注解类与方法

```java
@Controller
@RequestMapping("/admin")
public class TestController {
   @RequestMapping("/h1")
   public String test(){
       return "test";
  }
}
```

访问路径：http://localhost:8080 / 项目名/ admin /h1  , 需要先指定类的路径再指定方法的路径；

### 3.3 Restful风格

**概念**

Restful就是一个资源定位及资源操作的风格。不是标准也不是协议，**只是一种风格**。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。

**功能**

资源：互联网所有的事物都可以被抽象为资源

资源操作：使用POST、DELETE、PUT、GET，使用不同方法对资源进行操作。

分别对应 添加、 删除、修改、查询。

**传统方式操作资源**  ：通过不同的参数来实现不同的效果！方法单一，post 和 get

​	http://127.0.0.1/item/queryItem.action?id=1 查询,GET

​	http://127.0.0.1/item/saveItem.action 新增,POST

​	http://127.0.0.1/item/updateItem.action 更新,POST

​	http://127.0.0.1/item/deleteItem.action?id=1 删除,GET或POST

**使用RESTful操作资源** ：可以通过不同的请求方式来实现不同的效果！如下：请求地址一样，但是功能可以不同！

​	http://127.0.0.1/item/1 查询,GET

​	http://127.0.0.1/item 新增,POST

​	http://127.0.0.1/item 更新,PUT

​	http://127.0.0.1/item/1 删除,DELETE

#### 3.3.1 测试1，参数

新建一个类 RestFulController：

```java
@Controller
public class RestFulController {
    @RequestMapping("/add")
    public String test1(int a, int b, Model model){
        int result = a+b;
        model.addAttribute("msg","结果为："+result);
        return "test";
    }
}
```

我们看到这次我们在@RequestMapping的方法上除了model还加了int参数

打开Tomcat测试：

/add之后先是404错误，**发现还是1.2.2的那个问题，打开项目结构导入jar包就好了**

再/add发现是500错误，这是因为**我们在方法中定义了a和b两个参数**，前端没有传东西，当然访问不到

我们再次测试：http://localhost:8080/Springmvc01_war_exploded/add?a=1&b=2

成功：

![image-20220903100746931](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202209031008703.png)

但是在restful风格，我们的url其实并不需要这样写

在Spring MVC中可以使用  **@PathVariable** 注解，让**方法参数的值对应绑定**到一个URI模板变量上。

即代码变成：

```java
@Controller
public class RestfulController {
    @RequestMapping("/add/{a}/{b}")
    public String test1(@PathVariable int a,@PathVariable int b, Model model){
        int result = a+b;
        model.addAttribute("msg","结果为："+result);
        return "test";
    }
```

这样，前端url传递的参数{a}会传到方法中的a，b也一样，也就是说，这个时候我们的url就是：

http://localhost:8080/Springmvc01_war_exploded/add/1/2

而不是http://localhost:8080/Springmvc01_war_exploded/add?a=1&b=2（404）

修改参数类型是一样的：

```java
//映射访问路径
@RequestMapping("/commit/{p1}/{p2}")
public String index(@PathVariable int p1, @PathVariable String p2, Model model){

   String result = p1+p2;
   //Spring MVC会自动实例化一个Model对象用于向视图中传值
   model.addAttribute("msg", "结果："+result);
   //返回视图位置
   return "test";

}
```

http://localhost:8080/Springmvc01_war_exploded/add/1/a

这样会返回字符串 1a

#### 3.3.2 使用method属性指定请求类型

用于约束请求的类型，可以收窄请求范围。指定请求谓词的类型如GET, POST, HEAD, OPTIONS, PUT, PATCH, DELETE, TRACE等

比如我们现在把上面的代码改为：

```java
@RequestMapping(name = "/add/{a}/{b}",method = RequestMethod.DELETE)
```

这就限定了请求的方法只能是DELETE，我们使用浏览器地址栏进行访问默认是Get请求，如果我们直接用http://localhost:8080/Springmvc01_war_exploded/add/1/2会报错405：

![image-20220903102526070](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202209031025986.png)

这时再把代码改成：

```java
@RequestMapping(name = "/add/{a}/{b}",method = RequestMethod.GET)
```

就可以了。

方法级别的注解变体有如下几个，即组合注解：

```
@GetMapping
@PostMapping
@PutMapping
@DeleteMapping
@PatchMapping
```

@GetMapping 是一个组合注解，平时使用的会比较多！

它所扮演的是 @RequestMapping(name=xxx method =RequestMethod.GET) 的一个快捷方式。

相同的@PostMapping

**所有的地址栏请求默认都会是 HTTP GET 类型的。**我们现在用表单进行post方式提交：

web目录下创建a.jsp:

```jsp
<body>
    <form action="${pageContext.request.contextPath}/add/1/2" method="post">
        <input type="submit">
    </form>
</body>
```

约束请求类型为post方式：

```java
@Controller
public class RestfulController {
    @GetMapping("/add/{a}/{b}")
    public String test1(@PathVariable int a,@PathVariable int b, Model model){
        int result = a+b;
        model.addAttribute("msg","结果1为："+result);
        return "test";
    }
    @PostMapping("/add/{a}/{b}")
    public String test2(@PathVariable int a,@PathVariable int b, Model model){
        int result = a+b;
        model.addAttribute("msg","结果2为："+result);
        return "test";
    }
```

对比一下，我们先直接在地址栏请求：http://localhost:8080/Springmvc01_war_exploded/add/1/2

![image-20220903111444352](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202209031114701.png)

走了结果一

然后用提交表单的方式：

![image-20220903111547986](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202209031115879.png)

点提交：

![image-20220903111835706](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202209031118636.png)

走了结果2

我们发现url地址相同，但结果却不同