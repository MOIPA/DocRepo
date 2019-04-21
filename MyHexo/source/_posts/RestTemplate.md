title: RestTemplate
author: tr
tags:
  - RestTemplate
categories:
  - SpringCloud
  - ''
date: 2019-04-01 08:57:00
---
### 定义：访问第三方RESTful接口的网络请求框架

<!--more-->


#### 使用方法：

```java
/**
** 将访问的页面变为String
**/
@RestController
public ClassRestController(){
	@GetMapping("/testRest")
    public String testRest(){
   		RestTemplate restTemplate = new RestTemplate();
      return restTemplate.getForObject("http..",String.class);
    }

}

//可以自动变为对象
User user = restTemplate.getForObject("www....",User.class);
```