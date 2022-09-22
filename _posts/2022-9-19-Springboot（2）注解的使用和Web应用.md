---
layout: post
title: "Springboot（2）注解的使用和Web应用"
date:   2022-9-19
tags: [Springboot]
comments: true
author: Bo
---

## 2.1  启动类注解

### 2.1.1 @SpringBootApplication

是一个复合注解，由

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
```

组成

1.它拥有@Configuration的功能说明在

```java
@SpringBootApplication
public class Springboot01Application {

    public static void main(String[] args) {
        SpringApplication.run(Springboot01Application.class, args);
    }

}
```

这个类中可以去**声明对象**，对象能注入到容器中：

```java
@SpringBootApplication
public class Springboot01Application {

    public static void main(String[] args) {
        SpringApplication.run(Springboot01Application.class, args);
}
@Bean
public Student myStudent(){
    return new Student();
}
}
```

2.它拥有@EnableAutoConfiguration的功能

启用自动配置，也就是说，可以把java对象配置好，注入到容器中。例如你可以吧mybatis对象创建好，并注入倒容器中



## 2.2 创建MVC应用

配置文件名称有.properties和.yml两种，现在用yml多

- properties （key=value）
- yml  （key：value）

### 2.2.1 配置文件application.properties

在properties和yml文件都存在时，默认使用properties文件

先来一个properties配置文件：

```properties
# change Tomcat port
server.port=8082
# change initial site form '/' to '/myboot'
server.servlet.context-path=/myBoot

#  custom configurations
Student.name = bo
Student.site = ikun
```

**注意尽量不要写中文！注释也一样**

前两个注解的作用显而易见，而且是本来就有的，我们后面自己定义了两个key=value，那么怎么使用它们呢？

用@Value：

先写一个Controller：

```java
@Controller
public class BootController {
    
    @Value("${server.port}")
    private Integer port;

    @Value("${server.servlet.context-path}")
    private String contextPath;

    @Value("${Student.name}")
    private String name;
    
    @Value("${Student.site}")
    private String site;
    
    @RequestMapping("/query")
    @ResponseBody//不使用视图解析器
    public String queryData(){//获取配置文件的数据
        return "name:"+ name+"site"+site+"port"+port+"path"+contextPath;
    }
}
```

测试，成功输出信息！



```
@ConfigurationProperties(prefix = "student")
这个注解可以配置文件的数据映射为java对象，使用这个注解时注意导入以下依赖：
要注意prefix后面的必须都用小写，在写配置文件的时候就要注意
```

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

写一个school类：



```java
@Component
@ConfigurationProperties(prefix= "student")
public class School {
    private String name;
    private String site;
public String getName() {
    return name;
}

public void setName(String name) {
    this.name = name;
}

public String getSite() {
    return site;
}

public void setSite(String site) {
    this.site = site;
}

@Override
public String toString() {
    return "School{" +
            "name='" + name + '\'' +
            ", site='" + site + '\'' +
            '}';
}
```
记住@Component把这个对象注册进容器，@ConfigurationProperties(prefix= "student")表示这个对象的属性值来自于properties配置文件中的值

我们更改一下之前写的controller：

```java
@Controller
public class BootController {
    @Resource
    private School student1;

    @RequestMapping("/query")
    @ResponseBody//不使用视图解析器
    public String queryData(){//获取配置文件的数据
        return student1.toString();
    }
}
```

用@Resource注解就可以把配置文件中映射的java对象给取出来

测试：http://localhost:8082/myBoot/query

页面显示：School{name='bo', site='ikun'}



## 2.3 Web组件

### 2.3.1 拦截器

拦截器是springmvc中的一种对象，它能拦截对controller的请求

拦截器框架中有系统的拦截器，也可以自定义

**以后更新**





### 2.3.2 使用Servlet

在Springboot框架中使用servlet对象

步骤：

- 创建类继承HttpServlet
- 注册servlet
- 测试

servlet：

```java
public class Servlet01 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req,resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=utf-8");
        PrintWriter writer = resp.getWriter();
        writer.print("执行的servlet对象");
        writer.flush();
        writer.close();
    }
}
```



第一个步骤跟之前javaWeb差不多，主要是注册servlet

写一个配置类，加上Spring的配置注解@Configuration

```java
@Configuration
public class WebServletConfig {
    //定义方法用于注册
    @Bean
    public ServletRegistrationBean servletRegistrationBean(){
        ServletRegistrationBean Bean =
                new ServletRegistrationBean(new Servlet01(),"/s01");
        return Bean;
    }
}
```

方法的第一个参数是servlet对象，第二个是url。记住在方法上加上@Bean，这样才能注册到容器中

测试：http://localhost:8080/myBoot/s01

成功输出



### 2.3.3 字符集过滤器

在SpringMVC框架中，我们可以在web.xml文件中配置框架给的过滤器，解决乱码问题

在SpringBoot中，**应答默认的编码是ISO-8859-1**

这时我们使用**系统提供的字符集过滤类**

我们在刚刚的web配置文件中注册过滤类就可以了：

```java
@Bean
public FilterRegistrationBean filterRegistrationBean(){
    FilterRegistrationBean reg = new FilterRegistrationBean();
    //使用框架中的过滤器类
    CharacterEncodingFilter filter = new CharacterEncodingFilter();
    //指定编码
    filter.setEncoding("utf-8");
    //指定req，resp都使用utf-8
    filter.setForceEncoding(true);
    reg.setFilter(filter);
    //指定过滤的url地址
    reg.addUrlPatterns("/myBoot/*");
    return reg;
}
```



我们先用刚刚的servlet测试：

```java
@Override
protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    resp.setContentType("text/html");
    PrintWriter writer = resp.getWriter();
    writer.print("执行的servlet对象");
    writer.flush();
    writer.close();
}
```

输出乱码：

???servlet??

加上过滤器类之后：

**没有任何作用！**这是为何？

因为springboot中默认已经配置了CharacterEncodingFilter，并且默认编码是ISO-8859-1

我们要关闭默认，才能使用

你还需要在application.properties配置文件中加上：

```properties
server.servlet.encoding.enabled=false
```

成功输出：

`执行的servlet对象`

没有乱码

过滤器使用步骤总结：

- 配置字符集过滤器，注册到容器中
- 修改application.properties，关闭默认

#### 2.3.3.* 优化

我们刚刚学到springboot中默认已经配置了CharacterEncodingFilter，并且默认编码是ISO-8859-1，既然默认已经配置了CharacterEncodingFilter，已经有了过滤器了，我们去改这个过滤器就行，不用重写一个了

我们直接删掉刚刚的web配置文件中的过滤器，直接在application.properties中写：

```properties
server.servlet.encoding.charset=UTF-8
server.servlet.encoding.force=true
```

这样，req和resp就都使用utf-8的编码方式了