title: orika
author: tr
tags:
  - orika
categories:
  - SpringDataJPA
date: 2019-03-28 10:15:00
---
### orika 转换自定义实体类

orkia 类似 javaEE里的 beanutils 不过功能更加强大，用于po，vo，dto对象的转换。 下面的例子是使用jpa，@query自定义返回的实体，将这个实体转为domain里的vo对象。

1.首先注册，由于获得的query结果是List<Object[]>

```java
@Query(value = "SELECT prices_is_sale,CONCAT(ROUND(COUNT(prices_is_sale)/(SELECT COUNT(prices_is_sale) FROM DataSet)*100,2),'%')" +
        "   FROM DataSet" +
        "   GROUP BY prices_is_sale",nativeQuery = true)
List<Object[]> getSoldPercentage();
```

​	每条Object[] 0对应第一个字段 1对应第二个字段 以此类推，所以要注册时指定

```java
mapperFactory.classMap(Object[].class, SoldPercentageVO.class)
        .field("1", "percentage").field("0", "sale")
        .byDefault().register();
```



1. 转换代码

   ```java
       public PageVO<SoldPercentageVO> getSoldPercentage() {
           List<Object[]> soldPercentage = this.shoesDao.getSoldPercentage();
   //        SoldPercentageVO out = (SoldPercentageVO) ConveterUtil.mapOneTowOne(soldPercentage.get(0), SoldPercentageVO.class);
           AssertUtil.AssertNotNull(soldPercentage, ShoeErrorEnum.EMPTY_SHOE);
           List<SoldPercentageVO> list = soldPercentage.stream()
                   .map(data -> (SoldPercentageVO) ConveterUtil.mapOneTowOne(data, SoldPercentageVO.class))
                   .collect(Collectors.toList());
           PageVO<SoldPercentageVO> voList = new PageVO<>();
           voList.setTotal(list.size());
           voList.setData(list);
           return voList;
       }
   ```

