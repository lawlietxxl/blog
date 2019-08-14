---
title: leetcode中facebook的题目
tags: []
categories: [刷题]
keywords: []
date: 2019-08-11 21:47:15
---
记录leetcode里面facebook标签的题目，耗时，代码，时间空间占比.
格式为：
```markdown
# hard/med/easy

# [题号]. 题目名（链接）

## [时间占比] [空间占比] 解题耗时
[源码]
[解释]
```

源码中如果提交有错误，会标记  //bugfix
<!--more-->
# Hard
## [158. Read N Characters Given Read4 II - Call multiple times](https://leetcode.com/problems/read-n-characters-given-read4-ii-call-multiple-times/)

### 100.00% 100.00% 60mins

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


# Medium
## [91. Decode Ways](https://leetcode.com/problems/decode-ways/)
###  98.63% 100.00% 43min
```java
class Solution {
    public int numDecodings(String s) {
        int[] result = new int[s.length()+1];
        result[0] = 1;// bugfix
        if(s.charAt(0) == '0') return 0;
        else result[0+1] = 1;
        
        for(int sIndex = 1; sIndex < s.length(); ++sIndex){
            int rIndex = sIndex + 1;
            
            char c1 = s.charAt(sIndex-1), c2 = s.charAt(sIndex);
            int total = (c1-'0')*10 + (c2-'0');
            if(total == 0 || (c2 == '0' && total > 20)) return 0;
            //转换方程
            if(c2 != '0') result[rIndex] += result[rIndex-1];
            if(total >= 10 && total <= 26) result[rIndex] += result[rIndex-2];
        }
        return result[result.length - 1];
    }
    
}
```
此题目耗时较长，因为关于 '0' 的case没有想清楚，并且程序中存在一个bug。

计数思路：
+ 0开头，结果是0
+ 00相连，结果是0
+ 存在30，40，50...，结果是0
+ 动态规划，转换方程为上面的代码

有一次错误提交，bug见代码中注释。为了方便，我为result数组多申请了一个格子。这里初始化result[0]应该是1.

## [15. 3Sum](https://leetcode.com/problems/3sum/)
### TLE -- wrong answer
```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        Map<Integer, List<Integer[]>> map = new HashMap<>() ;
        for(int i = 0; i < nums.length-1; i ++)
            for(int j = i+1; j < nums.length; j++){
                int n = nums[i] + nums[j];
                if(!map.containsKey(n)) map.put(n, new ArrayList<>());
                Integer[] integers = new Integer[2];
                integers[0] = i;
                integers[1] = j;
                map.get(n).add(integers);
            }
        List<List<Integer>> result = new LinkedList<>();
        Set<Integer> set = new HashSet<>();
        for(int i = 0; i < nums.length; ++i){
            if(map.containsKey(-nums[i])){
                List<Integer[]> list = map.get(-nums[i]);
                for(Integer[] integers: list){
                    if(i <= integers[1]) continue;
                    List<Integer> addList = new LinkedList<Integer>();
                    int n1 = nums[integers[0]], n2 = nums[integers[1]], n3 = nums[i];
                    int i1 = Math.min(Math.min(n1, n2), Math.min(n1, n3));
                    int i3 = Math.max(Math.max(n1, n2), Math.max(n1, n3));
                    int i2 = n1 + n2 + n3 -i1 -i3;
                    addList.add(i1);
                    addList.add(i2);
                    addList.add(i3);
                    int hash = hash(addList);
                    if(!set.contains(hash)){
                        set.add(hash);
                        result.add(addList);
                    }
                }
            }
        }
        //bugfix 怎么去重
        return result;
    }
    
    private int hash(List<Integer> l){
        int result = 0;
        for(Integer i: l){
            result = result *  31 + i;
        }
        return result;
    }
}
```

**分析**：思路被2sum同化了。利用O(n2)的时间做索引，然后再遍历寻找结果。结果TLE了。后续需要思考。
PS：看了一下discuss发现都是O(n2)……

## [973. K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/)

### WRONG ANSER
没有注意审题，吭哧别都写了半天结果没看清楚题目……需要注意的是
1、返回K个最近的点，是全部返回，而不是只返回一个
2、保证只有唯一一个解（说明K+1这个点一定比K大）

