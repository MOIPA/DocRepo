title: maven 从远程库更新问题
tags:
  - >-
    maven 默认的中央仓库的速度不是很理想, 所以换成了阿里云的镜像, 但使用后发现, 无法正常更新索引了, Maven 的索引功能可以让 IDEA
    自动提示一些信息
categories:
  - maven
date: 2019-03-14 15:40:00
---



对于网上的大部分mirror配置都会导致Idea的update失败，使用下面的配置

<!--more-->

```xml
<mirrors>
    <mirror>
        <id>alimaven</id>
        <name>aliyun maven</name>
        <url>http://maven.aliyun.com/nexus/content/repositories/central</url>
        <mirrorOf>central</mirrorOf>        
    </mirror>
</mirrors>
```

