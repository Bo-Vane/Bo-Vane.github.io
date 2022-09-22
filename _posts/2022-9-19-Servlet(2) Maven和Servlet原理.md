---
layout: post
title: "Servlet(2) Maven和Servlet原理"
date:   2022-9-19
tags: [maven]
comments: true
author: Bo
---

> 为什么maven

1.在web开发中，需要手动导入大量jar包

2.maven可以帮我们自动导入和配置jar包

> 配置maven环境

![image-20220812154753189](https://raw.githubusercontent.com/Bo-Vane/picgo/main/img/202208121547234.png)

然后在PATH里面加%MAVEN_HOME%\bin即可

**修改配置文件**：

> 阿里云镜像加速下载

```xml
<mirror>
	<id>alimaven</id>
	<mirrorOf>central</mirrorOf>
	<name>aliyun maven</name>
	<url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
</mirror>
```

> 本地仓库

建立一个本地仓库：

```xml
<localRepository>F:\Environment\apache-maven-3.8.6-bin\apache-maven-3.8.6\maven-repo</localRepository>
```

## 在idea中使用maven

创建一个maven项目，直接用maven

![image-20220812172313213](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202208121723174.png)



要手动导入jar包时，maven可以直接通过在pom.xml中添加依赖

### 注意：约定大于配置

#### 配置文件无法导出

maven由于它的约定大于配置，我们之后可能遇到我们写的配置文件无法导出的问题，解决方案先放这：

**在build中配置resource**，来防止我们资源导出失败:

```xml
<build>
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
    </build>
```

## Maven仓库的使用

想要导入某个jar包的依赖，可以去https://mvnrepository.com/搜索复制

servlet和jsp的依赖：

```xml
<!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>4.0.1</version>
      <scope>provided</scope>
    </dependency>

    <!-- https://mvnrepository.com/artifact/javax.servlet.jsp/javax.servlet.jsp-api -->
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>javax.servlet.jsp-api</artifactId>
      <version>2.3.3</version>
      <scope>provided</scope>
    </dependency>
```



## Servlet开发

1.编写一个类，实现servlet接口

2.把开发好的java类部署到web服务器中



新建一个空的maven项目，把里面的src目录删掉，将这个空目录作为maven主工程，在里面建多个module即可。**此时需要在总的pom.xml导入全局依赖，这样就不用每次新建项目都导入依赖**

在项目下建maven的module即可

关于maven父子工程的理解：

建立子工程后，子工程也会有一个pom.xml,并且父工程会多一个：

```xml
	<modules>
        <module>servlet-01</module>
    </modules>
```

子工程会多一个：

```xml
 	<parent>
        <artifactId>javaweb-02-servlet</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
```

父项目中的java包(依赖)子工程可以使用，相当于java继承

注意：子工程中的框架就是一个完整的web项目的框架:

1.需要把web.xml更改为最新的：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0"
         metadata-complete="true">
</web-app>
```

2.完善maven的目录（main下添加java目录和resource目录）

接下来就是写servlet了：

servlet接口有两个默认的实现类，这里我们直接继承Httpservlet这个实现类

### 子工程注意事项

**每新建一个子工程，需要在Tomcat的Deployment中添加war包部署，不然无法生效。**

![image-20220814140712516](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202208141407499.png)

### servlet原理

![image-20220813104306564](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202208131043498.png)

### 关于mapping

```xml
/hello/hello
```

可以建多级

可以使用通配符！

```xml
/hello/*
/hello/?
```

