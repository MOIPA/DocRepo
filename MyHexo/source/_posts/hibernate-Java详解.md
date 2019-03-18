---
title: hibernate Java详解
date: 2019-03-08 15:56:59
tags: Hibernate
descriptions: javaee工程中是如何使用Hibernate的
categories: Hibernate
---



### 首先需要一个JavaBean在domain包下

<!--more-->

```java
package com.tr.domain;

/**
 * @author tr
 */
public class Customer {
//    create table `cst_customer`(
//            `cust_id` BIGINT(32) NOT NULL AUTO_INCREMENT comment '客户编号',
//            `cust_name` varchar(32) NOT NULL comment '客户名称',
//            `cust_source` varchar(32) DEFAULT NULL comment '客户信息来源',
//            `cust_industry` varchar(32) DEFAULT NULL comment '客户行业',
//            `cust_level` varchar(32) DEFAULT NULL comment '客户级别',
//            `cust_linkman` varchar(64) DEFAULT NULL comment '联系人',
//            `cust_phone` varchar(64) DEFAULT NULL comment '固定电话',
//            `cust_mobile` varchar(16) DEFAULT NULL comment '移动电话',
//    primary key (`cust_id`)
//)ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET =utf8;
    private long cust_id;
    private String cust_name;
    private String cust_source;
    private String cust_industry;
    private String cust_level;
    private String cust_linkman;
    private String cust_phone;
    private String cust_mobile;

    public long getCust_id() {
        return cust_id;
    }

    public void setCust_id(long cust_id) {
        this.cust_id = cust_id;
    }

    public String getCust_name() {
        return cust_name;
    }

    public void setCust_name(String cust_name) {
        this.cust_name = cust_name;
    }

    public String getCust_source() {
        return cust_source;
    }

    public void setCust_source(String cust_source) {
        this.cust_source = cust_source;
    }

    public String getCust_industry() {
        return cust_industry;
    }

    public void setCust_industry(String cust_industry) {
        this.cust_industry = cust_industry;
    }

    public String getCust_level() {
        return cust_level;
    }

    public void setCust_level(String cust_level) {
        this.cust_level = cust_level;
    }

    public String getCust_linkman() {
        return cust_linkman;
    }

    public void setCust_linkman(String cust_linkman) {
        this.cust_linkman = cust_linkman;
    }

    public String getCust_phone() {
        return cust_phone;
    }

    public void setCust_phone(String cust_phone) {
        this.cust_phone = cust_phone;
    }

    public String getCust_mobile() {
        return cust_mobile;
    }

    public void setCust_mobile(String cust_mobile) {
        this.cust_mobile = cust_mobile;
    }
}

```

### 添加一个utils包，将共有的方法抽象出来

​	这是由于SessionFactory最好在一个web中只存在一个的

```java
package com.tr.utils;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtils {

    private static SessionFactory sessionFactory;

    static{
        Configuration configure = new Configuration().configure(); //加载src下的主要配置文件
        sessionFactory = configure.buildSessionFactory(); //根据配置信息，创建sessionFactory
    }


    //获得session 1.全新  2.与线程绑定的
    public static Session openSession() {
        Session session = sessionFactory.openSession(); //获得新的session
        return session;
    }
    public static Session getCUrrentSession() {
        Session session = sessionFactory.getCurrentSession(); //获得新的session
        return session;
    }

    public static void closeSessionFactory() {
        sessionFactory.close();
    }
}
```

### 对于DAO层详解

``` java
package com.tr.dao;

import com.tr.bean.Customer;
import com.tr.utils.HibernateUtils;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.cfg.Configuration;

/**
 * sessionFactory: 1. 是线程安全的 消耗很多资源
 * 2. 要保证每个web项目生成一个Factory
 * 3. 工厂模式，用于读取文件配置，生成相应的session
 * session: 1. hibernate的核心对象 负责与数据库和hibernate的链接
 * 2. 可以对数据库做crud
 */
public class CustomerDao {
    //1.增
    public void insertCustomer(Customer customer) {
        //保存客户
        Configuration configure = new Configuration().configure(); //加载src下的主要配置文件
        SessionFactory sessionFactory = configure.buildSessionFactory(); //根据配置信息，创建sessionFactory
        Session session = sessionFactory.openSession(); //获得新的session
//    Session session = sessionFactory.getCurrentSession(); //获得与线程绑定的session

        //session获得事务操作的Transaction对象
//    Transaction transaction = session.getTransaction(); //只获得
        Transaction transaction = session.beginTransaction(); //不仅获得还开始事务操作
        //-----------
//        Customer customer = new Customer();
//        customer.setCust_name("测试公司");

        session.save(customer);
        //------------
        transaction.commit(); //提交事务
//    transaction.rollback();//回滚事务
        session.close();
        sessionFactory.close();
    }

    //2.查
    public void getCustomer(int id) {
        //保存客户
        Configuration configure = new Configuration().configure(); //加载src下的主要配置文件
        SessionFactory sessionFactory = configure.buildSessionFactory(); //根据配置信息，创建sessionFactory
        Session session = sessionFactory.openSession(); //获得新的session
//    Session session = sessionFactory.getCurrentSession(); //获得与线程绑定的session

        //session获得事务操作的Transaction对象
//    Transaction transaction = session.getTransaction(); //只获得
        Transaction transaction = session.beginTransaction(); //不仅获得还开始事务操作
        //-----------
//        Customer customer = new Customer();
//        customer.setCust_name("测试公司");

        Customer customer = session.get(Customer.class, (long) id);
        System.out.println(customer.getCust_name());
        //------------
        transaction.commit(); //提交事务
//    transaction.rollback();//回滚事务
        session.close();
        sessionFactory.close();
    }
//    3.改
public void updateCustomer(Customer customer,int id) {
        //保存客户
//        Configuration configure = new Configuration().configure(); //加载src下的主要配置文件
//        SessionFactory sessionFactory = configure.buildSessionFactory(); //根据配置信息，创建sessionFactory
//        Session session = sessionFactory.openSession(); //获得新的session
//    Session session = sessionFactory.getCurrentSession(); //获得与线程绑定的session

        //session获得事务操作的Transaction对象
//    Transaction transaction = session.getTransaction(); //只获得
    Session session = HibernateUtils.openSession();
    Transaction transaction = session.beginTransaction(); //不仅获得还开始事务操作

        session.update(customer);
//    System.out.println(customer.getCust_name());
        //------------
        transaction.commit(); //提交事务
//    transaction.rollback();//回滚事务
        session.close();
//        sessionFactory.close();
    }

    public void deleteCustomer(int id) {
        //保存客户
        Configuration configure = new Configuration().configure(); //加载src下的主要配置文件
        SessionFactory sessionFactory = configure.buildSessionFactory(); //根据配置信息，创建sessionFactory
        Session session = sessionFactory.openSession(); //获得新的session

        Transaction transaction = session.beginTransaction(); //不仅获得还开始事务操作
        //-----------
        Customer customer = session.get(Customer.class, (long) id);
        session.delete(customer);
        //------------
        transaction.commit(); //提交事务
//    transaction.rollback();//回滚事务
        session.close();
        sessionFactory.close();
    }
}
```

