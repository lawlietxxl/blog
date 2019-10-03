---
title: leetcode中facebook的题目 -- hard篇
tags: []
categories: []
keywords: []
date: 2019-08-15 23:34:39
mathjax: true
---
同上。
<!--more-->

# [158. Read N Characters Given Read4 II - Call multiple times](https://leetcode.com/problems/read-n-characters-given-read4-ii-call-multiple-times/)

## 100.00% 100.00% 60mins

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
    
    private char[] innerBuf = new char[4];
    private int innerOffset = 0;
    private int innerLength = 0;
    
    private boolean EOF = false;
    
    public int read(char[] buf, int n) {
        // result, n, innerOffset, readFromInner
        if(n <= 0 || EOF) return 0;
        int bufOffset = 0;
        
        // read from innerBuffer
        int min;
        min = Math.min(innerLength - innerOffset, n);
        for(int i = 0; i < min; i ++){
            buf[bufOffset++] = innerBuf[innerOffset++];
        }
        n -= min;
        if(innerOffset == innerLength){
            innerOffset = 0;
            innerLength = 0;
        }
        //bugfix
        if(n == 0) return bufOffset;
        
        // read from file
        int iter = n/4;
        for(int i = 0; i < iter; i++){
            int tmpResult = read4(innerBuf);
            copyArray(buf, bufOffset, innerBuf, 0, tmpResult);
            n -= tmpResult;
            bufOffset += tmpResult;
            if(tmpResult < 4){
                EOF = true;
                return bufOffset;
            }
        }
        // bugfix
        if(n == 0) return bufOffset;
        
        // write to innerBuffer
        // left is n
        
        //bugfix 没考虑tmpResult为0的情况
        int tmpResult = read4(innerBuf);
        min = Math.min(tmpResult, n);
        copyArray(buf, bufOffset, innerBuf, 0, min);
        bufOffset += min;
        if(tmpResult <= n){
            EOF = true;
        }else{
            innerOffset = n;
            innerLength = tmpResult;
        }
        return bufOffset;
    }
    
    public void copyArray(char[] dest, int destOffset, char[] src, int srcOffset, int length){
        for(int i = 0; i < length; i ++){
            dest[destOffset + i] = src[srcOffset + i];
        }
    }
}
```

有一次提交错误，存在bug，最后的时候没考虑tmpResult为0的情况，导致innerLength设置为0了。
思路：
+ 设置内部缓存，同时记录缓存offset和缓存长度
+ 顺序为先读缓存，再读文件，最后再写缓存。注意更新各种本地变量
感觉有点像tcp协议操作系统之类的实现

# [23. Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)

## 98.74 27.33 18mins
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        //bugfix
        if(lists.length == 0) return null;
        
        return merge(lists, 0, lists.length-1);
    }
    
    private ListNode merge(ListNode[] lists, int lo, int hi){  
        if(lo == hi) return lists[lo];
        int mid = lo + (hi - lo)/2;
        ListNode l1 = merge(lists, lo, mid);
        ListNode l2 = merge(lists, mid+1, hi);
        
        ListNode result = new ListNode(0);
        ListNode tail = result;
        
        while(true){
            if(l1 == null && l2 == null) break;
            if(l1 == null && l2 != null){
                tail.next = l2;
                break;
            }
            if(l1 != null && l2 == null){
                tail.next = l1;
                break;
            }
            
            if(l1.val < l2.val) {
                tail.next = l1;
                l1 = l1.next;
                tail.next.next = null;
                tail = tail.next;
            }else{
                tail.next = l2;
                l2 = l2.next;
                tail.next.next = null;
                tail = tail.next;
            }
        }
        
        result = result.next;
        return result;
    }
}
```
使用分治法的思想，一半一半的处理，把复杂度降低log。
但是内存占用较高，而且并不会分析算法复杂度。

# [301. Remove Invalid Parentheses](https://leetcode.com/problems/remove-invalid-parentheses/) (FAIL)

