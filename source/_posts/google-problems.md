---
title: google problems
tags: [leetcode]
categories: [leetcode google]
keywords: []
date: 2019-10-10 21:26:14
mathjax: true
---
记录leetcode里面google标签的题目，耗时，代码，时间空间占比.
格式为：
```markdown
# [题号]. 题目名（链接）

## [时间占比] [空间占比] 解题耗时
[源码]
[解释]
```

源码中如果提交有错误，会标记  //bugfix
<!--more-->

# HARD
## [57. Insert Interval](https://leetcode.com/problems/insert-interval/)
### 10.42 6.25 35mins
```java
class Solution {
    public int[][] insert(int[][] intervals, int[] newInterval) {
        // look for an interval start > <= newIntervar start
        // look for an interval end > <= newIntervar end
        
        TreeMap<Integer, Integer> start = new TreeMap<>();
        TreeMap<Integer, Integer> end = new TreeMap<>();
        
        for(int i = 0; i < intervals.length; i++) {
            start.put(intervals[i][0], i);
            end.put(intervals[i][1], i);
        }
        
        Integer ns1 = start.floorKey(newInterval[0]), 
        ns2 = start.ceilingKey(newInterval[0]), 
        ne1 = end.floorKey(newInterval[1]), 
        ne2 = end.ceilingKey(newInterval[1]);
        
        int ms, me;
        if(ns1 != null && intervals[start.get(ns1)][1] >= newInterval[0]) ms = start.get(ns1);
        else if (ns2 != null && intervals[start.get(ns2)][0] <= newInterval[1]) ms = start.get(ns2);
        else {
            List<int[]> res = new ArrayList<>();
            boolean inserted = false;
            for(int[] interval: intervals) {
                if(!inserted && newInterval[0] < interval[0]) {res.add(newInterval); inserted = true;}
                res.add(interval);
            }
            if(!inserted) res.add(newInterval);
            return res.toArray(new int[1][]);
        }
        
        if(ne2 != null && intervals[end.get(ne2)][0] <= newInterval[1]) me = end.get(ne2);
        else if(ne1 != null && intervals[end.get(ne1)][1] >= newInterval[0]) me = end.get(ne1);
        else {
            List<int[]> res = new ArrayList<>();
            boolean inserted = false;
            for(int[] interval: intervals) {
                if(!inserted && newInterval[0] < interval[0]) {res.add(newInterval); inserted = true;}
                res.add(interval);
            }
            if(!inserted) res.add(newInterval);
            return res.toArray(new int[1][]);
        }
        
        newInterval[0] = Math.min(newInterval[0], intervals[ms][0]);
        newInterval[1] = Math.max(newInterval[1], intervals[me][1]);
        
        List<int[]> res = new ArrayList<>();
        boolean inserted = false;
        for(int i = 0; i < intervals.length; i++) {
            int[] interval = intervals[i];
            if(!inserted && newInterval[0] < interval[0]) {res.add(newInterval); inserted = true;}
            if(i < ms || i > me) res.add(interval);
        }
        if(!inserted) res.add(newInterval);
        return res.toArray(new int[1][]);
        
    }
}
```

利用两个treeMap，分别保存start和end。然后分别取newInterval start 和 end，离的最近的两个interval。做了复杂的比较，然后进行merge。
时间和空间复杂度不好，TODO。

## [315. Count of Smaller Numbers After Self](https://leetcode.com/problems/count-of-smaller-numbers-after-self/)
### FAIL TODO
很经典的一道题目。
主要思路是对数组进行排序的过程中记录变量，得到结果。
一开始我用的quickSort，但是quickSort是不稳定的，是无法得到正确结果的（其实也没写出来）
需要使用**稳定的排序算法**，merge sort。记录index result。

