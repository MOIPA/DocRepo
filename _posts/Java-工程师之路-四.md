title: Java 工程师之路 四
author: tr
tags:
  - Java
categories:
  - Java
date: 2021-03-10 14:27:00
---
# 基础

## Java序列化

<!--more-->

序列化是将对象转换为可传输格式的过程，是一种数据持久化手段，一般广泛应用于网络传输，RMI和RPC场景。

反序列化就是逆操作，Java序列化特指JDK的序列化实现，这个实现不同于JSON序列化，是不跨语言的。

### Java对象的序列化

Java内创建的对象都是存在于JVM的堆内存中，只有JVM处于运行态这些对象才会存在，一旦JVM停止运行，这些对象状态也随之消失。但是在真是场景中我们需要将这些对象持久化下来，并且在需要的时候将对象重新读取出来。对象序列化可以很容易的在JVM中的活动对象和字节数组之间进行调换。

### Serializable接口

类通过实现java.io.Serializable接口以启用其序列化功能，当对一个对象进行序列化的时候，如果遇到不支持Serializable接口的对象，会抛出 NotSerializableException。

配置实体类
```java
import java.io.Serializable;
 @Data
public class User1 implements Serializable {
    private String name;
    private int age;
    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```
序列化实体类
```java
public class SerializableDemo1 {

    public static void main(String[] args) {
        //Initializes The Object
        User1 user = new User1();
        user.setName("test");
        user.setAge(1);
        System.out.println(user);

        //Write Obj to File
        ObjectOutputStream oos = null;
        try {
            oos = new ObjectOutputStream(new FileOutputStream("tempFile"));
            oos.writeObject(user);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            IOUtils.closeQuietly(oos);
        }

        //Read Obj from File
        File file = new File("tempFile");
        ObjectInputStream ois = null;
        try {
            ois = new ObjectInputStream(new FileInputStream(file));
            User1 newUser = (User1) ois.readObject();
            System.out.println(newUser);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } finally {
            IOUtils.closeQuietly(ois);
            try {
                FileUtils.forceDelete(file);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

    }
}

```

### Externalizable接口

Externalizable继承了Serializable接口，且定义了两个新的方法：writeExternal()和readExternal()，使用Externalizable接口进行序列化的时候要重写writeExternal()和readExternal()方法。

### serialVersionUID

序列化时将对象状态信息转为可存储或可传输的形式，虚拟机是否支持反序列化不仅取决于类的路径和代码一致，还要求两个类的序列化ID是否一致，即serialVersionUID

反序列化时：JVM将传递过来的serialVersionUID与本地对应的实体类的UID进行比较，相同即可反序列化。

这样是为了安全，因为文件存储的内容可能被篡改。

### Java序列化缺陷

1. 不能跨语言，如果两个不同语言编写的程序进行通讯，用Java序列化是无法完成序列化和反序列化的。

2. 不安全，Java的反序列化实质是通过ObjectInputStream类调用readObject()方法实现的，这个方法本质是个构造器，可以将实现了Serializable接口的类都实例化，所以在反序列化过程中该方法会执行任意代码。
以下是攻击代码：
```java
Set root = new HashSet();  
Set s1 = root;  
Set s2 = new HashSet();  
for (int i = 0; i < 100; i++) {  
   Set t1 = new HashSet();  
   Set t2 = new HashSet();  
   t1.add("test"); //使t2不等于t1  
   s1.add(t1);  
   s1.add(t2);  
   s2.add(t1);  
   s2.add(t2);  
   s1 = t1;  
   s2 = t2;   
} 
```
创建循环对象链，然后将序列化后的对象传输到程序中反序列化，这种情况会导致 hashCode 方法被调用次数呈次方爆发式增长, 从而引发栈溢出异常。

3. 序列化后的流很大

4. 序列化性能差 太慢

### 序列化对单例的破坏

实现一个双重校验锁的单例，对其序列化再反序列化，查看是否同一个对象。

双重校验锁单例看上一篇
验证：
```java
public static void main(String[] args) throws IOException, ClassNotFoundException {
        //Write Obj to file
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("tempFile"));
        oos.writeObject(Singleton.getSingleton());
        //Read Obj from file
        File file = new File("tempFile");
        ObjectInputStream ois =  new ObjectInputStream(new FileInputStream(file));
        Singleton newInstance = (Singleton) ois.readObject();
        //判断是否是同一个对象
        System.out.println(newInstance == Singleton.getSingleton());
    }
```

