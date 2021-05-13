title: Java工程师之路 一
author: tr
tags:
  - Java
categories:
  - Java
date: 2020-06-09 13:09:00
---
# 基础

## 面向对象

### 面向对象三大基本特征

1. 封装 这个不用多说
2. 继承 子类可以继承父类，对父类进行拓展，有两种继承：实现继承（直接使用父类方法），接口继承（重写父类属性和方法），实际代码编写中应该慎用继承，除非真是`is-a`关系 否则别用，用组合。
3. 多态 一个类的实例在不同情况下有不同表现，最常见就是子类传入父类参数中，运行调用父类时，实际的操作是其子类完成

### 面向对象5大基本原则 SOLID
<!--more-->

1. S：Single-Responsibility Principle 单一职责原则，一个类就做好一件事就好了，提高内聚降低耦合

2. O：Open-Closed Principle 开闭原则，类应该对扩展开发，对修改封闭，人话就是来了新需求不改动原来代码，直接对现在的代码进行拓展。举例来说，就是写类的时候让这个类继承一个父类或者接口，实现的时候覆写这个类或实现接口，下次有新的需求，不改动原来的子类，重新弄一个继承这个接口或者类的子类，来满足需求。

3. L：Liskov-Substitution Principle 里氏替换原则，子类必须能替换父类，为什么这样，因为继承是侵入性的，父类修改了变量或者方法，不满足替换原则的子类也会被变动，如果满足替换原则，父类随便怎么动都不会影响子类，
```
+子类必须实现父类的抽象方法，但不得重写（覆盖）父类的非抽象（已实现）方法。
+子类中可以增加自己特有的方法。
当子类覆盖或实现父类的方法时，方法的前置条件（即方法的形参）要比父类方法的输入参数更宽松。
+当子类的方法实现父类的抽象方法时，方法的后置条件（即方法的返回值）要比父类更严格。
```
举例：一个长方形类，有一个子类正方形类,如果正方形类里头重写了计算面积方法（边的平方），那么则无法满足原则，因为重写了非抽象方法

4. I：Interface-Segregation Principle 接口隔离原则，核心就是多个小接口，不要`fat interface`肥接口，接口应该单一的负责某件事。

5. D：Dependecy-Inversion Principle 依赖倒置原则，核心就是模块间只依赖接口，不依赖具体。

### 重载和重写

重载：Java的一个语法特性而已，有人觉得算多态，但只有在编译的时候会去判断调用哪个方法。

重写：运行时绑定，多态的体现，子类覆写父类方法，只有在运行的时候根据变量所指向的实际对象的类型调用方法。

### 变量

java里有三种变量：类变量（区），成员变量（堆），局部变量（栈）
```java
public class Variables {
    
        /**
         * 类变量
         */
        private static int a;
    
        /**
         * 成员变量
         */
        private int b;
    
        /**
         * 局部变量
         * @param c
         */
        public void test(int c){
            int d;
        }
    }
```
### 作用域
public：不解释

private：不解释

protected：只有类本身，统一包的其他类和子类能访问，其他包但不是子类的都不可以。

default：只有自己和其他包的类可以访问，其他包包括子类不可以访问

### 语言无关性

我们知道java可以write once run anywhere 这是靠jvm实现的，java编译为字节码，由jvm运行这些字节码文件，所以Java还是语言无关的，因为只要能弄出class文件，管他用什么语言写出来。
```
JVM支持：
1. Kotli
2. Groovy
3. Scala
4. Jruby
5. Jython
6. Fantom
7. Clojure（Lisp）
8. Rhino
9. Ceylon
```
### 值传递和引用传递**

什么是实参和形参：

形参：定义函数名和函数体的时候使用的参数，用来接受调用该函数传过来的参数

实参：调用有参函数时，主调函数传递过去的参数

```java
public static void main(String[] args) {
  ParamTest pt = new ParamTest();
  pt.sout("Hollis");//实际参数为 Hollis
}

public void sout(String name) { //形式参数为 name
  System.out.println(name);
}
```

