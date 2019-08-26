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

# [560. Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)

## 30.78 97.83 38mins

```java
class Solution {
    class Num {
        int index;
        int val;
        Num(int val, int index){
            this.val = val;
            this.index = index;
        }
    }
    
    public int subarraySum(int[] nums, int k) {
        if(nums.length == 0) return 0;
        if(nums.length == 1)
            if(nums[0] == k) return 1;
            else return 0;
        
        Num[] classNums = new Num[nums.length];
        classNums[0] = new Num(nums[0], 0);
        for(int i = 1; i < nums.length; i++) {
            nums[i] += nums[i-1];
            classNums[i] = new Num(nums[i], i);
        }
        
        //bugfix
        if(k >= 0)
            Arrays.sort(classNums, (n1, n2)->Integer.compare(n1.val, n2.val));
        else
            Arrays.sort(classNums, (n1, n2)->Integer.compare(n2.val, n1.val));
        
        int res = 0;
        // i >= j
        //bugfix
        if(k >= 0)
        for(int i = 0; i < nums.length;i++)
            for(int j = i; j < nums.length; j++){
                if(i == j && classNums[i].val == k) res++;
                if(classNums[j].val - classNums[i].val > k) break;
                if(classNums[j].val - classNums[i].val == k && classNums[j].index > classNums[i].index) res++;
            }
        else
        for(int i = 0; i < nums.length;i++)
            for(int j = i; j < nums.length; j++){
                if(i == j && classNums[i].val == k) res++;
                if(classNums[j].val - classNums[i].val < k) break;
                if(classNums[j].val - classNums[i].val == k && classNums[j].index > classNums[i].index) res++;
            }
        return res;
    }
}
```

**hint**: 在brute-force基础上做改进： 做累加 --> 排个序（可以少比几次）-->依次相减，但要判断index前后。

总结：可以看出这么写很容易出错。而且时间复杂度是N^2，空间复杂度是N。还不如直接累加完之后，brute-force。待优化。

用map是比较好的写法，可以看到solution里面有。很不错。

# [31. Next Permutation](https://leetcode.com/problems/next-permutation/)

## 100.00 31.00 40min
```java
class Solution {
    public void nextPermutation(int[] nums) {
        if(nums.length <= 1) return; 
        int i = nums.length - 2;
        while( i >= 0){
            if(nums[i] == nums[i+1]) i--;
            // bugfix
            else break;
        }
        
        if(i == -1) return;
        if(nums[i] < nums[i+1]){
            swap(nums, i, i+1);
            return;
        }
        
        while( i >= 0){
            if(nums[i] >= nums[i+1]) i--;
            // bugfix
            else break;
        }
        if(i == -1){
            swapAll(nums, 0, nums.length-1);
            return;
        }
        
        // wrong here, bugfix
        int j = findBigger(nums, i+1, nums[i]);
        swap(nums, i, j);
        swapAll(nums, i+1, nums.length-1);
    }
    
    private int findBigger(int[] nums, int start, int targ) {
        int result = start;
        for(int i = start; i < nums.length; i++){
            if(nums[i] <= targ) break;
            if(nums[i] <= nums[result]) result = i;
        }
        return result;
    }
    
    
    private void swap(int[] nums, int i, int j){
        int tmp = nums[i];
        nums[i] = nums[j];
        nums[j] = tmp;
    }
    
    private void swapAll(int[] nums, int start, int end){
        for(int i = start, j = end;i<j;i++,j--){
            swap(nums, i, j);
        }
    }
}
```
**tips**:
思考四种情况，分别处理。注意第四种，一开始处理方式不对，耽误了很长时间：找到后面大于flag的第一个值，swap后，把后面的全部swap。
{% asset_img 31.png %}

# [986. Interval List Intersections](https://leetcode.com/problems/interval-list-intersections/)

