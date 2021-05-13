title: Java 工程师之路 五
author: tr
tags:
  - Java
categories:
  - Java
date: 2021-03-24 16:34:00
---
# 并发编程

<!--more-->

## 并发和并行

并发：即OS中学到的分时操作系统，单cpu采取不同的cpu调度算法也可以让不同任务"同时"进行。

并行：两个CPU同时执行进程

进程切换的原理请看OS篇

进程是OS分配资源的基本单位，线程是基本执行单位。同一进程下的不同线程共享进程资源。

JVM就是一个进程，多个线程共享JVM资源。且多个线程可以并发执行。


## 线程的特点

### 线程实体

线程中的实体基本上不拥有系统资源，只是有一点必不可少的、能保证独立运行的资源。 线程的实体包括程序、数据和TCB。线程是动态概念，它的动态特性由线程控制块TCB（Thread Control Block）描述。TCB包括以下信息： （1）线程状态。 （2）当线程不运行时，被保存的现场资源。 （3）一组执行堆栈。 （4）存放每个线程的局部变量主存区。 （5）访问同一个进程中的主存和其它资源。 用于指示被执行指令序列的程序计数器、保留局部变量、少数状态参数和返回地址等的一组寄存器和堆栈。

### 共享资源

在同一进程中的各个线程，都可以共享该进程所拥有的资源，这首先表现在：所有线程都具有相同的地址空间（进程的地址空间），这意味着，线程可以访问该地址空间的每一个虚地址；此外，还可以访问进程所拥有的已打开文件、定时器、信号量机构等。由于同一个进程内的线程共享内存和文件，所以线程之间互相通信不必调用内核。

## 线程实现模型

即OS中的线程模型概念，内核与用户级线程的对应关系。

### 使用内核线程实现

内核线程即OS内核支持的线程，这种线程切换是由OS来操作的。OS通过调度器对线程进行调度，将线程的任务映射到各个CPU上。每个内核线程可以视为内核的分身，支持多线程的内核就叫多线程内核。

程序一般不适用内核线程，而是用内核线程的一种高级接口：轻量级进程（也就是我们通常指的线程），通常一个内核线程对应一个轻量级进程1：1的情况叫做一对一线程模型。

![upload successful](/images/pasted-40.png)

由于内核线程的支持，即使有一个轻量级进程被阻塞了，也不会影响整个进程。但缺点是：使用内核线程实现轻量级线程导致各种线程操作（创建，同步）会用到系统调用，需要从用户态切换到内核态，浪费资源，且一个轻量级进程都需要一个内核线程支持，数量有限。

### 用户线程实现

广义上：如果一个线程只要不是内核线程都可以认为是用户线程

侠义上：用户线程完全建立在用户空间的线程库，系统内核无法感知线程的存在，线程的操作都在用户态，这种1：N的关系叫做一对多线程模型。

使用用户线程的优势在于不需要系统内核支援，劣势也在于没有系统内核的支援，所有的线程操作都需要用户程序自己处理。线程的创建、切换和调度都是需要考虑的问题，而且由于操作系统只把处理器资源分配到进程，那诸如“阻塞如何处理”、“多处理器系统中如何将线程映射到其他处理器上”这类问题解决起来将会异常困难，甚至不可能完成。因而使用用户线程实现的程序一般都比较复杂 ，除了以前在不支持多线程的操作系统中（如DOS）的多线程程序与少数有特殊需求的程序外，现在使用用户线程的程序越来越少了，Java、Ruby等语言都曾经使用过用户线程，最终又都放弃使用它。

### 内核线程和用户线程的混合实现

线程除了依赖内核线程实现和完全由用户程序自己实现之外，还有一种将内核线程与用户线程一起使用的实现方式。在这种混合实现下，既存在用户线程，也存在轻量级进程。用户线程还是完全建立在用户空间中，因此用户线程的创建、切换、析构等操作依然廉价，并且可以支持大规模的用户线程并发。而操作系统提供支持的轻量级进程则作为用户线程和内核线程之间的桥梁，这样可以使用内核提供的线程调度功能及处理器映射，并且用户线程的系统调用要通过轻量级线程来完成，大大降低了整个进程被完全阻塞的风险。在这种混合模式中，用户线程与轻量级进程的数量比是不定的，即为N：M的关系，如图12-5所示，这种就是多对多的线程模型。

