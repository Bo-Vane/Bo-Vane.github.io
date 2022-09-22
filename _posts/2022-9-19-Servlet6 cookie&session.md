---
layout: post
title: "Servlet6 cookie&session"
date:   2022-9-19
tags: [servlet]
comments: true
author: Bo
---

### 6.1 会话

**会话**：用户打开了一个浏览器，点开多个超链接，访问多个web资源，然后关闭浏览器，这个过程可以称之为会话

**有状态会话：**

你怎么证明你是uestc的学生？

- 发票                  学校给你发票
- 学校有登记       学校登记你来过了

相似的，一个网站怎么证明你来过？

客户端=====》》服务端

- 服务端给客户端发票（**cookie**）客户端下次进入访问拿着发票就行

- 服务器登记（**session**）你来过了，下次访问的时候，服务器来匹配你

  

### 6.2 保存会话的两种技术

- cookie 客户端技术（响应，**req**）

- session 服务器技术，利用这个技术可以保存用户会话信息，我们客户可以把信息和数据放在session中

常见场景：网站登录后，下次不用再登录了

#### 6.2.1 cookie示例

```java
public class cookie01 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //服务器告诉你你来的时间，把这个时间封装为一个cookie给你，你下次带来就知道是你了

        req.setCharacterEncoding("utf8");
        resp.setCharacterEncoding("utf8");
        resp.setHeader("Content-type", "text/html;charset=utf-8");

        PrintWriter out = resp.getWriter();
        //服务端从客户端获取cookie，返回数组说明cookie可能存在多个
        Cookie[] cookies = req.getCookies();
        if(cookies!=null){
            for (Cookie cookie : cookies) {
                //获取cookie的名字如果等于服务器给的cookie名
                if (cookie.getName().equals("LastLoginTime")) {
                    //获取cookie的值
                    out.write(cookie.getValue());
                }
            }
        }else out.print("这是你第一次访问本站");

        //服务器可以给客户端响应一个cookie
        Cookie cookie = new Cookie("LastLoginTime",String.valueOf(System.currentTimeMillis()));
        resp.addCookie(cookie);

    }
```

配置好web.xml

打开Tomcat和浏览器测试，在审查元素中的application我们可以查看我们的cookie

![image-20220831124723052](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202208311247939.png)

我们可以看到，网页中呈现的是上一次（这里是第一次）访问浏览器时的cookie（上次访问的时间）浏览器审查元素中的cookie是这一次给我们的新的cookie（这次的时间）

这个时候我们把浏览器关闭再访问，我们可以发现：

![image-20220831125302152](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202208311253623.png)

**上次的cookie已经没用了!**

**这是因为我们关闭了浏览器，此次对话结束了**

但这个时候我们可以加一行代码让cookie继续有效

```java
//设置cookie的有效期为一天
cookie.setMaxAge(24*60*60);
```

#### 6.2.2 cookie总结

- 从请求中获取cookie信息
- 服务器响应给客户端cookie

方法：

```java
//获取cookie，返回数组
req.getCookies()
//得到cookie名
cookie.getName()
//得到cookie值
cookie.getValue()
//服务器响应给客户端cookie
    Cookie cookie = new Cookie("LastLoginTime",String.valueOf(System.currentTimeMillis()));
resp.addCookie(cookie);
```



#### 6.2.3 session示例

- 服务器会给每一个用户创建一个session对象
- 一个session独占一个浏览器，只要浏览器没有关闭，这个session就存在

- 用户登录之后，整个网站这个session都可以访问==》**保存用户的信息，保存购物车的信息之类**的

- session在浏览器打开的一瞬间就存在了，只是还没有存储东西



测试：

```java
public class session01 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //解决乱码
        req.setCharacterEncoding("utf-8");
        resp.setCharacterEncoding("utf-8");
        resp.setHeader("content-type","text/html;charset=utf-8");
        //得到session
        HttpSession session = req.getSession();
        //给session中存东西
        session.setAttribute("name","bo");
        //获取session的ID
        String id = session.getId();
        //判断session是不是新创建的
        if (session.isNew()){
            resp.getWriter().print("创建session成功，session的id为"+id);
        }else resp.getWriter().print("session已经在服务器中存在了");
    }

```



![image-20220831155604077](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202208311556664.png)



浏览器审查元素发现：

session中是空的，但cookie栏如图

```
Cookie: LastLoginTime=1661931727457; JSESSIONID=53B7AA7CF3D0D5D444D216E44ECEAC1C
```

所以:

```java
//session在创建的时候做了什么事情
Cookie cookie = new Cookie("JSESSIONID", id);
resp.addCookie(cookie);
```

我们在session01中存的东西还没用，我们再写一个servlet看看能不能取：

```java
//读取在session01中存在session中的名字
HttpSession session = req.getSession();
Object name = session.getAttribute("name");
System.out.println(name.toString());
```

运行后发现控制台输出了bo，成功

一个浏览器对应一个服务器只能有一个session，我们可以在xml文件中设置session默认的失效时间：

```xml
<session-config>
    <!--1分钟后session自动失效-->
    <session-timeout>1</session-timeout>
</session-config>
```

当然我们也可以手动注销session，再写一个servlet：

```java
HttpSession session = req.getSession();
session.removeAttribute("name");
session.invalidate();
```

removeAttribute可以把存进去的东西删掉，invalidate()可以直接让session失效，即注销（id就没了）

**手动注销和自动失效时间一般都要设置**，可以设想：

当用户登录后，用户可能会想注销账户，这个时候需要手动注销方法；当用户登录过太长时间没有访问，就不再保存他的登录信息



### 6.3 session和cookie的区别

- Cookie是把用户的数据写给用户的浏览器，浏览器保存（可以保存多个）
- session是服务器保存，**一个浏览器只存在一个**