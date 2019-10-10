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

# EASY