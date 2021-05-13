title: java 基础知识点（查漏补缺）
author: TR
tags:
  - Java
  - Lambda
categories:
  - Java
date: 2019-07-25 11:02:00
---
### java 复习（heard first java）

最近复习java基础知识:
* 位运算和基础盲点
* Lambda
* stream
<!--more-->

#### 基础知识点

1. 实例变量与局部变量：实例变量未被初始化可以通过编译，存在默认值，false，0，null。局部变量不初始化无法通过编译。

2. 进制转换：Integer.parseInt("1000",2) :1000转为二进制：8

3. 位异或运算（^）运算规则是：两个数转为二进制，然后从高位开始比较，如果相同则为0，不相同则为1。

	比如：8^11.8转为二进制是1000，11转为二进制是1011.从高位开始比较得到的是：0011.然后二进制转为十进制，就是Integer.parseInt("0011",2)=3;

4. 位与运算符（&）运算规则：两个数都转为二进制，然后从高位开始比较，如果两个数都为1则为1，否则为0。

	比如：129&128.
    129转换成二进制就是10000001，128转换成二进制就是10000000。从高位开始比较得到，得到10000000，即128.、
    
    `注意：&是位运算符，不是逻辑运算符，&&具有短路功能，在第一个条件不符合的情况下之后不会执行。
    但是&比较特殊，计算机中true是用二进制标识，所以可以进行与运算，使用&作为逻辑运算本质还是位运算，等所有条件运行后将结果与。（1>2 & 2>3) ==> (true & true)
    `
    
5. 位或运算符（|）运算规则：两个数都转为二进制，然后从高位开始比较，两个数只要有一个为1则为1，否则就为0。

	比如：129|128.
    
    129转换成二进制就是10000001，128转换成二进制就是10000000。从高位开始比较得到，得到10000001，即129.
    
    `注意：可作非短路，原因同上`
   
6. 位非运算符（~）运算规则：如果位为0，结果是1，如果位为1，结果是0.

	比如：~37
    
    在Java中，所有数据的表示方法都是以补码的形式表示，如果没有特殊说明，Java中的数据类型默认是int,int数据类型的长度是8位，一位是四个字节，就是32字节，32bit
    
7. 集合的add和addall方法区别：add添加一个集合相当于添加对应集合的指针，addall方法在添加一个集合元素时，会将这个集合里面的元素添加进去。 

##### 反射机制

java文件被编译生成字节码后并不是直接进入内存，需要类加载器（ClassLoader加载），这个加载器引导类进入内存变为类对象，并且将类中的所有区域变为对象，比如成员变量变为`Filed\[] field`,构造方法变为`Constractor\[] cons`对象数组，成员方法变为`Method\[] methods`。

那么反射就是可以直接操控类对象，获取Method里面的方法或者直接操作Filed内变量。比如IDE的代码自动提示，定义一个类`Person a`后，输入a会自动弹出后面的方法，这是因为IDE自动加载了`Person`类的字节码进入内存，获取里面的`Method`数组内的方法，从而展示在了弹窗中。

反射最大的好处是解耦。

#### java8

1. 内存中：方法区，栈，堆，实际上方法区存在于堆内的永久区，永久区也会被gc回收，只不过比较困难，gc工作在堆，但是8后jvm都取消了永久区
2. 1.7到8的升级点--concurrentHashMap：不在固定段16（因为不好确认多大，现在已经变为链表和红黑树），内部hash算法也变为了底层os支持的cas算法，快很多了

##### Lambda

3. 匿名函数，让函数代码可以作为参数传递

4. 在写lambda时需要函数式接口
	``` java
    
    接口类：
    @FunctionalInterface
    //双参数
    public interface MyFunctionTwo<T,R> {
        R calc(T t1, T t2);
    }
    
   ```
   ```java
   public class test {
    @Test
    public void test1() {
        int num = 10;
//        MyFunction myFunction = (x)->(Integer) x*(Integer)x;
        Integer operate = operate(num, x ->  x * x);
        System.out.println(operate);
    }

    @Test
    public void test2() {
        System.out.println(operate((long)19, (long)12, (x1, x2) -> x1 + x2));
    }

    private Integer operate(Integer num, MyFunction<Integer> myFunction) {
        return myFunction.calc(num);
    }

    private Long operate(Long num1, Long num2, MyFunctionTwo<Long, Long> myFunctionTwo) {
        return myFunctionTwo.calc(num1, num2);
    }

   ```
   
