title: SpringBoot
tags:
  - SpringBoot
categories:
  - Spring
date: 2019-03-19 09:53:00
---
### 配置Springboot

<!--more-->

### 入门

#### 1、简介
简化框架，整个spring技术栈的整合。

#### 2、微服务
是一种架构风格，一个应用应该是一组小型服务；服务间可以通过http方式进行互通，每一个功能元素都是一个可独立替换和升级的软件单元。

#### 3、入门使用

#### 1. 配置maven


进入maven的conf目录，配置settings文件，profiles中添加以下使得maven使用1.8编译项目：

```
    <profile>
      <id>jdk-1.8</id>

      <activation>
        <jdk>1.8</jdk>
      </activation>

	  <properties>
		<maven.compiler.source>1.8</maven.compiler.source>
		<maven.compiler.target>1.8</maven.compiler.target>
		<maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
	  </properties>
    </profile>

```

配置idea，setting中选择，将默认的maven换为自己下载的。

maven换源：

```
	<mirror>
	  <id>nexus-aliyun</id>
	  <mirrorOf>central</mirrorOf>
	  <name>Nexus aliyun</name>
	  <url>http://maven.aliyun.com/nexus/content/groups/public</url>
	</mirror>
	 
```

maven 修改本地仓库:
`<localRepository>D:\apache-maven-3.6.3\repository</localRepository>`


#### springhelloworld

##### 1. 配置一个maven工程，idea中新建maven工程

##### 2. 导入springboot相关依赖，官网有quickstart，可以直接copy下来，或者点击网站自动generate一个demo

```

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

```

##### 3. 编写启动主程序：

```java

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class HelloSpringApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    @GetMapping("/hello")
    public String hello(@RequestParam(value = "name", defaultValue = "World") String name) {
        return String.format("Hello %s!", name);
    }
}

```

##### 4. 启动

直接idea启动或者cli：`mvnw spring-boot:run`,可以添加`?name=tr`访问

主程序文件说明：

  1. `@RestController` ：`controller`标识，`spring`用来标记的
  2. `@RequestMapping` ： url标识
  3. `@EnableAutoConfiguration Annotation`：This annotation tells Spring Boot to “guess” how you want to configure Spring, based on the jar dependencies that you have added. Since spring-boot-starter-web added Tomcat and Spring MVC, the auto-configuration assumes that you are developing a web application and sets up Spring accordingly
  4. main方法：Spring应用程序启动入口，自动启动tomcat
  
 
 
##### 5. 自定义controller

```java
package com.tr.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HelloController {
    @ResponseBody
    @RequestMapping("/hi")
    public String hi() {
        return "hi";
    }
}


```

##### 6.部署

1. 添加部署插件,用于将应用打包成可执行jar包

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```
2. 点击右侧插件的package即可

3. `java -jar ***` 即可运行


##### 7. 备注

以上都是比较老的操作方法了，需要自己copy配置文件，新版的spring可以直接完成以上配置，`new project`，选择`spring initializr`，选择`web`或者其他自己要的场景，下一步即可

##### 8. 创建可执行jar

1. add the spring-boot-maven-plugin to our pom.xml

2. 使用mvn命令：package 会自动打包成war包

#### spring boot 初步探究

##### helloworld 探究

1. 父项目

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
```
其父项目是：

```xml
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.2.6.RELEASE</version>
    <relativePath>../../spring-boot-dependencies</relativePath>
  </parent>
```
用于管理所有依赖的版本号，这样以后写依赖不需要写版本号

2. 依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.2.5.RELEASE</version>
        </dependency>
```
`spring boot` 场景启动器:导入web需要的大部分依赖，tomcat啥的

3. 主程序类：

```java
@SpringBootApplication
@RestController
public class HelloSpringApplication {

    public static void main(String[] args) {

        SpringApplication.run(HelloSpringApplication.class, args);
    }
}
```
`@SpringBootApplication`：标注是`spring boot`主配置类，访问这个启动spring，进入查看

```java

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
```
`@SpringBootConfiguration`: 标识`springboot`配置类

`@Configuration`: 配置类，也是组件有@Component

`@EnableAutoConfiguration`: 开启自动配置

`@import`：底层注解，专门导入组件

将著配置类(@SpringBootApplication标注的类)的所在包和子包下的所有组件自动扫描到spring容器中

##### spring 初步

1. controller 注解`@Controller``@ResponseBody`是以前的controller类注解，现在使用`@RestController`代替

2. resources文件夹：

  + static：存放图片css等
  + templates：由于spring使用的嵌入式tomcat，不支持jsp，但是可以使用模板引擎
  + application.properties:配置文件，可以配置端口等，类似`server.port=1997`
  

### spring boot 配置

#### 配置文件

Spring boot 使用全局配置文件，文件名固定

 + application.properties
 + application.yml
 
 配置文件的作用：修改springboot自动配置的默认值；
 
 springboot底层都给我们自动配置好了
 
 yml:YAML 以数据为中心的配置文件
 
 ```yml
 server:
 	port: 8888
 ```
 
##### YAML语法
 
 ###### 1. 基本语法
 
 1. k:空格v 键值对标识，空格缩进标识层级关系,大小写敏感
 
 ```
 server:
 	port: 8111
   path: /hi
 ```
 
###### 2. 值的写法：
 
 普通数字、对象、map、数组等
 
 1. 普通字符 k: v
 
 字符串默认不用加上单引号或者双引号
 
 双引号：转义特殊字符,\n会变为换行
 
 单引号：不转义 输出\n等字符
 
 2. 对象、Map（键值对）
 
 k: v:还是kv模式，注意缩进
 
```yml
friends:
	name: tr
   age: 23
