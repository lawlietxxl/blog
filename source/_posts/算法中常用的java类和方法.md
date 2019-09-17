---
title: 算法中常用的java类和方法
tags: []
categories: []
keywords: []
date: 2019-08-16 22:48:22
---
有一些方法和类在算法中经常使用，总结一下，方便复习。
<!--more-->
# 排序
```java
List<Object> l = new ArrayList<>();
Collections.sort(l, (l1, l2)->{
    // compare method l1 < l2 ，返回-1，则l1在l2之前
});


int[] l = new int[3];
Arrays.sort(l, (i1, i2)-> Integer.compare(i1, i2););
```

# 数组初始化
**tips**:
+ 不要同时使用静态初始化和动态初始化，也就是说，不要在进行数组初始化时，既指定数组的长度，也为每个数组元素分配初始值。
```java
// 1 动态初始化
int[] intArr;
intArr = new int[]{1,2,3,4,5,9};
// 2 静态初始化
int[] price = new int[4];
// 3 简化的静态初始化
String[] strArr = {"张三","李四","王二麻"};
// 4 多维数组变长
int[][] a = new int[10][];
a[0] = new int[5];
a[1] = new int[7];

```

# PriorityQueue
三个方法：offer, poll, peek
默认的优先队列，先小后大。自定义需要重写Comparator

```java
        PriorityQueue<Integer> queue = new PriorityQueue<>(Collections.reverseOrder());
        PriorityQueue<Integer> priorityQueue = new PriorityQueue<>();
        priorityQueue.offer(3);
        priorityQueue.offer(1);
        priorityQueue.offer(2);

        System.out.println(priorityQueue.peek()); // 1
        while (priorityQueue.size() != 0) {
            System.out.println(priorityQueue.poll()); // 1 2 3
        }

        priorityQueue = new PriorityQueue<>(new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                if(o1.compareTo(o2) == 0) return 0;
                return o1.compareTo(o2) == 1 ? -1 : 1;
            }
        });
        priorityQueue.offer(3);
        priorityQueue.offer(1);
        priorityQueue.offer(2);

        System.out.println(priorityQueue.peek()); // 3
        while (priorityQueue.size() != 0) {
            System.out.println(priorityQueue.poll()); // 3 2 1
        }
```

# 泛型相关方法
下面是符合java语法的，int[] 不属于primitive type
```java
    List<int[]> a= new ArrayList<>();
    // 排序可以使用lambda，很easy
 
    Collections.sort(a, (a1, a2)->Integer.compare(a1[0], a2[0]));
```

# Stack
```java
    Stack<Integer> stack = new Stack<>();
    stack.push(1);
    stack.pop();
    stack.peek(); // throw new EmptyStackException();
```

# StringBuilder
```java
    StringBuilder stringBuilder = new StringBuilder();
    stringBuilder.insert(0, 'c');
```

# Integer
```java
int a = Integer.MAX_VALUE; // 0x7fffffff
```

# Queue
```java
Queue<Integer> queue = new LinkedList<>();
queue.offer(1);
queue.offer(2);
System.out.println(queue.poll()); // 1
System.out.println(queue.peek()); // 2
System.out.println(queue.size()); // 1
```

# List
```java
List<String> l = new ArrayList<>();
l.add(0, "abc");
String s = l.remove(0);
```