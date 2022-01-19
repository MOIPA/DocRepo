---
title: struts2学习
date: 2019-03-11 09:15:01
tags: struts2
categories: Struts2
---

# Struts2 learn

------

<!--more-->

### Configuration

#### basic configuration

1. 使用intellij 默认的maven项目管理依赖

2. 配置maven源镜像为国内阿里

3. pom.xml中添加以下依赖

   1. 在properties中添加struct和log4j变量

      ```xml
        <properties>
          <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
      
          <struts2.version>2.5.14.1</struts2.version>
          <log4j2.version>2.10.0</log4j2.version>
      
        </properties>
      ```

      ```
      2. 添加具体日志和struct依赖
      ```

      ```xml
          <dependency>
            <groupId>org.apache.struts</groupId>
            <artifactId>struts2-core</artifactId>
            <version>${struts2.version}</version>
          </dependency>
      
          <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>${log4j2.version}</version>
          </dependency>
          <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
            <version>${log4j2.version}</version>
          </dependency>
      
      ```

      1. 添加运行插件

   ```xml
     
             <plugin>
               <groupId>org.eclipse.jetty</groupId>
               <artifactId>jetty-maven-plugin</artifactId>
               <version>9.4.7.v20170914</version>
               <configuration>
                 <webApp>
                   <contextPath>/${build.finalName}</contextPath>
                 </webApp>
                 <stopKey>CTRL+C</stopKey>
                 <stopPort>8999</stopPort>
                 <scanIntervalSeconds>10</scanIntervalSeconds>
                 <scanTargets>
                   <scanTarget>src/main/webapp/WEB-INF/web.xml</scanTarget>
                 </scanTargets>
               </configuration>
             </plugin>
   ```

   ```
   4. 在intellij 中添加maven命令，命令格式：jetty:run
   ```

------

#### project structure

1. 添加java文件夹：src/main/java/<packageName>/controller 和dao层  

   这里的view是: src/main/webapp/index.jsp

2. 添加拦截器：用于将所有url访问拦截给struts框架 在src/main/resources/struts.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   
   <!DOCTYPE struts PUBLIC
           "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
           "http://struts.apache.org/dtds/struts-2.5.dtd">
   <struts>
       <constant name="struts.devMode" value="true" />
   
       <package name="basicstruts2" extends="struts-default">
           <action name="index">
               <result>/index.jsp</result>
           </action>
           <action name="hello" class="com.tr.controller.HelloWorldAction" method="execute">
               <result name="success">/HelloWorld.jsp</result>
               <result name="error">/error.jsp</result>
           </action>
       </package>
   </struts>
   ```

3. 添加log4j的配置：src/main/resources/log4j2.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <Configuration>
       <Appenders>
           <Console name="STDOUT" target="SYSTEM_OUT">
               <PatternLayout pattern="%d %-5p [%t] %C{2} (%F:%L) - %m%n"/>
           </Console>
       </Appenders>
       <Loggers>
           <Logger name="com.opensymphony.xwork2" level="debug"/>
           <Logger name="org.apache.struts2" level="debug"/>
           <Root level="warn">
               <AppenderRef ref="STDOUT"/>
           </Root>
       </Loggers>
   </Configuration>
   
   
   ```

------



#### example

1. 在webapp下添加index.jsp

   1. ```html
      <!DOCTYPE html>
      <%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
      <%@ taglib prefix="s" uri="/struts-tags"%>
      <html>
      <head>
          <meta charset="UTF-8">
          <title>Basic Struts 2 Application - Welcome</title>
      </head>
      <body>
      <h1>Welcome To Struts 2!</h1>
      <p><a href="<s:url action='hello'/> ">Link to hello</a> </p>
      
      <s:url action="hello" var="helloLink">
          <s:param name="userName">Bruce Phillips</s:param>
      </s:url>
      
      <p><a href="${helloLink}">Hello Bruce Phillips</a></p>
      
      
      </body>
      </html>
      
      ```