值传递：调用函数的时候将实际参数复制一份到函数中，这样对复制过来的参数随意修改不会修改实际参数

引用传递：调用函数的时候不是拷贝参数，而是将参数的地址传递过去，那么被调函数对这个参数修改会影响实际参数

*值传递和引用传递根本区别在于是否创建副本*

Java中只用值传递！

Java的求值策略：很多人误以为Java的对象传递是引用传递，实际上官方文档指出了，java就是值传递，只不过是把对象的引用当作值传递给方法，也就是说Java会将对象的地址的拷贝传递给被调函数，这个地址就是形式参数。这个方法是：传共享对象调用

## 基础数据类型

### 整型
byte 一个字节 正负
short 2字节 正负
int 4个字节 正负
long 8个字节 正负

Java中每个类型都有范围，如果溢出不会抛出异常，也没有提示所以同类型运算一定要注意数据溢出问题

### 浮点数

要给浮点数由两个数m和e表示：a = m* be m是尾数，b是基（操作系统给的基）p是保存的位数。

注意：不能用浮点数表示金额，计算机中保存的小数其实是十进制小数的近似值，并不是准确值，使用`BigDecimal`或者`Long`来表示金额

## 自动拆装箱

#### 为啥要有基本类型：

Java中存在基本类型（内置类型），之所以存在这些类型是因为在Java中new一个对象是存在堆里的，通过栈的引用使用这些对象，所以对象比较消耗资源，对于经常用的的数据类型 int之类的，每次new的话比较笨重。提供的基本数据类型每次创建都在栈中存储，更加高效


#### 为啥要有包装类型：

不同于scala 万物对象的概念，Java中的内置类型非对象，在遇到集合类的时候是无法将基础类型放入集合中的，为了解决这种问题，Java提供了和基本数据类型对应的包装类 都处于 `java.lang`包下，每个包装类多了一些属性和方法，更加方便操作。

#### 拆箱和装箱

有了基本类型和包装类型，肯定有要在其中转换的时候，包装类转基本类型就是拆箱，反之装箱（boxing），在Java5之前需要手动操作装箱：`Integer i = new Integer(10);`

在SE5中，为了方便开发，Java提供的自动拆箱和装箱功能。
```java
Integer i = 10;// 自动装箱
int b = i; // 自动拆箱
```

#### 自动拆装箱的实现原理

有如下自动拆装箱代码：
```java
Integer integer = 1; // boxing
int i = integer; //unboxing
```
反编译后：
```java
Integer integer = Integer.valueOf(1);
int i = integer.intValue();
```
可以看出自动装箱都是调用的`XXX.valueOf()`，自动拆箱通过`XXX.xxxValue()`实现

#### 啥时候自动帮你拆装箱

1. 数据进集合的时候
```java
List<Integer> list = new LinkedList<>();
list.add(1); // 没有报错，auto boxing
```

2. 包装类和基本类型比较的时候

包装类会先拆箱成基本类型然后进行比较

3. 包装类型的运算

两个包装类型进行运算也会自动拆箱成基本类型

4. 三目运算符

`int k = flag ? i.intValue() : j;`
只要当第二(i)和第三位（j）操作数为基本类型和对象的时候，对象就会拆箱。这时候如果i为null 那么拆箱会导致NPE问题

5. 函数参数和返回值

***NOTE： 一定要注意 在unboxing的时候如果包装类为null 会导致NPE 空指针异常

#### ***面试会考的玩意：自动拆装箱和缓存

```java

        Integer integer1 = 3;
        Integer integer2 = 3;

        if (integer1 == integer2)
            System.out.println("integer1 == integer2");
        else
            System.out.println("integer1 != integer2");

        Integer integer3 = 300;
        Integer integer4 = 300;

        if (integer3 == integer4)
            System.out.println("integer3 == integer4");
        else
            System.out.println("integer3 != integer4");
            
结果：
integer1 == integer2
integer3 != integer4
```

`==`用来比较对象引用 `equals`用来比较值，正常人会认为上面两个都是false，但是在SE5中Java为了节省内存，通过让整型对象使用相同对象实现了缓存和重用