```
或者：
```friends:{name: tr,age: 23}```

3. 数组（list，set）

```pets:
    - pig
    - monkey
```
或者行内写法 ```pets:[cat,dog]```

##### 配置注入

1. 导入配置文件处理器
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
        </dependency>
```
2. 需要注入的组件中

```java
package com.tr.bean;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.Date;
import java.util.List;
import java.util.Map;

/**
 * 将配置文件值映射到组件
 * @CinfugurationProperties
 * @Componen 标识是容器的组件，只有容器组件才能用spring功能
  默认找全局配置文件
 */
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private Integer age;
    private Boolean boss;
    private Date birth;

    private Map<String, Object> maps;
    private List<Object> lists;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Boolean getBoss() {
        return boss;
    }

    public void setBoss(Boolean boss) {
        this.boss = boss;
    }

    public Date getBirth() {
        return birth;
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }

    public Map<String, Object> getMaps() {
        return maps;
    }

    public void setMaps(Map<String, Object> maps) {
        this.maps = maps;
    }

    public List<Object> getLists() {
        return lists;
    }

    public void setLists(List<Object> lists) {
        this.lists = lists;
    }

    public Dog getDog() {
        return dog;
    }

    public void setDog(Dog dog) {
        this.dog = dog;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", boss=" + boss +
                ", birth=" + birth +
                ", maps=" + maps +
                ", lists=" + lists +
                ", dog=" + dog +
                '}';
    }

    private Dog dog;
}

```

3. application.yml中配置注入数据

```yml
server:
  port: 8081

person:
  age: 23
  boss: false
  birth: 1997/01/01
  maps: {k1: v1,k2: v2}
  lists:
    - tzq
    - tt
  dog:
    name: pp
    age: 2
  name: tr
```

4. properties配置类似

需要修改ide文件编码

注意的是：两张配置文件都存的情况下，properties会覆盖yml配置

5. 可以使用@Value注解注入值

！！！注意：只有注解类似`@Autowried`才生效，new出来的不生效！！！

`@Vaule("${person.name}")`

如果我们只是获取配置文件中某项值用`@Value` ,批量用`@ConfigurationProperties`

6. 加载指定配置文件

因为默认注入是找的全局配置文件：`application.properties`

指定文件：`@PropertySource(value="classpath:person.properties")`

如果是yml无法指定，因为@PropertySource目前不支持yml的解析

7. 导入spring配置文件

`@ImportResource`：导入自己写的配置文件，需要标注在配置类上，比如配置在@SpringBootApplication，`@ImportResource(locations = {"classpath:beans.xml"})`

spring boot 推荐给容器添加组件的方式：

a. 配置类===spring配置文件（不推荐）

b. @Configuration:在config包下写一个配置类

```java

/**
 * 标识当前类是配置类
 */
@Configuration
public class MyAppConfig {
    //将方法返回值添加到容器，容器的这个组件默认id为方法名
    @Bean
    public HelloService helloService() {
        return new HelloService();
    }
}
```

##### 配置文件占位符

1. ${random.value}随机数

2. 可以在配置文件中引用之前配置的属性`${person.name:默认值}`如果找不到赋值默认属性

##### Profile

不同环境不同profile

1. 多profile文件

编写主配置文件，文件名可以是 application-{dev}.yml/properties,激活对应配置文件需要在主配置文件（application.yml）中写入
```spring:
  profiles:
    active: dev
```

2. 文档块（只支持yml）

```yml
server:
  port: 8081

spring:
  profiles:
    active: dev2
---
server:
  port: 8888
spring:
  profiles: dev2
```

##### 配置文件加载位置

加载优先级（高到低）高的配置文件覆盖低优先级的

`classpath`为`resource`目录
`file`为项目根目录与`src`同级

+ file:./config/
+ file:./
+ classpath:/config
+ classpath:/


还可以通过`spring.config.location=D:/xxx.properties`改变more配置文件位置

项目打包后，可以使用命令行参数的形式指定配置文件位置，还可以外部配置文件的方式，不过这种工作一般运维做，以后遇到了再查文档