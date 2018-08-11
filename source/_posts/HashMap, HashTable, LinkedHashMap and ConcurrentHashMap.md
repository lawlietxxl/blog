---
title: HashMap, LinkedHashMap, HashTable and ConcurrentHashMap
date: 2018-08-05 14:17:58
categories: java10-source-reading
tags: HashMap
---
日常用HashMap相关的类最多，但对其内部实现并不了解。花时间读些源码，整理了一下。

## HashMap

### 注释阅读
+ HashMap不是同步的，而且允许存在key和value是null，这是与HashTable的区别所在。
+ "in particular, it does not guarantee that the order will remain constant over time."意思是，HashMap中的键值对的顺序不是永远不变的（resize可能会造成键值对顺序改变）
+ HashMap的效率跟两个值相关——初始化的容量以及load factor。load factor是做rehash的时候map的容量标准。
+ HashMap不同步，所以在对HashMap做结构化操作（删除或者添加Mapping）的时候，一定要在外部做同步。或者使用 ``Collections.synchronizedMap`` 方法获取Map
+ Iterator提供一个HashMap的视图，在iterator外对HashMap进行结构性修改会抛出一个 ``ConcurrentModificationException``异常，但是这并不能保证多线程安全。
<!--more-->
### 源码逻辑
#### fields and inner class
```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable{
    //...
    /* ---------------- Fields -------------- */
    transient Node<K,V>[] table; //hash表
    transient Set<Map.Entry<K,V>> entrySet; // entrySet() values() 等函数用到的视图
    transient int size; // map中保存的entry数量
    transient int modCount; // 修改次数
    int threshold; // resize的阈值，capacity * loadFactor
    final float loadFactor;
    //...
    
    //Node 结构如下
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
    }
    
    //当每个bucket内的node数目大于 TREEIFY_THRESHOLD 的时候，这个bucket下的链会treenify，为了提高查询效率
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
    }
}
```

#### get
对于每个key的hash，HashMap做了一些转化：因为table的buckets数目是2的幂，选取bucket的时候是取模的运算，如果仅仅做hash值的模运算，
冲突会很多（低位一致的object都会冲突），所以把高位的影响转化到低位。做了效率和碰撞概率的tradeOff。
```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable{
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
}
```

get Node的过程比较简单，首选通过入参 hash 判断Node在哪个bucket，判断当前bucket是treeNode还是Node，如果是树节点就调用树节点的方法，
如果是普通阶段就遍历。
```java
public class HashMap<K, V> extends AbstractMap<K, V> implements Map<K, V>, Cloneable, Serializable {
    final Node<K, V> getNode(int hash, Object key) {
        //...
        if ((tab = table) != null && (n = tab.length) > 0 &&
                (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && // always check first node
                    ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K, V>) first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                            ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
}
```

#### put
核心方法是
```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) 
```
代码较长，简要叙述过程：

put方法首先检查table是否进行了初始化，若否则调用resize进行初始化；
然后检查hash对应的bucket是否为null，若为null，则新建一个node填在bucket里面；若为treeNode就调用treeNode方法加入新节点，
若为普通node就弄一个新节点放到链表最后。普通node增加节点之后，判断节点数目是否大于 TREEIFY_THRESHOLD，若是则把链表树化``treeifyBin(tab, hash)``

最后

```java
++modCount; //修改次数+1
if (++size > threshold) //数目+1，如果大于阈值则调用resize
    resize();
```
#### remove
remove是put的反过程，核心函数是
```java
final Node<K,V> removeNode(int hash, Object key, Object value,boolean matchValue, boolean movable)
```
代码较长，简要叙述过程：

与put一样，得到该hash和key对应的bucket位置，为null则返回null；不为null，判断该bucket node类型，若为TreeNode，则调用treeNode的方法获取node；
若为普通Node，则遍历，得到该Node点。然后进行remove，见函数，

```java
if(node !=null&&(!matchValue ||(v =node.value)==value ||
            (value !=null&&value.equals(v))))
    {
        if (node instanceof TreeNode)
            ((TreeNode<K, V>) node).removeTreeNode(this, tab, movable); //treeNode调用remove方法
        else if (node == p)
            tab[index] = node.next; //node没有父节点，更新table
        else
            p.next = node.next; //node在链上
        ++modCount;
        --size;
        afterNodeRemoval(node);
        return node;
    }
```

#### resize
resize函数负责把table扩大两倍，并且改变node位置（所以Map是只扩大不缩小的）；
根据前面的阅读，一个Node的位置是 (n-1)&hash，所以一个node的位置要不变，要么位置*2。

大概做法是，搞两条链，low，high；然后特意强调了**preserve order**，把原节点的Node按照顺序放到这两条链中

