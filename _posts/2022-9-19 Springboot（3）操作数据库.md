---
layout: post
title: "Springboot（3）操作数据库"
date:   2022-9-19
tags: [Springboot]
comments: true
author: Bo
---

我们使用springboot添加mybatis起步依赖对数据库进行操作，我们就用当初学mybatis的时候创建的数据库。

springboot框架集成mybatis：

- mybatis起步依赖，完成对mybatis对象的自动配置，对象放在容器中

- pom.xml文件指定把src/main/java目录中的xml类型文件包含到classpath中

- 创建实体类User

- 创建Dao接口UserMapper接口

- UserMapper.xml

- 创建Service对象，创建UserService接口和它的impl去调用dao层

- 创建Controller对象，访问service

- 写application.properties配置文件

  配置数据库连接信息



## 3.1 起步依赖

创建一个新的Springboot的module，勾选web依赖，在sql中勾选mybatis框架和mysql Driver



## 3.2 Dao，Service ，Controller

> User实体类

在idea中连接数据库根据表创建实体类get/set toString

```java
public class User {
    private int id;
    private String name;
    private String pwd;
    }
```

> 创建Dao接口

写一个操作数据库的dao接口，**上面加上@Mapper注解，告诉mybais这是dao接口并创建此接口的代理对象**

```java
@Mapper
public interface UserMapper {
    //根据id查询
    User selectById(@Param("UserId") int id);
}
```



然后是UserMapper.xml，**注意资源导出问题！**在build中配置：

```xml
<resources>
    <resource>
        <directory>src/main/resources</directory>
        <includes>
            <include>**/*.properties</include>
            <include>**/*.xml</include>
        </includes>
        <filtering>true</filtering>
    </resource>
    <resource>
        <directory>src/main/java</directory>
        <includes>
            <include>**/*.properties</include>
            <include>**/*.xml</include>
        </includes>
        <filtering>true</filtering>
    </resource>
</resources>
```

UserMapper.xml：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.bo.dao.UserMapper">
    <select id="selectById"  resultType="com.bo.pojo.User">
        select * from user where id = #{UserId}
    </select>
</mapper>
```



> 业务层对象

```JAVA
public interface UserService {

    User queryUser(int id);
}
```

实现类：

```java
@Service
public class UserServiceImpl implements UserService {
    @Resource
    private UserMapper userMapper;

    public void setUserMapper(UserMapper userMapper) {
        this.userMapper = userMapper;
    }

    @Override
    public User queryUser(int id) {
        return this.userMapper.selectById(id);
    }
}
```

**业务层实现类要调用dao对象，用@Resource就可以注入**

在实现类上面别忘了@Service，声明这是业务对象



> 创建Controller

```java
@Controller
@RequestMapping("/user")
public class UserController {
    @Resource
    private UserService userService;

    @RequestMapping("/query")
    @ResponseBody
    public String queryUser(int id){
        return userService.queryUser(id).toString();
    }
}
```

Controller要调用service对象，还是用@Resource注入

> 配置文件application.properties

主要是做配置数据源

```properties
# connect to the database
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis?characterEncoding=UTF-8&useUnicode=true&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=123456
```

### 3.2.* 测试的错误

输入url测试：http://localhost:8080/user/query?id=1

404错误，

找了半天，**发现是我运行的是上一个工程的启动类，所以建立新工程时一定要注意**！

然后又报了一个500错误，控制台显示：

Invalid bound statement (not found): com.bo.dao.UserMapper.selectById

说明我们的useMapper.xml的映射出了问题，找了半天，代码没问题，看target文件夹，发现useMapper.xml未成功导出，但我前面在pom.xml加入了针对导出问题的过滤器的,很奇怪。

然后我在这个子module再加入一次resource插件：

```xml
<resources>
    <resource>
        <directory>src/main/java</directory>
        <includes>
            <include>**/*.properties</include>
            <include>**/*.xml</include>
        </includes>
        <filtering>true</filtering>
    </resource>
</resources>
```

再运行，成功了

**所以以后这种问题（not found）先看target文件夹！**



## 3.3 @Mapper和@Mapper-scan

我们从上面知道，把@Mapper放在dao层的接口上就可以告诉mybais这是dao接口并创建此接口的代理对象，但当我们有多个dao接口时不方便

这个时候我们用@Mapper-scan，注意是在主启动类加：

```java
@SpringBootApplication
@MapperScan(basePackages = "com.bo.dao")
public class Springboot03Application {

    public static void main(String[] args) {
        SpringApplication.run(Springboot03Application.class, args);
    }

}
```



## 3.4 Springboot中的事务控制

Spring框架中的事务：

1、管理事物的对象：事务管理器（接口，这个接口有很多实现类）

例如你要是用jdbc或者mybaits访问数据库，使用的事务管理器：DataSourceTransactionManager

2、声明式事务：在xml文件或者使用注解说明事务的内容

3、事务的处理fangs：

- Spring框架的注解@Transactional
- aspectj框架

### 3.4.1 使用springboot的注解

1、在**业务**方法的上面加入@Transactional，加入后方法就有事务功能了

2、明确的在主启动类加上@EnableTransactionManagement

在前面的基础上，添加一个addUser的方法，usermapper接口和xml配置好后，写**service层的实现类**：

```java
@Transactional
@Override
public int add(User user) {
    System.out.println("业务方法add User");
    int rows =  userMapper.add(user);
    System.out.println("执行sql语句");

    //抛出一个运行时异常，目的是回滚事务
    int m = 10/0;
    return rows
}
```

我们加了@Transactional注解，给这个service方法赋予了事务功能，并写了一个`int m = 10/0;`的运行时异常来测试，因为如果抛出异常就会回滚，没有异常就会自动提交事务。

然后写controller测试：

```java
    @RequestMapping("/add")
    @ResponseBody
    public String addUser(User user){
        userService.add(user);
        String s = "add successfully";
        return s;
    }
}
```

然后我们测试时：http://localhost:8080/user/add?name=hooxi&pwd=666666

发生了一个400错误，**控制台警告我们的user表限制id不能为空所以我们只能把参数设置为另外两个字段名而不能设置为对象！**

```java
    @RequestMapping("/add")
    @ResponseBody
    public String addUser(String name,String pwd){
        User user = new User();
        user.setName(name);
        user.setPwd(pwd);
        userService.add(user);
        String s = "add successfully";
        return s;
    }
}
```

这样就好了
