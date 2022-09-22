---
layout: post
title: "Servlet(3) ServletContext对象"
date:   2022-9-19
tags: [servlet]
comments: true
author: Bo
---

# Servlet(3) ServletContext对象

web容器在启动的时候，它会为每个web程序都创建一个对应的ServletContext对象，它代表了当前的web应用。

ServletContext可以存储多个servlet的数据

**特点：多个servlet共用一个ServletContext对象！**

## 共享数据

HelloServlet.java:

```java
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext context = this.getServletContext();
        String username = "bo";//数据
        context.setAttribute("username",username);//将一个数据保存在了ServletContext中，名字：username，值为：username

    }
}
```

GetServlet.java

```java
public class GetServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext context = this.getServletContext();
        String username = (String) context.getAttribute("username");//获取HelloServlet中保存的数据
        PrintWriter writer = resp.getWriter();
        writer.print(username);
    }
}
```

web.xml

```xml
    <servlet-mapping>
        <servlet-name>hello</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
	<servlet-mapping>
        <servlet-name>get</servlet-name>
        <url-pattern>/get</url-pattern>
    </servlet-mapping>
```

**注意事项，我们必须先访问/hello再访问/get**，因为直接访问get的话，ServletContext对象中还没有存储任何数据，会打印null

为了避免response返回内容识别乱码和格式问题，print前我们一般加上：

```java
        resp.setContentType("text/html");
        resp.setCharacterEncoding("utf-8");
```

 

## 获取初始化参数

```xml
<!-- 配置一些web应用初始化参数 -->
    <context-param>
        <param-name>url</param-name>
        <param-value>jdbc:mysql://localhost:3306/mybatis</param-value>
    </context-param>
```

ServletContext得到这些参数：

```java
		ServletContext context = this.getServletContext();
        String url = context.getInitParameter("url");//获取初始化参数
        resp.getWriter().print(url);
```

打开tomcat测试发现url被打印了出来。

## 请求转发

```java
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext context = this.getServletContext();
//        RequestDispatcher requestDispatcher = context.getRequestDispatcher("/gp");//转发的路径
//        requestDispatcher.forward(req,resp);//调用forward方法实现请求转发
        context.getRequestDispatcher("/gp").forward(req,resp);//简化
    }
```

就是getRequestDispatcher方法

部署后打开Tomcat测试，发现虽然进入了这个方法，但是页面却转发到了/gp,也就是初始化参数的那个页面

**转发和重定向**：

![image-20220813153753472](https://raw.githubusercontent.com/Bo-Vane/picgo/main/img/202208131537556.png)

A要找B拿资源，但这个资源只有C有。

转发：B去C拿资源，拿完给A，A不去找C

重定向：B告诉A只有C有，B不去C，A去C

## 读取资源文件

在resource下写一个资源文件db.properties：

```properties
username=root
password=123456
```

通过servletContext读取：

```java
public class Servlet05 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        InputStream is = this.getServletContext().getResourceAsStream("/WEB-INF/classes/db.properties");
		//读取资源文件，返回到输入流
        Properties properties = new Properties();
        properties.load(is);//properties加载输入流，因为db.properties已经有内容，可以直接加载
        String username = properties.getProperty("username");
        String password = properties.getProperty("password");

        resp.getWriter().print("username="+ username+"password="+password);
    }
}
```

注意：getResourceAsStream后面跟的路径**第一个/是代表当前web项目，不能省略**，返回值类型是流

配置好web.xml后测试即可。

