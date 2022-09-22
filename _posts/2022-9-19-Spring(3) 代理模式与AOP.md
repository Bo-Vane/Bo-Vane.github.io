---
layout: post
title: "Spring(3) 代理模式与AOP"
date:   2022-9-19
tags: [Spring]
comments: true
author: Bo
---

## 3.1 代理模式

代理模式的分类：

- 静态代理
- 动态代理

### 3.1.1 静态代理

角色分析：

- 抽象角色，一般会使用接口或者抽象类来解决
- 真实角色，被代理的角色
- 代理角色，代理真实角色，代理真实角色后会做一些附属操作
- 客户：访问代理对象的人

抽象角色是一个接口，真实角色和代理角色都要实现这个接口！

比如抽象角色是租房这个事，房东要租房，就要实现这个接口。但是房东只能有一件事，就是租房。但客户租房不会找房东，只会找中介，中介要帮房东租房，就要实现同一个租房接口，**客户要租哪套房，中介就代理哪个房东**。并且中介除了租房，还可以添加很多不同的功能。

静态代理好处：

- 可以使真实角色的操作更纯粹，不用关注一些业务

- 公共的业务由代理来完成 . 实现了业务的分工 ,
- 公共业务发生扩展时变得更加集中和方便 .

缺点 :

- 类多了 , 多了代理类 , 工作量变大了 . 开发效率降低 .

我们想要静态代理的好处，又不想要静态代理的缺点，所以 , 就有了动态代理 !

**我们在不改变原来的代码的情况下，实现了对原有功能的增强，这是AOP中最核心的思想！**



### 3.1.2 动态代理

- 动态代理的角色和静态代理的一样 .

- 动态代理的代理类**是动态生成的** . 静态代理的代理类是我们**提前写好的**

- 动态代理分为两类 : 一类是基于接口动态代理 , 一类是基于类的动态代理

- - 基于接口的动态代理----JDK动态代理
  - 基于类的动态代理--cglib
  - 现在用的比较多的是 javasist 来生成动态代理 . 百度一下javasist
  - 我们这里使用JDK的原生代码来实现，其余的道理都是一样的！、

**JDK的动态代理需要了解两个类**

核心 : InvocationHandler   和   Proxy  ， 打开JDK帮助文档看看

我们还是用房东租房的例子来理解：

租房（抽象角色）：

```java
//租房
public interface Rent {
    void rent();
}
```

房东（真实角色）：

```java
public class Host implements Rent{

    @Override
    public void rent() {
        System.out.println("房东租房了");
    }
}
```

现在我们需要一个代理角色：

```java
///等会我们用这个类动态生成代理类
public class ProxyInvocationHandler implements InvocationHandler {
    private Rent rent;
    //被代理的接口
    public void setRent(Rent rent) {
        this.rent = rent;
    }

    //生成得到代理类,参数依次为，loader，被代理的接口，和InvocationHandler对象
    public Object getProxy(){
       return Proxy.newProxyInstance(this.getClass().getClassLoader(),rent.getClass().getInterfaces(),this );
    }

    @Override
    //处理代理实例，并返回结果
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        //动态代理的本质就是通过反射实现的
        Object result = method.invoke(rent, args);
        return result;
    }
}
```

核心：**一个动态代理 , 一般代理某一类业务 , 一个动态代理可以代理多个类，代理的是接口！、**

> 深入理解

我们来使用动态代理实现代理我们后面写的UserService！

我们也可以编写一个通用的动态代理实现的类！所有的代理对象设置为Object即可！

```java
public class ProxyInvocationHandler implements InvocationHandler {
   private Object target;

    //要代理的东西
   public void setTarget(Object target) {
       this.target = target;
  }

   //生成代理类
   public Object getProxy(){
       return Proxy.newProxyInstance(this.getClass().getClassLoader(),
               target.getClass().getInterfaces(),this);
  }

   // proxy : 代理类
   // method : 代理类的调用处理程序的方法对象.这个method就可以理解成要代理的接口的某个方法
   public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
       //执行了接口的哪个，就会输出哪个方法的名字
       log(method.getName());
       Object result = method.invoke(target, args);
       return result;
  }

   public void log(String methodName){
       System.out.println("执行了"+methodName+"方法");
  }

}
```

> ##### 动态代理的好处

静态代理有的它都有，静态代理没有的，它也有！

