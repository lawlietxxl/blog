---
title: leetcode中facebook的题目 -- medium篇
tags: []
categories: []
keywords: []
date: 2019-08-15 23:34:31
---
同上。
<!--more-->

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

# [15. 3Sum](https://leetcode.com/problems/3sum/)
## TLE -- wrong answer
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

# [973. K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/)

## WRONG ANSER
没有注意审题，吭哧别都写了半天结果没看清楚题目……需要注意的是
1、返回K个最近的点，是全部返回，而不是只返回一个
2、保证只有唯一一个解（说明K+1这个点一定比K大）

## 36.35 90.68 11mins
```java
class Solution {
    public int[][] kClosest(int[][] points, int K) {
        PriorityQueue<PointData> q = new PriorityQueue<>((pd1, pd2)->{
            if(pd1.dis == pd2.dis) return 0;
            if(pd1.dis < pd2.dis) return 1;
            else return -1;
        });
        
        for(int[] point: points){
            PointData pd = new PointData(point);
            q.offer(pd);
            while(q.size() > K) q.poll();
        }
        
        int[][] result = new int[q.size()][];
        int i = 0;
        while(q.size() != 0){
            result[i++] = q.poll().point;
        }
        return result;
    }
    
    class PointData{
        public PointData(int[] point){
            this.point = point;
            this.dis = point[0]*point[0] + point[1]*point[1];
        }
        
        public int dis;
        public int[] point;
    }
}
```

使用java的PriorityQueue能够很快给出结果，但是时间上没有优势，待优化。
