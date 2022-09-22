---
layout: post
title: "Mybatis-Plus"
date:   2022-9-19
tags: [Mybatis-Plus]
comments: true
author: Bo
---

[MyBatis-Plus (opens new window)](https://github.com/baomidou/mybatis-plus)（简称 MP）是一个 [MyBatis (opens new window)](https://www.mybatis.org/mybatis-3/)的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。、

> 优势

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）
- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库
- **内置性能分析插件**：可输出 SQL 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作



我们直接根据官方文档学习即可：

## 快速开始

使用第三方组件的一般步骤：

- 导入对应依赖
- 研究依赖如何配置
- 代码如何编写
- 扩展技术

> 快速开始Mybatis-Plus步骤

1、建立数据库mybatis_plus

2、创建user表

对应的创建数据库和表的脚本：

```sql
DROP TABLE IF EXISTS user;

CREATE TABLE user
(
    id BIGINT(20) NOT NULL COMMENT '主键ID',
    name VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名',
    age INT(11) NULL DEFAULT NULL COMMENT '年龄',
    email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱',
    PRIMARY KEY (id)
);
```

其对应的数据库 Data 脚本如下：

```sql
DELETE FROM user;

INSERT INTO user (id, name, age, email) VALUES
(1, 'Jone', 18, 'test1@baomidou.com'),
(2, 'Jack', 20, 'test2@baomidou.com'),
(3, 'Tom', 28, 'test3@baomidou.com'),
(4, 'Sandy', 21, 'test4@baomidou.com'),
(5, 'Billie', 24, 'test5@baomidou.com');
```

3、新建Springboot项目，开始体验

4、导入依赖：

数据库驱动：

```xml
<dependency>
   <groupId>mysql</groupId>
   <artifactId>mysql-connector-java</artifactId>
</dependency>
```

然后根据官方文件导入依赖：

```xml
<dependency>
   <groupId>com.baomidou</groupId>
   <artifactId>mybatis-plus-boot-starter</artifactId>
   <version>3.0.5</version>
</dependency>
<dependency>
   <groupId>com.h2database</groupId>
   <artifactId>h2</artifactId>
   <scope>runtime</scope>
</dependency>
```

我们这里的mybatis-plus-boot-starter依赖没用最新版，为了使用一些原生的功能

一般使用，导入mybatis-plus即可，不要再导入mybatis依赖

然后就是配置文件application.properties连接数据库:

```properties
# connect with database
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis_plus?characterEncoding=UTF-8&useUnicode=true&serverTimezone=UTC
```



6、连接mybatis，配置mapper.xml文件，service， controller

写一个User类，就不说了，按照数据库来

按照我们原来的思路，我们要先写一个UserMapper接口，再写一个它的配置文件，再写service层和controller

但我们现在大可不必这样做，**我们直接在对应的mapper接口上面继承基本的类BaseMapper<T>   即可，这是泛型T可以是pojo中的任意类**

```java
public interface UserMapper extends BaseMapper<User> {}
```

至此，所有的crud操作都编写完成了！

记得在主启动类上加mapper-scan

```java
@SpringBootApplication
@MapperScan("com.bo.mapper")
public class MybatisPlusApplication {

   public static void main(String[] args) {
      SpringApplication.run(MybatisPlusApplication.class, args);
   }

}
```

我们可以去test里稍微测试一下crud：

```java
@SpringBootTest
class MybatisPlusApplicationTests {

   //继承了父类，所有的方法都来自父类，我们也可以编写自己的扩展方法
   @Autowired
   private UserMapper userMapper;
   @Test
   void contextLoads() {
      //这个方法的参数是一个wrapper，条件构造器，我们没有条件就设置为null，就可查询全部用户
      List<User> users = userMapper.selectList(null);
      users.forEach(System.out::println);
   }
```

第一次测试报错了：

`nested exception is org.apache.ibatis.executor.ExecutorException: No constructor found in com.bo.pojo.User matching`

这是因为我们写了有参构造没有无参构造，在User类中加入无参构造后便测试成功了

> 思考

那么sql是什么时候写的呢，方法又是哪来的？

都是mybatis-plus生成的



### 删除操作

#### 逻辑删除

- 物理删除：从数据库直接移除
- 逻辑删除：在数据库中没有被移除，**而是通过一个变量让它失效**

逻辑删除类似于回收站，是为了防止数据的流失

管理员可以查看被删除的记录

1、表中添加字段

我们首先在表里增加一个字段deleted，deleted默认值设为0，当它等于1的时候代表被逻辑删除

2、实体类中增加属性

```java
@TableLogic
private int deleted;
```

3、配置！

去官方文档中找到逻辑删除的相关组件

首先在MybatisPlusConfig的java配置类中将LogicSqlInjector对象注册到spring

```java
package com.bo.config;

import ...;

@EnableTransactionManagement
@Configuration
@MapperScan("com.bo.mapper")
public class MybatisPlusConfig {
    @Bean
    public ISqlInjector injector(){
        return new LogicSqlInjector();
    }
}
```

再配置application.properties:

```properties
# mybatis-plus
# logic delete
mybatis-plus.global-config.db-config.logic-delete-value=1
mybatis-plus.global-config.db-config.logic-not-delete-value=0
```

现在就可以测试一下逻辑删除了：

```java
@SpringBootTest
public class Mytest {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testDel(){
        userMapper.deleteById(1);
    }
}
```

**记得加上@SpringBootTest注解！**

删除之后我们发现表中的第一条数据还在，但是deleted变成了1！

那我们还能否查询到第一条数据呢？

```java
@Test
public void testDel(){
    System.out.println(userMapper.selectById(1));

}
```

结果：

```java
null
```

发现第一条已经没了。

明显是不行，因为以上的CRUD及其扩展操作都会在sql中加入where deleted=0

这个时候想要恢复数据，就去数据库将deleted改为0就ok
