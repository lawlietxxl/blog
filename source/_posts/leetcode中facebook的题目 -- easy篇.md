---
title: leetcode中facebook的题目 -- easy篇
tags: []
categories: [刷题]
keywords: []
date: 2019-08-11 21:47:15
---
记录leetcode里面facebook标签的题目，耗时，代码，时间空间占比.
格式为：
```markdown
# [题号]. 题目名（链接）

## [时间占比] [空间占比] 解题耗时
[源码]
[解释]
```

源码中如果提交有错误，会标记  //bugfix
<!--more-->

# [283. Move Zeroes](https://leetcode.com/problems/move-zeroes/)
## 100.00% 98.60% 7.5min
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
# [157. Read N Characters Given Read4](https://leetcode.com/problems/read-n-characters-given-read4/)
##  100.00% 100.00% 17min

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

# [278. First Bad Version](https://leetcode.com/problems/first-bad-version/)
## 99.48% 100.00% 20min

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

# [125. Valid Palindrome](https://leetcode.com/problems/valid-palindrome/)

## 96.21 92.86 20mins

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

# [1. Two Sum](https://leetcode.com/problems/two-sum/)
## 98.89 98.95 10mins
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

# [88. Merge Sorted Array](https://leetcode.com/problems/merge-sorted-array/)
## 100.00 100.00 15min

```java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int leftLength = m + n;
        //bugfix
        while(leftLength != 0){
            if(m == 0){
                for(int i = 0; i < n; i++){
                    nums1[i] = nums2[i];
                }
                break;
            }
            if(n == 0){
                break;
            }
            
            if(nums1[m-1] >= nums2[n-1]){
                //bugfix leftLength++ indexOutOfBound
                nums1[--leftLength] = nums1[--m];
            }else{
                nums1[--leftLength] = nums2[--n];
            }
        }
    }
}
```
**tips**:
+ 最好用length 而并非index（length比index多1）
+ 注意先减后用