## DFS ANSWER
```java
class Solution {
    public List<String> removeInvalidParentheses(String s) {
        List<String> res = new ArrayList<>();
        
        dfs(s, res, 0, 0, '(', ')');
        return res;
    }
    
    void dfs(String s, List<String> res, int i_last, int j_last, char c1, char c2) {
        int stack = 0;
        for(int i = i_last; i < s.length(); ++i){
            if(s.charAt(i) == c1) stack++;
            if(s.charAt(i) == c2) stack--;
            if(stack >= 0) continue;
            // stack < 0, dfs all of before
            //bugfix
            for(int j = j_last; j <= i; j++){
                if(s.charAt(j) == c2 && (j == j_last || s.charAt(j-1) != c2))
                    dfs(s.substring(0, j) + s.substring(j+1, s.length()), res, i, j , c1, c2);
            }
            //bugfix
            return;
        }
        //后缀已经正确，现在开始处理前缀
        //倒排序
        String reversed = new StringBuilder(s).reverse().toString();
        if(c1 == '(')
            dfs(reversed, res, 0, 0, ')', '(');
        else
            res.add(reversed);
    }
}
```

**tips**
+ 要看到本质。删除几个字符形成正确的结构 --> 有多种删除方法 --> 本质是组合的问题 --> 组合的问题都可以用DFS解。
+ 结论。组合问题 = DFS问题（比如八皇后）
+ 先解决后缀的正确性，再倒序后处理前缀。
+ 上文思路见图。
    + 注意 i 和 j：i前面的字符串已经是正确的了，j是为了防止重复。所以i 和 j要记录下来，不断移动。
{% asset_img 301.png %}

# [124. Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/)