```java
final Node<K,V>[] resize() {
    //新建 Node数组，更新各个field，略
    //遍历oldTable的每个节点，对不同类型节点进行判断，做不同的处理
    Node<K, V> e;
    if((e =oldTab[j])!=null){
        oldTab[j] = null;
        if (e.next == null)
            newTab[e.hash & (newCap - 1)] = e;
        else if (e instanceof TreeNode)
            ((TreeNode<K, V>) e).split(this, newTab, j, oldCap);
        else { // preserve order
            Node<K, V> loHead = null, loTail = null;
            Node<K, V> hiHead = null, hiTail = null;
            Node<K, V> next;
            do {
                next = e.next;
                if ((e.hash & oldCap) == 0) { //注意oldCap是 100000...，通过此方法可以判断e是保持bucket不变的点
                    if (loTail == null)
                        loHead = e;
                    else
                        loTail.next = e;
                    loTail = e;
                } else {
                    if (hiTail == null)
                        hiHead = e;
                    else
                        hiTail.next = e;
                    hiTail = e;
                }
            } while ((e = next) != null);
            if (loTail != null) {
                loTail.next = null;
                newTab[j] = loHead;
            }
            if (hiTail != null) {
                hiTail.next = null;
                newTab[j + oldCap] = hiHead; //这里j是oldTable的index
            }
        }
    }
}
```
如果是treeNode就调用treeNode的函数进行split，如果是普通链表就把链表进行拆分成两条链，放到新table的bucket里面。

### 为什么HashMap在多线程下会死循环（旧版jdk）
参考资料链接：https://coolshell.cn/articles/9606.html  原链接jdk可能有点老，本文粘贴的源码是jdk10的，所以有些不一样。

一句话概括就是**高并发下，HashMap出现了环形链表，导致get的时候出现了死循环**。涉及到的过程主要就是**resize**。

但是看了一下链接的图，出现环形链需要resize的时候，两个节点前后倒置；但是jdk10里面，新建了两条链，并且维持了顺序，不知道是否还会出现这种情况。不知如何分析。

有时间可以做一下多线程的实验，并且通过查看堆栈来看cpu占用率和在什么时候做了死循环。
### HashMap查询的复杂度为什么是 O(1)
参考链接：http://blog.zhaojie.me/2013/01/think-in-detail-why-its-o-1.html

O（1）其实是一种理想的状态，可以看出，每个bucket下其实可能退化成树。如果全部塞key值hash一样，那么hashMap复杂度很可能会退化成O(lgN)。
### tips
+ HashMap提供了三个方法，提供三个视图。值得注意的是，在这三个视图之外对HashMap进行修改，都会导致在下次操作iterator的时候modCount不一致抛出异常。
具体抛出异常的过程，iterator原理还有待研究。

```java
public Set<K> keySet();
public Collection<V> values();
public Set<Map.Entry<K,V>> entrySet();
```

+ HashMap提供了 ``static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V>``这个类，下面有很多算法，暂不研究细节。

### 总结
HashMap主要是算法和数据结构比较复杂，猜测选择几个默认值的时候进行的测试会比较多，
但没有特别复杂的逻辑点。涉及到的数据结构和算法有 hash，树，平衡树，链表。

## LinkedHashMap
### 注释阅读
+ LinkedHashMap维护一个双向链表保证塞进map的entry顺序稳定。
+ 一种特殊的 LinkedHashMap 提供了LRU的功能，当对map进行get put等操作的时候，那个Node会被放到前面。
+ 轮询LinkedHashMap时间跟table的真正size相关；轮询HashMap跟capacity相关。所以如果初始化一个较大的InitialCapacity，
后果没有HashMap那么严重。
### 源码逻辑
LinkedHashMap是HashMap的子类，它内部的Entry是HashMap.Node的子类
```java
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>
{
    static class Entry<K,V> extends HashMap.Node<K,V> {
            Entry<K,V> before, after;
            Entry(int hash, K key, V value, Node<K,V> next) {
                super(hash, key, value, next);
            }
        }
}
```
field包括
```java
    /**
     * The head (eldest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> head;

    /**
     * The tail (youngest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> tail;

    /**
     * The iteration ordering method for this linked hash map: {@code true}
     * for access-order, {@code false} for insertion-order.
     *
     * @serial
     */
    final boolean accessOrder;
```

这个类重载了HashMap的方法，例子如下，原理都是在**新增、替换、删除**等操作的时候，维护一下链表。
```java
    //新建Node之后，链在维护的链表尾部
    Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
        LinkedHashMap.Entry<K,V> p =
            new LinkedHashMap.Entry<>(hash, key, value, e);
        linkNodeLast(p);
        return p;
    }
```

#### LRU的LinkedHashMap
accessOrder默认是false，意思是，这个链表维护的是insertOrder。可以通过下面的构造函数置true，这样会维护accessOrder。
如下。
```java
    public LinkedHashMap(int initialCapacity,
                         float loadFactor,
                         boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }
    public V get(Object key) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return null;
    if (accessOrder)
        afterNodeAccess(e); // move node to last
    return e.value;
}
```

### tips
+ 可以看到，LinkedHashMap新建的三个视图，轮询的时候都是遍历链表，而不是遍历map，
+ 如果accessOrder是true，那么每次进行get操作，node都会转移到链表最后，所以这时候linkedHashMap是least-recently-used cache。
### 总结
LinkedHashMap跟HashMap相比，有效率和特性上的不同。通过重载各种函数，维护了一个链表；可以看到java面向对象编程的魅力。