- 可以使得我们的真实角色更加纯粹 . 不再去关注一些公共的事情 .
- 公共的业务由代理来完成 . 实现了业务的分工 ,
- 公共业务发生扩展时变得更加集中和方便 .
- 一个动态代理 , 一般代理某一类业务
- 一个动态代理可以代理多个类，代理的是接口！（**把setTarget(Object target) 里面换一下就可以了，只要target实现了特定的接口**）

、

## 3.2 AOP实现方式

AOP（Aspect Oriented Programming）意为：面向切面编程，利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

![图片](https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7JAeTYOaaH6rZ6WmLLgwQLHf5pmH30gj6mZm81PC7iauicFu55sicJtspU7K3vTCVdZCDTSHq7D5XHlw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



- 横切关注点：跨越应用程序多个模块的方法或功能。即是，与我们业务逻辑无关的，但是我们需要关注的部分，就是横切关注点。如日志 , 安全 , 缓存 , 事务等等 ....

- 切面（ASPECT）：横切关注点 被模块化 的特殊对象。即，它是一个类。

- 通知（Advice）：切面必须要完成的工作。即，它是类中的一个方法。

- 目标（Target）：被通知对象。

- 代理（Proxy）：向目标对象应用通知之后创建的对象。

- 切入点（PointCut）：切面通知 执行的 “地点”的定义。

- 连接点（JointPoint）：与切入点匹配的执行点。

  **即再不改变原有代码的情况下增加功能**

### 3.2.1 实现方式1 原生spring api接口

使用Spring实现Aop

需要导入依赖包：

```xml
<dependency>
   <groupId>org.aspectj</groupId>
   <artifactId>aspectjweaver</artifactId>
   <version>1.9.9.1</version>
</dependency>
```

首先编写我们的业务接口和实现类

```java
public interface UserService {

   public void add();

   public void delete();

   public void update();

   public void search();

}
```

```java
public class UserServiceImpl implements UserService{
    @Override
    public void add() {
        System.out.println("增加了一个用户");
    }
    @Override
    public void delete() {
        System.out.println("删除了一个用户");
    }
    @Override
    public void update() {
        System.out.println("更新一个用户");
    }
    @Override
    public void select() {
        System.out.println("查询一个用户");
    }
}
```

然后去写我们的增强类 , 我们编写两个 , 一个前置增强 一个后置增强

```java
public class Log implements MethodBeforeAdvice {

   //method : 要执行的目标对象的方法
   //objects : 被调用的方法的参数
   //target : 目标对象
   @Override
 public void before(Method method, Object[] args, Object target) throws Throwable {
        System.out.println(target.getClass().getName()+"执行了"+method.getName()+"方法");
    }
}
```

最后去spring的文件中注册 , 并实现aop切入实现 , 注意导入约束 .

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd">


    <!--注册bean-->
    <bean id="userService" class="com.bo.service.UserServiceImpl"/>
    <bean id="log" class="com.bo.log.log"/>
    <bean id="afterLog" class="com.bo.log.AfterLog"/>

    <!--配置AOP-->
    <aop:config>
        <!--切入点：在哪个地方执行,我们需要在execution()里面写一个表达式来定位-->
        <aop:pointcut id="pointcut" expression="execution(* com.bo.service.UserServiceImpl.*(..))"/>
        <!--执行环绕增加-->
        <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
        <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut"/>
    </aop:config>
</beans>
```

```xml
<aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
<aop:advisor advice-ref="afterLog" pointcut-ref="pointcut"/>
```

这两行表示要将切入类切入到哪个切入点

**advice就是要增强的功能，pointcut就是要被增强的方法，在通过advisor关联起来就ok了**

测试：

```java
public class MyTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

        //动态代理代理的是接口
        UserService userService = (UserService) context.getBean("userService");

        userService.add();
    }
}
```

### 3.2.2 自定义类来实现Aop

第一步 : 写我们自己的一个切入类：

```java
public class DiyPointcut {
    public void before(){
        System.out.println("方法执行前");
    }

    public void after(){
        System.out.println("方法执行后");
    }
}
```

可以看到这个类非常地清晰，把要加入的功能全写上了，接下来配置文件：

```xml
<bean id="diy" class="com.bo.demo01.diy.DiyPointcut"/>
    <aop:config>
        <!--自定义切面，ref加要应用的类-->
        <aop:aspect ref="diy">
            <!--切入点-->
            <aop:pointcut id="point" expression="execution(* com.bo.service.UserServiceImpl.*(..))"/>
            <!--通知：增强的方法-->
            <aop:before method="before" pointcut-ref="point"/>
            <aop:after method="after" pointcut-ref="point"/>
        </aop:aspect>
    </aop:config>