## 5.04 5.95 1h TODO
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
    public int maxPathSum(TreeNode root) {
        if(root == null) return 0;
        
        // exclude current root
        return traveralTree(root);
    }
    private int traveralTree(TreeNode root) {
        int l1 = computeLeft(root);
        int r1 = computeRight(root);
        int res = (l1+r1) - root.val;
        if(root.left != null) {
            int l = traveralTree(root.left);
            res = res > l ? res : l;
        }
        if(root.right != null) {
            int r = traveralTree(root.right);
            res = res > r ? res : r;
        }
        return res;
    }
    
    // include root, root should not be null
    private int computeLeft(TreeNode root) {
        int res = root.val;
        if(root.left != null) {
            int l = computeLeft(root.left);
            int r = computeRight(root.left);
            int more = Math.max(l, r);
            res += Math.max(0, more);
        }
        return res;
    }
    
    private int computeRight(TreeNode root) {
        int res = root.val;
        if(root.right != null) {
            int l = computeLeft(root.right);
            int r = computeRight(root.right);
            int more = Math.max(l, r);
            res += Math.max(0, more);
        }
        return res;
    }
}
```

遍历这棵树（traveralTree），分别计算经过每个node的最大值（通过计算左侧和右侧最大值进行间接计算），最后输出最大结果。
**todo**：
+ 感觉上有很多复杂计算，需要优化
+ 为什么内存占用会这么大，递归的内存占用分析不会。可以用这道题进行分析

# [42. Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) TODO
FAIL
## solution 1 98.21 92.47 TODO(多解)

```java
class Solution {
    public int trap(int[] height) {
        if(height.length <= 2) return 0;
        int[] res = new int[height.length];
        int max = height[0];
        for(int i = 0; i < height.length; i++)
            res[i] = max = Math.max(max, height[i]);
        max = height[height.length-1];
        
        for(int i = 0; i < height.length; i++){
            max = Math.max(max, height[height.length-1-i]);
            res[height.length-1-i] = Math.min(res[height.length-1-i], max);
        }
        
        int result = 0;
        for(int i = 0; i < height.length; i++)
            result += res[i] - height[i];
        return result;
    }
}
```
从左向右，取一次最大值；从右向左，取一次最大值。两者取小的，再进行减法。这算是动态规划的方法。

# [340. Longest Substring with At Most K Distinct Characters](https://leetcode.com/problems/longest-substring-with-at-most-k-distinct-characters/)

## 21.84 46.81 30mins
```java
class Solution {
    public int lengthOfLongestSubstringKDistinct(String s, int k) {
        if(k == 0) return 0;
        
        int res = 0;
        Map<Character, Integer> cache = new HashMap<>();
        
        int i = 0, j = 0;
        while(i < s.length()) {
            if(j == s.length()) return res;
            char c = s.charAt(j++);
            if(!cache.containsKey(c)) cache.put(c, 0);
            cache.put(c, cache.get(c)+1);
            // bugfix == -> <=
            if(cache.size() <= k) res = Math.max(res, j-i);
            while(cache.size() > k) {
                char ci = s.charAt(i++);
                if(cache.get(ci) == 1) cache.remove(ci);
                else cache.put(ci, cache.get(ci)-1);
                if(cache.size() == k) res = Math.max(res, j-i);
            }
        }
        return res;
    }
}
```

思路：两个pointer，i，j(excluded)，向右移动。分别计算里面的包含字符串个数 .但是有额外计算，导致时间复杂度较高

# [317. Shortest Distance from All Buildings](https://leetcode.com/problems/shortest-distance-from-all-buildings/) FAIL

# [76. Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/)
## 5.01 5.32 38mins
```java
// 漏掉了几种testcase
// "a" "a"
// "a" "aa"  没考虑到这种testcase 原理上就错了 
class Solution {
    public String minWindow(String s, String t) {
        Map<Character, Integer> set = new HashMap<>();
        for(char c: t.toCharArray()) {
            if(!set.containsKey(c)) set.put(c, 0);
            set.put(c, set.get(c)+1);
        }
    
        
        Map<Character, Integer> map = new HashMap<>();
        Queue<Integer> q = new LinkedList<>();
        
        String res = "";
        
        for(int i = 0; i < s.length(); ++i)
            if(set.containsKey(s.charAt(i))) {
                if(!map.containsKey(s.charAt(i))) map.put(s.charAt(i), 0);
                map.put(s.charAt(i), map.get(s.charAt(i)) + 1);
                // bugfix
                q.offer(i);
                
                // bugfix while(map.equals(set) && q.size() != 0) {
                // bugfix while(map.keySet().size() == set.keySet().size() && q.size() != 0) {
                while(mapContains(map, set) && q.size() != 0) {
                    int start = q.poll();
                    if(res.length() == 0) res = s.substring(start, i+1);
                    else res = res.length() < s.substring(start, i+1).length() ? res : s.substring(start, i+1);
                    map.put(s.charAt(start), map.get(s.charAt(start)) - 1);
                    if(map.get(s.charAt(start)) == 0) map.remove(s.charAt(start));
                }
                // bugfix q.offer(i);
            }
        return res;
    }
    
    private boolean mapContains(Map<Character, Integer> m1, Map<Character, Integer> m2) {
        if(m1.size() == m2.size()) {
            for(Character c: m1.keySet())
                if(m1.get(c) < m2.get(c)) return false;
        } else return false;
        return true;
    }
}
```
思路：
+ 从左向右扫描，记录每个可能点的位置。
+ 注意几个testcase，待优化

# [269. Alien Dictionary](https://leetcode.com/problems/alien-dictionary/) FAIL
这里遇到一个之前没了解过的知识点，[拓扑排序/Topological sorting](https://en.wikipedia.org/wiki/Topological_sorting#Kahn's_algorithm)，拓扑排序分为两种，BFS和DFS。待学习和写算法。

# [282. Expression Add Operators](https://leetcode.com/problems/expression-add-operators/) FAIL
有个问题：DFS和BACKTRACKING 到底有什么区别?

# [297. Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/)

## 42.59 54.28 20mins TODO(DFS BFS?)
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
public class Codec {

    // Encodes a tree to a single string.
    public String serialize(TreeNode root) {
        StringBuilder sb = new StringBuilder();
        Queue<TreeNode> q = new LinkedList<TreeNode>();
        q.offer(root);
        while(q.size() != 0) {
            TreeNode node = q.poll();
            if(node != null) {
                sb.append(node.val).append(" ");
                q.offer(node.left);
                q.offer(node.right);
            }
            else sb.append("null").append(" ");
        }
        return sb.substring(0, sb.length()-1);
    }

    // Decodes your encoded data to tree.
    public TreeNode deserialize(String data) {
        String[] vals = data.split(" ");
        Queue<TreeNode> q = new LinkedList<>();
        TreeNode res;
        if(vals[0].equals("null")) return null;
        else res = new TreeNode(Integer.valueOf(vals[0]));
        q.offer(res);
        for(int i = 1; i < vals.length; i++) {
            TreeNode n;
            if(vals[i].equals("null")) n = null;
            else n = new TreeNode(Integer.valueOf(vals[i]));
            
            TreeNode node = q.poll();
            node.left = n;
            if(n != null) q.offer(n);
            
            if(vals[++i].equals("null")) n = null;
            else n = new TreeNode(Integer.valueOf(vals[i]));
            
            node.right = n;
            if(n != null) q.offer(n);
        }
        return res;
    }
}

// Your Codec object will be instantiated and called as such:
// Codec codec = new Codec();
// codec.deserialize(codec.serialize(root));
```