# Easy
## [283. Move Zeroes](https://leetcode.com/problems/move-zeroes/)
### 100.00% 98.60% 7.5min
```java
class Solution {
    public void moveZeroes(int[] nums) {
        int firstZero = -1;
        for(int iter = 0; iter < nums.length; ++iter){
            if(nums[iter] == 0){
                if(firstZero < 0) firstZero = iter;
            }else{
                if(firstZero >= 0){
                    swap(nums, iter, firstZero);
                    firstZero++;
                }
            }
        }
    }
    
    private void swap(int[] nums, int i, int j){
        int tmp = nums[i];
        nums[i] = nums[j];
        nums[j] = tmp;
    }
}
```
## [157. Read N Characters Given Read4](https://leetcode.com/problems/read-n-characters-given-read4/)
###  100.00% 100.00% 17min

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
    public int read(char[] buf, int n) {
        if(n <= 0) return 0;
        
        int iter = n/4;
        
        int result = 0;
        char[] tmpBuf = new char[4];
        for(int i = 0; i < iter; i++){
            int tmpResult = read4(tmpBuf);
            result += tmpResult;
            for(int j = 0; j < tmpResult; j++){
                buf[4*i + j] = tmpBuf[j];
            }
        }
        //bugfix: 最后一次循环要特殊判断，取两者里面小的
        int left = n%4;
        if(left == 0) return result;
        
        int lastResult = read4(tmpBuf);
        int iter2 = left < lastResult?left:lastResult;
        
        for(int i = 0;i<iter2; i++){
            buf[4*iter + i] = tmpBuf[i];
        }
        //注意加的这个数值
        result += iter2;
        return result;
        
    }
}
```

消耗时间较多了，注意两个bug

## [278. First Bad Version](https://leetcode.com/problems/first-bad-version/)
### 99.48% 100.00% 20min

```java
/* The isBadVersion API is defined in the parent class VersionControl.
      boolean isBadVersion(int version); */

public class Solution extends VersionControl {
    public int firstBadVersion(int n) {
        //bugfix r = n + 1
        int l = 1, r = n, mid;
        while(l <= r){
            if(l == r) return l;
            mid = l + (r - l)/2;
            if(isBadVersion(mid)) r = mid;
            else l = mid + 1;
        }
        return l;
    }
}
```

**bugfix**:注意binary search的时候，边界是否包括结果可能对算法正确与否产生影响。比如如果认为边界不包括结果，则 r = n + 1。那么在后续判断的时候会令 r = mid+1; 那么有一种情况会造成死循环: [l][结果][r]。这时候会造成死循环。注意考虑这种边界条件.

## [125. Valid Palindrome](https://leetcode.com/problems/valid-palindrome/)

### 96.21 92.86 20mins

```java
class Solution {
    public boolean isPalindrome(String s) {
        if(s.length() == 0) return true;
        int l = 0, r = s.length() - 1;
        while(l != r){
            while(l < s.length() && !isAlphaNum(s.charAt(l))) l++;
            while(r >= 0 && !isAlphaNum(s.charAt(r))) r--;
            if(l >= r) return true;
            //bugfix ignore case
            if(getChar(s, l++) != getChar(s, r--)) return false;
        }
        
        return true;
    }
    
    private boolean isAlphaNum(char c){
        if(c >= 'a' && c <= 'z')
            return true;
        if(c >= 'A' && c <= 'Z')
            return true;
        if(c >= '0' && c <= '9')
            return true;
        return false;
    }
    
    private int getChar(String s, int i){
        char c = s.charAt(i);
        if(c >= '0' && c <= '9') return c;
        if(c >= 'a' && c <= 'z') return c - 'a';
        if(c >= 'A' && c <= 'Z') return c - 'A';
        return c;
    }
}
```
题目不难，但是被java语法卡住了
**tips**:
+ 移动指针的时候注意边界，不要越界
+ 判断边界条件及时返回
+ java中'a'+<int>编译器不能支持，但是'a'+10是支持的。（因为编译器编译时能判断后者没越界）所以需要使用另外一种方式替代ignorecase的判断。

## [1. Two Sum](https://leetcode.com/problems/two-sum/)
### 98.89 98.95 10mins
```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();
        for(int i = 0; i < nums.length; ++i){
            int n = nums[i];
            if(map.containsKey(n) && n*2 == target) return new int[]{map.get(n), i};
            if(map.containsKey(target-n)) return new int[]{map.get(target-n), i};
            map.put(n, i);
        }
        return new int[2];
    }
}
```
**tips**: 注意相等的情况，简单的题目暗藏对情况的全面考虑