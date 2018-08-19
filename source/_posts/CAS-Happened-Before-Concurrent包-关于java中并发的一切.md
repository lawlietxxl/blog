---
title: '锁, Happened-Before, volatile, Concurrent包--java并发'
date: 2018-08-16 20:31:18
categories: [java, 源码, 多线程]
tags: [锁, ConcurrentHashMap, CAS]
keywords: [java, 源码, 多线程, Concurrent, 锁, CAS]
---
{% asset_img lock.jpg %}
# 前言
多线程的知识涉及到的方面其实比较多，包括计算机系统、软件涉及理论等。我觉得在除了能够清晰了解java源码里涉及到的知识点之外，还要把深层次的理论知识多多补足。这边文章思路是从点到面，先把java10中Concurrent包中涉及到的点先拎出来说清楚，再讲包中的原子类，ConcurrentHashMap的原理等等。本篇文章中概念性的知识点，大多是引用和整理。
<!--more-->
# 锁，乐观锁，悲观锁
锁是实现事务隔离的方法。根据锁的策略，可以把锁分为两种——乐观锁和悲观锁。
乐观锁认为，多个事务可以在不互相冲突的情况下完成，所以他们在进行的时候不需要锁住资源；但是每个事务在最后进行提交的时候，需要确定没有其他的事务修改了资源产生冲突，如果产生了冲突，那么本事务就要回退。悲观锁认为，所有的事务都会产生冲突，所以从事务一开始（一般是数据读取）就要锁住资源，直到事务结束才会释放。
乐观锁的实现方法，简要言之就是在悲观锁锁资源的时候，乐观锁存储资源的状态而不是锁住。仅仅在提交事务的时候才锁一下，并与之前存储的资源状态进行对比，如果存在冲突证明之前已经有事务修改了该资源，当前事务进行回退。所以对于乐观锁来说，是"先改先生效"。
乐观锁会导致事务回滚和不可预期的问题，乐观锁适用于 **Read-Often Write-Sometimes** 这种数据竞争不激烈的情况。
悲观锁是“先取锁再访问”的保守策略，安全有保证，但是效率较低：加锁导致额外开销；可能产生死锁；还有会降低了并行性。悲观锁适用于数据争用激烈的环境，以及发生并发冲突时使用锁保护数据的成本要低于回滚事务的成本的环境中。


## 死锁
当两个以上的运算单元，双方都在等待对方停止运行，以获取系统资源，但是没有一方提前退出时，就称为死锁。
死锁的四个条件是：
+ **禁止抢占** no preemption - 系统资源不能被强制从一个进程中退出
+ **持有和等待** hold and wait - 一个进程可以在等待时持有系统资源
+ **互斥** mutual exclusion - 只有一个进程能持有一个资源
+ **循环等待** circular waiting - 一系列进程互相持有其他进程所需要的资源
死锁只有在这四个条件同时满足时出现。预防死锁就是至少破坏这四个条件其中一项。

> 扩展阅读，安全序列和银行家算法
> 安全序列："所谓系统是安全的，是指系统中的所有进程能够按照某一种次序分配资源，并且依次地运行完毕，这种进程序列{P1，P2，...，Pn}就是安全序列。如果存在这样一个安全序列，则系统是安全的；如果系统不存在这样一个安全序列，则系统是不安全的。"银行家算法是求得一个安全序列的算法。


# Happened-Before
Happened-Before是计算机科学的一个名词。在java中，happened-before关系意味着，A对内存的写操作对B可见，也就是说，在B开始读取之前，A已经完成了写操作。

# volatile关键字
volatile的声明可以保证：读取的时候，线程从主存中读取变量而并非cache；写入的时候，线程直接把变量写入主存，并不单单写入cache。这保证了，volatile关键字保证了在多线程中变量的**可见性**。（多cpu的环境中，每个cpu从主存中读取变量值，并且存入各自的cache中增加速度，然后再由cache存回主存）。

## java的Full Visibility Guarantee
java中的volatile还存在"Full volatile Visibility Guarantee"：
+ 线程A写一个volatile变量，线程B读一个volatile变量，那么线程A写变量之前的其他变量，在线程B读变量之后也是可见的。
+ 线程A读一个volatile变量，在线程A读一个volatile变量之后，所有对A可见的变量都会从主存中重新读取。

