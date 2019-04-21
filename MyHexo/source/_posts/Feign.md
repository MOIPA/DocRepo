title: Feign
author: tr
tags:
  - Feign
categories:
  - SpringCloud
date: 2019-04-08 16:15:00
---
### 声明式调用Feign

使用feign远程调度其他服务

<!--more-->


#### pom

```xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.eureka</groupId>
    <artifactId>eureka-fegion-client</artifactId>
    <version>1.0-SNAPSHOT</version>
    <parent>
        <groupId>com.eureka</groupId>
        <artifactId>tr</artifactId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../</relativePath>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-feign</artifactId>
            <version>1.4.6.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
            <version>1.4.6.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>


</project>
```

#### application.yml

```yml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/

spring:
  application:
    name: eureka-fegion-client

server:
  port: 8765

```
#### EurekaFegionClientApplication.java

```java
package com.tr;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

/**
 * @author tassa
 */
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
public class EurekaFegionClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaFegionClientApplication.class, args);
    }
}

```


### 开启feign 下面使用

1. FeignConfig.java

```java
package com.tr;

import feign.Retryer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FeignConfig {
    @Bean
    public Retryer feignRetryer() {
        return new Retryer.Default(100, 10, 5);
    }
}



```

2. EurekaClientFeign.java

```java
package com.tr;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

/**
 * value:要消费的服务名  自动负载均衡
 *configuration:配置bean
 */
@FeignClient(value = "eureka-client",configuration = FeignConfig.class)
public interface EurekaClientFeign {
    /**
     * value这里的GetMapping指的是被调用放的url
     * @param name
     * @return
     */
    @GetMapping(value = "/hello")
    String sayHiFromEurekaClientFeign(@RequestParam(value = "name")String name);
}


```

3. hi.java

```java
package com.tr.controller;

import com.netflix.discovery.converters.Auto;
import com.tr.service.HiService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class hi {
    @Autowired
    private HiService hiService;

    @GetMapping("/hi")
    public String sayHi(@RequestParam(value = "name",required = false) String name) {
        return this.hiService.sayHi(name);
    }
}

```