---
title: Hibernate配置
date: 2019-03-07 12:03:24
tags: Hibernate
---

### 使用maven+intellij配置基础环境

----

1. 修改intellij的自带maven源，改为国内镜像

   地址为：IntelliJ IDEA 2018.3.4\plugins\maven\lib\maven3\conf

2. 添加mirror节点 

<!--more-->

```xml
<mirror>
	<id>alimaven</id>
	<mirrorOf>central</mirrorOf>
	<name>aliyun maven</name>
	<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
</mirror>
```

3. 创建maven工程选择创建模板web, 添加pom节点

```xml
<dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-core</artifactId>
      <version>5.2.6.Final</version>
    </dependency>
```

4. 在main/resources下创建hibernate.cfg.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <property name="current_session_context_class">thread</property>

        <property name="hibernate.connection.driver_class">com.mysql.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://39.108.159.175/MyTest?characterEncoding=UTF-8</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">0800</property>

        <!-- 可以将向数据库发送的SQL语句显示出来 -->
        <property name="hibernate.show_sql">true</property>
        <!-- 格式化SQL语句 -->
        <property name="hibernate.format_sql">true</property>

        <!-- hibernate的方言 -->
        <property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>

        <property name="hibernate.hbm2ddl.auto">update</property>

        <!-- 配置hibernate的映射文件所在的位置 -->
        <mapping resource="Customer.hbm.xml" />

    </session-factory>
</hibernate-configuration>
```

5. 匹配自己的表对象  main/resources/Customer.hbm.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping>
    <class name="com.tr.dao.Customer" table="cst_customer">
        <id name="cust_id" column="cust_id">
            <!--generator 主键生成策略-->
            <generator class="native"></generator>
        </id>
        <property name="cust_name" column="cust_name"></property>
        <property name="cust_source" column="cust_source"></property>
        <property name="cust_industry" column="cust_industry"></property>
        <property name="cust_level" column="cust_level"></property>
        <property name="cust_linkman" column="cust_linkman"></property>
        <property name="cust_phone" column="cust_phone"></property>
        <property name="cust_mobile" column="cust_mobile"></property>
    </class>
</hibernate-mapping>

```



xml