```

`execution(* com.bo.service.UserServiceImpl.*(..))`

关于这个表达式，再解释一下，‘*’代表所有的东西，比如把UserServiceImpl换成星号，即表示service包下的所有东西都增强功能，都切入。（..）表示参数不限

### 3.2.3 注解实现AOP

通过@Aspect把一个类标志位切面：

```java
@Component
@Aspect
public class AnnotationPointCut {
    @Before("execution(* com.bo.service.UserServiceImpl.*(..))")
    public void before(){
        System.out.println("方法执行前");
    }
    @After("execution(* com.bo.service.UserServiceImpl.*(..))")
    public void after(){
        System.out.println("方法执行后");
    }
}
```

记得加上@Component注册到spring



## 3.3 spring整合mybatis

#### 3.3.1 回忆MyBatis

实体类（对应数据库）：

```java
public class User {
   private int id;  //id
   private String name;   //姓名
   private String pwd;   //密码
}
```

核心配置文件：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--configuration核心配置文件-->
<configuration>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=UTF8&amp;useUnicode=true&amp;serverTimezone=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

**UserDao接口编写**

```java
public interface UserMapper {
   public List<User> selectUser();
}
```

**接口对应的Mapper映射文件**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
       PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.kuang.dao.UserMapper">

   <select id="selectUser" resultType="User">
    select * from user
   </select>

</mapper>
```

**测试类**

```java
@Test
public void selectUser() throws IOException {

   String resource = "mybatis-config.xml";
   InputStream inputStream = Resources.getResourceAsStream(resource);
   SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
   SqlSession sqlSession = sqlSessionFactory.openSession();

   UserMapper mapper = sqlSession.getMapper(UserMapper.class);

   List<User> userList = mapper.selectUser();
   for (User user: userList){
       System.out.println(user);
  }

   sqlSession.close();
}
```

#### 3.3.2 整合一

**现在，MyBatis-Spring 会帮助你将 MyBatis 代码无缝地整合到 Spring 中。**

1.除了前面的mysql，mybatis等依赖，我们还需要导包：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.3.22</version>
</dependency>

<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.9.1</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>2.0.7</version>
</dependency>
```

要和 Spring 一起使用 MyBatis，需要在 **Spring 应用上下文中**定义至少两样东西：一个 SqlSessionFactory 和至少一个数据映射器类。

2.下面来写一个spring的配置文件**spring-dao.xml**

**3.更换DataSource数据源**，使用spring的数据源替换mybatis的
我们这里使用**spring提供的jdbc DriverManagerDataSource**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--DataSource数据源，使用spring的数据源替换mybatis的
    我们这里使用spring提供的jdbc DriverManagerDataSource-->
    <bean id="datasource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=UTF8&amp;useUnicode=true&amp;serverTimezone=UTC"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
    </bean>
    <!--sqlsessionFactory-->


</beans>
```

**4.注册sqlSessionFactory**

在 MyBatis-Spring 中，可使用SqlSessionFactoryBean来创建 SqlSessionFactory。**要配置这个工厂 bean，只需要把下面代码放在 Spring 的 XML 配置文件spring-dao.xml中**：

```xml
<!--sqlsessionFactory-->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource" />
</bean>
```

然后注意绑定mybatis配置文件：

```xml
 <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <!--绑定mybatis配置文件-->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <property name="mapperLocations" value="classpath:com/bo/mapper/*.xml"/>
    </bean>
```

**这样我们不需要再再mybatis-config中配置数据源和注册mapper了！**所以，mybatis-config.xml变成了:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--configuration核心配置文件-->
<configuration>
    <!--别名管理-->
    <typeAliases>
        <package name="com.bo.pojo"/>
    </typeAliases>
</configuration>
```

只有一点东西了

**5.现在我们有工厂了，但spring中还没有sqlSession**

```
SqlSessionTemplate就是我们之前用的sqlSession！！！
```

```xml
<!--注册sqlSessionTemplate , 关联sqlSessionFactory-->
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
   <!--利用构造器注入-->
   <constructor-arg index="0" ref="sqlSessionFactory"/>
</bean>
```

只能用构造器注入sqlSession，因为它没有set方法（源码）



**6.增加Dao（Mapper）接口的实现类；私有化sqlSessionTemplate**

```
我们的所有操作在原来使用sqlSession，现在用sqlSessionTemplate
```

```java
public class UserMapperImpl implements UserMapper{
    //我们的所有操作在原来使用sqlSession，现在用sqlSessionTemplate
    private SqlSessionTemplate sqlSession;