只适用于 -128 ~ +127 只适用于自动拆装箱，手动用构造函数的对象不适用

事实上这个范围可以在`java.lang.Integer.IntegerCache.high`中设置最大值

缓存不止存在于Integer，Byte、Short、Long的缓存范围和Integer一样，只有Character的缓存范围是0~127

#### 如何正确定义返回值：isSuccess/success

日常开发中定义布尔类型变量，比如RPC接口，会定义是否成功字段，一般有四种方式来定义：
```java
boolean success
boolean isSuccess
Boolean success
Boolean isSuccess
```
1. success还是isSuccess

阿里巴巴开发手册中规定任何布尔类型都不可加is，否则会导致序列化错误：定义为boolean isSuccess 的属性 它的方法也是 isSuccess(),RPC框架反向解析的时候以为对应的属性是success 导致获取不到。

自己在idea中生成两个属性试一下就知道了。首先关于getter和setter，如果是普通的参数就要生成get和set方法。但是如果是布尔类型，那么方法则是：is...和set...

所以如果命名boolean isSuccess 那么对应方法就是 isIsSuccess() 和 setIsSuccess(); 但是生成的是isSuccess()和setSuccess，如果普通情况下没什么问题，但是在序列化的时候就会出现问题。

如果实体类定义了 boolean isSuccess 在JSON序列化的时候，fastJson和jackson会序列化为：`{"xx":"xx","success":true}`，这是因为这两个工具在序列化的时候会通过反射获取所有的getter方法，的到了isSuccess方法后，根据Java的setter/getter命名原则，认为success是属性。而Gson结果是`{"isSuccess":true}`说明Gson遍历属性而不是getter方法

***结论： 用success 不要+is 可以导致很多灵异状况

2. boolean还是Boolean

现在还剩下两个情况：
```java
boolean success
Boolean success
```
首先这两个区别 基本类型的默认值是false，包装类的是null

阿里巴巴规范中：所有的POJO必须使用包装类，RPC方法返回值和参数使用包装类，推荐局部变量使用基本数据类型。

举个例子：如果调用RPC服务返回值，如果返回值是基本类型，那么如果调用失败或者异常退出，返回的值是0 而不是null。这是有问题的！ 如果使用包装类接受，拿到null值计算的时候会抛出异常提示用户，而不是用默认值计算误导用户。

总结： RPC的返回值和POJO中尽量使用包装类 定义的布尔类型不要+ is

## String 字符串

#### 字符串不可变

`String s = "abcd"` 这个s保存了String对象的引用（堆中）。`String s2 = s` 这个s2保存了相同的引用值。`s = s.concat("ef")` 这时候s保存的是新建的对象的引用

一旦一个String对象在堆中创建出来，就再也无法被修改，注意的是String的所有类方法都不是修改字符串，而是生成新的对象返回。如果需要一个可修改字符串对象，使用：`StringBuffer` or `StringBuilder`否则有大量时间浪费在GJ上

#### substring 原理

substring(int begin,int end)在不同版本是不同的，这是个字符串截取方法，左闭右开

在JDK6中：String是通过字符串数组实现的，String类中包含三个成员变量：char value[],int offset,int count,调用subString的时候会创建新的String对象，但是这个值依旧指向原来堆中的这个char数组，只不过count和offset不同。

这个版本这么操作会导致问题，如果一个很长的字符串做了截取操作，只截取了很短一点，这个截取的对象还是指向原来的那一长串，会导致原来的长的String对象无法释放，可能导致内存泄漏，所以在JDK6中会使用以下方法重新生成一个字符串：`x = x.substring(x,y)+""`
这个问题已经被记录在了java的bug database中

JDK7：现在这个问题得到了解决，可以在堆中生成新的数组了


#### replaceFirst,replaceAll,replace 区别
```java
//文字替换（全部） 
Pattern pattern = Pattern.compile("正则表达式"); 
Matcher matcher = pattern.matcher("正则表达式 Hello World,正则表达式 Hello World"); 
//替换所有符合正则的数据 
System.out.println(matcher.replaceAll("Java")); 
System.out.println("abac".replace("a", "\a")); //\ab\ac
```
1. replaceAll替换所有符合正则的文字