实现了类似leetcode的方式：\[1,2,3,null,null,4,5\] 但是时间和空间复杂度是50%左右。待优化。
# [273. Integer to English Words](https://leetcode.com/problems/integer-to-english-words/)
## 17.43 100.00 29mins
```java
class Solution {
    String[] dig = new String[]{"", "One", "Two", "Three", "Four", 
                               "Five", "Six", "Seven", "Eight", "Nine"};
    String[] dig2 = new String[] {"Ten", "Eleven", "Twelve", "Thirteen", "Fourteen", 
                                 "Fifteen", "Sixteen", "Seventeen", "Eighteen", "Nineteen"};
    String[] tens = new String[]{"", "", "Twenty", "Thirty", "Forty", "Fifty", "Sixty", 
                                "Seventy", "Eighty", "Ninety"};
    String[] unit = new String[]{"", "Thousand", "Million", "Billion"};
    
    //101-119
    // bugfix case: 0
    public String numberToWords(int num) {
        if(num == 0) return "Zero";
        
        Stack<String> stack = new Stack<>();
        int iter = 0;
        while(num != 0) {
            int mod = num % 1000;
            // bugfix
            if(mod == 0) {
                num /= 1000;
                iter++;
                continue;
            }
            stack.push(unit[iter]);
            
            int last2 = mod%100;
            if(last2 != 0) {
                if(last2 < 10) stack.push(dig[last2]);
                else if(last2 < 20) stack.push(dig2[last2-10]);
                else {
                    int ten = last2/10;
                    int one = last2%10;
                    if(one != 0) stack.push(dig[one]);
                    stack.push(tens[ten]);
                }
            }
            int first1 = mod/100;
            if(first1 != 0) {
                stack.push("Hundred");
                stack.push(dig[first1]);
            }
            
            num /= 1000;
            iter++;
        }
        
        StringBuilder sb = new StringBuilder();
        while(stack.size() != 0) {
            if(stack.peek().equals("")) {
                stack.pop();
                continue;
            }
            sb.append(stack.pop()).append(" ");
        }
        return sb.substring(0, sb.length()-1).toString();
    }
}
```
每三位处理一次，注意处理0和空值，注意空格。准备数据材料的时候，注意多种情况。

# [282. Expression Add Operators](https://leetcode.com/problems/expression-add-operators/)

