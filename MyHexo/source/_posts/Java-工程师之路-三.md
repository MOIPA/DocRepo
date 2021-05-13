title: Java 工程师之路 三
author: tr
tags:
  - Java
categories:
  - Java
date: 2021-03-02 16:31:00
---
# 基础篇

## IO

<!--more-->

### bit byte char

1 char = 2 byte = 16 bit

### 字节流

操作byte类型的主要操作类时OutputStream,InputStream的子类；不用缓冲区直接操作文件本身。

### 字符流

操作字符类型数据，主要操作类：Reader,Writer的子类，使用缓冲区，不关闭流就不输出实际内容。

### 转换

OutputStreamWriter：Writer的子类，将字符流变为字节流

InputStreamReader：Reader的子类，将输入的字节流变为字符流

### Linux下的5种IO模型

#### 阻塞式IO模型

用户线程发出IO请求，内核查看数据是否准备就绪，没有就等待数据，用户线程处于阻塞态，交出CPU，数据就绪后，内核将数据拷贝到用户线程，并将结果返回到用户线程，这时用户线程变为就绪态等待分配cpu时间片。

#### 非阻塞IO模型

非阻塞线程的用户线程会在发出IO请求后即使没有数据也不会让出cpu除非时间片到了，相当于在cpu时间内不断轮询数据。
```java
while(true){
    data = socket.read();
    if(data!= error){
        处理数据
        break;
    }
}
``` 
当然这会导致cpu占用率高的问题。

#### IO复用模型

多路复用IO模型是目前用的最多的，也是Java NIO的方式。

这个模型中，有一个线程不断轮询多个socket状态，只有当socket有真正的读写事件时，才调用实际的IO读写操作，Java NIO中，通过selector.select()去查询每个通道是否有到达事件如果没有线程阻塞。一个线程可以管理多个socket，因此多路复用适合连接数比较多的情况，而且线程轮询每个socket状态实在内核态完成的，效率高。

#### 信号驱动IO模型

用户线程发起IO请求，给对应socket发送信号函数，然后用户线程继续执行，知道内核准备的数据好了发送信号返回用户线程，用户线程接受信号调用IO读写。

#### 异步IO模型

用户线程发起read操作后就可以处理其他事情，内核等待数据准备完成，然后将数据拷贝到用户线程，完成后发送信号给用户线程通知read完成，也就是说用户线程只做了请求发送的动作就可以去使用数据了。

和信号驱动模型对比就是不需要用户线程去调用IO函数进行实际读写。

### BIO NIO AIO

Java中通过流的方式完成IO操作，所有的IO都被是为单个字节的移动，通过Stream对象一次移动一个字节，流可以和外部世界互动，也可以将对象转为字节，或者再转回来。

BIO： Java的Block IO 同步阻塞式IO，java.io包下实现。

NIO：NIO与普通的IO方式最大的区别时数据打包和传输方式，原来的IO以流的方式处理，NIO以块的方式处理。IO流一次一个字节处理，一个输入流产生一个字节的数据，一个输出流消费一个字节的数据，所以面向流的IO方式比较慢。

一个面向块的IO以块形式处理数据，每一个操作都在一次中产生或者消费一个数据库，速度要快得多。

并且NIO的方式时支持同步阻塞和同步非阻塞模式，比如来了100个网络连接调用accept等待响应，NIO会开一个线程会管理这个链接的读写请求，让主线程继续往下响应其他链接，就这样NIO通过一个管理线程轮询所有的链接（socket）的IO状态，某个socket数据准备好了那么会通知这个线程去处理数据。


AIO：异步非阻塞IO模型，真正的异步

### netty

Netty是一个非阻塞I/O客户端-服务器框架，主要用于开发Java网络应用程序，如协议服务器和客户端。异步事件驱动的网络应用程序框架和工具用于简化网络编程，例如TCP和UDP套接字服务器。Netty包括了反应器编程模式的实现。Netty最初由JBoss开发，现在由Netty项目社区开发和维护。