2. replaceFirst 替换第一个符合正则的

3. replace 替换字符串

#### String 对 + 的重载

String s = "a"+"b" 编译器会变成：String s = "ab"

建议使用StringBuilder的append方法，最后调用toString() 效率更高，特别是在循环中，使用+的话每次循环Java都会new一个StringBuilder进行append 然后toString返回对象


#### StringBuffer和StringBuilder

最大区别是StringBuffer是线程安全的，它的append方法前面加了synchronized

平时开发不是循环就用+就好了，如果并发下要用StringBuffer


#### 数字转字符

```java
int i = 5;
String i1 = ""+i
String i2 = String.valueOf(i);
String i3 = Integer.toString(i);
```

最后两个没区别，最糟糕的是使用+，会生成一个StringBuilder 调用append 然后toString()

#### switch 对String的支持

##### switch 和 整型

switch 对int的判断是直接比较整数的值

##### switch 和 char

实际上比较的是ascall码，编译器会将char转为int

##### switch 和 字符串

```java
public class switchDemoString {
    public static void main(String[] args) {
        String str = "world";
        switch (str) {
        case "hello":
            System.out.println("hello");
            break;
        case "world":
            System.out.println("world");
            break;
        default:
            break;
        }
    }
}
```
反编译：

```java
public class switchDemoString
{
    public switchDemoString()
    {
    }
    public static void main(String args[])
    {
        String str = "world";
        String s;
        switch((s = str).hashCode())
        {
        default:
            break;
        case 99162322:
            if(s.equals("hello"))
                System.out.println("hello");
            break;
        case 113318802:
            if(s.equals("world"))
                System.out.println("world");
            break;
        }
    }
}
```

switch 是通过 字符串的equals方法和hashcode方法实现的，返回的哈希值实际是int类型，之所以再次使用equals方法确保一样是因为哈希值可能会碰撞，实际上对string的switch确实性能没有整型或者字符型高，但是也没有那么差，i那位就多了一个equals方法，而且hashcode方法是有缓存的，在循环中不会有太大开销。

#### 字符串常量池

`String str = "abc"` 这种叫做字面量。

jvm为了减少相同字符串的重复创建，会单独开辟一个内存用来保存字符串常量，这个内存区叫做字符串常量池。当代码中有以上这种命名方法时，会先检查字符串常量池，看有没有相同内容的，有就返回这个字符串对象的引用，否则创建新的字符串对象，把引用让如字符串常量池。

此机制：字符串驻留或字符串池化

字符串常量池位置：JDK7以前放在永久代，7中放在堆，8中移除了永久代，元空间代替了永久代，就放在了元空间。

#### Class常量池

Java一共有三种常量池：字符串常量池，Class常量池，运行时常量池

##### class文件
可以通过`javap -v helloworld.class`查看常量池

如果将java代码如下编译为class文件
```java
public class HelloWorld {
    public static void main(String[] args) {
        String s = "Hollis";
    }
}
```
得到的HelloWorld.class ,用16进制打开，`vim HelloWorld.class` `:%!xxd`即可查看内容。

Class常量池其实就是Class文件中的资源库，Class文件包含了类的版本，字段，方法，接口，还有一个常量池，里面存放了各种字面量和符号引用。

不容的Class文件里的常量个数肯定不一样，所以在Class文件的入口会设置两个字节的常量池容量计数器，记录常量个数，常量池计数是从1开始的。

例子：
```
cafe babe  0000    0034    0011        0400...
魔数     次版本号 主版本号 常量池计数   常量池数据区
```

常量池里存了：

+ 字面量：int a = 123 这个123就是字面量，即整数，字符，浮点数

+ 符号引用：类和接口的全名，字段的名称和描述，方法的名称和描述

##### 有什么用

