---
title: leetcode dp题目
date: 2020-02-21 19:44:48
tags: [leetcode]
mathjax: true
---

要求是写出每道题目的状态转换方程，和时间空间复杂度


<!--more-->

# [746. Min Cost Climbing Stairs](https://leetcode.com/problems/min-cost-climbing-stairs/)

```java
class Solution {
    public int minCostClimbingStairs(int[] cost) {
        int r0 = 0, r1 = 0;
        
        for(int i = 2; i < cost.length; i++) {
            int r2 = Math.min(r0 + cost[i-2], r1 + cost[i-1]);
            r0 = r1;
            r1 = r2;
        }
        
        return Math.min(r0+cost[cost.length-2], r1+cost[cost.length-1]);
    }
}
```

$f(n)=min(f(n-2)+v(n-2), f(n-1)+v(n-1))$

# [1218. Longest Arithmetic Subsequence of Given Difference](https://leetcode.com/problems/longest-arithmetic-subsequence-of-given-difference/)

## $O(n^2)$的解法
```java
class Solution {
    public int longestSubsequence(int[] arr, int difference) {
        int[] r = new int[arr.length];
        for(int i = 0; i < r.length; i++) r[i] = 1;
        for(int i = 0; i < r.length; i++){
            for(int j = i+1; j < r.length; j++)
                if(arr[i] + difference == arr[j]) {
                    r[j] = Math.max(r[j], r[i]+1);
                    break;
                }
        }
        int res = 1;
        for(int rr: r) res = Math.max(res, rr);
        return res;
    }
}
```
超时了

## dp的解法
```java
class Solution {
    public int longestSubsequence(int[] arr, int difference) {
        Map<Integer, Integer> dp = new HashMap<>();
        int res = 1;
        for(int a: arr) {
            int lastDp = dp.getOrDefault(a-difference, 0);
            dp.put(a, Math.max(lastDp+1, dp.getOrDefault(a, 0)));
            res = Math.max(res, dp.get(a));
        }
        return res;
    }
}
```
空间：$O(n)$
时间：$O(n)$
状态方程：$f(x)=Math.max(f(x), f(x-difference)+1)$
> 强顺序性，保证了方程的正确性