## [410. Split Array Largest Sum](https://leetcode.com/problems/split-array-largest-sum/)
### FAIL 超时了
```java
class Solution {
    public int splitArray(int[] nums, int m) {
        return split(nums, 0, nums.length, m);
    }
    
    int split(int[] nums, int l, int r, int m) {
        if(l >= r ||m > r - l || m == 0) return -1;
        if(m == r - l) {
            int res = 0;
            for(int i = l; i < r; i++) res = Math.max(res, nums[i]);
            return res;
        }
        if(m == 1) {
            int res = 0;
            for(int i = l; i < r; i++) res += nums[i];
            return res;
        }
        int res = Integer.MAX_VALUE, firstSum = 0;
        
        for(int i = l; i < r; i++) {
            firstSum += nums[i];
            int tmp = split(nums, i+1, r, m-1);
            if(tmp != -1) res = Math.min(res, Math.max(tmp, firstSum));
        }
        return res;
    }
}
```

这个复杂度 = brute-force了。所以过不去。 TODO

# MEDIUM
## [200. Number of Islands](https://leetcode.com/problems/number-of-islands/)

### 5.28 84.65
```java
class Solution {
    public int numIslands(char[][] grid) {
        if(grid.length == 0) return 0;
        DSU dsu = new DSU(grid);
        for(int i = 0; i < grid.length; i++)
            for(int j = 0; j < grid[0].length; j++)
                if(grid[i][j] == '1') {
                    if(i > 0 && grid[i-1][j] == '1')
                        dsu.union(i, j, i-1, j);
                    if(j > 0 && grid[i][j-1] == '1')
                        dsu.union(i, j, i, j-1);
                }
        Map<Integer, Set<Integer>> map = new HashMap<>();
        for(int i = 0; i < grid.length; i++)
            for(int j = 0; j < grid[0].length; j++) {
                int[] p = dsu.find(i, j);
                if(p[0] != -1)
                    map.computeIfAbsent(p[0], x->new HashSet<>()).add(p[1]);
            }
        int res = 0;
        for(Map.Entry<Integer, Set<Integer>> e: map.entrySet())
            for(Integer i: e.getValue()) res++;
        return res;
    }
    
    class DSU {
        int[][] rows, cols;
        DSU(char[][] grid) {
            int r = grid.length, c = grid[0].length;
            rows = new int[r][c];
            cols = new int[r][c];
            for(int i = 0; i < r; i++)
                for(int j = 0; j < c; j++)
                    if(grid[i][j]=='1') {
                        rows[i][j] = i;
                        cols[i][j] = j;
                    } else {
                        rows[i][j] = -1;
                        cols[i][j] = -1;
                    }
        }
        
        int[] find(int i, int j) {
            if(rows[i][j] == -1) return new int[]{-1, -1};
            if(i == rows[i][j] && j == cols[i][j]) return new int[]{i, j};
            
            int[] res = find(rows[i][j], cols[i][j]);
            rows[i][j] = res[0];
            cols[i][j] = res[1];
            return res;
        }
        
        void union(int i1, int j1, int i2, int j2) {
            int[] p1 = find(i1, j1), p2 = find(i2, j2);
            rows[p1[0]][p1[1]] = p2[0];
            cols[p1[0]][p1[1]] = p2[1];
        }
    }
}
```

二维的union find。有了前面DSU的经验，这次基本结构构建的比较快。但是时间复杂度较高。TODO。需要看看别人的做法