5. 现在可以直接使用java内置的四个基本接口，如有特殊需要再自定义接口

``` java

import org.junit.Test;

import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Supplier;

/**
 * java8 四大内置核心函数接口
 *
 * Consumer<T> 消费性接口
 *      void accept(T t)
 * Supplier<T> 供给型接口
 *      T get();
 * function<T,R> 函数型接口
 *      R apply(T t)
 * Predicate<T> 断言
 *      boolean test(T t);
 *
 */
public class Examples {

    @Test
    public void testConsumer() {
        LogConsumer("hello consumer", x-> System.out.println("log:"+x));
    }

    private void LogConsumer(String message, Consumer<String> consumer) {
        consumer.accept(message);
    }

    @Test
    public void testSupplier() {
        Supplier<Integer> supplier = ()->(int)(Math.random()*100);
        System.out.println(supplier.get());
    }

    @Test
    public void testFunction() {
        Function<String, Integer> function = (x) -> Integer.parseInt(x);
        System.out.println(function.apply("11"));
        }

}

```

* 需要注意的是：匿名内部类不可以直接修改外部数据，但是可以操作对象调用其修改方法！

* 如果4个基本接口不够，可以试试看子类接口（提供更多参数）API为Bi+4接口

##### REF
1. 配合lambda的方法，若lambda的内容已经被实现可以引用

2. 使用示例

```java

import java.util.Comparator;
import java.util.function.*;

/**
 * 1.方法引用：三种引用格式
 * 对象::实例
 * <p>
 * 类::静态方法
 * <p>
 * 类::实例方法
 * 2. 构造器引用
 * ClassName::new
 * <p>
 * 3. 数组引用
 * Type::new
 */
public class refTest {


    // 对象::实例
    @Test
    public void test1() {
        Consumer<String> consumer = x -> System.out.println(x);
        //System.out.println实际被实现了，可以引用
        //注意：引用的时候，被引用方法/实例 的参数和返回值要和接口一致
        //这里自动将入参给System.out.print(),返回值和Consumer一样为void
        consumer = System.out::println;

    }

    @Test
    public void test2() {
        Employee employee = new Employee();
        int test = 1;
        Supplier<String> supplier = () -> {
            employee.setName("tst");
            return employee.getName();
        };
        System.out.println(supplier.get());
        //REF
        employee.setAge(8);
        Supplier<Integer> supplierRef = employee::getAge;
        System.out.println(supplierRef.get());
    }

    //类::静态方法名
    @Test
    public void test3() {
        Comparator<Integer> com = (x, y) -> Integer.compare(x, y);
        //REF
        com = Integer::compare;
    }

    //类::实例方法
    @Test
    public void test4() {
        //注意：这里的方法体也被实现了，String的equals方法，但这是一个实例对象方法
        BiPredicate<String, String> predicate = (x, y) -> x.equals(y);
        //REF
        // java规定，如果第一个参数是调用函数者，第二个是被调者，可以用类名::方法名
        predicate = String::equals;

    }

    // 构造器引用
    @Test
    public void test5() {
        Supplier<Employee> supplier = () -> new Employee();
        //REF  无参数构造
        supplier = Employee::new;

        //REF 有参数构造
        //这里由于BiFunction接受2参数，返回一个，而Employee三个参数，所以手动写一个参数
        BiFunction<String, Integer, Employee> bf = (x, y) -> new Employee(x, y, 10);
        //REF 调用几个参数的构造器取决于函数接口的参数个数
        bf = Employee::new;//调用2参数构造器
        Employee tr = bf.apply("tr", 23);
        System.out.println(tr.toString());

    }

    //数组引用
    @Test
    public void test6() {
        Function<Integer, String[]> fun = x -> new String[x];
        String[] apply = fun.apply(3);
        //REF  和构造器引用一致
        fun = String[]::new;
        fun.apply(10);
    }
}


```

#### Stream 操作

1. java8的stream分为：创建steam，中间操作，终端操作。
实质是将数据源转为流，对流进行一系列操作后生成结果。