如何解决反序列化对单例的破坏呢：在单例类中定义readResolve即可添加方法到单例中：

```java
  private Object readResolve() {
        return singleton;
    }
```

原理：调用栈挺多的，实质就是在底层方法里有个判断目标对象是否有readResolve方法，如果有，就通过反射的方式调用readResolve方法返回一个对象用于反序列化最终返回对象。所以单例的readResolve方法直接返回自身。

### Apache-Commons-Collections的反序列化漏洞


这个漏洞只出现在3.2.1版本以下

#### 复现

Commons Collections中提供了一个Transformer接口，主要是用来进行类型转换，这个接口有一个实现类：InvokerTransformer，里面有个transform方法，核心代码就三行，通过反射将传入的对象进行实例化。然后执行其iMethodName方法。


![upload successful](/images/pasted-38.png)

## 注解

### 元注解

定义其他注解的注解

元注解有六个:@Target（表示该注解可以用于什么地方）、@Retention（表示再什么级别保存该注解信息）、@Documented（将此注解包含再javadoc中）、@Inherited（允许子类继承父类中的注解）、@Repeatable（1.8新增，允许一个注解在一个元素上使用多次）、@Native（1.8新增，修饰成员变量，表示这个变量可以被本地代码引用，常常被代码生成工具使用）。

### Java常用注解

@Override 表示覆写父类方法

@Deprecated 表示当前方法已过时

@SuppressWarnings 表示关闭一些警告信息

@SafeVarargs 表示专门为抑制堆污染警告提供的

@FunctionalInterface 表示用来指定某个接口必须是函数式接口

### Spring常用注解

@Configuration：会将一个类作为IoC容器，它的某个方法头上如果注册了@Bean就会作为这个Spring容器中的Bean。

@Scope 作用域

@Lazy(true) 延迟初始化

@Service 标注业务层组件

@Controller 标注控制层组件

@Repository 标注数据访问组件，即DAO组件

@Component 泛指组件，当组件不好归类的时候可以用这个注解

@PostConstruct用于指定初始化方法（用在方法上）

@PreDestory用于指定销毁方法（用在方法上）

@DependsOn：定义Bean初始化及销毁时的顺序

@Primary：自动装配时当出现多个Bean候选者时，被注解为@Primary的Bean将作为首选者，否则将抛出异常

@Autowired 默认按类型装配，如果我们想使用按名称装配，可以结合@Qualifier注解一起使用。如下： @Autowired @Qualifier("personDaoBean") 存在多个实例配合使用

@Resource默认按名称装配，当找不到与名称匹配的bean才会按类型装配。

@PostConstruct 初始化注解

@PreDestroy 摧毁注解 默认 单例 启动就加载

### 如何定义注解

注解和接口的定义差不多，注解多了个@符号

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
public @interface EnableAuth{
	String name() default "tr"; 
}
```
可以定义成员变量，还可以添加默认值，实例如上。

Target: 指定注解修饰什么东西（类，方法，字段）

Rentention: 指定修饰的注解保留多长时间，分别SOURCE（注解仅存在于源码中，在class字节码文件中不包含）,CLASS（默认的保留策略，注解会在class字节码文件中存在，但运行时无法获取）,RUNTIME（注解会在class字节码文件中存在，在运行时可以通过反射获取到）三种类型，* 如果想要在程序运行过程中通过反射来获取注解的信息需要将Retention设置为RUNTIME ！*

Documented：指定被修饰的注解类将被javadoc工具提取成文档。

Inherited：指定注解是否有继承性

### 通过反射使用自定义注解

如何获取类的方法，字段上的注解，以及获取注解的值：

```java
Class<?> clz = bean.getClass();
Method[] methods = clz.getMethods();
for (Method method : methods) {
    if (method.isAnnotationPresent(EnableAuth.class)) {
        String name = method.getAnnotation(EnableAuth.class).name();
    }
}
```

实体类：

```java
@Data
@ToString
public class Person {
    @EnableAuth
    private String stra;
    private String strb;
    private String strc;

