title: springBoot+mybatis+多数据源动态配置
author: tr
tags:
  - Java
categories:
  - Java
date: 2019-08-01 11:24:00
---
### 说明

公司项目需要多数据源选择，尝试过以下方式配置静态数据源,但是项目已经使用了接口和sql一起的方式，使用以下方式需要sql写为xml分离，故使用`AbstractRoutingDataSource`尝试动态修改数据源

<!--more-->

`MyBatisConfig.java`
```java
package com.sichuan.sichuanproject.config;

import lombok.extern.slf4j.Slf4j;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;

import javax.annotation.Resource;
import javax.sql.DataSource;

@Slf4j
public class MyBatisConfig {
    /**
     * 第一个数据库 SqlSessionFactory && SqlSessionTemplate 创建
     */
    @Configuration
    @MapperScan(basePackages = {"com.sichuan.sichuanproject.mapper.primary"},
            sqlSessionFactoryRef = "sqlSessionFactoryOne",
            sqlSessionTemplateRef = "sqlSessionTemplateOne")
    public static  class DBOne{
        @Resource
        DataSource dbOneDataSource;
        @Bean
        public SqlSessionFactory sqlSessionFactoryOne() throws Exception {
            log.info("sqlSessionFactoryOne 创建成功。");
            SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
            factoryBean.setDataSource(dbOneDataSource);
//            factoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/one/*.xml"));
            return factoryBean.getObject();
        }
        @Bean
        public SqlSessionTemplate sqlSessionTemplateOne() throws Exception {
            SqlSessionTemplate template = new SqlSessionTemplate(sqlSessionFactoryOne()); // 使用上面配置的Factory
            return template;
        }
    }
    /**
     * 第二个数据库 SqlSessionFactory && SqlSessionTemplate 创建
     */
    @Configuration
    @MapperScan(basePackages = {"com.sichuan.sichuanproject.mapper.second"},
            sqlSessionFactoryRef = "sqlSessionFactoryTwo",
            sqlSessionTemplateRef ="sqlSessionTemplateTwo" )
    public static class DBTwo{
        @Resource
        DataSource dbTwoDataSource;
        @Bean
        public SqlSessionFactory sqlSessionFactoryTwo() throws Exception {
            log.info("sqlSessionFactoryTwo 创建成功。");
            SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
            factoryBean.setDataSource(dbTwoDataSource);
            factoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/two/*.xml"));
            return factoryBean.getObject();
        }
        @Bean
        public SqlSessionTemplate sqlSessionTemplateTwo() throws Exception {
            SqlSessionTemplate template = new SqlSessionTemplate(sqlSessionFactoryTwo()); // 使用上面配置的Factory
            return template;
        }
    }
}

```

思路：在service层使用mapper之前修改datasource（通过注解和aop）

### 1.配置数据源yum

spring工程resource目录下`application.yum`
```xml
#application.yml

server:
  port: 8080

spring:
  datasource:
    jdbc-url: jdbc:mysql://59.225.206.13:3711/sc_fxyj?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=UTF-8
    username: root
    password: Scfxyjpasswd741*
    driver-class-name: com.mysql.cj.jdbc.Driver
  second-datasource:
    jdbc-url: jdbc:mysql://59.225.206.13:3501/isgs-home?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=UTF-8
    username: rewi
    password: Schlw+Rewis@201907
    driver-class-name: com.mysql.cj.jdbc.Driver

mybatis:
  configuration:
    map-underscore-to-camel-case: true

```

### 2.关闭数据源自动扫描

springboot启动文件`工程名Application.java` 添加

`@SpringBootApplication(exclude = {
        DataSourceAutoConfiguration.class
})`

如下：

```java
package com.sichuan.sichuanproject;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;


@SpringBootApplication(exclude = {
        DataSourceAutoConfiguration.class
})
@RestController
public class SichuanprojectApplication {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    public static void main(String[] args) {
        SpringApplication.run(SichuanprojectApplication.class, args);
    }

}

```

### 3.配置注解和aop拦截器

1.先配置一个enum解耦

`DsEnum.java`