2. 值得注意的是：只是转为stream没有任何意义，只有对stream进行终端操作，jvm才会执行stream中间操作。

3. 首先是生成stream，最常用的应该是collection系列集合的stream方法吧

##### 创建stream

```java

 // 创建stream
    @Test
    public void test1() {
//        1. 可以通过collection系列集合提供的steam（）或者 parallelStream（）获取流，前者串行流，后者并行流
        List<String> list = new ArrayList<String>();
        Stream<String> stream = list.stream();
//        2. 通过Arrays中的静态方法stream（）获取数组流
        Employee[] array = new Employee[3];
        Stream<Employee> stream1 = Arrays.stream(array);
//        3. 通过stream中的of方法创建流
        Stream<String> stream2 = Stream.of("a", "bb", "ccc");
//        4. 创建无限流
        //迭代
        Stream<Integer> stream3 = Stream.iterate(0, (x) -> x + 2);
//        stream3.forEach(System.out::println);
        // stream只有在被终端操作的时候才会执行
        stream3.limit(3).forEach(System.out::println);
        // 生成
        Stream<Double> stream4 = Stream.generate(() -> Math.random());
        stream4.limit(10).forEach(System.out::println);
    }



```

##### 流操作

对stream流的一些常用操作(流操作)：`skip` `filter` `limit` `map` `flatmap` `distinct` `sorted`

``` java


/**
 * 获取stream后的操作
 * 1. filter 筛选
 * 2. limit 截断
 * 3. skip 跳过
 * 4. distinct 去重
 */
public class SecondOperateStream {

    List<Employee> employees = Arrays.asList(
            new Employee("test1", 18, 999.9),
            new Employee("test2", 28, 999.9),
            new Employee("test3", 38, 999.9),
            new Employee("test4", 48, 999.9),
            new Employee("test4", 48, 999.9),
            new Employee("test4", 48, 999.9)
    );

    @Test
    public void test1() {
        // limit
        // 中间操作：不执行
        Stream<Employee> employeeStream = employees.stream()
                .filter((e) -> e.getAge() > 28)
                .limit(2);// 一旦数据量到达，不再执行后续操作：短路
        //终端操作：一次性执行全部内容，惰性求值
        employeeStream.forEach(System.out::println);
    }

    @Test
    public void test2() {
        //skip
        employees.stream()
                .filter((e) -> e.getAge() > 0)
                .skip(1)
                .distinct() // 注意 去重的对象要重写hashcode和equals，看employee类实现
                .forEach(System.out::println);
    }

    //Map操作：将函数作用于每个元素输出
    @Test
    public void test3() {
        List<String> list = Arrays.asList("111", "bbb", "ccc");
        list.stream().map((x) -> x.toUpperCase())
                .forEach(System.out::println);
        employees.stream().map(Employee::getName)
                .forEach(System.out::println);
    }

    //flatMap操作:将流中的每个元素转为流，再将流拼接
    @Test
    public void test4() {
        List<String> list = Arrays.asList("111", "bbb", "ccc");
//        Stream<char[]> stream = list.stream().flatMap((ele) -> List.of(ele.toCharArray()).stream());
        Stream<Character> stream = list.stream()
                .flatMap((ele) -> {
                    List<Character> temp = new ArrayList<>();
                    for (Character c : ele.toCharArray()) temp.add(c);
                    return temp.stream();
                });
        stream.forEach(System.out::println);

    }

    //排序
    //sorted()自然排序
    //sorted(Comparator com)定制排序
    @Test
    public void test5() {

        //自然排序
        List<String> list = Arrays.asList("aaa", "bbb", "ccc");
        list.stream().sorted().forEach(System.out::println);

        //定制排序
        employees.stream().sorted((x1,x2)->{
            if (x1.getAge()==x2.getAge())
                return x1.getName().compareTo(x2.getName());
            return x2.getAge() - x1.getAge();
        }).forEach(System.out::println);
    }

}



```


##### 终止操作

查找匹配，规约收集等