    public void setSqlSessionTemplate(SqlSessionTemplate sqlSessionTemplate) {
        this.sqlSession = sqlSessionTemplate;
    }

    //得到sqlSessionTemplate之后，我们就可以用了
    @Override
    public List<User> selectUser() {
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        return mapper.selectUser();
    }
}
```

我们在spring容器中已经注册了一个sqlSessionTemplate对象，我们只需要再注册一个实现类对象**，用ref把sqlSessionTemplate对象set进去就行**

测试类：

```java
public class Mytest {
    @Test
    public void test1(){
        ApplicationContext context = new ClassPathXmlApplicationContext("spring-dao.xml");
        //只需要拿到那个实现类就可以操作
        UserMapperImpl userMapper = (UserMapperImpl) context.getBean("userMapper");
        for (User user : userMapper.selectUser()) {
            System.out.println(user);
        }

    }
}
```

##### 3.3.2.* 理解和报错

第一次测试，结果报错

```
org.apache.ibatis.binding.BindingException: Type interface com.bo.mapper.UserMapper is not known to the MapperRegistry.
```

我们看到UserMapper is not known to the MapperRegistry.我们是通过usermapper.xml来进行映射，将UserMapper注册进去，**说明usermapper.xml在spring中的配置路径有问题或者UserMapper.xml的绑定接口有错误**，这两种情况，看了一下spring配置文件，没问题，那就只能是第二种情况：

```xml
<mapper namespace="com.bo.dao.UserMapper">

    <select id="selectUser" resultType="User">
        select * from user
    </select>

</mapper>
```

果然，包名写错了,改为：

```xml
<mapper namespace="com.bo.mapper.UserMapper">
```

成功输出



一个理解：

我们知道学mybatis的时候说usermapper.xml这个文件其实就相当于UserMapper接口的实现类了，那为什么还要加一个实现类呢？

可以理解为usermapper.xml这个文件其实是dao层，而加的实现类是业务层，业务层调用dao层实现业务

其实我们可以调整一下配置文件的分布来理解这个问题：

原先我们只用一个spring-dao.xml来配置所有bean，我们现在加一个总的配置文件applicationContext.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd">
    
<!--DataSource数据源，使用spring的数据源替换mybatis的
我们这里使用spring提供的jdbc DriverManagerDataSource-->
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=UTF8&amp;useUnicode=true&amp;serverTimezone=UTC"/>
    <property name="username" value="root"/>
    <property name="password" value="123456"/>
</bean>
        <!--sqlsessionFactory-->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
<property name="dataSource" ref="dataSource" />
<!--绑定mybatis配置文件-->
<property name="configLocation" value="classpath:mybatis-config.xml"/>
<property name="mapperLocations" value="classpath*:com/bo/mapper/*.xml"/>
</bean>

        <!--注册sqlSessionTemplate , 关联sqlSessionFactory-->
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
<!--利用构造器注入-->
<constructor-arg index="0" ref="sqlSessionFactory"/>
</bean>
    
</beans>
```

这里面放的都是不变的代码，以后整合mybatis把这段复制进去就好了。

变化的代码就是我们的业务层不同的业务有不同的实现类，就要注册不同的mapper，所有spring-dao.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd">

<import resource="applicationContext.xml"/>

    <bean id="userMapper" class="com.bo.mapper.UserMapperImpl">
        <property name="sqlSessionTemplate" ref="sqlSession"/>
    </bean>
</beans>
```

把applicationContext.xml   import进去就可。



#### 3.3.3 整合二

 dao继承Support类 , 直接利用 getSqlSession() 获得 , 然后直接注入SqlSessionFactory . 比起方式1 , 不需要管理SqlSessionTemplate , 而且对事务的支持更加友好 . 可跟踪源码查看

示例：

我们再加一个mapper接口的实现类(继承SqlSessionDaoSupport)：

```java
public class UserMapperImpl2 extends SqlSessionDaoSupport implements UserMapper {
    @Override
    public List<User> selectUser() {
        System.out.println("这是方式2，不需要注册sqlSession");
        return getSqlSession().getMapper(UserMapper.class).selectUser();
    }
}
```

然后注册实现类即可：

```xml
<bean id="userMapper2" class="com.bo.mapper.UserMapperImpl2">
    <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
</bean>
```

