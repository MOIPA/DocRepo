---
title: entityManagerFactory error log
date: 2019-03-19 09:24:41
tags: error
categories: Log
descriptions: Cannot resolve reference to bean 'entityManagerFactory' while setting constructor argument 失败 - 代码日志
---



### Spring数据jpa-没有定义名为“entityManagerFactory”的bean;注入自动连接的依赖失败

<!--more-->

Spring数据JPA默认情况下查找名为entityManagerFactory的EntityManagerFactory。



```xml
<bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
```

想这样配置为entityManagerFactory会自动找到



否则手动匹配

```xml
<!--spring data jpa 配置-->
    <!--扫描springdatajpa接口所在的dao包-->
    <jpa:repositories base-package="com.tr" entity-manager-factory-ref="entityManagerFactory"/>
```