许多UNIX系列的操作系统，如Solaris、HP-UX等都提供了N：M的线程模型实现。

![upload successful](/images/pasted-41.png)


### 线程的6态转换

具体看OS篇

### Java设置线程优先级

Java虚拟机采用抢占式调度模型。也就是说他会给优先级更高的线程优先分配CPU。

虽然Java线程调度是系统自动完成的，但是我们还是可以“建议”系统给某些线程多分配一点执行时间，另外的一些线程则可以少分配一点——这项操作可以通过设置线程优先级来完成。

Java语言一共设置了10个级别的线程优先级（Thread.MIN_PRIORITY至Thread.MAX_PRIORITY），在两个线程同时处于Ready状态时，优先级越高的线程越容易被系统选择执行。

Java 线程优先级使用 1 ~ 10 的整数表示。默认的优先级是5。

> 最低优先级 1：Thread.MIN_PRIORITY

> 最高优先级 10：Thread.MAX_PRIORITY

> 普通优先级 5：Thread.NORM_PRIORITY

在Java中，可以使用Thread类的setPriority()方法为线程设置了新的优先级。getPriority()方法返回线程的当前优先级。当创建一个线程时，其默认优先级是创建该线程的线程的优先级。

以下代码演示如何设置和获取线程的优先：
```java
package threadPriority;

public class Main {
    public static void main(String[] args) {
        Thread t = Thread.currentThread();
        System.out.println("Main Thread  Priority:" + t.getPriority());

        Thread t1 = new Thread();
        System.out.println("Thread(t1) Priority:" + t1.getPriority());
        t1.setPriority(Thread.MAX_PRIORITY - 1);
        System.out.println("Thread(t1) Priority:" + t1.getPriority());

        t.setPriority(Thread.NORM_PRIORITY);
        System.out.println("Main Thread  Priority:" + t.getPriority());

        Thread t2 = new Thread();
        System.out.println("Thread(t2) Priority:" + t2.getPriority());

        // Change thread t2 priority to minimum
        t2.setPriority(Thread.MIN_PRIORITY);
        System.out.println("Thread(t2) Priority:" + t2.getPriority());

    }
}

```
在上面的代码中，Java虚拟机启动时，就会通过main方法启动一个线程，JVM就会一直运行下去，直到以下任意一个条件发生：

调用了exit()方法，并且exit()有权限被正常执行。
所有的“非守护线程”都死了(即JVM中仅仅只有“守护线程”)。

### 线程调度

Linux线程调度

在Linux中，线程是由进程来实现，线程就是轻量级进程（ lightweight process ），因此在Linux中，线程的调度是按照进程的调度方式来进行调度的，也就是说线程是调度单元。

Linux这样实现的线程的好处的之一是：线程调度直接使用进程调度就可以了，没必要再搞一个进程内的线程调度器。在Linux中，调度器是基于线程的调度策略（scheduling policy）和静态调度优先级（static scheduling priority）来决定那个线程来运行。

在Linux中，主要有三种调度策略。分别是：

SCHED_OTHER 分时调度策略，（默认的）

SCHED_FIFO 实时调度策略，先到先服务

SCHED_RR 实时调度策略，时间片轮转

Windows线程调度：

Windows 采用基于优先级的、抢占调度算法来调度线程。

用于处理调度的 Windows 内核部分称为调度程序，Windows 调度程序确保具有最高优先级的线程总是在运行的。由于调度程序选择运行的线程会一直运行，直到被更高优先级的线程所抢占，或终止，或时间片已到，或调用阻塞系统调用（如 I/O）。如果在低优先级线程运行时，更高优先级的实时线程变成就绪，那么低优先级线程就被抢占。这种抢占使得实时线程在需要使用 CPU 时优先得到使用。

