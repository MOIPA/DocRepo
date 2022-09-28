title: Spring Basic
tags:
  - spring
categories:
  - Spring
date: 2019-03-11 17:12:00
---
# spring Basic



<!--more-->

> `Spring` 的核心：`AOP`,`IOC`
>
> 带来了以下的优势
>
> > 1. 为什么说`Spring`解耦了？
> >
> > 原始`javaEE`的开发时，内部所有对象都是自己手动声明的，因此类和类的耦合也重，`Spring`带来的第一个好处就是类的解耦，`Spring`将所有程序中需要生成的对象由他自己控制，每个对象(`bean`)的创建到死亡都由`Spring`来管理
> >
> > 这样带来的好处是每个对象想要使用其他对象时，只需要问`Spring`要就行了，类和类之间不在有直接的耦合关系
>
> 
>
> > 2. AOP 的好处？
> >
> > 面向切面编程，提供了一个统一管理接口的方法
>
> 
>
> > 3. 事务开发？
> >
> > 原来的事务开发是真正的编程式的，在开启一个事务前你得写一些控制代码，结束了事务也得写，Spring带来了配置方式，只要配置好那些方法需要事务即可，不需要像原来那样每个方法写事务控制代码
>
> 
>
> > 4. 方便测试？
> >
> > spring带来的junit4等工具可以方便快捷的编写测试用例
>
> 
>
> > 5. spring还集成了许多其他框架如 `Structs`,`Hibernate`等
>
> 
>
> > 6. 封装了很多工具的API如JDBC，JDBC原始的代码是很麻烦的，Spring封装填写了很多参数



# Spring IOC内容



## 配置bean



> 上面说了，以后所有的对象都不再由我们手动new出来了，都交给spring来生成，我们拿过来用就行了，那么spring怎么知道哪些对象(Bean)要生成出来呢，我们需要提供一个xml配置文件告诉它
>
> 不过首先要知道怎么定位一个类文件，在java开发中，每个类文件属于一个包，我们可以通过`包名+文件名`找到类，假如一个文件名为`test.java`处在`com.tr.example`包下，那么我们可以通过`com.tr.example.test`定位
>
> 
>
> 光说无法体会，以下以一个例子说明



## 搭建spring环境



> 1. 先用maven导入spring的包
> 2. 创建对象
> 3. 配置文件内配置对象
> 4. 通过spring获取对象



> 新建maven工程，打开idea，选择maven，随便选择启动器都行，反正pom文件自己配置就行了
>
> 在pom文件中添加spring的核心依赖：`spring-context` 这个依赖包含了5个核心包，写好后点击同步等待包下载即可

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.3.18</version>
        </dependency>
    </dependencies>
```

> 配置好依赖后，创建interface接口和对应的实现，众所周知interface接口可以对应很多个实现，具体对应哪个我们可以xml中配置，先创建interface代码如下

```java
package com.tr.example;

public interface UserDao {
    public void save();
}
```

> 具体实现

```java
package com.tr.example.impl;

import com.tr.example.UserDao;

public class UserDaoImpl implements UserDao {

    @Override
    public void save() {
        System.out.println("hello from user dao");
    }
}
```

> 编写配置bean 文件名就叫ApplicationConfig.xml放在resources目录下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userDao" class="com.tr.example.impl.UserDaoImpl"/>
</beans>
```

> 这步bean的配置定义基本完成了，现在其他类可以轻松的从spring中获取这些bean了

```java
package com.tr.example.controller;

import com.tr.example.UserDao;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class UserDaoDemo {

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("ApplicationConfig.xml");
        UserDao userDao = (UserDao) context.getBean("userDao");
        userDao.save();
    }
}
```

> 执行代码即可看到效果
>
> 这里可以看到使用类的所有对象都不是自己生成的，都是从spring获取的

## Spring的配置文件

### Bean标签的基本配置

> 所有对象交由Spring创建，默认情况下Spring创建是调用无参构造函数的类似`new A()`
>
> id ：唯一性标识用来标志类，不可以重复
>
> class：类的全路径标识



