title: 从0到1完成前后端分离的web项目
author: tr
tags:
  - Spring
categories:
  - Sping
date: 2018-04-22 10:57:00
---
# 从0到1完成前后端分离的web项目

最简最少技术栈创建一个完整项目
<!--more-->

技术栈：

> 后端： springboot + mybatis（注解形式）
> 前端： jquery+layUI

## 项目启动

### 创建spring项目

打开idea 点击新建项目，选择spring initializr

![upload successful](/images/pasted-49.png)

一步一步往下填写相关信息，填错了也无所谓，可以复制我的pom文件

#### pom文件配置

配置mybatis和必要启动组件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.0.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.dql</groupId>
    <artifactId>retailmanager</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>retailmanager</name>
    <description>team work</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.4</version>
        </dependency>
        <!--mysql数据库驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
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

#### 配置基础文件结构


![upload successful](/images/pasted-50.png)

### 数据库配置

选择一个数据库，这里选择mysql，无所谓。

#### 创建utf8的数据库

`CREATE DATABASE retail_manager DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;`

#### 创建用户表

可以用命令行，或者navicate或者idea自带的管理工具

![upload successful](/images/pasted-51.png)
![upload successful](/images/pasted-53.png)

### 创建实体类和mapper

实体类bean应该遵守阿里的相关约定，dto vo po 以及互相转换的orika

这里推荐选择lombok，这个插件的好处是不用写getter和setter，可以让代码更加精简。

mapper需要配置mapper的resource文件，这个xml文件格式可以百度下，创建xml文件 填入模板

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.yourproject">


</mapper>
```

注意：生成的UserDao里面需要加上`@Mapper`注解否则无法被扫描到

#### 创建service

注意sevice层的注解

```java
package com.dql.retailmanager.service;

import com.dql.retailmanager.dao.mapper.UserDao;
import com.dql.retailmanager.entity.User;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

@Service
public class UserService {
    @Resource
    UserDao userDao;

    /**
     * select user by primary key
     * @param id
     * @return
     */
    public User findUserById(int id) {
        return userDao.selectByPrimaryKey(id);
    }

    /**
     * add new user to db
     * @param user
     * @return
     */
    public int createUser(User user) {
        return userDao.insert(user);
    }

    /**
     * delete a user by primary key
     * @param id
     * @return
     */
    public int deleteUserById(int id) {
        return userDao.deleteByPrimaryKey(id);
    }

}

```

#### 创建controller

```java
package com.dql.retailmanager.controller;

import com.dql.retailmanager.entity.User;
import com.dql.retailmanager.service.UserService;
import org.springframework.web.bind.annotation.*;

import javax.annotation.Resource;

@RestController
@RequestMapping("/user")
public class UserController {
    @Resource
    UserService userService;

    @GetMapping("/getUserById")
    public User getUserById(@RequestParam int id) {
        return userService.findUserById(id);
    }

    @GetMapping("/deleteUserById")
    public int deleteUserById(@RequestParam int id) {
        return userService.deleteUserById(id);
    }

    @PostMapping("/addUser")
    public int addUser(@RequestBody User user) {
        return userService.createUser(user);
    }

    @PostMapping("/updateUser")
    public int updateUser(@RequestBody User user) {
        return userService.updateUser(user);
    }
}

```

#### 配置文件配置

```properties
server:
    port: 8081
spring:
#数据库连接配置
    datasource:
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://47.107.105.158:3306/test?characterEncoding=utf-8&useSSL=false
        username: root
        password: 123456

#mybatis的相关配置
mybatis:
    #mapper配置文件
    mapper-locations: classpath:mapper/*.xml
    #开启驼峰命名
    configuration:
        map-underscore-to-camel-case: true
```

#### 完成启动


![upload successful](/images/pasted-55.png)

#### 测试后台


![upload successful](/images/pasted-56.png)


#### 单点登陆校验

数据库配置session，本地使用localstorage存放sessionid，一般这个单点登录和分布式集群是用redis来管理的，但是这里为了不引入更多技术就用数据库了，问题不大。
```js
login.js:

import urlInfo from './baseURL.js'
    $(function () {
      layui.use('form', function () {
        var form = layui.form;
        layer.msg('login system....', function () {
          form.on('submit(login)', function (data) {
            // 生成token
            var rod = Math.round(1, 100);
            var md5rod = $.md5(rod + 'author');
            console.log(md5rod);
            $.get(urlInfo.baseUrl + urlInfo.login, {
              name: data.field.username,
              pwd: data.field.password,
              sessionToken: md5rod
            }, function (data, status) {
              if (data != null && data != "") {
                localStorage.setItem("token", md5rod);
                localStorage.setItem("userId", data.id)
                localStorage.setItem("user", data);
                location.href = 'index.html';
              } else {
                alert("error user or password");
                $("#login-name").val("");
                $("#login-password").val("");
              }
            });
            return false;
          });
        });
        //监听提交

      });
    })
    
    
check js:

// 判断是否登陆
        import urlInfo from './baseURL.js'
        layui.use(['element', 'layer'], function () {
            var element = layui.element;
            $.get(urlInfo.baseUrl + urlInfo.sessionCheck, {
                userId: localStorage.getItem("userId"),
                token: localStorage.getItem("token")
            }, function (data, status) {
                if (data == -1) {
                    // 用户未登录过 跳转到登陆页面
                    layer.msg('please login');
                    setTimeout(() => {
                        window.location.href = "./login.html";
                        window.event.returnValue = false;
                    }, 1500);
                } else {
                    console.log("logined user");
                }
            });
        });
```

## 前后端分离

### 配置nginx反向代理 解决跨域

```

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
	
	server{
		listen 			9999;
		server_name  localhost;
		location / {  
			proxy_pass http://localhost:8081;
		}
		location /user/ {
			proxy_pass http://localhost:8080;
		}
		location /session/ {
			proxy_pass http://localhost:8080;
		}
	}

}

```


![upload successful](/images/pasted-58.png)

执行reload命令生效配置

## 配置分页器

### pom

```
<dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper-spring-boot-starter</artifactId>
            <version>1.2.5</version>
        </dependency>
```

### yml

```
# pagehelper   
pagehelper:
    helperDialect: mysql
    reasonable: true
    supportMethodsArguments: true
    params: count=countSql
```