2. 在webapp下添加HelloWorld.jsp

   ```html
   <%--
     Created by IntelliJ IDEA.
     User: tr
     Date: 19-3-4
     Time: 下午4:18
     To change this template use File | Settings | File Templates.
   --%>
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <%@ taglib prefix="s" uri="/struts-tags"%>
   <html>
   <head>
       <title>hello world</title>
   </head>
   <body>
       <h2><s:property value="messageStoreSuccess.message" /></h2>
   </body>
   </html>
   
   ```

3. 在webapp下添加error.jsp

   ```html
   <%--
     Created by IntelliJ IDEA.
     User: tr
     Date: 19-3-5
     Time: 上午10:06
     To change this template use File | Settings | File Templates.
   --%>
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <%@ taglib prefix="s" uri="/struts-tags" %>
   <html>
   <head>
       <title>error page</title>
   </head>
   <body>
       <h1>error info <s:property value="messageStoreError.message"/></h1>
   </body>
   </html>
   
   ```

4. 在java包内的controller文件夹建立HelloWorldAction.java

   ```java
   package com.tr.controller;
   
   import com.opensymphony.xwork2.ActionSupport;
   import com.tr.dao.MessageStore;
   
   import java.util.Random;
   
   /***
    * @author tr
    */
   public class HelloWorldAction extends ActionSupport {
   
       private MessageStore messageStoreSuccess = null;
       private MessageStore messageStoreError = null;
       private int count = 1;
       private int mgNumber = 2;
   
       @Override
       public String execute() throws Exception {
           Random random = new Random();
           count = random.nextInt();
           if(count%mgNumber==0){
               messageStoreSuccess = new MessageStore("Welcome you are the: "+count);
               return SUCCESS;
           }
           else {messageStoreError = new MessageStore("error count value is: " + count);
           return ERROR;}
       }
   
       public MessageStore getMessageStoreSuccess() {
           return messageStoreSuccess;
       }
   
       public MessageStore getMessageStoreError() {
           return messageStoreError;
       }
   
   
   }
   
   ```

5. 在java包内的dao文件夹建立MessageStore.java

   ```java
   package com.tr.dao;
   
   /**
    * @author tr
    */
   public class MessageStore {
       private String message;
   
       public MessageStore(String message) {
           this.message = message;
       }
   
       public String getMessage() {
           return message;
       }
   
   }
   
   ```

6. 运行maven命令 输入 url   类似 0.0.0.:8080/<ProjectName>/index.action

------

### Taglib

#### url tag with param

```html
<s:url action="hello" var="helloLink">
  <s:param name="userName">Bruce Phillips</s:param>
</s:url>

<p><a href="${helloLink}">Hello Bruce Phillips</a></p>
```

这会产生url的参数 ？userName=Bruce Phillips



#### struts2 form tag

```html
<p>Get your own personal hello by filling out and submitting this form.</p>

<s:form action="hello">
  <s:textfield name="userName" label="Your name" />
  <s:submit value="Submit" />
</s:form>
```

这会产生一个post请求到./hello.action

#### struts2 property tag

```html
<s:property value="messageStore.message" />
```

自动调用对应action的getMessageStore方法 再获取内部message

注意的是每个url请求对应的action对象不同，如果希望这些不同对象同享变量，变量要加static

**如果getMessageStore返回的messageStore对象执行他的.method()返回不同结果**

1. 返回的是string 不做变换
   ​    	2. 返回的是int 变为string
   1. 返回的是对象，自动调用toString

其实最后给人的感觉是可以直接调用java文件里的成员变量

------

### Coding actions

#### action mapping in struts.xml

```xml
<action name="hello" class="com.tr.controller.HelloWorldAction" method="execute">
    <result name="success">/HelloWorld.jsp</result>
</action>
```

action和class关联 并且确定了对view的提供



#### action接受输入的参数

```xml
<s:form action="hello">
    <s:textfield name="userName" label="Your name" />
    <s:submit value="Submit" />
</s:form>
```

需要在控制hello的java类中创建一个userName属性并且实现setter和getter

------

### Processing Forms



#### Forms and a Java model class