```java

package com.Java8.stream;

import com.Java8.Employee;
import com.Java8.Status;
import org.junit.Test;

import java.util.*;
import java.util.stream.Collectors;

/**
 * @author tr
 * @date 2020/4/14 10:55
 * <p>
 * 对stream的终止操作
 * <p>
 * 查找与匹配
 * allMatch--检查是否匹配所有元素
 * anyMatch--检查是否少匹配一个元素
 * noneMatch--检查是否没有匹配所有元素
 * findFirst--返回第一个元素
 * findAny--返回当前流中的任意元素
 * count--返回流中的元素总数
 * max--返回流中的最大值
 * min--返回流中最小值
 * 规约，收集
 */
public class TerminateOperateStream {

    List<Employee> employees = Arrays.asList(
            new Employee("test1", 18, 919.9, Status.FREE),
            new Employee("test2", 17, 299.9, Status.BUSY),
            new Employee("test3", 28, 599.9, Status.VOCATION),
            new Employee("test4", 38, 1999.9, Status.FREE),
            new Employee("test5", 58, 599.9, Status.BUSY),
            new Employee("test6", 48, 499.9, Status.VOCATION)
    );

    /**
     * .......
     */
    @Test
    public void test1() {
        // 是否所有人都忙  检查匹配所有元素
        boolean ifAllBusy = employees.stream()
                .allMatch((employee -> employee.getStatus().equals(Status.BUSY)));
        System.out.println(ifAllBusy);
        // 是否有人忙
        boolean ifAnyBusy = employees.stream().anyMatch((employee -> employee.getStatus().equals(Status.BUSY)));
        System.out.println(ifAnyBusy);

        //是否没人忙
        boolean noneBusy = employees.stream().noneMatch(employee -> employee.getStatus().equals(Status.BUSY));
        System.out.println(noneBusy);

        //第一个  工资排序
        // Optional 容器类，内部不为空，若为空有替代
        Optional<Employee> first = employees.stream().sorted(((t1, t2) -> Double.compare(t1.getSalary(), t2.getSalary())))
                .findFirst();
        System.out.println(first.get());

        // 随便找个空闲的人
        Optional<Employee> any = employees.stream().filter(employee -> employee.getStatus().equals(Status.FREE))
                .findAny();
        System.out.println(any.get());

    }

    @Test
    public void test2() {
        //count
        long count = employees.stream().count();
        System.out.println(count);
        //max
        Optional<Employee> max = employees.stream().max((t1, t2) -> Double.compare(t1.getSalary(), t2.getSalary()));
        System.out.println(max.get());
        //min
        Optional<Double> min = employees.stream().map(Employee::getSalary)
//                .min((t1, t2) -> Double.compare(t1, t2));
                .min(Double::compare);
        System.out.println(min.get());
    }

    /**
     * 规约  map  reduce
     */
    @Test
    public void test3() {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        // reduce  先结合0作为x
        Integer sum = list.stream()
                .reduce(0, (x, y) -> x + y);
        System.out.println(sum);

        //获取所有员工工资总和
        Optional<Double> salarySum = employees.stream()
                .map(Employee::getSalary)
//                .reduce((x, y) -> x + y);
                .reduce(Double::sum);
        System.out.println(salarySum.get());
    }

    /**
     * 收集 collect
     */
    @Test
    public void test4() {
        // 提取所有员工名字 放入集合
        List<String> nameList = employees.stream().map(Employee::getName)
                .collect(Collectors.toList());
        nameList.forEach(System.out::println);
//        System.out.println(nameList);

        //放入自定义Collection集合
        HashSet<Integer> collect = employees.stream()
                .map(Employee::getAge)
                .collect(Collectors.toCollection(HashSet::new));
    }

    /**
     * collectors
     * 平均值
     * 和
     * 最大最小
     */
    @Test
    public void test5() {
        //收集方法的 计算总数  个数！方法
        Long collect = employees.stream()
                .collect(Collectors.counting());

        // 工资平均值
//        Double collect1 = employees.stream()
//                .map(Employee::getSalary)
//                .collect(Collectors.averagingDouble((x) -> Double.valueOf(x)));
        Double collect1 = employees.stream().collect(Collectors.averagingDouble(Employee::getSalary));
        System.out.println(collect1);


        // 工资总和
        Double collect2 = employees.stream().collect(Collectors.summingDouble(Employee::getSalary));
        System.out.println(collect2);

        // 最大值
        Optional<Employee> collect3 = employees.stream().collect(Collectors.maxBy((t1, t2) -> Double.compare(t1.getAge(), t2.getAge())));
        System.out.println(collect3.get());
    }

    /**
     * 分组
     */
    @Test
    public void test6() {
        // 按状态分组
        Map<Status, List<Employee>> collect = employees.stream()
                .collect(Collectors.groupingBy(Employee::getStatus));
        System.out.println(collect);
        // 多列分组 ，按状态分完按年龄，20以下一组，20-40一组 40以上一组
        Map<Status, Map<String, List<Employee>>> collect1 = employees.stream()
                .collect(Collectors.groupingBy(Employee::getStatus, Collectors.groupingBy((employee)->{
                    if (employee.getAge()<=20) return "青年";
                    else if (employee.getAge()<=40&&employee.getAge()>20) return "中年";
                    else return "老年";
                })));
        System.out.println(collect1);
    }

    /**
     * 分区
     * 分区 满足条件的一个区，不满足的一个区
     */
    @Test
    public void test7() {
        // 满足工资大于800的一个区
        Map<Boolean, List<Employee>> collect = employees.stream()
                .collect(Collectors.partitioningBy((employee -> employee.getSalary() > 800)));
        System.out.println(collect);
    }

    /**
     *  collectors除了以上方法，还有很多其他方法获得平均值，最大最小
     */
    @Test
    public void test8() {

        DoubleSummaryStatistics collect = employees.stream()
                .collect(Collectors.summarizingDouble(Employee::getAge));
        System.out.println(collect.getAverage());
        System.out.println(collect.getMax());
        System.out.println(collect.getMin());
    }

    /**
     * join
     * 连接字符串
     */
    @Test
    public void test9() {
        String collect = employees.stream()
                .map(Employee::getName)
                .collect(Collectors.joining(",","-","="));
        System.out.println(collect);
    }
}


```

