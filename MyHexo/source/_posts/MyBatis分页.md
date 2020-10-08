title: MyBatis分页
author: tr
tags:
  - MyBatis
  - 分页
categories:
  - MyBatis
date: 2020-04-21 17:28:00
---
### MyBatis分页

分页问题：分页的几种实现，前后端分页分别是干嘛的

<!--more-->

#### 分页

顾名思义，分页是为了将数据库查到的大量数据分成多页送出，原来我以为回送所有数据，分页操作由前端来做，在此之前都是自己在前端分页。然而工作中遇到的分页都是后端做的，前端的分页只是简单的在调用接口的时候将`currentPage` `pageSize`即当前页，一页大小作为参数发送给后端，后端只给部分分页数据。

前端：用户点击的前端页面，点击了第二页，此时每页大小20，那么前端发送`.../xxx?currentPage=2&pageSize=20`,后端接受，计算。

后端：从数据库中查到100条数据，生成一个`list`，将其中的`(currentPage-1)*pageSize 到 currentPage*pageSize`数据送出 ，即`(2-1)*20=20 到 2*20=40`，20-40号送出

可以直接使用`list.sublist(start,end)`完成

#### mybatis分页

1. 每次取出全部数据，分出其中需要的数据

```java
		// 获取所有数据
        List<GraphVO> list = mapper.showGraphList(name);
  		//设置全部数据条数
        page.setTotal(list.size());
        long currentPage = page.getCurrent();
        long pageSize = page.getSize();
		// 给出当前数据
        list =  list.subList((int) ((currentPage-1)*pageSize), (int) (currentPage*pageSize)); //直接在list中截取

        page.setRecords(list);
        return new PageUtils(page);
```

2. 在sql中分页,mapper传入参数

limit是mysql的 ，oracle得用rownum实现

```xml
<select id="showList" parameterType="map" resultMap="GraphVO">
        select * from Graph limit #{currIndex} , #{pageSize}
</select>
```

3. 其他分页暂时没用到，用到再写