## [394. Decode String](https://leetcode.com/problems/decode-string/)
### 71.16 100.00 44mins
```java
class Solution {
    public String decodeString(String s) {
        Stack<String> st = new Stack<>();
        Tokenizer t = new Tokenizer(s);
        while(!t.end()) {
            String token = t.next();
            if("]".equals(token)) {
                String word = st.pop();
                st.pop();
                String num = st.pop();
                StringBuilder sb = new StringBuilder();
                int number = Integer.valueOf(num);
                for(int i = 0; i < number; i++) sb.append(word);
                // bugfix
                while(!st.isEmpty() && Character.isAlphabetic(st.peek().charAt(0)))
                    sb.insert(0, st.pop());
                st.push(sb.toString());
            }
            // bugfix
            else if (Character.isAlphabetic(token.charAt(0))) {
                StringBuilder sb = new StringBuilder();
                sb.insert(0, token);
                while(!st.isEmpty() && Character.isAlphabetic(st.peek().charAt(0)))
                    sb.insert(0, st.pop());
                st.push(sb.toString());
            }
            else
                // [ number word
                st.push(token);
        }
        // bugfix
        StringBuilder res = new StringBuilder();
        while(!st.isEmpty())
            res.insert(0, st.pop());
        return res.toString();
    }
    
    class Tokenizer {
        int cur;
        String s;
        Tokenizer(String s) {
            this.s = s;
            cur = 0;
        }
        
        boolean end() {
            return cur == s.length();
        }
        
        String next() {
            if(s.charAt(cur) == '[') {
                cur++;
                return "[";
            }
            else if(s.charAt(cur) == ']') {
                cur++;
                return "]";
            }
            else if(Character.isDigit(s.charAt(cur))) {
                int start = cur;
                for(; cur < s.length(); cur++) 
                    if(!Character.isDigit(s.charAt(cur)))
                        break;
                return s.substring(start, cur);
            } else {
                int start = cur;
                for(; cur < s.length(); cur++) 
                    if(!Character.isAlphabetic(s.charAt(cur)))
                        break;
                return s.substring(start, cur);
            }
        }
    }
}
```

注意bugfix一栏。这里主要是没有考虑一种case 即 "abc" 或者 "5[abc]bc" 这种没有中括号的场景。需要注意。

## [981. Time Based Key-Value Store](https://leetcode.com/problems/time-based-key-value-store/)
### FAIL 20mins
```java
class TimeMap {

    /** Initialize your data structure here. */
    
    Map<String, Node> map = new HashMap<>();
    
    public TimeMap() {
        
    }
    
    public void set(String key, String value, int timestamp) {
        if(map.get(key) == null) {
            Node n = new Node();
            n.value = value;
            n.timestamp = timestamp;
            map.put(key, n);
        } else {
            map.get(key).insert(value, timestamp);
        }
    }
    
    public String get(String key, int timestamp) {
        if(map.get(key) == null) return "";
        return map.get(key).get(timestamp);
    }
    
    class Node {
        String value;
        int timestamp;
        Node left, right;
        
        String get(int ts) {
            if(timestamp == ts) return value;
            if(timestamp > ts)
                if(left == null) return "";
                else return left.get(ts);
            else
                if(right == null) return value;
                else if(right.timestamp > ts) return value;
                else return right.get(ts);
        }
        
        void insert(String v, int ts){
            if(ts == timestamp) value = v;
            if(ts > timestamp)
                if(right == null) {
                    Node n = new Node();
                    n.value = v;
                    n.timestamp = ts;
                    right = n;
                } else right.insert(v, ts);
            if(ts < timestamp)
                if(left == null) {
                    Node n = new Node();
                    n.value = v;
                    n.timestamp = ts;
                    left = n;
                } else left.insert(v, ts);
        }
    }
}

/**
 * Your TimeMap object will be instantiated and called as such:
 * TimeMap obj = new TimeMap();
 * obj.set(key,value,timestamp);
 * String param_2 = obj.get(key,timestamp);
 */
```

自己写了一个bst。但是由于不平衡，所以会超时。看了答案，答案用了jdk中的binarySearch和treeMap的floorKey。TODO：正确答案 和 平衡搜索树模板

## [163. Missing Ranges](https://leetcode.com/problems/missing-ranges/)

