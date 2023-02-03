title: orika使用
author: tr
date: 2023-01-31 07:03:02
tags:
---
# orika

 一、Orika背景介绍
 　　

　　Orika是java Bean映射框架，可以实现从一个对象递归拷贝数据至另一个对象。在开发多层应用程序中非常有用。在这些层之间交换数据时，通常为了适应不同API需要转换一个实例至另一个实例。

　　有很多方法可以实现：硬代码拷贝或Dozer实现bean映射等。总之，需要简化不同层对象之间映射过程。

　   Orika使用字节码生成器创建开销最小的快速映射，比其他基于反射方式实现（如，Dozer）更快。之前使用Bean Copy 性能非常慢，发现在这个领域业界还是有很多新秀的。 Orika 应该就算一个比较好的吧。


 二、优势


1. 性能
　　大概是Dozer的8-10 倍， 这个上面的已经做了描述

2. 内存消耗
　　大概是Dozer内存消耗的一半多点。 为什么做到这点的还没想清楚， 估计是因为运行期不需要维护复杂的Mapping 关系。 不需要大量的Mapping 关系查找以及需要的对这些查找优化所消耗的空间。

3. 简单
　　Orika的代码短小精悍， 而且可读性非常强， Dozer如果要加减一个功能， 不才完全没有信心， Orika 我还是可以偶尔在Orika里面打几个酱油的。


三、基础使用

Maven项目依赖包：POM文件直接依赖进去即可。
```xml
<dependency>
    <groupId>ma.glasnost.orika</groupId>
    <artifactId>orika-core</artifactId>
    <version>1.4.6</version>
</dependency>
```
创建两个Bean对象，用于复制使用。
　　
```java
// UserVo对象
@Data //Set GET方法
@Accessors(chain = true)
public class UserVo {
private String id;
private String name;
}
// User对象

@Data
@Accessors(chain = true)
public class User {
private String id;
private String name;
}
使用场景（如果对象中属性太多，普通逻辑处理代码太多）
A对象复制到B对象（对象复制）
//业务场景，将A对象复制B对象中
//1.普通逻辑处理
User A = new User().setId("123").setName("1231");
UserVo B = new UserVo().setId(A.getId()).setName(A.getName());
System.out.println("普通方式将A对象复制B对象中："+B);

//使用orika复制工具将A复制到B对象中
MapperFactory mapperFactory = new DefaultMapperFactory.Builder().build();
UserVo map = mapperFactory.getMapperFacade().map(A, UserVo.class);
System.out.println("orika复制对象:"+map);
由上得出控制台输出如下：
普通方式将A对象复制B对象中：UserVo(id=123, name=1231)

orika复制对象:UserVo(id=123, name=1231)
```

2.A集合复制到B集合（集合复制）

```java
//1.普通逻辑处
//A对象
List<User> A = Arrays.asList(new User().setId("123").setName("张三"));
//B对象
List<UserVo> B = new ArrayList<>();
//将A集合数据复制到B集合中
A.forEach(x->{
B.add( new UserVo().setId(x.getId()).setName(x.getName()));
});
System.out.println("将A集合中数据set到B集合中数据打印");

//使用orika复制工具将A集合复制到B集合中
MapperFactory mapperFactory = new DefaultMapperFactory.Builder().build();
List<UserVo> userVo = mapperFactory.getMapperFacade().mapAsList
mapperFactory.getMapperFacade().mapAsList(A, UserVo.class);
System.out.println("orika直接复制对象集合打印结果:"+userVo);
由上得出控制台输出如下：
`将A集合中数据set到B集合中数据打印：[UserVo(id=123, name=张三)]`

`orika直接复制对象集合打印结果:[UserVo(id=123, name=张三)]`
到此为止您已经算入门了，以上是Orika的基础使用，由此发现如果对象中属性如果20个，那么用普通的逻辑处理需要20个set过去，如果用Orika两行代码搞定。话不多说关键步骤总结一波（关键必须牢记）：
MapperFactory mapperFactory = new DefaultMapperFactory.Builder().build();
mapperFactory.getMapperFacade().map(操作对象)
mapperFactory.getMapperFacade().mapAsList(操作集合对象)
```

四、高级使用方法

 创建两个Bean对象，用于复制使用。
　　　UserVo对象
```java
@Data //Set GET方法
@Accessors(chain = true)
public class UserVo {
private String id;
private String userName;
private int ageOne;
}
```

　　User对象
  
```java
@Data
@Accessors(chain = true)
public class User {
private String id;
private String name;
private int age;
}
```
使用场景（对象A与对象B中属性不一致时，如果对象中属性太多，普通逻辑处理代码太多）
　　1.A对象复制到B对象，对象中属性不一样时

```java
//业务场景，将A对象复制B对象中,A对象中的字段与B对象中的字段不一致
//1.普通逻辑处理
User A = new User().setId("123").setName("张三").setAge(20);
UserVo B = new UserVo().setId(A.getId()).setUserName(A.getName()).setAgeOne(A.getAge());
System.out.println("普通方式将A对象处理B对象中："+B);.

//使用orika复制工具将A复制到B对象中
MapperFactory mapperFactory = new DefaultMapperFactory.Builder().build();
mapperFactory.classMap(User.class, UserVo.class)
.field("name", "userName")
.field("age", "ageOne")
.byDefault().register();
UserVo userVo = mapperFactory.getMapperFacade().map(A, UserVo.class);
System.out.println("orika复制对象:"+userVo);
由上得出控制台输出如下：
`普通方式将A对象处理B对象中：UserVo(id=123, userName=张三, ageOne=20)`

`orika复制对象:UserVo(id=123, userName=张三, ageOne=20)`
```