    public Person(String str1,String str2,String str3){
        super();
        this.stra = str1;
        this.strb = str2;
        this.strc = str3;
    }

}
```

获取注解

```java
public class MyTest {
    public static void main(String[] args) {
        //初始化全都赋值无注解
        Person person = new Person("无注解","无注解","无注解");
        //解析注解
        doAnnoTest(person);
        System.out.println(person.toString());
    }

  private static void doAnnoTest(Object obj) {
        Class clazz = obj.getClass();
        Field[] declareFields = clazz.getDeclaredFields();
        for (Field field:declareFields) {
            //检查该字段是否使用了某个注解
            if(field.isAnnotationPresent(EnableAuth.class)){
                EnableAuth anno = field.getAnnotation(EnableAuth.class);
                if(anno!=null){
                    String fieldName = field.getName();
                    try {
                        Method setMethod = clazz.getDeclaredMethod("set" + fieldName.substring(0, 1).toUpperCase() + fieldName.substring(1),String.class);
                        //获取注解的属性
                        String annoValue = anno.value();
                        //将注解的属性值赋给对应的属性
                        setMethod.invoke(obj,annoValue);
                    }catch (NoSuchMethodException e){
                        e.printStackTrace();
                    } catch (IllegalAccessException e) {
                        e.printStackTrace();
                    } catch (InvocationTargetException e) {
                        e.printStackTrace();
                    }

                }
            }
            
        }
    }

}
```

## 泛型

泛型是JDK5引入的新特性，允许在定义类和接口的时候使用类型参数。

泛型最大的好处是复用，比如一个List接口，可以将String，Integer等类型放入，如果不用泛型，存String要写一个对应的接口，使用泛型就可以解决这个问题。

### Java的类型擦除

Java语言中的泛型只在程序源码中存在，在编译后的字节码文件中，就已经被替换为原来的原生类型（Raw Type，也称为裸类型）了，并且在相应的地方插入了强制转型代码，因此对于运行期的Java语言来说，ArrayList<Integer>与ArrayList<String>就是同一个类。所以说泛型技术实际上是Java语言的一颗语法糖，Java语言中的泛型实现方法称为类型擦除，基于这种方法实现的泛型被称为伪泛型。
  
  类型擦除指的是通过类型参数合并，将泛型类型实例关联到同一份字节码上。编译器只为泛型类型生成一份字节码，并将其实例关联到这份字节码上。类型擦除的关键在于从泛型类型中清除类型参数的相关信息，并且再必要的时候添加类型检查和类型转换的方法。 类型擦除可以简单的理解为将泛型java代码转换为普通java代码，只不过编译器更直接点，将泛型java代码直接转换成普通java字节码。 类型擦除的主要过程如下： 1.将所有的泛型参数用其最左边界（最顶级的父类型）类型替换。 2.移除所有的类型参数。
  
  
### eg

code:

```java
public static void main(String[] args) {  
    Map<String, String> map = new HashMap<String, String>();  
    map.put("name", "test");  
    map.put("age", "1");  
    System.out.println(map.get("name"));  
    System.out.println(map.get("age"));  
}  
```

反编译：

```java
public static void main(String[] args) {  
    Map map = new HashMap();  
    map.put("name", "test");  
    map.put("age", "1"); 
    System.out.println((String) map.get("name"));  
    System.out.println((String) map.get("age"));  
}  
```

可以看到泛型类型都转为了基本类型

### 泛型带来的问题

#### 泛型不可重载
```java
public class GenericTypes {  

    public static void method(List<String> list) {  
        System.out.println("invoke method(List<String> list)");  
    }  