### Bean的标签范围配置

> scope：对象的作用范围
>
> + singleton：默认的,单例  在加载配置文件的时候，实例就已经被创建了，当应用停止时才销毁
> + prototype：多例的  只有bean被获取时才创建一个新的返回，无指针引用就被gc回收了
> + request：Spring创建后会将对象存入request中
> + session：创建后存入session中
> + global session：创建后存入Portle环境，若无Portle环境则存入session
>
> 
>
> 测试scope很简单，只要修改例子中的xml配置即可，可以在代码内打印bean的地址查看`<bean id="userDao" class="com.tr.example.impl.UserDaoImpl" scope="prototype"/>`

### Bean的声明周期

> 可以在xml中给类声明创建和销毁时调用的方法

```java

public class UserDaoImpl implements UserDao {
    public UserDaoImpl() {
        System.out.println("user dao impl created");
    }
    public void init(){
        System.out.println("init user dao");
    }
    public void destroy(){
        System.out.println("destroy user dao");
    }
    @Override
    public void save() {
        System.out.println("hello from user dao");
    }
}
```

```xml
    <bean id="userDao" class="com.tr.example.impl.UserDaoImpl" init-method="init" destroy-method="destroy"/>
```

### Bean实例化的三种方式

> + 无参构造
> + 工厂静态方法实例化
> + 工厂实例方法实例化
>
> 
>
> 无参构造很简单，提供构造函数即可，下面研究工厂方法，首先创建静态工厂类

```java
package com.tr.example.factory;

import com.tr.example.UserDao;
import com.tr.example.impl.UserDaoImpl;

public class StaticFactory {

    public static UserDao getUserDao(){
        return new UserDaoImpl();
    }
}
```

> 然后告诉spring，创建这个对象使用工厂来创建

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

<!--    <bean id="userDao" class="com.tr.example.impl.UserDaoImpl" init-method="init" destroy-method="destroy"/>-->
        <bean id="userDao" class="com.tr.example.factory.StaticFactory" factory-method="getUserDao"/>
</beans>
```

> 这就是静态工厂的方式了
>
> 
>
> 再来看看动态工厂，创建工厂

```java
package com.tr.example.factory;

import com.tr.example.UserDao;
import com.tr.example.impl.UserDaoImpl;

public class DynamicFactory {
    public  UserDao getUserDao(){
        return new UserDaoImpl();
    }
}
```

> 配置xml，首先要声明一个Factory的bean，我们目标bean需要和Factory的bean关联

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--    <bean id="userDao" class="com.tr.example.impl.UserDaoImpl" init-method="init" destroy-method="destroy"/>-->
    <!--        <bean id="userDao" class="com.tr.example.factory.StaticFactory" factory-method="getUserDao"/>-->
    <bean id="myFactory" class="com.tr.example.factory.DynamicFactory"/>
    <bean id="userDao" factory-bean="myFactory" factory-method="getUserDao"/>
</beans>
```

> 为什么要用工厂？ 不觉得麻烦吗？其实是因为很多对象创建的时候不是单单一个new那么简单，创建的对象可能引用了很多其他对象，需要再new更多对象作为参数传入，使用工厂就相当于有人帮我们完成了复杂的创建逻辑



### Bean的依赖注入

> 先配置好三层架构：`controller,dao,service`，在xml中配置好service和dao层的bean

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--    <bean id="userDao" class="com.tr.example.impl.UserDaoImpl" init-method="init" destroy-method="destroy"/>-->
    <!--        <bean id="userDao" class="com.tr.example.factory.StaticFactory" factory-method="getUserDao"/>-->
    <!--    <bean id="myFactory" class="com.tr.example.factory.DynamicFactory"/>-->
    <!--    <bean id="userDao" factory-bean="myFactory" factory-method="getUserDao"/>-->
    <bean id="userDao" class="com.tr.example.dao.impl.UserDaoImpl"/>
    <bean id="userService" class="com.tr.example.service.impl.UserServiceImpl"/>