```java
package com.sichuan.sichuanproject.config;

/**
 * @author: tr
 * @Date: 2019-08-01
 * @Description:数据库枚举类型，存放数据库标识
 */
public enum DsEnum {
    /**
     * 数据库枚举
     */
    FIRST_DS("firstDataSource","001","first数据库"),
    SECOND_DS("secondDataSource","002","second数据库"),
    AUTO_DS("auto","003","预留字段后期自动获取数据库"),
    NONE("error","999","BASE ERROR")
            ;

    private String ds;
    private String baseid;
    private String message;

    public static DsEnum createDSBybaseid(String cid) {
        for(DsEnum val : DsEnum.values()) {
            if(val.getBaseid().equalsIgnoreCase(cid)) {
                return val;
            }
        }
        return DsEnum.NONE;
    }

    DsEnum(String ds, String baseid) {
        this.ds = ds;
        this.baseid = baseid;
    }

    DsEnum(String ds, String baseid, String message) {
        this.ds = ds;
        this.baseid = baseid;
        this.message = message;
    }

    public String getDs() {
        return ds;
    }

    public String getBaseid() {
        return baseid;
    }

    public String getMessage() {
        return message;
    }


}

```

2.配置一个DynamicDataSource(重要)：java文件只要继承`	AbstractRoutingDataSource`即可。spring在选择时会走`AbstractRoutingDataSource`代码选择一个datasource。需要config提前设置一个内部map

`DynamicDataSource.java`

```java
package com.sichuan.sichuanproject.config;

import lombok.extern.slf4j.Slf4j;
import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;
import org.springframework.lang.Nullable;

/**
 * @author: tr
 * @Date: 2019-08-01
 * @Description:数据源路由器，动态数据源设置，负责提取线程中的当前数据库,这是关键，实现动态选择
 * 查看了AbstractRoutingDataSource的源码，发现本质其也是维护一个map，通过key（Object）来选择datasource
 * 所以通过aop注解事先修改key内容，spring选择数据源时过来用key抽取对象
 * 如果传入的对象为null，即key为空，spring会自动选择hikaridatasource作为默认数据源
 * 可在22行断点查看框架代码执行
 */
@Slf4j
public class DynamicDataSource extends AbstractRoutingDataSource {
    @Nullable
    @Override
    protected Object determineCurrentLookupKey(){
        log.info("数据源为:{}",DataSourceContextHolder.getDB());
        String db = DataSourceContextHolder.getDB();
        return db;
    }
}

```

3.设置一个用于存放key的线程安全类

`DataSourceContextHolder.java`

```java
package com.sichuan.sichuanproject.config;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

/**
 * @author: tr
 * @Date: 2019-8-01
 * @Description:设置线程安全类，存放数据库源标识(String)，用于动态切换数据库源
 */
@Slf4j
@Component
public class DataSourceContextHolder {

    private static final ThreadLocal<String> contextHolder = new ThreadLocal<>();
    public static final DsEnum DEFAULT_DS = DsEnum.FIRST_DS;
    public static void setDB(DsEnum db) {
        log.info("切换到{}数据源",db.getMessage());
        contextHolder.set(db.getDs());
    }

    public static String getDB() {
        return contextHolder.get();
    }

    public static void clearDB() {
        contextHolder.remove();
    }
}

```

4.配置aop拦截器，作用：所有使用DS注释的service前切换数据库

`DataAspect.java`

```java
package com.sichuan.sichuanproject.config;

import com.sichuan.sichuanproject.service.DS;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import java.lang.reflect.Method;

/**
 * @author: tr
 * @Date: 2019-08-01
 * @Description:拦截器
 */
@Aspect
@Order(-1)
@Component
@Slf4j
public class DataAspect {

    @Before("@annotation(com.sichuan.sichuanproject.service.DS)")
    public void beforeDSAnnotation(JoinPoint point){
        //获得当前访问的class
        Class<?> className = point.getTarget().getClass();
        //获得访问的方法名
        String methodName = point.getSignature().getName();
        //得到方法的参数的类型
        Class[] argClass = ((MethodSignature)point.getSignature()).getParameterTypes();
        DsEnum dsEnum = null;
        try {
            // 得到访问的方法对象
            Method method = className.getMethod(methodName, argClass);
            // 判断是否存在@DS注解
            if (method.isAnnotationPresent(DS.class)) {
                DS annotation = method.getAnnotation(DS.class);
                //如果为自动，则根据baseId自动选择数据库
                if(DsEnum.AUTO_DS == annotation.value()) {
                    //预留功能 获取baseid的值
                    //Object[] args = point.getArgs();
                    String baseid = "200";
                    dsEnum = DsEnum.createDSBybaseid(baseid);
                }
                // 取出注解中的数据源名
                dsEnum = annotation.value();
                //切换数据源
                DataSourceContextHolder.setDB(dsEnum);
            }
        } catch (Exception e) {
            e.printStackTrace();
            log.error("{} 类出现了异常，异常信息为：{}",getClass().getSimpleName(),e.getMessage());
        }
    }

    @After("@annotation(com.sichuan.sichuanproject.service.DS)")
    public void afterDSAnnotation() {
        //还原数据源
        DataSourceContextHolder.clearDB();
    }
}

```