    public static void method(List<Integer> list) {  
        System.out.println("invoke method(List<Integer> list)");  
    }  
}  
```
代码中可以看到，两个重载函数唯一区别是`List<String>` 和 `List<Integer>` 但是在编译后泛型类型都会被擦除变为原始类型，导致两个方法变为一样的方法，所以编译不通过。

#### 泛型的catch

还是由于类型擦除问题，catch多个自定义泛型类的时候可能会重复。

#### 泛型的静态变量

```java
public class StaticTest{
    public static void main(String[] args){
        GT<Integer> gti = new GT<Integer>();
        gti.var=1;
        GT<String> gts = new GT<String>();
        gts.var=2;
        System.out.println(gti.var);
    }
}
class GT<T>{
    public static int var=0;
    public void nothing(T x){}
}
```

答案是——2！由于经过类型擦除，所有的泛型类实例都关联到同一份字节码上，泛型类的所有静态变量是共享的。

### 使用泛型注意点总结

泛型本质就是一个java提供的语法糖，在虚拟机中只有普通类和普通方法，所有泛型类的类型参数在编译的时候都会被擦除

创建泛型对象的时候要指明类型让编译器尽早做参数检查。

所有静态变量是被泛型类的所有实例共享的

泛型的类型参数不能用在catch里

尽量不要使用原始态类型（Map 不指定泛型类型）可能导致ClassCastException

### 使用泛型

限定通配符和非限定通配符

+ <? extends T> ：可以赋值给任意T及T的子类集合，上界为T，取出来的类型带有泛型限制，向上强制转型为T。null 可以表示任何类型，所以null除外，任何元素都不得添加进<? extends T>集合内。
+ <? super T> : 可以复制T及任何T的父类集合，下界为T。再生活中，投票选举类似于<? super T>的操作。选举投票时，你只能往里投票，取数据时，根本不知道时是谁的票，相当于泛型丢失。


```java
public class Food {}
public class Fruit extends Food {}
public class Apple extends Fruit {}
public class Banana extends Fruit{}

public class GenericTest {

    public void testExtends(List<? extends Fruit> list){

        //报错,extends为上界通配符,只能取值,不能放.
        //因为Fruit的子类不只有Apple还有Banana,这里不能确定具体的泛型到底是Apple还是Banana，所以放入任何一种类型都会报错
        //list.add(new Apple());

        //可以正常获取
        Fruit fruit = list.get(1);
    }

    public void testSuper(List<? super Fruit> list){

        //super为下界通配符，可以存放元素，但是也只能存放当前类或者子类的实例，以当前的例子来讲，
        //无法确定Fruit的父类是否只有Food一个(Object是超级父类)
        //因此放入Food的实例编译不通过
        list.add(new Apple());
//        list.add(new Food());

        Object object = list.get(1);
    }
}


public class Test {
    public static void main(String[] args) {
        List<Apple> apples = new LinkedList<>();
        List<Food> foods = new LinkedList<>();
        putFruits(foods);
        eatFruits(apples);
    }

    private static void putFruits(List<? super Fruit> list) {

    }

    private static void eatFruits(List<? extends Fruit> list) {

    }
}

```

`<? extends Fruit>`虽然不能添加元素，但可以在初始化的时候，接受一个已经定义好的list，而该list存放的类型一定相同，因此，List<? extends T>可直接接受一个定义好的list。

`<? super T>`:专门用来存，存的数据只能是本身或者子类，指向的时候只能指向父类

`<? extends T>`: 专门用来消费，得到的类型都是本身，指向的时候只能指向子类

## 异常

自定义异常最好继承Exception

### try-with-resources

一般操作文件io，数据库链接比较耗费资源，用完后应该及时关闭，一般我们是在try catch finally的finally里面调用close方法释放资源。

但是从jdk7开始提供了更好的方法：try-with-resources，新的syntax sugar，在try后面接上对io流的定义即可。

```java
public static void main(String... args) {
    try (BufferedReader br = new BufferedReader(new FileReader("c:\\ test.xml"))) {
        String line;
        while ((line = br.readLine()) != null) {
            System.out.println(line);
        }
    } catch (IOException e) {
        // handle exception
    }
}
```
### finally和return的执行顺序

假如try里面有个return 那么后面的finally会不会执行，什么时候执行？

如果try里有return 那么finally的代码还是会执行，return表示整个方法的返回，所以finally会在return之前执行。

但是return前执行的finally块内，对数据的修改效果对于引用类型和值类型会不同

```java
// 测试 修改值类型
static int f() {
    int ret = 0;
    try {
        return ret;  // 返回 0，finally内的修改效果不起作用
    } finally {
        ret++;
        System.out.println("finally执行");
    }
}

