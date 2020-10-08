title: Nginx
author: tr
tags:
  - Nginx
categories:
  - Nginx
date: 2020-04-30 11:23:00
---
### nginx 

后台不修改的情况下前端跨域问题

<!--more-->

#### nginx 反向代理

##### 原理


![upload successful](/images/pasted-18.png)

浏览器访问nginx入口，nigx代理到实际前端页面，前端想访问跨域资源时访问nginx代理，这样对用户浏览器来说一直都是nginx端口，所以不会产生跨域。

##### 配置

前端web iframe访问跨域

`<iframe id='svg-show' name='svg-show' src="http://localhost:8981/api/tumo/svgOperator.jsp?fileId=123123" class="layui-col-md12" style="height: 600px"></iframe>`

使用intellij 内置jetty启动

![upload successful](/images/pasted-20.png)


后端接口：`http://localhost:8080/pssc_service_graph/tumo/svgOperator.jsp?fileId=F92F43DD-36C0-4AB4-A5E9-A5D4821D784C`

nginx配置：

```
	server{
        listen 8981;
        server_name  localhost;
 
        location /{
            proxy_pass http://localhost:63342;
        }
 
        location /api{
            proxy_pass http://localhost:8080/pssc_service_graph;
        }
    }
```