Java线程调度：

可以看到，不同的操作系统，有不同的线程调度策略。但是，作为一个Java开发人员来说，我们日常开发过程中一般很少关注操作系统层面的东西。

主要是因为Java程序都是运行在Java虚拟机上面的，而虚拟机帮我们屏蔽了操作系统的差异，所以我们说Java是一个跨平台语言。

在操作系统中，一个Java程序其实就是一个进程。所以，我们说Java是单进程、多线程的！

前面关于线程的实现也介绍过，Thread类与大部分的Java API有显著的差别，它的所有关键方法都是声明为Native的，也就是说，他需要根据不同的操作系统有不同的实现。

在Java的多线程程序中，为保证所有线程的执行能按照一定的规则执行，JVM实现了一个线程调度器，它定义了线程调度模型，对于CPU运算的分配都进行了规定，按照这些特定的机制为多个线程分配CPU的使用权。

主要有两种调度模型：协同式线程调度和抢占式调度模型。

#### 协同式线程调度

协同式调度的多线程系统，线程的执行时间由线程本身来控制，线程把自己的工作执行完了之后，要主动通知系统切换到另外一个线程上。协同式多线程的最大好处是实现简单，而且由于线程要把自己的事情干完后才会进行线程切换，切换操作对线程自己是可知的，所以没有什么线程同步的问题。

#### 抢占式调度

抢占式调度的多线程系统，那么每个线程将由系统来分配执行时间，线程的切换不由线程本身来决定。在这种实现线程调度的方式下，线程的执行时间是系统可控的，也不会有一个线程导致整个进程阻塞的问题。

系统会让可运行池中优先级高的线程占用CPU，如果可运行池中的线程优先级相同，那么就随机选择一个线程，使其占用CPU。处于运行状态的线程会一直运行，直至它不得不放弃CPU。

Java虚拟机采用抢占式调度模型。

虽然Java线程调度是系统自动完成的，但是我们还是可以“建议”系统给某些线程多分配一点执行时间，另外的一些线程则可以少分配一点——这项操作可以通过设置线程优先级来完成。Java语言一共设置了10个级别的线程优先级（Thread.MIN_PRIORITY至Thread.MAX_PRIORITY），在两个线程同时处于Ready状态时，优先级越高的线程越容易被系统选择执行。

不过，线程优先级并不是太靠谱，原因是Java的线程是通过映射到系统的原生线程上来实现的，所以线程调度最终还是取决于操作系统，虽然现在很多操作系统都提供线程优先级的概念，但是并不见得能与Java线程的优先级一一对应。

### debug 多线程代码

IDEA的断点设置里，右击选择Thread

![upload successful](/images/pasted-42.png)

执行代码时就会进入每一个线程。

### 守护线程

Java中有两类线程：UserThread用户线程、DaemonThread守护线程。用户线程执行用户级任务，守护线程就是后台线程，用来执行后台任务，比如GC 垃圾回收器

这两种线程其实是没有什么区别的，唯一的区别就是Java虚拟机在所有“用户线程”都结束后就会退出。也就是如果JVM只剩下了守护线程那么JVM就会退出。

我们可以通过使用setDaemon()方法通过传递true作为参数，使线程成为一个守护线程。我们必须在启动线程之前调用一个线程的setDaemon()方法。否则，就会抛出一个java.lang.IllegalThreadStateException。

可以使用isDaemon()方法来检查线程是否是守护线程。

```java
public class Main {
    public static void main(String[] args) {

        Thread t1 = new Thread();
        System.out.println(t1.isDaemon()); // false
        t1.setDaemon(true);
        System.out.println(t1.isDaemon());// true
        t1.start();
        t1.setDaemon(false);// 抛出异常，JVM已退出
    }
}
```
代码中设置了子线程为守护线程，所以当代码执行后只有子线程，JVM退出，无法设置。

## 如何创建线程


### 继承Thread类创建线程

继承Thread类并且重写run方法即可。注意的是：在主线程中，调用了子线程的Start()方法后，主线程是不会等待而是继续执行。也可以不调用Start()方法而直接调用run方法，这时候的run方法不过是个普通方法。

