title: SpringDataJPA
tags:
  - SpringDataJPA
categories:
  - SpringDataJPA
author: tr
date: 2019-03-15 09:38:00
---
### 简介 

1. springdata下的一个项目 提供了一套JPA标准的数据库操作方案，底层还是HibernateJPA

2. 我们只需要定义接口然后继承SpringDataJpa中的接口即可，不用写实现类，意思dao不需要实现

<!--more-->

![](/images/SpringJPA继承图0.png)

### startUp

#### 在SpringHibernateJPA基础上修改

1. pom.xml文件添加

```xml
<dependencies>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.8.0-beta4</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.8.0-beta4</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-jpa</artifactId>
        <version>2.1.5.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-commons</artifactId>
        <version>2.1.5.RELEASE</version>
    </dependency>
    <dependency>
```




2. 修改配置文件applicationContext.xml

   1. 开启命名空间 

      ```xml
      xmlns:jpa="http://www.springframework.org/schema/data/jpa"        http://www.springframework.org/schema/data/jpa      http://www.springframework.org/schema/data/jpa/spring-jpa.xsd
      ```


   2. 添加SpringDataJPA配置

   ```xml
   <!--spring data jpa 配置-->
       <!--扫描springdatajpa接口所在的dao包-->
       <jpa:repositories base-package="com.tr.dao"/>
   ```

#### 完整的配置文件

1. /resources/applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:jpa="http://www.springframework.org/schema/data/jpa"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/data/jpa
        http://www.springframework.org/schema/data/jpa/spring-jpa.xsd
">


    <!--配置读取properites文件的工具类-->
    <context:property-placeholder location="jdbc.properties"></context:property-placeholder>
    <!--配置ic3p0数据库连接池-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="jdbcUrl" value="${jdbc.url}"></property>
        <property name="driverClass" value="${jdbc.driver.class}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
    <!--配置hibernate的sessionFactory 改为JPA-->
    <!--spring整合jpa 配置EntityanagerFactory-->
    <bean id="entityManager" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="jpaVendorAdapter">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
                <!--配置数据库类型-->
                <property name="database" value="MYSQL"/>
                <!--自动创建表-->
                <property name="generateDdl" value="true"/>
                <!--配置显示sql-->
                <property name="showSql" value="true"/>
            </bean>
        </property>
        <!--配置实体扫描包-->
        <property name="packagesToScan">
            <list>
                <value>com.tr</value>
            </list>
        </property>
    </bean>
    <!--配置hibernate JPA的事务管理器
        注意：换事务管理器了
    -->
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManager"/>
    </bean>
    <!--配置开启注解事务处理-->
    <tx:annotation-driven transaction-manager="transactionManager"/>
    <!--配置springIOC的注解模式-->
    <context:component-scan base-package="com.tr"/>

<!--spring data jpa 配置-->
    <!--扫描springdatajpa接口所在的dao包-->
    <jpa:repositories base-package="com.tr.dao"/>

</beans>
```

2. /resources/jdbc.properties

```xml
jdbc.url=jdbc:mysql://1.11.11.111:3306/MyTest
jdbc.driver.class=com.mysql.cj.jdbc.Driver
jdbc.username=root
jdbc.password=123
```

3. pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>SpringDataJPA</groupId>
    <artifactId>SpringDataJPA</artifactId>
    <version>1.0-SNAPSHOT</version>

<dependencies>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.8.0-beta4</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.8.0-beta4</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-jpa</artifactId>
        <version>2.1.5.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-commons</artifactId>
        <version>2.1.5.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-entitymanager</artifactId>
        <version>5.4.1.Final</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.8.13</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.0.11.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-expression</artifactId>
        <version>5.0.11.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>5.0.11.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>5.0.11.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.0.11.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-tx</artifactId>
        <version>5.0.11.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>5.0.11.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>5.0.11.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjrt</artifactId>
        <version>1.8.13</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-orm</artifactId>
        <version>5.0.11.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aspects</artifactId>
        <version>5.0.11.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>antlr</groupId>
        <artifactId>antlr</artifactId>
        <version>2.7.7</version>
    </dependency>
    <dependency>
        <groupId>dom4j</groupId>
        <artifactId>dom4j</artifactId>
        <version>1.6.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.geronimo.specs</groupId>
        <artifactId>geronimo-jta_1.1_spec</artifactId>
        <version>1.1.1</version>
    </dependency>
    <dependency>
        <groupId>org.hibernate.common</groupId>
        <artifactId>hibernate-commons-annotations</artifactId>
        <version>5.0.1.Final</version>
    </dependency>
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>5.2.17.Final</version>
    </dependency>
    <dependency>
        <groupId>org.hibernate.javax.persistence</groupId>
        <artifactId>hibernate-jpa-2.1-api</artifactId>
        <version>1.0.2.Final</version>
    </dependency>
    <dependency>
        <groupId>org.jboss</groupId>
        <artifactId>jandex</artifactId>
        <version>2.0.3.Final</version>
    </dependency>
    <dependency>
        <groupId>org.javassist</groupId>
        <artifactId>javassist</artifactId>
        <version>3.22.0-GA</version>
    </dependency>
    <dependency>
        <groupId>org.jboss.logging</groupId>
        <artifactId>jboss-logging</artifactId>
        <version>3.3.2.Final</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.13</version>
    </dependency>
    <dependency>
        <groupId>com.mchange</groupId>
        <artifactId>c3p0</artifactId>
        <version>0.9.5-pre8</version>
    </dependency>
    <dependency>
        <groupId>com.mchange</groupId>
        <artifactId>mchange-commons-java</artifactId>
        <version>0.2.7</version>
    </dependency>
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-c3p0</artifactId>
        <version>5.4.1.Final</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
</dependencies>

</project>
```



