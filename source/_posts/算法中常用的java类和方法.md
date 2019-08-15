---
title: 算法中常用的java类和方法
tags: []
categories: []
keywords: []
date: 2019-08-16 22:48:22
---
有一些方法和类在算法中经常使用，总结一下，方便复习。
<!--more-->
# PriorityQueue
三个方法：offer, poll, peek
默认的优先队列，先小后大。自定义需要重写Comparator

```java
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