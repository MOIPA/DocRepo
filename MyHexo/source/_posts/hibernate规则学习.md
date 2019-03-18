---
title: hibernate规则学习
date: 2019-03-11 09:14:34
tags: Hibernate
categories: Hibernate
---

### hibernate 规则

<!--more-->

#### 1. 实体类创建注意事项

```
	a. 成员变量私有，设置setter，getter公共方法
	b. 属性都是指的get，set方法 
	c. 需要无参数构造方法
	d. 尽量使用包装类型 Integer ...
	e. 持久化类需要提供oid 与数据库的主键对应   如果表没有主键 不能用hibernate
	f. 不要用final修饰class 因为hibernate使用cglib代理生成代理对象，代理对象是继承关系
```

### 2. 主键类型

```
	a. 自然主键（少见）  表的业务列中符合必须有，且不重复特征，该列可以为主键
	b. 代理主键（常见）  表没有任何意义的列，用于区别
```

### 3. 主键生成策略generator

```
	a. 每条记录录入时主键生成的规则 
			identity:自增，由数据库自己维护主键值
            increment：自增，（不建议使用）由hibernate维护，每次插入前找表中id最大的，+1
            sequence：oracle中的主键生成策略
            hilo:高低位算法（不建议使用），自增，hibernate，确保不会重复
            native：（建议）hilo+sequence+identity 自动三选一
            uuid：（主键类型必须为String）用来产生永远不重复的随机字符串最为主键
            assigned：自然属性生成策略，就是没有策略，用户自己指定主键
```

### 4. hibernate 中对象的状态

```
save方法其实是为了生成oid值 所以需要执行insert语句
持久化状态会自动将任何变化同步到数据库，
	1. 对象分为三种状态： 	瞬时状态
								没有id ,没有与session关联就是 不在session缓存
							持久化状态
								有id，有session关联
							游离状态
								有id，没有与session关联
								
	2. 结论：将我们希望同步到数据库的数据，对应的对象转换为持久化状态
			在get后可以直接修改状态，不用调用update
			后期有个重要方法：saveOrUpdate 可以将对象变为持久化状态
```

### 5. hibernate 一级缓存

```
	例子：比如取5次某个对象，这5个对象引用地址一样，因为hibernate先查看缓存
	hibernate取出的时候会产生两个对象，一个放入缓存，一个放入快照，commit的时候对比快照
	改变就产生sql
```

### 6. hibernate事务

```
	1. 原子性 a	要么全成功 。。。
	2. 一致性 c	数据总量提交前和提交后不变
	3. 隔离性 i 事务并发
	4. 持久性 d 数据必须立即写入硬盘
	
	事务并发问题： 1脏读 读的是正在操作的数据
					2不可重复读：两次读不一致
					3幻读：自己删了有人添加，一查还在 感觉没删
	事务隔离级别：读为提交 可能：123
					读已提交：23
					可重复读（mysql默认级别)：3
					串行化：没问题  效率低

	指定hibernate隔离级别：			
		<!--指定hibernate隔离级别
			1|2|4|8
			0001 1 读为提交
			0010 2  读已提交
			0100 4 可重复读
			1000 8 串行化
		-->
		<property name="hibernate.connection.isolation">4</property>

	项目中如何管理事务：
		1. 业务开始之前打开事务，业务执行之后提交事务，执行中出现异常回滚
		2. dao和service层用的同一个session，service负责开启和提交事务，将session绑定到本线程，
			调用session.getCurrentSession方法即可获得当前线程绑定的session对象
			注意:必须在著配置文件中配置才能用这个方法
			```xml				
				<!--指定session与线程绑定-->
				<property name="hibernate.current_session_context_class">thread</property>
```
		**example**
		service 开启事务，关闭和回滚
		```java
		
			public void save(Customer customer) {
	
				//获得session   这时候customer没有id，不和session关联  瞬时状态
				Session session = HibernateUtils.getCUrrentSession();
				//执行事务
				Transaction transaction = session.beginTransaction();
	
				try {
					dao.save(customer);
				} catch (Exception e) {
					e.printStackTrace();
					transaction.rollback();
				}
	
				//提交事务
				transaction.commit();
				session.close(); //有id 无session 游离状态
	
			}
		```
		dao 执行事务
		```java
			/**
			 * save方法其实是为了生成oid值 所以需要执行insert语句
			 * 持久化状态会自动将任何变化同步到数据库，
			 */
			public class CustomerDaoImpl implements CustomerDao {
				@Override
				public void save(Customer customer) {
					//获得session   这时候customer没有id，不和session关联  瞬时状态
					Session session = HibernateUtils.getCUrrentSession();
					//保存
					session.save(customer); //持久化状态
				}
			}
```

​	

```
	```
```

### 7. hibernate中批量查询

```
	1. hql   可以多表查询 但是大部分都是单表查询
		 * 书写HQL
		 * String hql = "from com.tr.domain.Customer where cust_id = 1";  //如果类名不重复 可以不用写包名
		 * Query query = session.createQuery(hql);
		 * List<Custromer> list =  query.list(); //查询所有结果
		 * query.uniqueResult() ; //查询唯一结果(有where的情况下)
		 *
		  * 带参数？的
		 *
		 * String hql = "from Custromer where cust_id=?"
		 * //创建查询对象
		 * Query query = session.createQuery(hql);
		 *  //设置参数  第一个是参数的位置
		 *  query.setLong(0,12") //可以换成setParameter就不用写类型
		 *  Customer c  =query.uniqueResult();
		 *
		  * 命名占位符
		 *  String hql = "from Custromer where cust_id = :cust_id"
		 *  Query query = session.createQuery(hql);
		 *  query.setParameter("cust_id",12);
		 *
		 * 分页
		 *  String hql = "from Custromer"
		 *  Query query = session.createQuery(hql);
		 *   //设置分页信息
		 *   query.setFirstResult(0);  //从哪开始抓
		 *   query.setMaxResults(10);   //抓多少
		 *  query.list();
		 *
		 
	2. Criteria:hibernate自创的查询，都不用写hql或则sql  用于单表查询
			现在已经被抛弃了
	3. sql  复杂业务还是sql
```

​			

```
		video 1 和video 2看完了  但是3，4没看完，且2的最后两个没怎么看。留以后有余力看
					hibernate现在用的少，我选择先看 spring data jpa
```

​		
```