### 使用idea的实体映射自动生成实体类

 	1. 在项目的faclet中添加JPA组件和Hibernate，且JPA组件在下拉栏中选择Hibernate作为提供者，右键JPA选择generate Persistence Mapping ==> ByDataBaseSchema
 	2. 在ImportDatabaseSchema中选中左下角：AddToPersistenceUnit



### 编写dao层接口

1. 使用SpringDataJPA不需要对dao层接口做实现

2. 继承extends JpaRepository<Users,Integer> 

3. 第一个参数：操作的实体类（实体类隐射了数据库表）

   第二个参数：这个实体类的主键类型

### 编写测试类

**千万不要忘记@ContextConfiguration!!!!**

```java
package com.tr.test;

import com.tr.dao.UsersDao;
import com.tr.domain.Users;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import javax.transaction.Transactional;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class testUsersDao {

    @Autowired
    private UsersDao usersDao;

    @Test
    @Transactional(rollbackOn = Exception.class)
    public void testInsertUsers() {
        Users users = new Users("sdj", "sdj", "sdj", "sdj", "sdj", "sdj", "sdj");
        this.usersDao.save(users);
    }
}

```

### 接口使用

#### Repository接口  

​	**注意点：继承的包org.springframework.data.repository.Repository;**

​	**且继承后要写泛型里面的类型**

​	**读取下划线字段findByCust_name会出错，使用@Column转驼峰法**

 1. 是SpringDataJPA提供的顶层接口  是一个标识接口

 2. 提供了两种查询方式的支持

     1. 基于方法命名规则查询

        规则：findBy(关键字)+属性名称（属性名称首字母大写）+	查询条件（首字母大写）

        ```java
         /**
             * 使用用户名作为查询条件
             */
            @Test
            public void testGetUserByName() {
                /**
                 * findBy : 规定写法
                 * Cust_name ：属性首字母大写
                 * 条件有三种标识： 1. 默认不写会使用相等判断
                 *                  2. Is
                 *                  3. equal
                 */
                List<Users> list = this.usersDaoWithRepository.findByNameIs("tzq");
                for (Users u : list) {
                    System.out.println(u);
                }
            }
        ```

        ```java
        /**
         * 多命令：查询名称为tzq且linkman为tr
         * 缺陷：简单查询可以，条件复杂不可
         */
        @Test
        public void testAnd() {
            List<Users> list = this.usersDaoWithRepository.findByNameLikeAndLinkmanLike("t%","tr");
            for (Users u :
                    list) {
                System.out.println(u);
            }
        }
        ```

        ```java
        package com.tr.dao;
        
        import com.tr.domain.Users;
        import org.springframework.data.repository.Repository;
        
        import java.util.List;
        
        public interface UsersDaoWithRepository extends Repository<Users,Integer> {
        
            List<Users> findByNameIs(String tzq);
        
            List<Users> findByNameLike(String s);
        
            List<Users> findByLinkmanIs(String tr);
        
            List<Users> findByNameLikeAndLinkmanLike(String s, String tr);
        }
        ```

     2. 基于@Query注解查询

        1. 通过JPQL语句查询（基于HQL改来的）

           ```java
           //使用@Query注解查询
           @Query(value = "from Users where name = ?1")
           List<Users> queryUserByNameUseJPQL(String name);
           ```

        2. 通过SQL语句查询

           ```java
           //Sql
           @Query(value = "select * from cst_customer where cust_name=?", nativeQuery = true)
           List<Users> queryUsersBySql(String name);
           ```

        3. 更新语句

           ```java
               @Query("update Users set name=?2 where id=?1")
               @Transactional(rollbackOn = Exception.class)
               @Rollback(false)
               @Modifying  //更新语句
               void updateNameById(int i, String jpa);
           ```

#### CureRepository接口

1. 继承了接口后可以直接crud了

   ```java
     /**
        * 添加单条数据
        */
       @Test           //不需要加@Transcational 因为执行的代理对象内部已经处理了
       @Rollback(false)
       public void test1() {
           Users users = new Users();
           users.setName("CrudJpa");
           this.usersDao.save(users);
       }
       /**
        * 批量添加
        */
       @Test
       @Rollback(true)
       public void test2() {
           Users users = new Users();
           users.setName("CrudJpa1");
           Users users2 = new Users();
           users2.setName("CrudJpa2");
           List<Users> list = new LinkedList<>();
           list.add(users);
           list.add(users2);
           this.usersDao.saveAll(list);
       }
   
       /**
        * 查询单条数据
        */
       @Test
       public void test3() {
           Optional<Users> users = this.usersDao.findById(1);
           System.out.println(users.get());
       }
       /**
        * 查询全部数据
        */
       @Test
       public void test4() {
           Iterable<Users> users = this.usersDao.findAll();
           for (Users u :
                   users) {
               System.out.println(u);
           }
       }
   
       /**
        * 更新数据
        */
       @Test
       @Transactional
       @Rollback(false)
       public void test5() {
           Optional<Users> byId = this.usersDao.findById(1); //持久化状态
           Users users = byId.get();
           users.setName("changed");   //持久化可以直接修改 前提是手动改@Transcation
   //        this.usersDao.save(users);
       }
   ```

    

#### PagingAndSortingRepository接口