#### java8的optional

optional用于快速定位或者防止空指针

```java

package com.Java8.Optional;

import com.Java8.Employee;
import org.junit.Test;

import java.util.Optional;

/**
 * @author tr
 * @date 2020/4/14 17:14
 *
 * optional  用于避免空指针
 *
 * optional 容器类的常用方法
 * Optional.of(T t) 创建一个optional实例
 * Optional.empty() 创建一个空的实例
 * Optional.ofNullable(T t) 如果t不为空null，创建option实例，否则创建空实例
 * isPresent 判断是否包含值
 * orElse(T t) 如果对象包含实例，返回该值，否则返回t
 * orElseGet(supplier s) 如果对象包含值，返回，否则返回s获取的值
 * map(Function f) 如果有值对其处理，并返回处理后的 Optional 否则返回Optional.empty()
 * flatMap(Function mapper) 类似mapper 要求返回值为Optional
 *
 */
public class TestOptional {
    @Test
    public void test1() {
        // 如果of里面传入的null 立刻发送异常，用于快速定位问题
        Optional<Employee> op = Optional.of(new Employee());

        Employee employee1 = op.get();
        System.out.println(employee1);

        // 如果一定要optional内为null
        Optional<Employee> empty = Optional.empty();
        // null get不到值
        System.out.println(empty.get());

        // 构建空optional的另一个方法
        Optional<Employee> empty2 = Optional.ofNullable(null);
        System.out.println(empty2.get());

        //判断是否存在值
        if(empty2.isPresent()) System.out.println(empty.get());

        // 没值代替一个
        Employee employee2 = op.orElse(new Employee());

        // flatMap 要求返回用optiona容器包装
        Optional<String> s = op.flatMap(x -> Optional.of(x.getName()));
        System.out.println(s);
    }
}


```

#### java8 接口新特性

新增了接口的静态方法和default修饰方法。

现在可以接口中实现默认方法，若一个类同时继承了某个同名方法类，且implements了这个接口，那么子类调用时，类优先。

若两个接口都定义了getName()那么子类实现这两个接口的话会报错要求实现类必须实现方法，如果extends了一个类又实现了同名方法的接口，extends的类优先。

```java
package com.Java8.TheInterface;
/**
 * @author tr
 * @date 2020/4/14 18:16
 *
 * java8 支持接口中实现默认方法
 *
 */
public interface IMyFun {
    default String getName() {
        return "hello";
    }
    public static void show(){
    	System.out.println("接口的静态方法");
    }
}
```

#### Java的新的时间类

