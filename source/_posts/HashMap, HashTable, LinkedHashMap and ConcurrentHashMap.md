---
title: HashMap, HashTable, LinkedHashMap,  and ConcurrentHashMap
date: 2018-08-05 14:17:58
tags: java10-source-reading
---
## HashMap

### Comment Reading
+ HashMap不是同步的，而且允许存在key和value是null，这是与HashTable的区别所在。
+ "in particular, it does not guarantee that the order will remain constant over time."意思是，HashMap中的键值对的顺序不是永远不变的（resize可能会造成键值对顺序改变）
+ HashMap的效率跟两个值相关——初始化的容量以及load factor。load factor是做rehash的时候map的容量标准。
+ HashMap不同步，所以在对HashMap做结构化操作（删除或者添加Mapping）的时候，一定要在外部做同步。或者使用 ``Collections.synchronizedMap`` 方法获取Map
+ Iterator提供一个HashMap的视图，在iterator外对HashMap进行结构性修改会抛出一个 ``ConcurrentModificationException``异常，但是这并不能保证多线程安全。