5.配置注解

`DS.java`

```java
package com.sichuan.sichuanproject.service;

import com.sichuan.sichuanproject.config.DsEnum;

import java.lang.annotation.*;

/**
 * @author: tr
 * @Date: 2019-08-01
 * @Description:  用于service切换数据库的ds注解
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Target({ElementType.METHOD})
public @interface DS {
    DsEnum value() default DsEnum.FIRST_DS;
}

```

### 4.spring启动配置数据源

`DatasourceConfig.java`

```java
package com.sichuan.sichuanproject.config;

import org.mapstruct.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

import javax.sql.DataSource;
import java.util.HashMap;

/**
 * 数据源配置类
 *
 * @author tr
 * @create 2019-07-31 15:36
 **/
@Configuration
public class DataSourceConfig {
//    @Primary
    @Bean(name = "firstDataSource")
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource firstDataSrouce(){
        return DataSourceBuilder.create().build();
    }

    @Bean(name="sendDataSource")
    @ConfigurationProperties(prefix = "spring.second-datasource")
    public DataSource secondDatasource(){
        return DataSourceBuilder.create().build();
    }

    /**
     * 动态数据源，将两个datasource存放在map里，后期使用时通过数据源名提取
     * @return
     */
    @Primary
    @Bean(name = "dynamicDataSource")
    public DataSource dynamicDataSource(){
        DynamicDataSource dynamicDataSource = new DynamicDataSource();
        //配置默认数据源
        dynamicDataSource.setDefaultTargetDataSource(firstDataSrouce());
        //配置多数据源
        HashMap<Object, Object> dsMap = new HashMap<>();
        dsMap.put("firstDataSource", firstDataSrouce());
        dsMap.put("secondDataSource", secondDatasource());
        dynamicDataSource.setTargetDataSources(dsMap);
        return dynamicDataSource;
    }
}

```

### service层调用流程

```java
package com.sichuan.sichuanproject.service.impl;

import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;
import com.sichuan.sichuanproject.config.DsEnum;
import com.sichuan.sichuanproject.domain.WarningSignal;
import com.sichuan.sichuanproject.dto.StaRewiDTO;
import com.sichuan.sichuanproject.mapper.primary.WarningSignalMapper;
import com.sichuan.sichuanproject.mapper.second.StaRewiMapper;
import com.sichuan.sichuanproject.service.DS;
import com.sichuan.sichuanproject.service.WarningSignalService;
import com.sichuan.sichuanproject.vo.WarningSignalVO;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.List;

/**
 * @author
 */

@Component
public class WarningSignalServiceImpl implements WarningSignalService {

    @Autowired
    private WarningSignalMapper warningSignalMapper;

    @Autowired
    private StaRewiMapper staRewiMapper;

    @DS(value = DsEnum.FIRST_DS)
    @Override
    public Integer addWarningSignal(WarningSignal warningSignal) {
        Integer result = warningSignalMapper.addWarningSignal(warningSignal);
        return result;
    }

//    @DS(value = DsEnum.FIRST_DS)
    @Override
    public PageInfo<WarningSignalVO> getWarningSignal(Integer pageNum, Integer pageSize) {
        PageHelper.startPage(pageNum, pageSize);
        List<WarningSignalVO> warningSignalVOList = warningSignalMapper.getWarningSignal();
        PageInfo pageInfo = new PageInfo(warningSignalVOList);

        return pageInfo;
    }

    @DS(value = DsEnum.SECOND_DS)
    @Override
    public List<StaRewiDTO> getStaRewi() {
        return staRewiMapper.getWarningSignal();
    }
}

```

### 总结说明：

1. Spring启动时执行@Configuration的java：由于停用了自动扫描且配置了动态数据库链接`DataSourceConfig.java`所以spring连接数据库会优先使用@Primary注解的dynamicDataSource，在代码中创建不同数据库的两个datasource，且将这两个对象放入spring框架map数据结构里，map的key是DsEnum里的key。

2. service层调用时：由于启动了注解，（么得注解就使用默认数据源）扫描到注解后执行DataAspect，将DataContextHolder里面存放的key改为现在需要数据源的key。执行完毕，执行DynamicDataSource，由于继承了框架AbstractRoutingDataSource，spring自动执行内部的resolveSpecifiedLookupKey方法，这个方法通过DataContextHolder的key取出内部map的数据源。