## 99.93 78.38 15mins
```java
class Solution {
    public int[][] intervalIntersection(int[][] A, int[][] B) {
        if(A.length == 0 || B.length == 0) return new int[0][];
        
        List<int[]> res = new ArrayList<>();
        for(int i = 0, j = 0; i < A.length && j < B.length;) {
            int[] tmp = inter(A[i], B[j]);
            if(null != tmp) res.add(tmp);
            if(A[i][1] >= B[j][1]) j++;
            else i++;
        }
        
        return res.toArray(new int[1][]);
    }
    
    int[] inter(int[] a, int[] b) {
        // 判断是否存在交集，利用线段中心距离 和 线段长度之和做比较
        if( Math.abs((a[0]+a[1]) - (b[0]+b[1])) <= (b[1]-b[0] + a[1] - a[0])){
            return new int[]{Math.max(a[0], b[0]), Math.min(a[1], b[1])};
        } else return null;
    }
}
```

# [98. Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/)
## 100.00 79.54 20mins
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public boolean isValidBST(TreeNode root) {
        if(root == null) return true;
        boolean leftB = true, rightB = true;
        TreeNode lr = leftRight(root);
        TreeNode rl = rightLeft(root);
        if(root.left != null)
            // bugfix
            if(root.left.val < root.val && (lr == null || lr.val < root.val)) leftB = isValidBST(root.left);
            else return false;
        if(root.right != null)
            // bugfix
            if(root.right.val > root.val && (rl == null || rl.val > root.val)) rightB = isValidBST(root.right);
            else return false;
        return leftB&&rightB;
    }
    
    TreeNode leftRight(TreeNode root) {
        if(root.left == null) return null;
        TreeNode result = root.left;
        while(result.right != null) result = result.right;
        return result;
    }
    
    TreeNode rightLeft(TreeNode root) {
        if(root.right == null) return null;
        TreeNode result = root.right;
        while(result.left != null) result = result.left;
        return result;
    }
}
```
**思路**：
+ 对于图1的三元节点，保证 左<中<右 的大小顺序
+ 左子树的最右边节点 < 中 < 右子数的最左边节点哈罗芬瑟到了可热了狂热冷酷日

# [199. Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/)

##  98.32 100.00 12mins
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public List<Integer> rightSideView(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        if(root == null) return res;
        
        TreeNode sentinal = new TreeNode(0);
        TreeNode lastNode = null;
        Queue<TreeNode> q = new LinkedList<>();
        
        q.offer(root);
        q.offer(sentinal);
        
        while(q.size() != 0) {
            TreeNode tmp = q.poll();
            if(tmp != sentinal) lastNode = tmp;
            else {
                res.add(lastNode.val);
                if(q.size() != 0) q.offer(tmp);
                continue;
            }
            if(tmp.left != null) q.offer(tmp.left);
            if(tmp.right != null) q.offer(tmp.right);
        }
        
        return res;
    }
}
```

使用队列和哨兵位，注意哨兵归放到队尾的时候要判断一下当前队列是否为空

# [621. Task Scheduler](https://leetcode.com/problems/task-scheduler/)

## AFTER FAIL: 41.12 86.76 26mins
```java
class Solution {
    public int leastInterval(char[] tasks, int n) {
        if(tasks.length == 0) return 0;
        int[] data = new int[26];
        for(char c: tasks)
            data[c-'A']++;
        PriorityQueue<Integer> q = new PriorityQueue<>(Collections.reverseOrder());
        for(int i: data)
            if(i != 0)
                q.offer(i);
        int result = 0;
        while(q.size() != 0) {
            List<Integer> buf = new ArrayList<>();
            // bugfix n+1 ||
            for(int i = 0; i < n+1 && (buf.size()!=0 || q.size()!=0); i++){
                result ++;
                //bugfix
                if(q.size() != 0) {
                    int t = q.poll();
                    if(--t > 0) buf.add(t);
                }
            }
            for(Integer i: buf) 
                q.offer(i);
        }
        
        return result;
    }
}
```

使用PriorityQueue做。第一次没想出来。
**思路**：
+ 优先队列整理数据
+ 依次取前n个数字，放到结果中，如果数字用完就不放回去，如果没用完则放回队列，重复此操作。
+ 注意几个bugfix细节点

最后的时间只有42，还有优化空间。复杂度是 nlog(n)?