Class和字符串常量池不同，这是Class文件中包含的各种常量，是在JVM运行加载的时候使用，而字符串常量池不过是在运行中动态在元空间创建的数据。

Javac编译的时候并不像C那样有链接，而是在JVM加载class文件的时候动态链接，虚拟机运行的时候需要从常量池获取对应的符号引用，再在类创建时翻译到具体的内存地址。

class是用来保存常量的一个地方，JVM载入文件时需要把常量池里的常量加载到内存里。

#### 运行时常量池

运行时常量池存储在方法区，每当一个类或者接口被创建，就会存储至对应的运行时常量池，包括了不同常量：数值字面量，方法或者字段的引用。jdk1.8中没有了永久代，只有元空间，而方法区存在永久代或者元空间，所以现在的运行时常量池也在元空间。

运行时常量池包含 Class常量池的常量，字符串常量池的内容

#### 三个常量池的关系

虚拟机启动时，会将各个Class文件的常量池载入到运行时常量池（元空间内），所以Class常量池只是一个暂时存放数据的地方，类似硬盘，启动的时候从里面取数据加载到内存。

字符串常量池就类似运行时常量池的分支，加载时，对于class文件里的常量池，里面是字符串的部分都会被装入字符串常量池

#### intern

显式使用常量池，使用这个方法会将字符串内容放入字符串常量池(如果找不到相同字符串)。

`String str = "ab".intern();`

#### String的长度

定义时的长度限制：

之前已经了解了，String类实际由三个部分组成，byte bytes[], int offset, int length。

这里面的length是整型，所以String定义的时候最大长度是int的最大范围：2^31-1。

但是实际不可能真的能定义到这么长，这是因为我们定义的字符串都要保存进字符串常量池中，字符串常量池本身就是有限制的，这个大小是2字节：2^16-1 = 65535 但是真正编译的时候javac规定了 > 65535就会抛出异常编译失败。所以我们真正的最大值是 65534

运行时的长度限制：int的范围，可以写一个for循环试试。所以在开发中要注意，如果对大的图片或者文本做加密 base64或者其他，容易让string超出范围。

## Java的关键字


#### transient

Java中的集合类 ArrayList和Vector都是通过数组实现的，只在定义element时不同，Arraylist使用了transient

`private transient Object[] elememntData`

意思就是被transient修饰的值在序列化的时候会被忽视，反序列化后，被修饰的变量或被设置为默认值 int 就是0，对象就是null

#### instanceof

`if(A instanceof ArrayList)` 用来测试是否是右边类的实例。


#### volatile 重要

volatile 被认为时轻量级的 synchronized，volatile只能修饰变量。如果一个变量可能被多个线程访问，就可以修饰。

```java
public class Singleton {  
    private volatile static Singleton singleton;  
    private Singleton (){}  
    public static Singleton getSingleton() {  
    if (singleton == null) {  
        synchronized (Singleton.class) {  
        if (singleton == null) {  
            singleton = new Singleton();  
        }  
        }  
    }  
    return singleton;  
    }  
}  
```

##### 原理

cpu和内存之间是存在多级缓存的 L1cache L2chache。但是缓存之间会存在缓存不一致问题，比如一个双核cpu: CORE ONE 对应的 L1Cache 和 CORE TWO 对应的 L1Cache 里面存了同样的变量，这两个缓存的这个变量可能不同步。写回主存的时候可能会覆写。

但是对于volatile变量，从cache获得这个变量进行写的时候JVM会向cpu发一条lock前缀的指令将这个缓存中的变量先写回主存。但是其他的缓存可能还存的旧值，所以为了在多处理器下的各个缓存保持一致，会实现`缓存一致性协议`。

`缓存一致性协议`: 最出名的就是Intel 的MESI协议，MESI协议保证了每个缓存中使用的共享变量的副本是一致的。它核心的思想是：当CPU写数据时，如果发现操作的变量是共享变量，即在其他CPU中也存在该变量的副本，会发出信号通知其他CPU将该变量的缓存行置为无效状态，因此当其他CPU需要读取这个变量时，发现自己缓存中缓存该变量的缓存行是无效的，那么它就会从内存重新读取。