### 100 100 50mins
```java
// 设计各种test case，比如 [-1] 0, -1; [-1] -1 0
// 考虑+1时候的溢出 比如 [2147483647] 0 2147483647
class Solution {
    public List<String> findMissingRanges(int[] nums, int lower, int upper) {
        List<String> res = new ArrayList<>();
        if(nums.length == 0) {
            res.add(outputInclusive(lower, upper));
            return res;
        }
        
        int ind1 = bSearchLess(nums, lower, 0, nums.length-1);
        int ind2 = bSearchMore(nums, upper, 0, nums.length-1);
        
        
        if(ind1 == -1) {
            if(lower+1 < nums[0] && notMax(lower)) res.add(""+lower+"->"+(nums[0]-1));
            else if(lower+1 == nums[0] && notMax(lower)) res.add(""+lower);
            // bugfix
            ind1 = 0;
        }
        boolean more = false;
        if(ind2 == -1) {
            more = true;
            ind2 = nums.length-1;
        } else nums[ind2] = upper;
        for(int i = ind1; i < ind2; i++)
            if(nums[i] + 1 < nums[i+1] && notMax(nums[i]))
                if(nums[i] + 2 == nums[i+1]) res.add(""+(nums[i]+1));
                else if(notMax(nums[i])) res.add(""+(nums[i]+1)+"->"+(nums[i+1]-1));
        if(more)
            if(nums[nums.length-1] + 1 < upper && notMax(nums[nums.length-1])) res.add(""+(nums[nums.length-1]+1) + "->" + upper);
            else if(nums[nums.length-1] + 1 == upper) res.add("" + upper);
        return res;
    }
    
    int bSearchLess(int[] nums, int lower, int start, int end) {
        if(nums[start] >= lower) return -1;
        if(nums[end] < lower) return end;
        int mid = start + (end-start)/2;
        if(nums[mid] >= lower) return bSearchLess(nums, lower, start, mid-1);
        else return bSearchLess(nums, lower, mid, end);
    }
    
    int bSearchMore(int[] nums, int upper, int start, int end) {
        if(nums[end] <= upper) return -1;
        if(nums[start] > upper) return start;
        if(start > end) return -1;
        int mid = start + (end-start)/2;
        if(nums[mid] <= upper) return bSearchMore(nums, upper, mid+1, end);
        else return bSearchMore(nums, upper, start, mid);
    }
    
    String outputInclusive(int l, int r) {
        if(l == r) return ""+l;
        else return ""+l+"->"+r;
    }
    
    boolean notMax (int i) {
        return (i ^ (i+1)) > 0;
    }
}
```
TODO 看别人的写法。

这里需要注意非常多的testcase，尤其是溢出问题，需要格外注意。

## [221. Maximal Square](https://leetcode.com/problems/maximal-square/)

### FAIL
一开始想成了那个求最大面积的题目了，发现并不完全一样。而且忘记了求最大面积最后的方法。TODO

### dp
```java
public class Solution {
    public int maximalSquare(char[][] matrix) {
        int rows = matrix.length, cols = rows > 0 ? matrix[0].length : 0;
        int[][] dp = new int[rows + 1][cols + 1];
        int maxsqlen = 0;
        for (int i = 1; i <= rows; i++) {
            for (int j = 1; j <= cols; j++) {
                if (matrix[i-1][j-1] == '1'){
                    dp[i][j] = Math.min(Math.min(dp[i][j - 1], dp[i - 1][j]), dp[i - 1][j - 1]) + 1;
                    maxsqlen = Math.max(maxsqlen, dp[i][j]);
                }
            }
        }
        return maxsqlen * maxsqlen;
    }
}
```

值得注意的是 dp可以多建立一行的空数据。TODO 自己实现

