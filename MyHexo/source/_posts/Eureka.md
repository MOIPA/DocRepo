title: Eureka
author: tr
tags:
  - eureka
categories:
  - SpringCloud
date: 2019-03-29 08:46:00
---
### SpringCloud--Eureka

1. 什么是eureka：服务注册和发现组件

<!--more-->

#### 配置Eureka-server

1. 创建maven主工程且pom.xml如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>tr</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
        <relativePath></relativePath>
    </parent>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

</project>
```

2. 创建eureka-server，使用spring-initlizaor maven版本
	pom如下
    
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>tr</artifactId>
        <version>1.0-SNAPSHOT</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <groupId>com.eureka</groupId>
    <artifactId>eureka-server</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>eureka-server</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
            <version>1.4.6.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

3. eureka-server的application.yml

```yml
server:
  port: 8761
eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

4. 在Application文件里，在类上添加@EnableEurekaServer

#### 配置Eureka-client

1. 同上创建client工程，pom里面父亲为主工程，添加自己的依赖
```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
            <version>1.4.6.RELEASE</version>
        </dependency>
```

2. 配置client的application.yml

```yml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
spring:
  application:
    name: eureka-client
server:
  port: 8762
  
```

3. 在Application类中开启@EnableEurekaClient

#### Eureka 名词

1. Register服务注册，EurekaClient想EUrekaServer注册时，会提供自己的元数据信息和圆口，ip等信息。
2. Renew 服务续约：eurekaclient每隔30s就发送一次心跳来进行续约，如果server90秒内没有收到心跳则会将这个client实例从注册列表中删除
3. FetchRegistries获取服务注册列表信息：client从server获取其他服务信息，从而远程调用，client和server可以使用json和xml数据格式进行通信
4. cancel服务下线：client在程序关闭时可以向eurekaserver发送下线请求，那么该客户端的示例信息从server服务注册列表中删除。需要调用代码

	```java
    DiscoverManager.getInstance().shutdownComponent();
   ```
5. Eviction服务剔除：90秒没有发送到server心跳就被剔除

#### Eureka 概念

1. EurekaClient实际上分为：ApplicationService，ApplicationClient
2. server通过在配置文件指向其他节点的defaultZone可以将对方视为伙伴节点，之后会同步节点之间的注册过了的服务

#### Eureka 继承在一个maven工程

1. client和server的配置大致相似，也和上面的差不多，下面是client的pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.eureka</groupId>
        <artifactId>tr</artifactId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../</relativePath> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.eureka</groupId>
    <artifactId>eureka-client</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>eureka-client</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
            <version>1.4.6.RELEASE</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

2. 给出主模块的pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.eureka</groupId>
    <artifactId>tr</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>pom</packaging>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
        <relativePath></relativePath>
    </parent>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <modules>
        <module>eureka-server</module>
        <module>eureka-client</module>
    </modules>

</project>
```

#### 运行示例

1. 我在示例中写了两个模块，每个模块生成了一个jar包，在server的模块的jar包目录处使用：

	java -jar eureka-server-0.0.1-SNAPSHOT.jar --spring.profiles.active=peer2

	java -jar eureka-server-0.0.1-SNAPSHOT.jar --spring.profiles.active=peer2

	这样通过使用这个命令，选择了对应的profile，启动了两个端口不同互相为同伴节点的server
   
2. 这时候启动client，client向server：peer1注册，但是peer2很快也会同步

3. localhost:8761 查看结果