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