</beans>
```

> 在controller层中新建方法使用service的bean

```java
package com.tr.example.controller;

import com.tr.example.service.UserService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class UserController {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("ApplicationConfig.xml");
        UserService userService = (UserService) context.getBean("userService");
        userService.save();
    }
}
```

> 这里其实有问题，我们代码中从spring那里获取了service的bean，调用了save方法，执行到save方法的时候又去spring里获取了dao的bean，然后调用save方法，可以看出service层的这个类是引用了dao层的类的，他们存在耦合关系，但是当前做法是从外部将dao层的bean注入进去的：即手动获取调用
>
> 但是这样不好，我们更希望不要自己手动去spring里面获取bean，让spring把需要的bean送过来，我只需要声明一下`private 类 a` 像这样 spring自动把对应的bean注入到`a`中，spring提供了在内部结合的方式DI(Dependency injection)，通过xml配置和set的方式注入
>
> 
>
> 先配置下我们需要被注入的类，写上set方法给spring调用

```java
public class UserServiceImpl implements UserService {

    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public void save() {
        System.out.println("do service");
        userDao.save();
    }
}
```

> 我们需要通知spring，把UserDao的bean调用set方法传递进来

```xml
    <bean id="userDao" class="com.tr.example.dao.impl.UserDaoImpl"/>
    <bean id="userService" class="com.tr.example.service.impl.UserServiceImpl">
        <property name="userDao" ref="userDao"></property>
    </bean>
```

> 这样就完成了bean注入，直接运行controller的方法可以看到没问题，不过这么写xml有点啰嗦，Spring还提供了命名空间p可以精简xml书写
>
> 首先添加命名空间：`xmlns:p="http://www.springframework.org/schema/p"`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userDao" class="com.tr.example.dao.impl.UserDaoImpl"/>
    <bean id="userService" class="com.tr.example.service.impl.UserServiceImpl" p:userDao-ref="userDao"/>
</beans>
```





> Bean还提供了另一种注入方式，构造方式注入，先给个构造函数

```java

public class UserServiceImpl implements UserService {

    private UserDao userDao;

    public UserServiceImpl(UserDao userDao) {
        this.userDao = userDao;
    }

    public UserServiceImpl() {
    }

    @Override
    public void save() {
        System.out.println("do service");
        userDao.save();
    }
}
```

> 惯例，配置xml让spring调用构造函数的时候把bean注入进来

```xml
<bean id="userDao" class="com.tr.example.dao.impl.UserDaoImpl"/>
<bean id="userService" class="com.tr.example.service.impl.UserServiceImpl">
    <constructor-arg name="userDao" ref="userDao"></constructor-arg>
</bean>
```

> 测试即可
>
> 要注意的是，set的时候 name属性指的是set方法后面的小写开头的名字，构造函数模式下name指的是参数接口的名字

### Bean依赖注入的数据类型

> Bean可以注入哪些数据类型？
>
> 除了对象，普通数据和集合都可以，这里以dao层的类为例，给dao层的类注入普通数据

```java
package com.tr.example.dao.impl;

import com.tr.example.dao.UserDao;

public class UserDaoImpl implements UserDao {

    private String name;
    private int age;

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public UserDaoImpl() {
        System.out.println("user dao impl created");
    }

    public void init(){
        System.out.println("init user dao");
    }

    public void destroy(){
        System.out.println("destroy user dao");
    }

    @Override
    public void save() {
        System.out.println("hello from user dao");
        System.out.println(name+age);
    }
}

```

> 配置xml 唯一不同的是 不用ref 改用 value

```xml
<bean id="userDao" class="com.tr.example.dao.impl.UserDaoImpl">
        <property name="name" value="tr"></property>
        <property name="age" value="24"></property>
</bean>
<bean id="userService" class="com.tr.example.service.impl.UserServiceImpl">
    <constructor-arg name="userDaoBkk" ref="userDao"></constructor-arg>
</bean>
```



