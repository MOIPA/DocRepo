title: 熔断器Hystrix
author: tr
tags:
  - Hystrix
categories:
  - SpringCloud
date: 2019-04-09 09:27:00
---
### 熔断器

在分布式系统中，服务与服务之间的依赖错综复杂， 一种不可避免的情况就是某些服务会 出现故障，导致依赖于它们的其他服务出现远程调度的线程阻塞。 Hystrix 是 Netflix 公司开 源的一个项目，它提供了熔断器功能，能够阻止分布式系统中出现联动故障。 Hystrix 是通过 隔离服务的访问点阻止联动故障的，并提供了故障的解决方案，从而提高了整个分布式系统 的弹性。

<!--more-->

某个服务的单个点的请求故障会导致用户的请求处于阻塞状态，最终的结果就是整个服务 的线程资源消耗殆尽。由于服务的依赖性，会导致依赖于该故障服务的其他服务也处于线程阻 塞状态，最终导致这些服务的线程资源消耗殆尽， 直到不可用，从而导致整个问服务系统都不 可用，即雪崩效应。
为了防止雪崩效应，因而产生了熔断器模型。 Hystrix 是在业界表现非常好的一个熔断器 模型实现的开源组件，它是 Spring Cloud 组件不可缺少的一部分。

#### 在RestTemplate和Ribbon上使用熔断器

一下配置在eureka-client-ribbon基础上配置

首先在工程的 porn 文件中引用 Hystrix 的起步依赖 spring-cloud-starter-hystrix

然后在 Spring Boot 的启动类 EurekaRibbonClientApplication 加上＠EnableHystrix 注解开启
Hystrix 的熔断器功能


修改 RibbonService 的代码， 在 hiO方法上加＠HystrixCommand 注解。有了@HystrixCommand
注解， hi（）方法就启用 Hystrix 熔断器的功能， 其中 ， fallbackMethod 为处理回退（fallback）逻 辑的方法。在本例中， 直接返回了一个字符串。在熔断器打开的状态下，会执行 fallback 逻辑。 fall back 的逻辑最好是返回一些静态的字符串，不需要处理复杂的逻辑，也不需要远程调度其 他服务，这样方便执行快速失败，释放线程资源。 如果一定要在 fallback 逻辑中远程调度其他 服务，最好在远程调度其他服务时，也加上熔断器。

```java
package com.eureka.eurekaclientribbon.service;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class RibbonService {
    @Autowired
    RestTemplate restTemplate;

    @HystrixCommand(fallbackMethod = "hiError")
    public String hi(String name) {
        return restTemplate.getForObject("http://eureka-client/hello?name=" + name, String.class);
    }

    public String hiError(String name) {
        return "hi"+name+"error";
    }
}

```
