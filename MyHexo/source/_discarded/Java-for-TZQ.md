title: Java for TZQ
author: tzq
tags:
  - Java
categories:
  - Java
date: 2020-09-19 18:23:00
---
## JAVA FOR TZQ

### Java 基础

#### static 和入口程序 main

```java
package HelloWorld;
public class HelloWorld{
    public static void main(String[] args){
        System.out.print("hello world!\n");

        SaySomeThing saySomeThing = new SaySomeThing();
        saySomeThing.sayName();

        SaySomeThing saySomeThing2 = new SaySomeThing();
        saySomeThing2.sayName();

        SaySomeThing.staticMethod();


    }

    public static void test(){
        System.out.print("obj");
    }
}


class SaySomeThing{

    private int val = 0;

    public void sayName(){
        System.out.print("name\n");
    }

    public static void staticMethod(){
        System.out.print("i am static!");
    }

}
```

![upload successful](/images/pasted-29.png)

说明： 如果类加了static，或者类里头的方法加了static

1. 类加了static方法：那么无论怎么new 这个类，生成的实例化对象都是模板默认的实例化对象，所有对象都是这个默认对象。

2. 方法加了static：那么new这个类，会生成新的对象，但是使用static的方法会调用模板默认实例化对象的方法。