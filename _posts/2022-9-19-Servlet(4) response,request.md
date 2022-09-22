---
layout: post
title: "Servlet(4) response,request"
date:   2022-9-19
tags: [servlet]
comments: true
author: Bo
---

# Response

## 响应验证码界面到浏览器的模拟：

```java
public class Servlet06 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //如何让浏览器3秒刷新一次：响应头
        resp.setHeader("refresh","3");
        //在内存中创建一个图片
        BufferedImage image = new BufferedImage(80,20,BufferedImage.TYPE_INT_RGB);
        //得到图片
        Graphics2D g = (Graphics2D) image.getGraphics();//画笔
        //设置图片颜色
        g.setColor(Color.white);
        g.fillRect(0,0,80,20);
        //给图片写数据
        g.setColor(Color.blue);
        g.setFont(new Font(null,Font.BOLD,20));
        g.drawString(mkNum(),0,20);

        //告诉浏览器这个响应以图片的方式打开
        resp.setContentType("image/jpg");
        //网站存在缓存，不让浏览器缓存
        resp.setHeader("Cache-Control","no-cache");
        //把图片写给浏览器
        boolean write = ImageIO.write(image,"jpg", resp.getOutputStream());

    }
        //生成随机数
        private String mkNum(){
            Random random = new Random();
            String num = random.nextInt(9999999) + " ";
            StringBuffer sb = new StringBuffer();
            for (int i = 0; i < 7 - num.length(); i++) {
                sb.append("0");//num不足七位数时，用零来补成七位
            }
            return num+sb.toString();

        }

}
```

在web.xml中配置部署：

```xml
	<servlet>
        <servlet-name>s6</servlet-name>
        <servlet-class>com.bo.servlet.Servlet06</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>s6</servlet-name>
        <url-pattern>/s6</url-pattern>
    </servlet-mapping>
```





## 重定向

重定向的意思前面已有提及，所以直接上事例：

```java
public class RedirectServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.sendRedirect("/s1/s6");
    }
}
```

重定向就是调用response中的sendRedirect方法，注意括号里的路径，因为重定向默认是定向到：

localhost:8080/(括号路径)，/s1是我们添加的项目路径，必须加上。

跳转到localhost:8080/s6会找不到资源。

配置好web.xml(/s7),发现跳转到s6的验证码页面:

![image-20220814095916887](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202208140959097.png)



# Request

## 获取参数

index.jsp中写一个前端表单,提交action设置为/login：

```jsp
<html>
<body>
<h2>Hello World!</h2>
<%@ page contentType="text/html; charset=UTF-8"%>
<%--这里提交的路径需要寻找到项目的路径--%>
<%--${pageContext.request.contextPath}表示当前项目路径--%>
<form action="${pageContext.request.contextPath}/login" method="get">
    用户名: <input type="text" name="username"> <br>
    密码: <input type="password" name="password"> <br>
    提交： <input type="submit">
</form>

</body>
</html>
```

**${pageContext.request.contextPath}表示当前项目路径**，提交后的路径在后面再加即可。



编写获取参数，并重定向到登录页面的servlet：

```java
public class RequestTest extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String username = req.getParameter("username");
        String password = req.getParameter("password");
        System.out.println("username:"+username);
        System.out.println("\\n");
        System.out.println("password:"+password);

        //提交表单后重定向
        resp.sendRedirect("/s1/success.jsp");
    }

```

其中的success.jsp:

```jsp
<html>
<head>
    <title>success</title>
</head>
<body>
    <h1>success!</h1>
</body>
</html>
```

web.xml:

```xml
	<servlet>
        <servlet-name>request</servlet-name>
        <servlet-class>com.bo.servlet.RequestTest</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>request</servlet-name>
        <url-pattern>/login</url-pattern>
    </servlet-mapping>
```

浏览器输入数据，提交表单：

![image-20220814105835917](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202208141058101.png)

重定向跳转：

![image-20220814105913242](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202208141059275.png)

终端打印request从前端获取的参数：

```java
username:admin
password:password
```

此外req还可以get到其他的很多参数：

![image-20220814113256209](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202208141132238.png)

## request转发

我们建一个新的子工程，写一个新的login：

```java
public class LoginServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String username = req.getParameter("username");
        String password = req.getParameter("password");

        //获取多选框等有多个选项的参数
        String[] hobbies = req.getParameterValues("hobby");
        for (String s :
                hobbies) {
            System.out.println(s);
        }
        System.out.println("=========================");
        System.out.println("username="+username);
        System.out.println("password="+password);
        //通过请求转发
        req.getRequestDispatcher("/req/success.jsp").forward(req,resp);
        //括号里路径记得加上项目路径，后面不要忘记forward
    }


    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}

```

**index.jsp**,这次用post:

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>登录</title>
</head>
<body>

    <h1>登录</h1>
<div style="text-align: center">
    <%--以post方式提交表单--%>
    <form action="${pageContext.request.contextPath}/login" method="post">
        用户名：<input type="text" name="username"> <br>
        密码：<input type="password" name="password"> <br>
        爱好：
        <input type="checkbox" name="hobby" value="sing">sing
        <input type="checkbox" name="hobby" value="dance">dance
        <input type="checkbox" name="hobby" value="rap">rap
        <input type="checkbox" name="hobby" value="basketball">basketball
        <br>
        <input type="submit">
    </form>
</div>

</body>
</html>

```

web.xml:

```xml
<servlet>
        <servlet-name>login</servlet-name>
        <servlet-class>com.bo.servlet.LoginServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>login</servlet-name>
        <url-pattern>/login</url-pattern>
    </servlet-mapping>
```

配置好一切后，一定记得在Tomcat添加war包：

![](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202208141407499.png)

添加完后开启Tomcat它还是会默认走/s1，这时把/s1改成/req就行：

![image-20220814141936169](https:/raw.sevencdn.com/Bo-Vane/picgo/main/img/202208141419049.png)

这时就看到了我们新的登录页面。

提交表单后可以看到~~转发成功~~：

### 成功拿到404

![image-20220814142745106](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202208141427246.png)

404，/req/success.jsp未找到，说明我们在servlet中写的路径可能有问题

```java
//通过请求转发
        req.getRequestDispatcher("/req/success.jsp").forward(req,resp);
        //括号里路径记得加上项目路径，后面不要忘记forward
```

我们把这句代码中/req去掉试试。

![image-20220814143113095](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202208141431601.png)

成功解决！得到教训：

### 转发和重定向路径的区别

转发时"/"代表的是本应用程序的根目录   重定向时"/"代表的是webapps目录

所以，

- resp重定向时，括号内路径要加上项目
- req转发时，不用加项目，直接/资源名 即可



![image-20220828181057607](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202208281810453.png)