除了作为异步网络应用程序框架，Netty还包括了对HTTP、HTTP2、DNS及其他协议的支持，涵盖了在Servlet容器内运行的能力、对WebSockets的支持、与Google Protocol Buffers的集成、对SSL/TLS的支持以及对用于SPDY协议和消息压缩的支持。自2004年以来，Netty一直在被积极开发

从版本4.0.0开始，Netty在支持NIO和阻塞Java套接字的同时，还支持使用NIO.2作为后端。

本质：JBoss做的一个Jar包

目的：快速开发高性能、高可靠性的网络服务器和客户端程序

优点：提供异步的、事件驱动的网络应用程序框架和工具

## 反射

反射：程序在运行时能够获取自身的信息，在Java中，只要给定类的名字，就能通过反射机制获取属性和方法。

### 用处

1. 运行时判断任意一个对象所属的类

2. 运行时判断任意一个类所具有的成员变量和方法

3. 运行时任意调用一个对象的方法

4. 运行时构造任意一个类的对象

### Class类

Java的Class类时Java反射机制的基础，通过Class可以获得一个类的相关信息

每个类都会有一个Class对象，运行程序时，JVM首先检查是否所有要加载的类对应的Class对象都已经被加载，如果没有，会根据类名查找.class文件，并将其Class对象载入。

### 反射+工厂实现IOC

首先不带反射的普通工厂模式：

```java
interface fruit{
	public abstract void eat();
}
class Apple implements fruit{
	public void eat(){
    	System.out.println("Apple");
    }
}
class Orange implements fruit{
	public void eat(){
    	System.out.println("Orange");
    }
}
// 工厂类 以后有其他新的水果类要修改工厂
class Factory{
	public static fruit getInstance(String fruitName){
    	fruit f = null;
        if("Apple".equals(fruitName)){
        	t = new Apple();
        }
        if("Orange".equals(fruitName)){
        	t = new Orange();
        }
        return f;
    }
}
// 主类
class Main{
	public static void main(String[] a){
    	fruit f = Factory.getInstance("Orange");
        f.eat();
    }
}
```
反射机制实现：

```java
interface fruit{
	public abstract void eat();
}
class Apple implements fruit{
	public void eat(){
    	System.out.println("Apple");
    }
}
class Orange implements fruit{
	public void eat(){
    	System.out.println("Orange");
    }
}
// 工厂类 以后有其他新的水果类要修改工厂
class Factory{
	public static fruit getInstance(String className){
    	fruit f = null;
        try{
        	f = (fruit)Class.forName(className).newInstance();
        }catch(Exception e){
        	e.printStackTrace();
        }
        return f;
    }
}
// 主类
class Main{
	 public static void main(String[] a){
        fruit f=Factory.getInstance("Reflect.Apple");
        if(f!=null){
            f.eat();
        }
    }
}
```
配置properties文件
```properties
apple=Reflect.Apple
orange=Reflect.Orange
```
编写操作属性文件类

```java
class init{
	public static Properties getPro() throws FileNotFoundException, IOException{
        Properties pro=new Properties();
        File f=new File("fruit.properties");
        if(f.exists()){
            pro.load(new FileInputStream(f));
        }else{
            pro.setProperty("apple", "Reflect.Apple");
            pro.setProperty("orange", "Reflect.Orange");
            pro.store(new FileOutputStream(f), "FRUIT CLASS");
        }
        return pro;
    }
}

class Main{
    public static void main(String[] a) throws FileNotFoundException, IOException{
        Properties pro=init.getPro();
        fruit f=Factory.getInstance(pro.getProperty("apple"));
        if(f!=null){
            f.eat();
        }
    }
}
```

运行以上代码可以看出反射动态生成对象的好处。

### IOC容器

IOC最基本的就是反射，使用这个方式，用户只要在spring中配置要生成的对象文件。提高灵活性和可维护性。只有用反射才能做出框架。