## FAIL
```java
class Solution {
    public List<String> addOperators(String num, int target) {
        return dfs(num, target);
    }
    
    private List<String> dfs(String num, int target) {
        List<String> res = new ArrayList<>();
        if(num.length() == 0) return res;
        int sum = 0;
        for(int i = 0; i < num.length(); i++) {
            char c = num.charAt(i);
            sum = sum*10 + (c-'0');
            if(i == num.length()-1 && sum == target) {
                res.add(num);
                return res;
            }
            if(i != num.length()-1) {
                List<String> l1 = dfs(num.substring(i+1), sum-target);
                List<String> l2 = dfs(num.substring(i+1), target-sum);
                if(!l1.isEmpty()) res.addAll(insert(l1, num.substring(0, i+1)+"-"));
                if(!l2.isEmpty()) res.addAll(insert(l2, num.substring(0, i+1)+"+"));
                
                // 处理乘号
                int sum2 = 0;
                for(int j = i+1; j < num.length(); j++) {
                    char c2 = num.charAt(j);
                    sum2 = sum2*10 + (c2-'0');
                    // bugfix
                    if(j == num.length()-1 && sum*sum2 == target) {
                        res.add(num.substring(0, i+1)+"*"+num.substring(i+1, j+1));
                        return res;
                    }
                    l1 = dfs(num.substring(j+1), sum*sum2-target);
                    l2 = dfs(num.substring(j+1), target-sum*sum2);
                    if(!l1.isEmpty()) res.addAll(insert(l1, num.substring(0, i+1)+"*"+num.substring(i+1, j+1)+"-"));
                    if(!l2.isEmpty()) res.addAll(insert(l2, num.substring(0, i+1)+"*"+num.substring(i+1, j+1)+"+"));
                    
                    // 处理连乘 ...无穷无尽
                    for(int k = j+1; k < num.length(); k++) {
                        char c3 = num.charAt(c3);
                        sum3 = sum3*10+(c3-'0');
                        if(k == num.length()-1 && sum * sum2 * sum3 == target) {
                            res.add(num.substring(0, i+1)+"*"+num.substring(i+1, j+1)+"*"+num.substring(j+1));
                            return res;
                        }
                        
                    } 
                }
            }
        }
        return res;
    }
    
    private List<String> insert(List<String> l, String s) {
        for(int i = 0; i < l.size(); i++) l.set(i, s+l.get(i));
        return l;
    }
}
```

用dfs处理加减是可以的。但是加上乘法，因为两者优先级不一样，所以就无法使用dfs了。不然就会循环套循环无穷无尽。

## FAIL2
```java
class Solution {
    public List<String> addOperators(String num, int target) {
        List<String> res = all(num, target);
        Collections.sort(res);
        return res;
    }
    
    // 只有加减
    private List<String> dfs(String num, int target) {
        List<String> res = new ArrayList<>();
        if(num.length() == 0) return res;
        int sum = 0;
        for(int i = 0; i < num.length(); i++) {
            char c = num.charAt(i);
            sum = sum*10 + (c-'0');
            if(i == num.length()-1 && sum == target) {
                res.add(num);
                return res;
            }
            if(i != num.length()-1) {
                List<String> l1 = dfs(num.substring(i+1), sum-target);
                List<String> l2 = dfs(num.substring(i+1), target-sum);
                if(!l1.isEmpty()) res.addAll(insert(l1, num.substring(0, i+1)+"-"));
                if(!l2.isEmpty()) res.addAll(insert(l2, num.substring(0, i+1)+"+"));
                // 处理乘号
                //List<String> l3 = dfsMultiply(num.substring(i+1), target, sum, num.substring(0, i+1));
                //if(!l3.isEmpty()) res.addAll(insert(l3, num.substring(0, i+1)+"*"));
            }
            // bugfix
            if(i == 0 && c == '0') break;
        }
        return res;
    }
    
    private List<String> all(String num, int target) {
        List<String> res = new ArrayList<>();
        int sum = 0;
        for(int i = 0; i < num.length(); i++) {
            char c = num.charAt(i);
            sum = sum*10 + (c-'0');
            if(i == num.length()-1 && sum == target) {
                res.add(num);
                return res;
            }
            if( i != num.length() -1) {
                res.addAll(dfsMultiply(num.substring(i+1), target, sum, num.substring(0, i+1)));
                List<String> l1 = all(num.substring(i+1), sum-target);
                List<String> l2 = all(num.substring(i+1), target-sum);
                if(!l1.isEmpty()) res.addAll(insert(l1, num.substring(0, i+1)+"-"));
                if(!l2.isEmpty()) res.addAll(insert(l2, num.substring(0, i+1)+"+"));
            }
            // bugfix
            if(i == 0 && c == '0') break;
        }
        return res;
    }
    
    //开头是乘号
    private List<String> dfsMultiply(String num, int target, int m1, String m1s) {
        List<String> res = new ArrayList<>();
        if(num.length() == 0) return res;
        int sum = 0;
        for(int i = 0; i < num.length(); i++) {
            char c = num.charAt(i);
            sum = sum*10 + (c-'0');
            if(i == num.length()-1 && sum*m1 == target) {
                res.add(m1s+"*"+num);
                return res;
            }
            
            List<String> l1 = dfs(num.substring(i+1), m1*sum-target);
            List<String> l2 = dfs(num.substring(i+1), target-m1*sum);
            if(!l1.isEmpty()) res.addAll(insert(l1, m1s+"*"+num.substring(0, i+1)+"-"));
            if(!l2.isEmpty()) res.addAll(insert(l2, m1s+"*"+num.substring(0, i+1)+"+"));
            
            List<String> l3 = dfsMultiply(num.substring(i+1), target, m1*sum, m1s+"*"+num.substring(0, i+1));
            //if(!l3.isEmpty()) res.addAll(insert(l3, m1s+"*"));
            if(!l3.isEmpty()) res.addAll(l3);
            // bugfix
            if(i == 0 && c == '0') break;
        }
        return res;
    }
    
    
    private List<String> insert(List<String> l, String s) {
        for(int i = 0; i < l.size(); i++) l.set(i, s+l.get(i));
        return l;
    }
}
```

