---
title: SpringBoot
date: 2019-03-19 09:53:11
tags: SpringBoot
categories: Spring
---

### 配置Springboot

<!--more-->



### 使用maven配置  pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>SpringBootDemo</groupId>
  <artifactId>SpringBootDemo</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.5.RELEASE</version>
  </parent>
  <name>SpringBootDemo Maven Webapp</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-autoconfigure</artifactId>
      <version>2.0.7.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot</artifactId>
      <version>2.0.7.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot</artifactId>
      <version>2.0.7.RELEASE</version>
    </dependency>
  </dependencies>

  <build>
    <finalName>SpringBootDemo</finalName>
    <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
      <plugins>
        <plugin>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
        <plugin>
          <artifactId>maven-clean-plugin</artifactId>
          <version>3.1.0</version>
        </plugin>
        <!-- see http://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_war_packaging -->
        <plugin>
          <artifactId>maven-resources-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.8.0</version>
        </plugin>
        <plugin>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>2.22.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-war-plugin</artifactId>
          <version>3.2.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-install-plugin</artifactId>
          <version>2.5.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-deploy-plugin</artifactId>
          <version>2.8.2</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
</project>
```



### first start web application Java Code

```java
package com.tr.controller;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@EnableAutoConfiguration
public class ExampleController {
    @RequestMapping("/")
    String Home() {
        return "hello";
    }

    public static void main(String[] args) {
        SpringApplication.run(ExampleController.class, args);
    }

}
```

1. @RestController ：controller标识，spring用来标记的
2. @RequestMapping ： url标识
3. @EnableAutoConfiguration Annotation：This annotation tells Spring Boot to “guess” how you want to configure Spring, based on the jar dependencies that you have added. Since spring-boot-starter-web added Tomcat and Spring MVC, the auto-configuration assumes that you are developing a web application and sets up Spring accordingly
4. main方法：Spring应用程序启动入口，自动启动tomcat

5. 其他启动方式：maven命令-> spring-boot:run

### 创建可执行jar

1. add the spring-boot-maven-plugin to our pom.xml

2. 使用mvn命令：package 会自动打包成war包



### Starter模板

1. springboot提供了很多依赖模板，使得我们不用再去配置不同的maven依赖
   1. 例子：如果配置SpringDataJPA，不用写那么多依赖了直接：添加spring-boot-starter-data-jpa依赖即可
   2. starter的依赖命名都是spring-boot-starter-*
2. 很多模板比如jdbc，tomcat，jetty都可以找到



### 目录结构

1. com.example.myapplication/Application.java 存放main入口

   ​						  /Custromer/Customer.java

   ​						  /CustomerController.java

   ​						  /CustomerService.java


2. Application.java 里面的方法定义：

   ​	

### Bean注解

1. 可以使用@ComponentScan 这个可以将所有bean自动识别为SpringBeans且不需要参数
2. 也可以使用spring的@Component, @Service, @Repository, @Controller

3. 使用@Autowired注入 

4. • @EnableAutoConfiguration: enable Spring Boot’s auto-configuration mechanism 

   • @ComponentScan: enable @Component scan on the package where the application is located (see the best practices)

   • @Configuration: allow to register extra beans in the context or import additional configuration classes

   The @SpringBootApplication annotation is equivalent to using @Configuration, @EnableAutoConfiguration, and @ComponentScan with their default attributes, as

### 开发

#### Actuator

 	1. 配置pom：spring-boot-starter-actuator
 	2. 