1. create a javabean，在dao层创建Person.java

   ```java
   package com.tr.dao;
   
   /**
    * @author tr
    */
   public class Person {
       private String firstName;
       private String lastName;
       private String email;
       private int age;
   
       public String getFirstName() {
           return firstName;
       }
   
       public void setFirstName(String firstName) {
           this.firstName = firstName;
       }
   
       public String getLastName() {
           return lastName;
       }
   
       public void setLastName(String lastName) {
           this.lastName = lastName;
       }
   
       public String getEmail() {
           return email;
       }
   
       public void setEmail(String email) {
           this.email = email;
       }
   
       public int getAge() {
           return age;
       }
   
       public void setAge(int age) {
           this.age = age;
       }
       
       public String toString() {
           return "First Name: " + getFirstName() + " Last Name:  " + getLastName() +
                   " Email:      " + getEmail() + " Age:      " + getAge() ;
       }
   }
   ```

2. view层**register.jsp**

```html
<%@ taglib prefix="s" uri="/struts-tags" %>
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>Register</title>
  </head>
  <body>
    <h3>Register for a prize by completing this form.</h3>

    <s:form action="register">
      <s:textfield name="personBean.firstName" label="First name" />
      <s:textfield name="personBean.lastName" label="Last name" />
      <s:textfield name="personBean.email"  label ="Email"/>  
      <s:textfield name="personBean.age"  label="Age"  />
      <s:submit/>
    </s:form>	
  </body>
</html>
```

1. controller层的Register.java

```java
import com.opensymphony.xwork2.ActionSupport;
import org.apache.struts.register.model.Person;

public class Register extends ActionSupport {
    
    private static final long serialVersionUID = 1L;
    
    private Person personBean;


    public String execute() throws Exception {
        //call Service class to store personBean's state in database
        
        return SUCCESS;
    }
    
    public Person getPersonBean() {
        return personBean;
    }
    
    public void setPersonBean(Person person) {
        personBean = person;
    }

}
```

1. 注意事项：struts2会首先实例化personBean然后赋值，执行execute的时候已经有personBean这个实例了，不需要手动实例化
2. 添加thankyou.jsp

```html
<%@ taglib prefix="s" uri="/struts-tags" %>
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>Registration Successful</title>
  </head>
  <body>
    <h3>Thank you for registering for a prize.</h3>

    <p>Your registration information: <s:property value="personBean" /> </p>

    <p><a href="<s:url action='index' />" >Return to home page</a>.</p>
  </body>
</html>
```

------

### Form Validation

**总结一下就是写个validate，在struts.xml里添加**

```xml
<result name="input">/register.jsp</result>
```

然后会自己自动显示错误信息，以下为官方详细说明：

重写action里的validata方法，这样会自动调用 方法例子：

```java
    public void validate(){
        if (personBean.getFirstName().length() == 0) {
            addFieldError("personBean.firstName", "First name is required.");
        }

        if (personBean.getEmail().length() == 0) {
            addFieldError("personBean.email", "Email is required.");
        }

        if (personBean.getAge() < 18) {
            addFieldError("personBean.age", "Age is required and must be 18 or older");
        }
    }
```

If any errors have been added then Struts 2 will not proceed to call the execute method. Rather the Struts 2 framework  will return `input` as the result of calling the action.



#### Handle Input Being Returned

So what should we do if Struts 2 returns `input`  indicating that the user’s input in the form is not valid? In most  cases  we will want to redisplay the web page that has the form and include in  the form error messages to inform the user what is wrong.

To handle the return value of `input` we need to add the following result to our action node in `struts.xml`.

```
<result name="input">/register.jsp</result>
```

The above result node goes just after the success result node for the register action and before the closing of the action node.

#### Error Messages

So when validation fails and Struts 2 returns input, the Struts 2 framework will redisplay the `register.jsp`. Since we  used Struts 2 form tags, automatically Struts 2 will add the error messages. These error messages are the ones we specified  in the `addFieldError` method call. The addFieldError method takes two arguments. The first is the form field name to which  the error applies and the second is the error message to display above that form field.

So the following `addFieldError` method call:

```
addFieldError("personBean.firstName", "First name is required.")
```

will cause the message *First name is required* to be displayed above the `firstName` field on the form.

#### 如何修改自己form表单错误提示的css

在html的head里添加<s:head />