2.A集合对象复制到B集合对象，对象中属性不一样时

```java
//1.普通逻辑处里A对象中与B对象字段不一致处理
//A对象
List<User> A = Arrays.asList(new User().setId("123").setName("张三").setAge(20));
//B对象
List<UserVo> B = new ArrayList<>();
//将A集合数据复制到B集合中
A.forEach(x->{
B.add(new UserVo().setId(x.getId()).setUserName(x.getName()).setAgeOne(x.getAge()));
});
System.out.println("将A集合中数据set到B集合中数据打印"+B);

//使用orika复制工具将A集合复制到B集合中
MapperFactory mapperFactory = new DefaultMapperFactory.Builder().build();
mapperFactory.classMap(User.class, UserVo.class)
.field("name", "userName")
.field("age", "ageOne")
.byDefault().register();
List<UserVo> userVos = mapperFactory.getMapperFacade().mapAsList(A, UserVo.class);
System.out.println("orika复制对象:"+userVos);
由上得出控制台输出如下：
`将A集合中数据set到B集合中数据打印[UserVo(id=123, userName=张三, ageOne=20)]`

`orika复制对象:[UserVo(id=123, userName=张三, ageOne=20)]`
话不多说关键步骤总结一波（关键必须牢记）：
MapperFactory mapperFactory = new DefaultMapperFactory.Builder().build();
mapperFactory.classMap(User.class, UserVo.class)
.field("name", "userName")
.field("age", "ageOne")
.byDefault().register();
//集合复制--使用mapAsList
List<UserVo> userVos = mapperFactory.getMapperFacade().mapAsList(A, UserVo.class);
//对象复制--使用map
UserVo userVos = mapperFactory.getMapperFacade().map(A, UserVo.class);
```

五、总结工具类

```java
/**
* 映射工具类
*/
public enum MapperUtils {

INSTANCE;

/**
* 默认字段工厂
*/
private static final MapperFactory MAPPER_FACTORY = new DefaultMapperFactory.Builder().build();

/**
* 默认字段实例
*/
private static final MapperFacade MAPPER_FACADE = MAPPER_FACTORY.getMapperFacade();

/**
* 默认字段实例集合
*/
private static Map<String, MapperFacade> CACHE_MAPPER_FACADE_MAP = new ConcurrentHashMap<>();

/**
* 映射实体（默认字段）
*
* @param toClass 映射类对象
* @param data 数据（对象）
* @return 映射类对象
*/
public <E, T> E map(Class<E> toClass, T data) {
return MAPPER_FACADE.map(data, toClass);
}

/**
* 映射实体（自定义配置）
*
* @param toClass 映射类对象
* @param data 数据（对象）
* @param configMap 自定义配置
* @return 映射类对象
*/
public <E, T> E map(Class<E> toClass, T data, Map<String, String> configMap) {
MapperFacade mapperFacade = this.getMapperFacade(toClass, data.getClass(), configMap);
return mapperFacade.map(data, toClass);
}

/**
* 映射集合（默认字段）
*
* @param toClass 映射类对象
* @param data 数据（集合）
* @return 映射类对象
*/
public <E, T> List<E> mapAsList(Class<E> toClass, Collection<T> data) {
return MAPPER_FACADE.mapAsList(data, toClass);
}


/**
* 映射集合（自定义配置）
*
* @param toClass 映射类
* @param data 数据（集合）
* @param configMap 自定义配置
* @return 映射类对象
*/
/* public <E, T> List<E> mapAsList(Class<E> toClass, Collection<T> data, Map<String, String> configMap) {
T t = data.stream().findFirst().orElseThrow(() -> new ResourceNotExistException("映射集合，数据集合为空"));
MapperFacade mapperFacade = this.getMapperFacade(toClass, t.getClass(), configMap);
return mapperFacade.mapAsList(data, toClass);
}*/

/**
* 获取自定义映射
*
* @param toClass 映射类
* @param dataClass 数据映射类
* @param configMap 自定义配置
* @return 映射类对象
*/
private <E, T> MapperFacade getMapperFacade(Class<E> toClass, Class<T> dataClass, Map<String, String> configMap) {
String mapKey = dataClass.getCanonicalName() + "_" + toClass.getCanonicalName();
MapperFacade mapperFacade = CACHE_MAPPER_FACADE_MAP.get(mapKey);
if (Objects.isNull(mapperFacade)) {
MapperFactory factory = new DefaultMapperFactory.Builder().build();
ClassMapBuilder classMapBuilder = factory.classMap(dataClass, toClass);
configMap.forEach(classMapBuilder::field);
classMapBuilder.byDefault().register();
mapperFacade = factory.getMapperFacade();
CACHE_MAPPER_FACADE_MAP.put(mapKey, mapperFacade);
}
return mapperFacade;
}

}
```