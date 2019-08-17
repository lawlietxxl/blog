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

# [215. Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/)

## 32.77 78.76 40mins
```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        return sort(nums, 0, nums.length-1, nums.length-k);
    }
    
    private int sort(int[] nums, int l, int r, int i){
        int mid = partition(nums, l, r);
        if(mid == i) return nums[mid];
        if(mid < i) return sort(nums, mid+1, r, i);
        return sort(nums, l, mid-1, i);
    }
    
    private int partition(int[] nums, int l, int r){
        //bugfix
        if(l == r) return l;
        int i = l, j = r + 1;
        while(true){
            while(nums[++i] < nums[l])
                if(i == r) break;
            while(nums[l] < nums[--j])
                if(j == l) break;
            if(i >= j) break;
            exchange(nums, i, j);
        }
        exchange(nums, l, j);
        return j;
    }
    
    private void exchange(int[] nums, int i, int j){
        int tmp = nums[i];
        nums[i] = nums[j];
        nums[j] = tmp;
    }
}
```

使用快速排序的变通方法可以得到结果，注意控制边界（bugfix）。
**tips***：
+ 快排内有玄机，详情见常用算法模板
+ 快拍时间复杂度仅仅为30+，需要优化

# [56. Merge Intervals](https://leetcode.com/problems/merge-intervals/)

## 25.19 36.23 18mins
```java
class Solution {
    public int[][] merge(int[][] intervals) {
        if(intervals.length <= 1) return intervals;
        //bugfix wrong: Collections.sort
        Arrays.sort(intervals, (l1, l2)->Integer.compare(l1[0], l2[0]));
        
        List<int[]> result = new ArrayList<>();
        result.add(intervals[0]);
        
        for(int i = 1; i < intervals.length; ++i){
            int[] in = intervals[i];
            int[] ou = result.get(result.size()-1);
            if(ou[1] < in[0]) result.add(in);
            else {
                result.remove(result.size()-1);
                result.add(new int[]{ou[0], Math.max(ou[1], in[1])});
            }
        }
        
        int[][] resultArray = new int[result.size()][];
        for(int i = 0; i < result.size(); ++i){
            resultArray[i] = result.get(i);
        }
        return resultArray;
    }
}
```

**思路**
+ 按照第一位排序，这样排序好之后，就可以按照顺序依次merge了
+ TODO: 空间、时间都不优，待优化

## 95.63 71.02
```java
class Solution {
	public int[][] merge(int[][] intervals) {
        int length=intervals.length;
        if(length<=1)
            return intervals;
    
        int[] start = new int[length];
        int[] end = new int[length];
        for(int i=0;i<length;i++){
            start[i]=intervals[i][0];
            end[i]=intervals[i][1];
        }
        Arrays.sort(start);
        Arrays.sort(end);
        int startIndex=0;
        int endIndex=0;
        List<int[]> result = new LinkedList<>();
        while(endIndex<length){
            //as endIndex==length-1 is evaluated first, start[endIndex+1] will never hit out of index
            if(endIndex==length-1 || start[endIndex+1]>end[endIndex]){
                result.add(new int[]{start[startIndex],end[endIndex]});
                startIndex=endIndex+1;
            }
            endIndex++;
        }
        return result.toArray(new int[result.size()][]);
    }
}
```

**tips**:
+ 看上去似乎经过了两次排序，但是在java中，对基础类型排序要比对object排序快很多。
+ 思路：
    + 1.start一组排序，end一组排序；类似一个栈，当出现的start和end相等的时候就可以进行结果插入；
    + 2.或者是算法中所示，取endIndex下一个start为计算标识，如果大就进行上一个interval的结果插入

{% asset_img 56.png %}

# [253. Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/)

## 100.00 75.64
```java
class Solution {
    public int minMeetingRooms(int[][] intervals) {
        if(intervals.length <= 1) return intervals.length;
        int[] start = new int[intervals.length];
        int[] end = new int[intervals.length];
        for(int i = 0; i < intervals.length; i++){
            start[i] = intervals[i][0];
            end[i] = intervals[i][1];
        }
        Arrays.sort(start);
        Arrays.sort(end);
        
        int result = 0, tmpResult = 0;
        // i means startIndex, j means endIndex
        int i = 0,j = 0;
        
        while(i < start.length){
            if(end[j] <= start[i]){
                tmpResult --;
                j++;
            }else{
                tmpResult ++;
                i ++;
                result = Math.max(result, tmpResult);
            }
        }
        return result;
    }
}
```

**思路**:此题目跟[56. Merge Intervals](https://leetcode.com/problems/merge-intervals/)思路一样
+ start排序，end排序
+ 两者取比较小的数进行计数，start小，则result++；end小于等于，则result--。（注意取临时变量）

# [238. Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/)

## 100.00 92.13 15min
```java
class Solution {
    public int[] productExceptSelf(int[] nums) {
        int[] n1 = new int[nums.length];
        int[] n2 = new int[nums.length];
        n1[0] = 1;
        for(int i = 1; i < nums.length; i++) {
            // bugfix
            n1[i] = nums[i-1] * n1[i-1];
        }
        n2[n2.length-1] = 1;
        for(int i = 1; i < nums.length; i++) {
            n2[n2.length-1-i] = n2[n2.length-i] * nums[nums.length-i];
        }
        
        int[] result = new int[nums.length];
        for(int i = 0; i < result.length; i++) {
            result[i] = n1[i] * n2[i];
        }
        
        return result;
    }
}
```

**tips**:
建立两个数组，一个表示前面的乘积，一个表示后面的乘积。(注意数组长度)
按照提示思考一下 constant 空间复杂度的解法

## 100.00 100.00 8mins
```java
class Solution {
    public int[] productExceptSelf(int[] nums) {
        int[] result = new int[nums.length];
        result[0] = 1;
        for(int i = 1; i < nums.length; ++i){
            result[i] = result[i-1] * nums[i-1];
        }
        
        int mul = 1;
        for(int i = 1; i < nums.length; ++i){
            mul *= nums[nums.length-i];
            result[nums.length-1-i] *= mul;
        }
        return result;
    }
}
```
constant 空间复杂度的解法