IOC容器的工作模式可以看作是工厂模式的升华，工厂要生成的对象都在配置文件中给出定义，然后利用反射机制生成对象。

## 设计模式 代理模式

### 静态代理

定义一个简单接口和实现类

```java
public interface HelloService {
    public void say();
}

public class HelloServiceImpl implements HelloService{

    @Override
    public void say() {
        System.out.println("hello world");
    }
}
```
定义代理对象
```java
public class HelloServiceProxy implements HelloService{

	private HelloService target;
    public HelloServiceProxy(HelloService target){
    	this.target = target;
    }
     @Override
    public void say() {
        System.out.println("记录日志");
        target.say();
        System.out.println("清理数据");
    }
}
```

测试
```java
public class Main {
    @Test
    public void testProxy(){
        //目标对象
        HelloService target = new HelloServiceImpl();
        //代理对象
        HelloServiceProxy proxy = new HelloServiceProxy(target);
        proxy.say();
    }
}
```

这个代理模式的所有角色，代理对象，目标对象，目标对象接口都已经在编译器确定了。


### 动态代理

静态代理会要写很多代码，动态代理则可以省去且在运行期动态生成。

有两种实现方式：

1. JDK动态代理：在java.lang.reflect包中的proxy类和InvocationHandle接口提供了生成动态代理类的能力，但是代理对象必须实现一个或多个接口。

2. Cglib动态代理：code generation library，第三方的代码生成类库，许多apo框架都在用比如spring aop，底层是通过使用一个小而快的字节码处理框架ASM来转换字节码生成新的类。且目标类不需要实现接口。


#### JDK动态代理

```java
public class MyInvocationHandler implements InvocationHandler {

    private Object target;

    public MyInvocationHandler(Object target) {

        super();
        this.target = target;

    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        PerformanceMonior.begin(target.getClass().getName()+"."+method.getName());
        //System.out.println("-----------------begin "+method.getName()+"-----------------");
        Object result = method.invoke(target, args);
        //System.out.println("-----------------end "+method.getName()+"-----------------");
        PerformanceMonior.end();
        return result;
    }

    public Object getProxy(){

        return Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(), target.getClass().getInterfaces(), this);
    }

}

public static void main(String[] args) {

  UserService service = new UserServiceImpl();
  MyInvocationHandler handler = new MyInvocationHandler(service);
  UserService proxy = (UserService) handler.getProxy();
  proxy.add();
}

```

#### Cglib 动态代理

记得在maven添加依赖
```java
public class CglibProxy implements MethodInterceptor{  
 private Enhancer enhancer = new Enhancer();  
 public Object getProxy(Class clazz){  
  //设置需要创建子类的类  
  enhancer.setSuperclass(clazz);  
  enhancer.setCallback(this);  
  //通过字节码技术动态创建子类实例  
  return enhancer.create();  
 }  
 //实现MethodInterceptor接口方法  
 public Object intercept(Object obj, Method method, Object[] args,  
   MethodProxy proxy) throws Throwable {  
  System.out.println("前置代理");  
  //通过代理类调用父类中的方法  
  Object result = proxy.invokeSuper(obj, args);  
  System.out.println("后置代理");  
  return result;  
 }  
}  

public class DoCGLib {  
 public static void main(String[] args) {  
  CglibProxy proxy = new CglibProxy();  
  //通过生成子类的方式创建代理类  
  UserServiceImpl proxyImp = (UserServiceImpl)proxy.getProxy(UserServiceImpl.class);  
  proxyImp.add();  
 }  
}
```

### AOP

Spring AOP 中的动态代理有两种方式，JDK动态代理和CGLIB动态代理

JDK动态代理通过反射来接收被代理的类，并且要求被代理的类必须实现一个接口。JDK动态代理的核心是InvocationHandler接口和Proxy类。

如果目标类没有实现接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。

CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成某个类的子类，注意，CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。