```java
class SubClassThread extends Thread{
	@Override
    public void run(){
    	System.out.println(getName());
    }
}
```

### 实现Runnable接口

```java
class MyThread implements Runnable{
	@Override
    public void run(){
	System.out.println(Thread.currentThread().getName());
    }
}
```

一般用这个方法，继承的话只能继承一个，接口就无所谓了，两种方式创建的线程没区别基本。

但是这两种方式创建的线程是无法在执行完后返回值给主线程的。

如果想要获取子线程的值，要用Callable和FutureTask

### Callable & FutureTask

这两是JDK1.5之后的

```java
package thread;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class MultiThreads {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyCallableThread myCallableThread = new MyCallableThread();
        FutureTask futureTask = new FutureTask<>(myCallableThread);
        new Thread(futureTask).start();
        System.out.println(futureTask.get());
    }
}

class MyCallableThread implements Callable {

    @Override
    public Object call() throws Exception {
        System.out.println(Thread.currentThread().getName());
        for (int i = 0; i < 30; i++) {
            System.out.println("doing something");
            Thread.sleep(100);
        }
        return "my callable thread";
    }
}
```


![upload successful](/images/pasted-43.png)


FutureTask可用于异步获取执行结果或取消执行。FutureTask非常适合用于耗时的计算，主线程可以在启动FutureTask后去做其他事，在其余事情做完后调用get方法获取结果。

调用get方法会阻塞主线程，直到子线程返回值，可以通过isDone()来判断子线程是否执行完成。

可以改造代码如下：

```java
 public static void main(String[] args) throws InterruptedException {
        CallableThread callableThread = new CallableThread();
        FutureTask futureTask = new FutureTask<>(callableThread);
        new Thread(futureTask).start();

        System.out.println("主线程先做其他重要的事情");
        if(!futureTask.isDone()){
            // 继续做其他事儿
        }
        System.out.println(future.get()); // 可能会阻塞等待结果

```

一般会将Callable放入线程池，让线程池执行Callable的代码。手动new Thread(futureTask)还是有开销的。

### 线程池创建线程

Java提供的线程池创建线程方式，调用ThreadPoolExecutor即可

```java
  ExecutorService executorService = new ThreadPoolExecutor(1, 1, 60L, TimeUnit.SECONDS,
            new ArrayBlockingQueue<Runnable>(10));
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName());
            }
        });
```

线程池本质是个HashSet，多余的任务存入阻塞队列中

#### 线程池的主要参数

```java
public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), defaultHandler);
}
```
1、corePoolSize（线程池基本大小）：当向线程池提交一个任务时，若线程池已创建的线程数小于corePoolSize，即便此时存在空闲线程，也会通过创建一个新线程来执行该任务，直到已创建的线程数大于或等于corePoolSize时，（除了利用提交新任务来创建和启动线程（按需构造），也可以通过 prestartCoreThread() 或 prestartAllCoreThreads() 方法来提前启动线程池中的基本线程。）

2、maximumPoolSize（线程池最大大小）：线程池所允许的最大线程个数。当队列满了，且已创建的线程数小于maximumPoolSize，则线程池会创建新的线程来执行任务。另外，对于无界队列，可忽略该参数。

3、keepAliveTime（线程存活保持时间）当线程池中线程数大于核心线程数时，线程的空闲时间如果超过线程存活时间，那么这个线程就会被销毁，直到线程池中的线程数小于等于核心线程数。

4、workQueue（任务队列）：用于传输和保存等待执行任务的阻塞队列。

5、threadFactory（线程工厂）：用于创建新线程。threadFactory创建的线程也是采用new Thread()方式，threadFactory创建的线程名都具有统一的风格：pool-m-thread-n（m为线程池的编号，n为线程池内的线程编号）。

5、handler（线程饱和策略）：当线程池和队列都满了，再加入线程会执行此策略。


为什么使用阻塞队列：如果是非阻塞队列，需要线程不停判断是否有事件，一直占用CPU资源，不好。