这次做了改进，区分了只有加减号，和开头是乘号 这两种情况。目前返回时正确的了。但是有一种存在0的情况没有排除。
case: 105, 5。这种做法会返回 1*05.需要特殊处理0. 于是加了 bugfix两句话。但是有case过不去。
"123456789"
45
发现输出少了很多个解。
TODO

## 55.92 56.76 40mins
```java
class Solution {
    public List<String> addOperators(String num, int target) {
        List<String> res = new ArrayList<>();
        // bugfix start with iteration
        for(int i = 0; i < num.length(); i++) {
            StringBuilder sb = new StringBuilder();
            if(i == 0 && num.charAt(i) == '0') {
                dfs(res, sb.append('0'), num, i, target, 0, 0);
                break;
            }
            String part = num.substring(0, i+1);
            // num overflow?? use long!!
            long partVal = Long.parseLong(part);
            sb.append(part);
            dfs(res, sb, num, i, target, partVal, partVal);
        }
        return res;
    }
    
    private void dfs(List<String> res, StringBuilder sb ,String num, int pos, long target, long val, long multi) {
        // bugfix
        if(pos == num.length()-1) {
            if(val == target) res.add(sb.toString());
            return;
        }
        StringBuilder tmp;
        for(int i = pos+1; i < num.length(); i++) {
            if(i == pos+1 && num.charAt(i) == '0') {
                // bugfix tmp = sb
                tmp = new StringBuilder(sb);
                dfs(res, tmp.append('+').append('0'), num, i, target, val, 0);
                tmp = new StringBuilder(sb);
                dfs(res, tmp.append('-').append('0'), num, i, target, val, 0);
                tmp = new StringBuilder(sb);
                dfs(res, tmp.append('*').append('0'), num, i, target, val-multi, 0);
                return;
            }
            String part = num.substring(pos+1, i+1);
            long partVal = Long.parseLong(part);
            tmp = new StringBuilder(sb);
            dfs(res, tmp.append('+').append(part), num, i, target, val+partVal, partVal);
            tmp = new StringBuilder(sb);
            dfs(res, tmp.append('-').append(part), num, i, target, val-partVal, -partVal);
            tmp = new StringBuilder(sb);
            dfs(res, tmp.append('*').append(part), num, i, target, val-multi+multi*partVal, multi*partVal);
        }
    }
}
```
参考后面的答案写出来的结果，比原答案复杂不少，原因是dfs中 pos的含义与原答案中不一样。我写的dfs中pos意思是"当前位置"，但是答案中dfs中的pos意思是，"下一个位置"。总结一下dfs的一贯思路

