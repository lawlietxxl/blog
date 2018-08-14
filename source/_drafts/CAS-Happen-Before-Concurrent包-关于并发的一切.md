---
title: 'CAS,Happen-Before,Concurrent包,ConcurrentHashMap...关于并发的一切'
tags:
---

## ConcurrentHashMap
其实ConcurrentHashMap放在这里并不太合适，应该放到Concurrent包里说的，这里暂且看一下，当个预习吧。

### 注释阅读
1. get不锁表，get会取到最近的update的结果。某个key值的update类型操作对读取有着“happen-before”的关系。那些iterator视图，取到的都是某个时间点的值，并非实时的.如果做统计是足够的，但是不能用来做程序判断。
对于`putAll clear`这种操作，读取操作可能只能反应这个过程中部分entry被处理后的状态。同样的，几个`iterator`视图也是这样的，只表现视图新建的那一刻的状态，所以它其实是给单线程做设计的。
2. 什么是**Happened-Before**？https://en.wikipedia.org/wiki/Happened-before   链接里面写的比较清楚，“ In Java specifically, a happens-before relationship is a guarantee that memory written to by statement A is visible to statement B, that is, that statement A completes its write before statement B starts its read.”
就是说这个``ConcurrentHashMap``写操作对于读操作来说，它的内存是透明的，都操作看到写操作完全结束之后才会去读取。
1. `ConcurrentHashMap`可以做一个频率直方表，具体做法待定
1. bulk operation这段没看懂啊，花了最后一大段讲，还设计到了forkJoinPool，好像是ConcurrentHashMap可以放ForkJoinTask？
1. 不想浪费空间存储lock，所以把每个桶的第一个Node当作lock，用synchronized这种方法（synchronized什么原理）。resize的时候，每个桶的转移都需要bin lock。其他线程会加入线程一起做resize而不是一直等这锁释放。

### 源码阅读
1. constant
与HashMap有不同。
2. 用了native方法读取数组的元素，具体含义未知。
```java
    private static final Unsafe U = Unsafe.getUnsafe();
    static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
        return (Node<K,V>)U.getObjectAcquire(tab, ((long)i << ASHIFT) + ABASE);
    }

    static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                        Node<K,V> c, Node<K,V> v) {
        return U.compareAndSetObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
    }

    static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
        U.putObjectRelease(tab, ((long)i << ASHIFT) + ABASE, v);
    }
```

3. fields有些不同
关于CAS和volatile的介绍见三个链接
https://www.cnblogs.com/gxjz/p/5726979.html
http://www.cnblogs.com/Mainz/p/3556430.html
https://blog.csdn.net/ls5718/article/details/52563959