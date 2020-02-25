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

# [121. Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)

```java
class Solution {
    public int maxProfit(int[] prices) {
        if(prices.length <= 1) return 0;
        int res = 0, l = 0;
        for(int i = 1; i < prices.length; i++)
            if(prices[i] <= prices[l]) l = i;
            else res = Math.max(res, prices[i] - prices[l]);
        return res;
    }
}
```
循环的时候，```prices[i]-prices[l]```为**当天卖出**获得的最大利润。

# [62. Unique Paths](https://leetcode.com/problems/unique-paths/)
```java
class Solution {
    public int uniquePaths(int m, int n) {
        if(m == 0 || n == 0) return 0;
        if(m == 1 || n == 1) return 1;
        int[] res = new int[n];
        Arrays.fill(res, 1);
        for(int i = 2; i <= m; i++)
            for(int j = 2; j <= n; j++)
                res[j-1] = res[j-2] + res[j-1];
        return res[n-1];
    }
}
```

$f(m,n)=f(m-1,n)+f(m,n-1)$
$S=O(n)$
$T=O(mn)$

# [63. Unique Paths II](https://leetcode.com/problems/unique-paths-ii/)
```java
class Solution {
    public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        if(obstacleGrid.length == 0 || obstacleGrid[0].length == 0) return 0;
        // fix 最后一个格子是路障，则返回0
        //if(obstacleGrid[0][0] == 1 || obstacleGrid[obstacleGrid.length-1][obstacleGrid[0].length-1] == 1) return 0;
        
        int[] res = new int[obstacleGrid[0].length];
        // fix 需要考虑通用的更新方法
        for(int i = 0; i < obstacleGrid.length; i++)
            for(int j = 0; j < obstacleGrid[0].length; j++)
                if(obstacleGrid[i][j] == 1) res[j] = 0;
                else if(i == 0 && j == 0) res[j] = 1;
                else if(i == 0) res[j] = res[j-1];
                else if(j == 0) res[j] = res[j];
                else res[j] = res[j-1] + res[j];
        return res[res.length-1];
    }
}
```

错了n次 要写通用一点的判断方法

# [120. Triangle](https://leetcode.com/problems/triangle/)
```java
class Solution {
    public int minimumTotal(List<List<Integer>> triangle) {
        if(triangle.size() == 0 || triangle.get(0).size() == 0) return 0;
        int m = triangle.size();
        int[][] dp = new int[m][m];
        for(int i = 0; i < m; i++)
            for(int j = 0; j <= i; j++)
                if(i == 0) dp[i][j] = triangle.get(i).get(j);
                else if(j == 0) dp[i][j] = dp[i-1][j] + triangle.get(i).get(j);
                else if(j == i) dp[i][j] = dp[i-1][j-1] + triangle.get(i).get(j);
                else dp[i][j] = Math.min(dp[i-1][j-1], dp[i-1][j]) + triangle.get(i).get(j);
        int res = dp[m-1][0];
        for(int r : dp[m-1])
            res = Math.min(res, r);
        return res;
    }
}
```
方程：$f(m,n) = Math.min(f(m-1,n)+f(m-1,n-1)) + v(m,n)$
T：$O(mn)$
S: $O(mn)$ 明显可以优化为 $O(m)$