dfs思路
+ 入参很关键，要把dfs的入参设计好
+ 入参一定是跟**结果** 和 **前序输入**相关。一定不能和后续输入相关。（比如本题，因为有了一个乘号，所以之前考虑dfs的时候会考虑后续的输入，但是这是不对的。答案使用了一个multiply变量输入dfs，解决了这个问题。让后续依赖前序，而非相反。
+ dfs需要注意如果dfs内部有多个分支，那么不同分支之间一定不要互相影响，要回退一些临时数据。比如本题中的stringBuilder，要每次进行copy

## ANSWER
```java
public class Solution {
    public List<String> addOperators(String num, int target) {
        List<String> rst = new ArrayList<String>();
        if(num == null || num.length() == 0) return rst;
        helper(rst, "", num, target, 0, 0, 0);
        return rst;
    }
    public void helper(List<String> rst, String path, String num, int target, int pos, long eval, long multed){
        if(pos == num.length()){
            if(target == eval)
                rst.add(path);
            return;
        }
        for(int i = pos; i < num.length(); i++){
            if(i != pos && num.charAt(pos) == '0') break;
            long cur = Long.parseLong(num.substring(pos, i + 1));
            if(pos == 0){
                helper(rst, path + cur, num, target, i + 1, cur, cur);
            }
            else{
                helper(rst, path + "+" + cur, num, target, i + 1, eval + cur , cur);
                
                helper(rst, path + "-" + cur, num, target, i + 1, eval -cur, -cur);
                
                helper(rst, path + "*" + cur, num, target, i + 1, eval - multed + multed * cur, multed * cur );
            }
        }
    }
}
```

# [489. Robot Room Cleaner](https://leetcode.com/problems/robot-room-cleaner/)
## 5.45 8.00 50mins
```java
/**
 * // This is the robot's control interface.
 * // You should not implement it, or speculate about its implementation
 * interface Robot {
 *     // Returns true if the cell in front is open and robot moves into the cell.
 *     // Returns false if the cell in front is blocked and robot stays in the current cell.
 *     public boolean move();
 *
 *     // Robot will stay in the same cell after calling turnLeft/turnRight.
 *     // Each turn will be 90 degrees.
 *     public void turnLeft();
 *     public void turnRight();
 *
 *     // Clean the current cell.
 *     public void clean();
 * }
 */
class Solution {
    private Map<Integer, Set<Integer>> map = new HashMap<>();
    
    public void cleanRoom(Robot robot) {
        dfs(robot, 0, 0, 0, -1);
    }
    
    private void visit(int x, int y) {
        map.computeIfAbsent(x, i->new HashSet<Integer>()).add(y);
    }
    private boolean visited(int x, int y) {
        return map.computeIfAbsent(x, i->new HashSet<Integer>()).contains(y);
    }
    
    // x 左右，y上下
    // 方向回位
    // bugfix 方向会影响，需要输入方向
    private void dfs(Robot robot, int x, int y, int xd, int yd) {
        if(visited(x, y)) return;
        visit(x, y);
        robot.clean();
        move(robot, x+xd, y+yd, xd, yd);
        robot.turnRight();
        move(robot, x+yd, y-xd, yd, -xd);
        robot.turnRight();
        move(robot, x-xd, y-yd, -xd, -yd);
        robot.turnRight();
        move(robot, x-yd, y+xd, -yd, xd);
        robot.turnRight();
    }
    
    private boolean move(Robot robot, int x, int y, int xd, int yd) {
        boolean res = false;
        if(robot.move()) {
            res = true;
            dfs(robot, x, y, xd, yd);
            robot.turnRight();
            robot.turnRight();
            robot.move();
            robot.turnRight();
            robot.turnRight();
        }
        return res;
    }
}
```

有了上一题dfs的经验，这一题会做的简单一些。需要注意的是每次dfs要输入方向，同时dfs之后要回退到当前位置，方向也要回位。

# [65. Valid Number](https://leetcode.com/problems/valid-number/)

就是考虑边界条件特别多，TODO吧