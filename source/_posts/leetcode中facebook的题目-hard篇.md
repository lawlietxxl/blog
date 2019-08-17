---
title: leetcode中facebook的题目 -- hard篇
tags: []
categories: []
keywords: []
date: 2019-08-15 23:34:39
---
同上。
<!--more-->

# [158. Read N Characters Given Read4 II - Call multiple times](https://leetcode.com/problems/read-n-characters-given-read4-ii-call-multiple-times/)

## 100.00% 100.00% 60mins

```java
/**
 * The read4 API is defined in the parent class Reader4.
 *     int read4(char[] buf); 
 */
public class Solution extends Reader4 {
    /**
     * @param buf Destination buffer
     * @param n   Number of characters to read
     * @return    The number of actual characters read
     */
    
    private char[] innerBuf = new char[4];
    private int innerOffset = 0;
    private int innerLength = 0;
    
    private boolean EOF = false;
    
    public int read(char[] buf, int n) {
        // result, n, innerOffset, readFromInner
        if(n <= 0 || EOF) return 0;
        int bufOffset = 0;
        
        // read from innerBuffer
        int min;
        min = Math.min(innerLength - innerOffset, n);
        for(int i = 0; i < min; i ++){
            buf[bufOffset++] = innerBuf[innerOffset++];
        }
        n -= min;
        if(innerOffset == innerLength){
            innerOffset = 0;
            innerLength = 0;
        }
        //bugfix
        if(n == 0) return bufOffset;
        
        // read from file
        int iter = n/4;
        for(int i = 0; i < iter; i++){
            int tmpResult = read4(innerBuf);
            copyArray(buf, bufOffset, innerBuf, 0, tmpResult);
            n -= tmpResult;
            bufOffset += tmpResult;
            if(tmpResult < 4){
                EOF = true;
                return bufOffset;
            }
        }
        // bugfix
        if(n == 0) return bufOffset;
        
        // write to innerBuffer
        // left is n
        
        //bugfix 没考虑tmpResult为0的情况
        int tmpResult = read4(innerBuf);
        min = Math.min(tmpResult, n);
        copyArray(buf, bufOffset, innerBuf, 0, min);
        bufOffset += min;
        if(tmpResult <= n){
            EOF = true;
        }else{
            innerOffset = n;
            innerLength = tmpResult;
        }
        return bufOffset;
    }
    
    public void copyArray(char[] dest, int destOffset, char[] src, int srcOffset, int length){
        for(int i = 0; i < length; i ++){
            dest[destOffset + i] = src[srcOffset + i];
        }
    }
}
```

有一次提交错误，存在bug，最后的时候没考虑tmpResult为0的情况，导致innerLength设置为0了。
思路：
+ 设置内部缓存，同时记录缓存offset和缓存长度
+ 顺序为先读缓存，再读文件，最后再写缓存。注意更新各种本地变量
感觉有点像tcp协议操作系统之类的实现

# [23. Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)

## 98.74 27.33 18mins
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        //bugfix
        if(lists.length == 0) return null;
        
        return merge(lists, 0, lists.length-1);
    }
    
    private ListNode merge(ListNode[] lists, int lo, int hi){  
        if(lo == hi) return lists[lo];
        int mid = lo + (hi - lo)/2;
        ListNode l1 = merge(lists, lo, mid);
        ListNode l2 = merge(lists, mid+1, hi);
        
        ListNode result = new ListNode(0);
        ListNode tail = result;
        
        while(true){
            if(l1 == null && l2 == null) break;
            if(l1 == null && l2 != null){
                tail.next = l2;
                break;
            }
            if(l1 != null && l2 == null){
                tail.next = l1;
                break;
            }
            
            if(l1.val < l2.val) {
                tail.next = l1;
                l1 = l1.next;
                tail.next.next = null;
                tail = tail.next;
            }else{
                tail.next = l2;
                l2 = l2.next;
                tail.next.next = null;
                tail = tail.next;
            }
        }
        
        result = result.next;
        return result;
    }
}
```
使用分治法的思想，一半一半的处理，把复杂度降低log。
但是内存占用较高，而且并不会分析算法复杂度。