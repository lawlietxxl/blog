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
# [题号]. 题目名（链接）

## [时间占比] [空间占比] 解题耗时
[源码]
[解释]
```
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

# [91. Decode Ways](https://leetcode.com/problems/decode-ways/)

##  98.63% 100.00% 43min
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