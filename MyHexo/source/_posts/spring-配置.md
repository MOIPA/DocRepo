---
title: spring 配置
date: 2019-03-11 16:50:38
tags: spring
---

<!--more-->

### maven的pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.swb</groupId>
  <artifactId>helloSpring</artifactId>
  <packaging>war</packaging>
  <version>0.0.1-SNAPSHOT</version>
  <name>helloSpring Maven Webapp</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>${springVersion}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>${springVersion}</version>
    </dependency>
  </dependencies>
  <build>
    <finalName>helloSpring</finalName>
  </build>
  <properties>
      <springVersion>3.2.5.RELEASE</springVersion>
  </properties>
</project>
```



### 配置resources/applicationContext.xml

##### 包含了所有bean对象的创建和注入

##### spring使用配置文件通过控制反转和依赖注入使得将类和类之间解耦

```xml

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--配置user对象 交给spring容器
    Bean元素：使用该元素需要spring管理的对象
        name：给对象起名用与获得，可以重复
        class: 被管理对象的完整类名
        id(早期)：和name一样，名称不可重复
    -->
    <!--创建方式1：使用空参构造创建
                2：静态工厂
                3：实例工厂
    -->
    <!--scope 1:singleton单例对象 被标识为单例对象，这也是默认属性 大部分使用默认
              2：prototype 每次创建都是新的  如果和struts配合 由于action每次都是新的 要用prototype
              3：request 和request生命周期一样      （不用
              4：session对象与session生命周期一致  （不用
    -->
    <!--生命周期属性 1. init-method 配置一个方法作为生命周期初始化方法，在对象创建后立即调用
                    2. destory-mothod 配置。。。销毁。。。               关闭时销毁
    -->
    <!--分模块配置 1.import 导入其他配置文件
        <import resource="applicationContext.xml" />
    -->

    <bean name="user" class="com.tr.bean.User" scope="prototype" init-method="init" destroy-method="destory"></bean>

    <!--属性注入-->
    <bean name="user2" class="com.tr.bean.User">
        <!--为user对象中名为name的属性注入tom作为值-->
        <property name="name" value="tom"/>
        <property name="age" value="18"/>
        <!--注入对象 得先配置此对象 引用类型用ref-->
        <property name="car" ref="car"/>
    </bean>
    <!--构造用于被引用的对象-->
    <bean name="car" class="com.tr.bean.Car">
        <property name="name" value="奔驰"></property>
        <property name="color" value="white"></property>
    </bean>

    <!--复杂对象注入-->
    <!--<bean name="cb" class="com.tr"-->
    <bean name="cu" class="com.tr.bean.ComplexUser">
        <!--<property name="arr" value="tom"></property>  &lt;!&ndash;只注入一个值&ndash;&gt;-->
        <property name="arr">
            <array>
                <value>tom</value>
                <value>jerry</value>
            </array>
        </property>
        <property name="list">
            <list>
                <value>test</value>
                <value>test</value>
            </list>
        </property>
        <property name="map">
            <map>
                <entry key="tr" value="1"/>
                <entry key="car" value-ref="car2"/>    <!--值 放入对象 -->
                <entry key-ref="car2" value-ref="car2"/>    <!--键 放入对象 -->
            </map>
        </property>
        <property name="properties">
            <props>
                <prop key="tr">111</prop>
                <prop key="tr">111</prop>
            </props>
        </property>
    </bean>
<!--================-->
    <!--构造函数注入-->
    <!--index:因为两个构造函数由于入参顺序不同变为重载函数，可以指定哪个参数先入
            0，1，2，3，4
         type: 指定入参类型 比如 java.lang.Integer
    -->
    <bean name="car2" class="com.tr.bean.Car" >
        <constructor-arg name="color" value="white"/>
        <constructor-arg name="name" value="benz"/>
    </bean>
</beans>
```



### 创建和使用容器对象

```java
public class Demo {
    @Test
    public void func1() {
        //1 创建容器对象
        ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
        //2 向容器要javaBean
//        User user = (User) ac.getBean("user");
        User user = (User) ac.getBean("user2");
        //3打印user对象
        System.out.println(user);

        //通过构造函数配置的对象
        Car car2 = (Car) ac.getBean("car2");

        ComplexUser cu = (ComplexUser) ac.getBean("cu");
        System.out.println(cu.getArr()[0]);
        System.out.println(cu.getArr()[1]);
        //4(可选) 销毁操作
        ac.close();
    }
}
```



#### 在Action或controller中使用

 1. 需要将ClassPathXmlApplication对象配置在Application域中，生命周期和项目一致，放在action对象里会导致创建n个

    这需要在web.xml中配置监听器 

    ```xml
    <!-- 可以让spring容器随着项目启动而启动，销毁而销毁-->
    <listener>
    	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <!--指定加载spring配置文件的位置-->
    <context-param>
    	<para-name>contextConfigLocation</para-name>
        <para-value>classpath:applicationContext.xml</para-value>
    </context-param>
    ```

 2. struts 项目 在把spring放入容器于项目application后，再取出使用

    ```java
    //1.获得servletContext
    ServletContext sc = ServletActionContext.getServletContext();
    //2.从sc中获得spring容器
    WebApplicationContext ac = WebApplicationContextUtils.getWebApplicationContext(sc);
    ac.getBean("customerService");
    ```