> 然后是集合的注入，dao层的方法中新增以下三个，并且写好set方法

```java
private List<String> strList;
private Map<String, User> userMap;
private Properties properties;
```

> 新建一个User类，这里用了Lombok插件的`@Data`少写点get set方法

```java
package com.tr.example.domain;

import lombok.Data;

@Data
public class User {
    private String name;
    private int age;
}
```

> 然后配置下xml，把dao里面用到的都注入进去

```xml
<bean id="userDao" class="com.tr.example.dao.impl.UserDaoImpl">
    <property name="strList">
        <list>
            <value>aaa</value>
            <value>bbb</value>
            <value>ccc</value>
        </list>
    </property>
    <property name="userMap">
        <map>
            <entry key="u1" value-ref="user1"></entry>
            <entry key="u2" value-ref="user2"></entry>
        </map>
    </property>
    <property name="properties">
        <props>
            <prop key="p1">p1</prop>
            <prop key="p2">p2</prop>
        </props>
    </property>
</bean>

<bean id="user1" class="com.tr.example.domain.User">
    <property name="age" value="24"/>
    <property name="name" value="tr"/>
</bean>

<bean id="user2" class="com.tr.example.domain.User">
    <property name="age" value="24"/>
    <property name="name" value="tzq"/>
</bean>

<bean id="userService" class="com.tr.example.service.impl.UserServiceImpl">
    <constructor-arg name="userDaoBkk" ref="userDao"></constructor-arg>
</bean>
```

### 引入其他配置文件（分模块）

> xml配置随便写写就容易过大，必须要分，不然很影响阅读
>
> 这里新增一个user的xml，然后主配置文件导入即可

```xml
<import resource="ApplicationConfig-user.xml">/import>
```



## Spring的api

> applicationContext ：接口类型，代表应用上下文，可以用来获取容器的bean，他有三个实现类分别是
>
> + ClassPathXmlApplicationContext：从根路径（resources目录）加载xml配置
> + FileSystemXmlApplicationContext：从磁盘路径读取配置
> + AnnotationConfigApplicationContext：使用注解配置对象时候用的，用来读取注解



### getBean()

> getBean有两种方式使用，直接传递bean的id，另一种是通过传递字节码找`context.getBean(UserService.class)`这样的好处是不用强制转换但是要注意，如果配置了两个都是UserService的bean，只是id不同，那么这种情况只能用传id的方式
>
> 不过最新版本还可以用`context.getBean(id,UserService.class)`传id和类型，也不用强转



## 数据源



### 手动读取配置生成数据源对象

> 数据源即连接池的作用：提高性能，能提前初始化资源
>
> 常见的数据源：DBCP,DRUID,C3P0

> 开发步骤
>
> + 导入依赖
> + 创建对象
> + 设置参数，驱动，数据库地址，用户名密码等

> 导入依赖

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.15</version>
</dependency>
<dependency>
    <groupId>c3p0</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.1.2</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.10</version>
</dependency>
```

> 创建对象使用

```java
package com.tr;


import com.alibaba.druid.pool.DruidDataSource;
import com.mchange.v2.c3p0.ComboPooledDataSource;
import org.junit.Test;

import java.sql.Connection;

public class DatasourceTest{


    // 测试Druid管理工具
    @Test
    public void test0()throws Exception {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://192.168.10.85:3306/user?useSSL=false");
        dataSource.setUsername("root");
        dataSource.setPassword("0800");
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        connection.close();
    }

    // 测试C3P0 管理工具
    @Test
    public void test1() throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setDriverClass("com.mysql.cj.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://192.168.10.85:3306/user?useSSL=false");
        dataSource.setUser("root");
        dataSource.setPassword("0800");
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        connection.close();
    }
}
```

> 在这两个测试用例中我们可以看到，数据库的信息配置都是手动配置上去的，后期如果修改，必须要找到相关代码配置，可以利用上节的bean注入里面的注入技术把配置信息注入

> 这里不建议xml存放配置，用peoperties文件，简单易懂
>
> 在`resources`目录建立 `jdbc.properties`

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://192.168.10.85:3306/user?useSSL=false
jdbc.username=root
jdbc.password=0800
```

