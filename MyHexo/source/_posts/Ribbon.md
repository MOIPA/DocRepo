title: Ribbon
author: tr
tags:
  - Ribbon
categories:
  - SpringCloud
date: 2019-04-01 09:10:00
---
### Ribbon 均衡负载器

#### 概念

Ribbon 是 Netflix 公司开源的一个负载均衡的组件，它属于上述的第二种方式，是将负载 均衡逻辑封装在客户端中，并且运行在客户端的进程里。 Ribbon 是一个经过了云端测试的 IPC 库，司以很好地控制 HTTP 和 TCP 客户端的负载均衡行为。 在 Spring Cloud 构建的微服务系统中， Ribbon 作为服务消费者的负载均衡器，有两种使用
方式，一种是和 RestTemplate 相结合，另一种是和 Feign 相结合。Feign 已经默认集成了 Ribbon

<!--more-->

#### 使用Ribbon和RestTemplate

 **本次示例说明**
 
 现有一个服务名为eureka-client的服务，启动了两个，一个是8762，一个是8763端口，现在启动一个名为eureka-client-ribbon端口，访问eureka-client服务，使用ribbon技术自动选择两个端口的一个。
 
 添加依赖：
 
 ```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-ribbon</artifactId>
            <version>1.4.6.RELEASE</version>
        </dependency>

```

1. 编写一个RibbonConfigBean，只需要在程序的 IoC 容器中注入一个restTemplate 的 Bean ， 并在这个 Bean 上加上@LoadBalanced 注解，此时 RestTemplate 就结合了 Ribbon 开启了负载均衡功能，代码如下：

	```java
    package com.eureka.eurekaclientribbon.bean;

    import org.springframework.cloud.client.loadbalancer.LoadBalanced;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.web.client.RestTemplate;

    @Configuration
    public class RibbonConfig {
        @Bean
        @LoadBalanced
        RestTemplate restTemplate() {
            return new RestTemplate();
        }
    }
   ```
2. 写一个RibbonService 类，在该类的 hi（）方法用 restTemplate 调用 eureka-client 的 API 接口 ， 此时 Uri 上不需要使用硬编码（例如 IP 地址），只需要写服务名 eureka-client 即可，代码如下：
	
    ```java
    package com.eureka.eurekaclientribbon.service;

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Service;
    import org.springframework.web.client.RestTemplate;

    @Service
    public class RibbonService {
        @Autowired
        RestTemplate restTemplate;

        public String hi(String name) {
            return restTemplate.getForObject("http://eureka-client/hello?name=" + name, String.class);
        }
    }

    ```
3. 编写RibbonController 给出调用接口，本身访问其他接口
	
    ```java
    	package com.eureka.eurekaclientribbon.controller;

      import com.eureka.eurekaclientribbon.service.RibbonService;
      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.RequestParam;
      import org.springframework.web.bind.annotation.RestController;

      @RestController
      public class HiController {

          @Autowired
          RibbonService ribbonService;

          @GetMapping("/hello")
          public String hello(@RequestParam(required = true, defaultValue = "tr") String name) {
              return ribbonService.hi(name);
          }

      }

    ```
4. 代码总结：
	本质上是在调用者的controller使用一个RestTemplate通过http请求其他服务。这个RestTemplate需要自己写成一个bean，然后注解上@LoadBalanced即可使用。
    
    **千万不要忘记开启@EnableEurekaServer和@EnableEurekaClient**