## [222. Count Complete Tree Nodes](https://leetcode.com/problems/count-complete-tree-nodes/)
### 8.99 92.68 6mins
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
    public int countNodes(TreeNode root) {
        int res = 0;
        Queue<TreeNode> q = new LinkedList<>();
        q.offer(root);
        while(q.size() != 0) {
            TreeNode n = q.poll();
            if(n == null) break;
            res++;
            q.offer(n.left);
            q.offer(n.right);
        }
        return res;
    }
}
```

TODO 使用队列可以很简单的实现。但是很明显，没有利用完整二叉树的特性，待优化。TODO。

思路：
+ 其实这是一道 binary search 的题目
+ 利用最left和最right，让树分别向左还是向右走
TODO 二叉树每一层的个数

## [247. Strobogrammatic Number II](https://leetcode.com/problems/strobogrammatic-number-ii/)

### 18.76 57.14 60mins
```java
class Solution {
    public List<String> findStrobogrammatic(int n) {
        return findN(n);
    }
    
    List<String> findN(int n) {
        List<String> res = new ArrayList<>();
        if(n <= 0) return res;
        if(n == 1) {
            res.add("0");
            res.add("1");
            res.add("8");
            return res;
        }
        if(n == 2) {
            res.add("11");
            res.add("69");
            res.add("88");
            res.add("96");
            return res;
        }
        if(n%2 == 1) {
            List<String> lastRes = findN(n-1);
            for(String s: lastRes) {
                res.add(s.substring(0, s.length()/2)+"0"+s.substring(s.length()/2));
                res.add(s.substring(0, s.length()/2)+"1"+s.substring(s.length()/2));
                res.add(s.substring(0, s.length()/2)+"8"+s.substring(s.length()/2));
            }
            // 不能计算其他的了 否则会重复
            return res;
        } else {
            int tmpN = 2;
            while(n-tmpN >= 0) {
                List<String> lastRes = findN(n-tmpN);
                if(lastRes.size() == 0) lastRes.add("");
                for(String s: lastRes) {
                    res.add("1"+rep0(tmpN/2-1)+s+rep0(tmpN/2-1)+"1");
                    res.add("6"+rep0(tmpN/2-1)+s+rep0(tmpN/2-1)+"9");
                    res.add("8"+rep0(tmpN/2-1)+s+rep0(tmpN/2-1)+"8");
                    res.add("9"+rep0(tmpN/2-1)+s+rep0(tmpN/2-1)+"6");
                }
                tmpN += 2;
            }
            return res;
        }
        
    }
    
    String rep0(int n) {
        String res = "";
        for(int i = 0; i < n; i++) res += "0";
        return res;
    }
}
```

花费的时间比较长。而且时间效率低，TODO.
这里需要注意的点在于，0 这个数字的特殊性。抽象来看
+ 最后结果的输出中，开头和结尾不应该包括0
+ 但是计算的过程中，需要对0进行处理。


## [1007. Minimum Domino Rotations For Equal Row](https://leetcode.com/problems/minimum-domino-rotations-for-equal-row/)

### FAIL
```java
class Solution {
    public int minDominoRotations(int[] A, int[] B) {
        if(A.length <= 1) return 0;
        int resA = Integer.MAX_VALUE, resB = Integer.MAX_VALUE;
        int res1 = flip(A, B);
        if(res1 != -1) resA = res1;
        int res2 = flip(B, A);
        if(res2 != -1) resB = res2;
        
        if(res1 == -1 && res2 == -1) return -1;
        return Math.min(resA, resB);
    }
    
    // -1 B -> A
    int flip(int[] A, int[] B){
        int flag = A[0], res = 0;
        for(int i = 1; i < A.length; i++)
            if(A[i] == flag) continue;
            else if (B[i] == flag) res++;
            else return -1;
        return res;
    }
}
```
这里做的不对。因为我只考虑了两种情况：全部转成A[0]且，面向上；全部转成B[0]，且面向下。
还需要考虑另外两种：全部转成A[0]，且面向下；全部转B[0]，且面向上。

### 64.07 100.00
```java
class Solution {
    public int minDominoRotations(int[] A, int[] B) {
        if(A.length <= 1) return 0;
        int res1 = flip(A, B);
        int res2 = flip(B, A);
        if(res1 == -1 && res2 == -1) return -1;
        if(res1 == -1 || res2 == -1) return res1 + res2 + 1;
        return Math.min(res1, res2);
    }
    
