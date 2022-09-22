---
layout: post
title: "Springboot（1）创建项目"
date:   2022-9-19
tags: [Springboot]
comments: true
author: Bo
---

> 为什么要使用Springboot

- 因为  spring，springmvc**都需要大量配置文件**（xml文件）

- 还需要配置各种对象，把使用的对象放入spring容器中才能使用对象

- 还需要了解其他框架的配置规则

SpringBoot就相当于不需要配置文件的spring和springmvc。常用的框架和第三方库拿来就可以使用了，开发效率高。



## 1.1JavaConfig

JavaConfig：**使用Java类作为xml配置文件的替代**，是配置spring容器的纯java方式，在这个Java类中可以创建Java对象，把对象放入spring容器中。

使用两个注解：

1）@Configuration  放在一个类上面，表示这个类是作为配置文件使用的

2）@Bean  声明对象，把这个对象注入到容器中

示例：

新建一个空的项目，在项目中新建一个maven的module：

> pom.xml依赖:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.3.18</version>
    </dependency>
   
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    
    <dependency>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.8.1</version>
    </dependency>

```

### 使用xml配置文件配置容器

> 新建一个Student类

```java
public class Student {
    private String name;
    private Integer age;
    private String sex;
<getter/setter>
    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", sex='" + sex + '\'' +
                '}';
    }
}    
```

> 在resources中创建一个配置文件beans.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--声明bean对象-->
    <bean id="myStudent" class="com.bo.springboot.Student">
        <property name="name" value="李四" />
        <property name="age" value= "20" />
        <property name="sex" value="男" />
    </bean>
</beans>
```

> 在test文件夹中创建一个测试类来测试拿到对象

```java
public class MyTest {
    /*
    使用xml作为容器配置文件
     */
    @Test
    public void test01(){
        String config="beans.xml";
        ApplicationContext ctx = new ClassPathXmlApplicationContext(config);
        Student student = (Student) ctx.getBean("myStudent");
        System.out.println("容器中的对象："+student.toString());
    }
}
```

输出结果：

```
容器中的对象：Student{name='李四', age=20, sex='男'}
```

没有问题。



## 1.2 Springboot特性

Springboot是spring中的一个成员，它可以简化spring和springmvc开发，本质还是IOC容器

特点：

- 创建spring应用
- 内嵌Tomcat
- 提供了starter起步依赖，简化应用配置
  - 比如使用mybatis框架，在spring项目中需要配置mybatis的对象，在springboot项目中，只需在pom.xml中加入mybatis的starter依赖即可



## 1.3 创建springboot项目

step1：新建项目

### 1.3.1 第一种方式，使用spring提供的初始化器，就是向导创建springboot应用

**向导地址：https://start.spring.io**

新建项目，选择spring initialer 设置好java版本（8）再设置好需要的依赖即可。

然后可以看看目录：可以发现差不多是一个**标准的maven工程**

![image-20220827145705830](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202208271457265.png)

在pom.xml中发现有字段爆红：

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>//爆红
                <artifactId>spring-boot-maven-plugin</artifactId>//爆红
```

在下面加上版本号即可

```xml
               <version>2.7.3</version>
```

### 1.3.2 第二种方式创建Springboot项目

第一种向导地址由于是国外的，访问会很慢甚至有时会失败，这时我们可以用国内的镜像向导地址：

**向导地址：https://start.springboot.io**

![image-20220827151219676](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202208271512680.png)

在server URL自己设置即可

## 1.4 基于springboot的web例子

在controller层写好我们的代码：

```java
@Controller
public class HelloSpringboot {

    @RequestMapping("/hello")
    @ResponseBody
    public String helloSpringboot(){
        return "welcome to springboot";
    }

}
```

不需要导入其他依赖，不需要配置Tomcat，直接启动它给我们的Application类中的main方法，发现Tomcat已经启动

浏览器发现异常：

# Whitelabel Error Page

This application has no explicit mapping for /error, so you are seeing this as a fallback.

原因：IDEA目录结构的问题，Application启动类的位置不对.要将Application类放在最外侧,即包含所有子包 。而我的controller则放在了最外层的包里面。导致找不到页面。

所以我们的结构应该是这样的：

![image-20220827160959108](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202208271610412.png)