> 代码里使用我们的配置

```java
// 测试C3P0 管理工具 加载配置文件形式
@Test
public void test2() throws Exception{
    // 读取配置文件
    ResourceBundle bundle = ResourceBundle.getBundle("jdbc");
    String driver = bundle.getString("jdbc.driver");
    String url = bundle.getString("jdbc.url");
    String username = bundle.getString("jdbc.username");
    String password = bundle.getString("jdbc.password");

    ComboPooledDataSource dataSource = new ComboPooledDataSource();
    dataSource.setDriverClass(driver);
    dataSource.setJdbcUrl(url);
    dataSource.setUser(username);
    dataSource.setPassword(password);
    Connection connection = dataSource.getConnection();
    System.out.println(connection);
    connection.close();
}
```



### 用spring配置数据源

> 可以发现不论是`druid`还是`c3p0`都是set什么东西，这就可以利用spring帮我们导入这些配置

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.cj.jdbc.Driver"></property>
    <property name="jdbcUrl" value="jdbc:mysql://192.168.10.85:3306/user?useSSL=false"/>
    <property name="user" value="root"/>
    <property name="password" value="0800"/>
</bean>
```

> 来测试下这种方式获取bean链接

```java
@Test
public void test3() throws Exception{
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationConfig.xml");
    ComboPooledDataSource dataSource = context.getBean("dataSource", ComboPooledDataSource.class);
    Connection connection = dataSource.getConnection();
    System.out.println(connection);
    connection.close();
}
```

### Spring读取properties配置数据源

> 上节的方式是在xml内配置数据源，对每个bean写死了配置，一般是通过读取properties文件的方式动态配置每个bean

> 引入命名空间，只需要改xml就够了

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:context="http://www.springframework.org/schema/context"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

<!--    加载配置 -->
<context:property-placeholder location="classpath:jdbc.properties"/>

<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
<property name="driverClass" value="${jdbc.driver}"></property>
<property name="jdbcUrl" value="${jdbc.url}"/>
<property name="user" value="${jdbc.username}"/>
<property name="password" value="${jdbc.password}"/>
</bean>

</beans>
```



## Spring 注解

> 刚刚的那一系列操作已经可以看到xml配置文件写起来真的非常麻烦，注解是代替xml配置的一种方式

### Spring早期版本注解



>  原始注解，主要用来替代bean配置

> + @Component 使用在类上，用于实例化Bean
> + @Controller 用在web层类
> + @Service 用在service层
> + @Repository 用在dao层
> + @Autowired 用于引用注入
> + @Qualified 接口@Autowired用于根据名称注入
> + @Resource 相当于@Autowired+@Qualified
> + @Value 注入普通属性
> + @Scope 标记bean范围(单例多例)
> + @PostConstruct bean的初始化方式指明
> + @PreDestroy 指明销毁方法

> 还是构建三层架构 controller，service，dao，分别写上方法，如同上节的bean学习，写好UserDao，UserService等方法。这不过这次不再使用xml配置文件了

```java
// UserDao的方法 加上@Component标签，相当于在xml中加入了<bean id="userDao" class="" />
package com.tr.dao.impl;

import com.tr.dao.UserDao;
import org.springframework.stereotype.Component;

@Component("userDao")
public class UserDaoImpl implements UserDao {
    @Override
    public void save() {
        System.out.println("save running");
    }
}
```

```java
package com.tr.service.impl;

import com.tr.dao.UserDao;
import com.tr.service.UserService;
import lombok.Data;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

@Component("userService")
public class UserServiceImpl implements UserService {

    @Autowired
    @Qualifier("userDao")
    private UserDao userDao;

    @Override
    public void save() {
        userDao.save();
    }
}
```

> 还需要配置组件扫描

```xml
```