    // -1 B -> A
    int flip(int[] A, int[] B){
        int flag = A[0], res = 0, tmpRes = 0;
        for(int i = 0; i < A.length; i++)
            if(A[i] == flag) continue;
            else if (B[i] == flag) tmpRes++;
            else {
                tmpRes = -1;
                break;
            };
        res = tmpRes;
        tmpRes = 0;
        for(int i = 0; i < A.length; i++)
            if(B[i] == flag) continue;
            else if (A[i] == flag) tmpRes++;
            else {
                tmpRes = -1;
                break;
            }
        if(res == -1 && tmpRes == -1) return -1;
        if(res == -1 || tmpRes == -1) return res + tmpRes + 1;
        return Math.min(res, tmpRes);
    }
}
```

写的很丑陋。需要改进。TODO.

# EASY

## [05. Isomorphic Strings](https://leetcode.com/problems/isomorphic-strings/)

### 6.29 100.00 7mins
```java
class Solution {
    public boolean isIsomorphic(String s, String t) {
        char[] cs1 = s.toCharArray();
        char[] cs2 = t.toCharArray();
        
        Map<Character, Character> m = new HashMap<>();
        // bugfix 1 for 1
        Map<Character, Character> m2 = new HashMap<>();
        for(int i = 0; i < cs1.length; i++) {
            if(m.containsKey(cs1[i])) {
                if(!m.get(cs1[i]).equals(cs2[i])) return false;
            }
            if(m2.containsKey(cs2[i])) {
                if(!m2.get(cs2[i]).equals(cs1[i])) return false;
            }
            
            m.put(cs1[i], cs2[i]);
            m2.put(cs2[i], cs1[i]);
        }
        return true;
    }
}
```

TODO 时间太差。

## [299. Bulls and Cows](https://leetcode.com/problems/bulls-and-cows/)
### 23.16 100.00 12mins
```java
class Solution {
    public String getHint(String secret, String guess) {
        int A=0, B=0;
        char[] s1 = secret.toCharArray(), s2 = guess.toCharArray();
        Set<Integer> inds = new HashSet<>();
        Map<Character, Integer> m = new HashMap<>();
        
        for(int i = 0; i < s1.length; i++)
            if(s1[i] == s2[i]) {
                A++;
                inds.add(i);
            } 
            else if(!m.containsKey(s1[i])) m.put(s1[i], 1);
            else m.put(s1[i], m.get(s1[i]) + 1);
        
        for(int i = 0; i < s2.length; i++)
            if(inds.contains(i)) continue;
            else if(m.containsKey(s2[i])) {
                B++;
                if(m.get(s2[i]) == 1) m.remove(s2[i]);
                else m.put(s2[i], m.get(s2[i]) - 1);
            }
        return ""+A+"A"+B+"B";
    }
}
```
主要是用map和set。但是时间上效率不够高，todo。

## [482. License Key Formatting](https://leetcode.com/problems/license-key-formatting/)

### 21.92 100.00 8mins
```java
class Solution {
    public String licenseKeyFormatting(String S, int K) {
        char[] s = S.toCharArray();
        StringBuilder res = new StringBuilder();
        int n = 0;
        for(int i = s.length-1; i >= 0; i--) {
            char c = s[i];
            if(c == '-') continue;
            res.insert(0, Character.toUpperCase(c));
            if(++n == K && i != 0) {
                res.insert(0, '-');
                n = 0;
            }
        }
        // bugfix
        if(res.length() != 0 && res.charAt(0) == '-') return res.substring(1);
        return res.toString();
    }
}
```

有个bug，没有考虑 '-' 开头的情况。
但是时间效率太差了，看效率分布图，需要优化。 TODO
{% asset_img 482.png %}