这样，当一个volatile变量被修改，会被立刻刷回主存，并通知其他cache的缓存行状态为无效。这样就可以包装volatile的变量在所有cache中是可见的。

可见性：一个线程修改了这个变量，其他线程能立刻看到。通过缓存一致性解决了。

有序性：cpu是会对输入代码乱序执行的。比如对a+1和对b+1这两个指令是随机执行的。volatile会让cpu顺序执行。比如volatile修饰a变量， 执行a+1 a-1,这两个指令只要是有a出现的cpu不会变动顺序，一个一个执行。

原子性：一个操作要么全部执行完，要么不执行。对于volatile来说无法保证原子性。

为什么volatile无法保证原子性：对于i++这样的操作，cpu不是一步完成的，是分作读取，自增， 写入这三步骤，被volatile修饰的变量修改写入可以作为原子操作。当两个线程同时执行i++,假设A线程读取了i的值，B线程也同时读取了i的值，这时候他们都将i复制到了自己的工作内存，假如A修改了值，虽然满足可见性，其他线程的这个值都变了，但是对于B线程来说除非再读取一次主存内容，不然工作内存的i还是原来的值。总之两个cpu各自读取了值，共同写入cache中，这时候无所谓可见性。

load-compute-store-其他线程可见，这四步，前面三步是不安全的

#### synchronized

synchronized 可以修饰方法，代码块，被修饰的这些同一时间只能被一个线程访问。

##### 原理

>> 方法级的同步是隐式的。同步方法的常量池中会有一个ACC_SYNCHRONIZED标志。当某个线程要访问某个方法的时候，会检查是否有ACC_SYNCHRONIZED，如果有设置，则需要先获得监视器锁，然后开始执行方法，方法执行之后再释放监视器锁。这时如果其他线程来请求执行方法，会因为无法获得监视器锁而被阻断住。值得注意的是，如果在方法执行过程中，发生了异常，并且方法内部并没有处理该异常，那么在异常被抛到方法外面之前监视器锁会被自动释放。

>> 同步代码块使用monitorenter和monitorexit两个指令实现。可以把执行monitorenter指令理解为加锁，执行monitorexit理解为释放锁。 每个对象维护着一个记录着被锁次数的计数器。未被锁定的对象的该计数器为0，当一个线程获得锁（执行monitorenter）后，该计数器自增变为 1 ，当同一个线程再次获得该对象的锁的时候，计数器再次自增。当同一个线程释放锁（执行monitorexit指令）的时候，计数器再自减。当计数器为0的时候。锁将被释放，其他线程便可以获得锁。

>> 无论是ACC_SYNCHRONIZED还是monitorenter、monitorexit都是基于Monitor实现的，在Java虚拟机(HotSpot)中，Monitor是基于C++实现的，由ObjectMonitor实现。

>> ObjectMonitor类中提供了几个方法，如enter、exit、wait、notify、notifyAll等。sychronized加锁的时候，会调用objectMonitor的enter方法，解锁的时候会调用exit方法

monitorenter 和 monitorexit都是原子性的操作。加锁解锁，死锁，和cpu调度问题都是操作系统原理，默认都掌握了。

#### final

final变量：变量会变成常量

final方法：子类无法覆写

final类：无法被继承


#### static

修饰变量和方法，也可以形成静态代码块。

静态变量：一个类的静态变量不属于任何类的对象或者实例，不是线程安全的，所以一般用final修饰静态变量，如果静态变量不是private，可以通过类名.变量名访问。

静态方法：和静态变量一样只属于类，静态方法稚嫩该调用静态变量和静态方法，静态方法一般是用来给其他类使用的公共方法所以不需要创建实例。

静态代码块：一组指令在类加载的时候在内存中由Java ClassLoader执行。一般用来初始化类的静态变量，在类装载的时候创建静态资源，Java不允许在静态块中使用非静态变量。

静态类：挺少用的，大部分用来作配置。

#### const 

Java的预留关键字 和 final相似 不怎么用