例子如下
写volatile：在线程A执行,`this.days=days`这句话的时候，所有的三个变量会一起写回主存。
```java
public class MyClass {
    private int years;
    private int months;
    private volatile int days;

    public void update(int years, int months, int days){
        this.years  = years;
        this.months = months;
        this.days   = days;
    }
}
```
读volatile：在执行`int total=this.days`的时候，所有的变量，会冲主存中重新读一次。
```java
public class MyClass {
    private int years;
    private int months;
    private volatile int days;

    public int totalDays() {
        int total = this.days;
        total += months * 30;
        total += years * 365;
        return total;
    }

    public void update(int years, int months, int days){
        this.years  = years;
        this.months = months;
        this.days   = days;
    }
}
```

## java的Happens-Before Guarantee
java编译器为了优化执行，可能会reorder执行语句的顺序。这就有可能导致 full visibility guarantee 不能成功。所以java编译器保证了如果其他变量的语句在读/写votaile之后/前，不能把它们的语句调整到后面。

# CAS -- compare and swap 和 concurrent.atomic包中的典型实现
CAS其实是一种乐观锁，即实现的是一种 **check and set** 模式。简单例子如下

```java
class MyLock {

    private boolean locked = false;

    public boolean lock() {
        if(!locked) {
            locked = true;
            return true;
        }
        return false;
    }
}
```
这是一种简单的check and set模式，显然在多线程条件下，这个函数是不能正确实现的，所以我们引入一个类 -- `AtomicBoolean`
```java
public static class MyLock {
    private AtomicBoolean locked = new AtomicBoolean(false);

    public boolean lock() {
        return locked.compareAndSet(false, true);
    }

}
```
`public final boolean compareAndSet(boolean expectedValue, boolean newValue)`函数，实现了check and set的方法。如果能够成功比较并修改，那么它返回的就是true，否则返回false。CAS实现方法主要是java利用了现代计算机cpu的cas指令并结合JNI的进行调用。

## AtomicBoolean
{% asset_img boolean.png %}
`VarHandle`是java反射用到的核心类，从java9开始引入，代替了之前的`Unsafe`。核心用法如下
```java
    public final boolean compareAndSet(boolean expectedValue, boolean newValue) {
        return VALUE.compareAndSet(this,
                                   (expectedValue ? 1 : 0),
                                   (newValue ? 1 : 0));
    }
```

## AtomicInteger
{% asset_img int.png %}
可以看到注释，这里本来想用VarHandle处理反射的，但是因为循环引用的问题还是使用了Unsafe。但其实用法是类似的，具体区别暂不了解。

## DoubleAccumulator, DoubleAdder, LongAccumulator, DoubleAdder
这四个类是java8新增的，目的是在大规模并发的时候，对double和long进行增加的时候，减少竞争，提高效率。适用环境是数据统计，某个数字需要频繁增加，但不太经常get的情况。
具体原理用四个字概括就是：热点分离。可以看到，类中包含一个数组的数据结构`Cell[] cells`。当线程足够多，直到cells中的某个栏位出现CAS失败的时候，就会开辟新的栏位给竞争线程使用。这样到后来，几乎每个线程都有自己的一个栏位，在最后的时候用"map-reduce"的思想，把每个栏位相加，得到最后的总和。
XXXAdder其实是XXXAccumulator的一种特殊情况，它可以输入一个函数进行计算方法的定制。

# concurrent.locks



# 自问自答
1. 什么是乐观锁、悲观锁？分别适用于什么情况？
1. 以java代码为例使用乐观锁和悲观锁。

# 引用
+ https://docs.jboss.org/jbossas/docs/Server_Configuration_Guide/4/html/TransactionJTA_Overview-Pessimistic_and_optimistic_locking.html
+ https://docs.jboss.org/hibernate/orm/4.0/devguide/en-US/html/ch05.html
+ http://www.hollischuang.com/archives/934
+ https://zh.wikipedia.org/wiki/死锁
+ https://www.ibm.com/developerworks/cn/java/j-lo-deadlock/index.html
+ https://blog.csdn.net/abigale1011/article/details/6450845
+ https://en.wikipedia.org/wiki/Happened-before
+ http://tutorials.jenkov.com/java-concurrency/volatile.html
+ DoubleAccumulator原理 https://dzone.com/articles/how-longaccumulator-and

