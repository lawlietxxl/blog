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