// 测试 修改引用类型
static int[] f2(){
    int[] ret = new int[]{0};
    try {
        return ret;  // 返回 [1]，finally内的修改效果起了作用
    } finally {
        ret[0]++;
        System.out.println("finally执行");
    }
}
```

## 时间

### SimpleDateFormat的线程安全问题

强制：SimpleDateFormat 是线程不安全的，一般不要定义为static变量，如果定义static变量必须加锁或者使用DateUtils工具类。

Date类型转为String并且指定输出格式：

```java
// Date转String
Date data = new Date();
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
String dataStr = sdf.format(data);
System.out.println(dataStr);
```

String 转Date：`sdf.parse(dataStr);`

时区问题：由于存在时区，不同地区时间不同，可以在代码中手动设置时区，获得当地时间，如中国的时间是GTM+8

```java
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
sdf.setTimeZone(TimeZone.getTimeZone("America/Los_Angeles"));
System.out.println(sdf.format(Calendar.getInstance().getTime()));
```

为什么会导致线程不安全？：

因为观察源码可以发现，
![upload successful](/images/pasted-39.png)

calendar.setTime(date)是没有线程安全保证的，所以当设置SimpleDateForat为静态变量时，所有子线程都可以访问，可能A线程刚刚设置完calendar.setTime(date)就被B线程再调用修改时间了。

如何解决？

1. 使用局部变量。

2. 或者一定要将SimpleDateFormat作为共享变量的话，加同步锁。
`synchronized(simpleDateFormat){...`

3. 或者使用使用ThreadLocal

```java
/**
 * 使用ThreadLocal定义一个全局的SimpleDateFormat
 */
private static ThreadLocal<SimpleDateFormat> simpleDateFormatThreadLocal = new ThreadLocal<SimpleDateFormat>() {
    @Override
    protected SimpleDateFormat initialValue() {
        return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    }
};

//用法
String dateString = simpleDateFormatThreadLocal.get().format(calendar.getTime());
```

如果是Java8. 换成DateTimeFormatter即可，线程安全。

### 为什么选择ThreadLocal

ThreadLocal使同一变量在每一个线程中有各自的副本，不就意味着这个变量是不共享数据的，那不共享数据的话，为什么我不把这个变量变成自定义线程类的成员域，如果可以，ThreadLocal类的作用是啥?

Spring采用Threadlocal的方式，来保证单个线程中的数据库操作使用的是同一个数据库连接，同时，采用这种方式可以使业务层使用事务时不需要感知并管理connection对象，通过传播级别，巧妙地管理多个事务配置之间的切换，挂起和恢复。

其实本质是将变量的值都存在thread中，为什么不将变量定义为线程的成员变量呢，还是因为资源节省，每个线程都用成员变量而不用单例的话会造成大量浪费。而且假如有链路的话，需要共享一些变量比如用户信息，这时候如果定义成线程成员变量，那么不同线程请求得将信息传递过去，但是如果使用静态变量那么直接import后不需要传参直接threadlocal获取。

### Java8时间处理

在Java8中， 新的时间及⽇期API位于java.time包中， 该包中有哪些重要的类。 分别代表了什么？

Instant： 时间戳

Duration： 持续时间， 时间差

LocalDate： 只包含⽇期， ⽐如： 2016-10-20

LocalTime： 只包含时间， ⽐如： 231210

LocalDateTime： 包含⽇期和时间， ⽐如： 2016-10-20 231421

Period： 时间段

ZoneOffset： 时区偏移量， ⽐如： +8:00

ZonedDateTime： 带时区的时间

Clock： 时钟， ⽐如获取⽬前美国纽约的时间

新的java.time包涵盖了所有处理日期，时间，日期/时间，时区，时刻（instants），过程（during）与时钟（clock）的操作。

#### 获取当前时间

```java
LocalDate today = LocalDate.now();
int year = today.getYear();
int month = today.getMonthValue();
int day = today.getDayOfMonth();
System.out.printf("Year : %d Month : %d day : %d t %n", year,month, day);
```

#### 创建指定日期

`LocalDate date = LocalDate.of(2018, 01, 01);`

#### 格式化时间+时间戳转换
```java
        LocalDateTime localDateTime = Instant.ofEpochMilli(1617257935413L).atZone(ZoneOffset.ofHours(8)).toLocalDateTime();
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        String time = localDateTime.format(formatter);
        System.out.println(time);
        System.out.println(LocalDateTime.of(2021,3,11,0,0).toInstant(ZoneOffset.ofHours(8)).toEpochMilli());

```

#### 闰年判断

```java
LocalDate nowDate = LocalDate.now();
//判断闰年
boolean leapYear = nowDate.isLeapYear();
```

#### 计算日期间天数和月数

`Period period = Period.between(LocalDate.of(2018, 1, 5),LocalDate.of(2018, 2, 5));
`