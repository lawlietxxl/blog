---
title: facebook problems
tags: [leetcode]
categories: [leetcode facebook]
keywords: []
date: 2019-10-03 14:30:05
mathjax: true
---
记录leetcode里面facebook标签的题目，耗时，代码，时间空间占比.
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

## [158. Read N Characters Given Read4 II - Call multiple times](https://leetcode.com/problems/read-n-characters-given-read4-ii-call-multiple-times/)

### 100.00% 100.00% 60mins

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

## [23. Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)

### 98.74 27.33 18mins
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

## [301. Remove Invalid Parentheses](https://leetcode.com/problems/remove-invalid-parentheses/) (FAIL)

### DFS ANSWER
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

## [124. Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/)

### 5.04 5.95 1h TODO
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

## [42. Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) TODO
FAIL
### solution 1 98.21 92.47 TODO(多解)

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

## [340. Longest Substring with At Most K Distinct Characters](https://leetcode.com/problems/longest-substring-with-at-most-k-distinct-characters/)

### 21.84 46.81 30mins
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

## [317. Shortest Distance from All Buildings](https://leetcode.com/problems/shortest-distance-from-all-buildings/) FAIL

## [76. Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/)
### 5.01 5.32 38mins
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

## [269. Alien Dictionary](https://leetcode.com/problems/alien-dictionary/) FAIL
这里遇到一个之前没了解过的知识点，[拓扑排序/Topological sorting](https://en.wikipedia.org/wiki/Topological_sorting##Kahn's_algorithm)，拓扑排序分为两种，BFS和DFS。待学习和写算法。

## [282. Expression Add Operators](https://leetcode.com/problems/expression-add-operators/) FAIL
有个问题：DFS和BACKTRACKING 到底有什么区别?

## [297. Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/)

### 42.59 54.28 20mins TODO(DFS BFS?)
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
## [273. Integer to English Words](https://leetcode.com/problems/integer-to-english-words/)
### 17.43 100.00 29mins
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

## [282. Expression Add Operators](https://leetcode.com/problems/expression-add-operators/)

### FAIL
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

### FAIL2
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

### 55.92 56.76 40mins
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

### ANSWER
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

## [489. Robot Room Cleaner](https://leetcode.com/problems/robot-room-cleaner/)
### 5.45 8.00 50mins
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

## [65. Valid Number](https://leetcode.com/problems/valid-number/)

就是考虑边界条件特别多，TODO吧

# MEDIUM

## [91. Decode Ways](https://leetcode.com/problems/decode-ways/)
###  98.63% 100.00% 43min
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

## [15. 3Sum](https://leetcode.com/problems/3sum/)
### TLE -- wrong answer
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

## [973. K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/)

### WRONG ANSER
没有注意审题，吭哧别都写了半天结果没看清楚题目……需要注意的是
1、返回K个最近的点，是全部返回，而不是只返回一个
2、保证只有唯一一个解（说明K+1这个点一定比K大）

### 36.35 90.68 11mins
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

## [215. Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/)

### 32.77 78.76 40mins
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

## [56. Merge Intervals](https://leetcode.com/problems/merge-intervals/)

### 25.19 36.23 18mins
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

### 95.63 71.02
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

## [253. Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/)

### 100.00 75.64
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

## [238. Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/)

### 100.00 92.13 15min
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

### 100.00 100.00 8mins
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

## [560. Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)

### 30.78 97.83 38mins

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

## [31. Next Permutation](https://leetcode.com/problems/next-permutation/)

### 100.00 31.00 40min
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

## [986. Interval List Intersections](https://leetcode.com/problems/interval-list-intersections/)

### 99.93 78.38 15mins
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

## [98. Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/)
### 100.00 79.54 20mins
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

## [199. Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/)

###  98.32 100.00 12mins
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

## [621. Task Scheduler](https://leetcode.com/problems/task-scheduler/)

### AFTER FAIL: 41.12 86.76 26mins TODO
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

## [15. 3Sum](https://leetcode.com/problems/3sum/)
### 8.82 41.34 60mins TODO
```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        Arrays.sort(nums);
        for(int i = 0; i < nums.length-1; i++) {
            // duplicate, continue
            if(i != 0 && nums[i] == nums[i-1]) continue;
            Map<Integer, List<List<Integer>>> sum2Map = new HashMap<>();
            Set<Integer> done = new HashSet<>();
            for(int j = i + 1; j < nums.length; j++){
                // bugfix
                // if(j != i + 1 && nums[j] == nums[j-1]) continue;
                
                if(done.contains(nums[j])) continue;
                if(sum2Map.containsKey(-nums[j])){
                    done.add(nums[j]);
                    for(List<Integer> l: sum2Map.get(-nums[j])){
                        l.add(nums[j]);
                        res.add(l);
                    }
                }
                int sum2 = nums[i] + nums[j];
                // bugfix
                if(sum2Map.containsKey(sum2)) continue;
                if(!sum2Map.containsKey(sum2)) sum2Map.put(sum2, new ArrayList<>());
                
                // 注意顺序
                List<Integer> tmpL = new ArrayList<>();
                tmpL.add(nums[i]);
                tmpL.add(nums[j]);
                sum2Map.get(sum2).add(tmpL);
            }
        }
        return res;
    }
}
```

算法复杂度为 O(n^2)。要点在于排重。我是靠**排序**作为排重手段的。
**流程**：
+ 先排序，然后O(n^2)遍历
+ 注意i的排重逻辑是判断跟前一个是否一样。
+ 轮询到j的时候，应该先算结果（看看本次ij循环前面，是否包含结果）；再进行sum的排重。

## [133. Clone Graph](https://leetcode.com/problems/clone-graph/)(FAIL)

### 正解
```java
/*
// Definition for a Node.
class Node {
    public int val;
    public List<Node> neighbors;

    public Node() {}

    public Node(int _val,List<Node> _neighbors) {
        val = _val;
        neighbors = _neighbors;
    }
};
*/
class Solution {
    public Node cloneGraph(Node node) {
        Map<Node, Node> cache = new HashMap<>();
        return clone(node, cache);
    }
    
    private Node clone(Node node, Map<Node, Node> cache) {
        if(cache.containsKey(node)) return cache.get(node);
        Node copy = new Node(node.val, new ArrayList<>());
        cache.put(node, copy);
        if(node.neighbors == null) return copy;
        for(Node n: node.neighbors)
            copy.neighbors.add(clone(n, cache));
        return copy;
    }
}

```
一开始的思路相当复杂：
每个无向图相当于有向图，只不过边是双向的而已。之前做个一个类似的“图的复制”题目，记得当时题目的解法是：在每个节点后面加一个哨兵节点，最后把所有的哨兵节点连接起来，就跟之前的图相同。但是我这么做之后，会被判为返回引用，很奇怪。

此题目的正解也十分简单：使用map保留新图和旧图中node的对应关系，使用dfs即可。

**图的复制**一类的题目都可以向上面两个思路靠。后者简单一些。

## [173. Binary Search Tree Iterator](https://leetcode.com/problems/binary-search-tree-iterator/)
### 18.55 92.59 7mins
```java
class BSTIterator {
    Queue<Integer> q = new LinkedList<>();

    public BSTIterator(TreeNode root) {
        if(root == null) return;
        traverse(root);
    }
    
    /** @return the next smallest number */
    public int next() {
        return q.poll();
    }
    
    /** @return whether we have a next smallest number */
    public boolean hasNext() {
        return !q.isEmpty();
    }
    
    private void traverse(TreeNode root) {
        if(root == null) return;
        traverse(root.left);
        q.offer(root.val);
        traverse(root.right);
    }
}
```

利用 队列先进先出的特性 + 先序遍历。可以实现。但是时间复杂度较高，TODO 其他解法。TODO

## [161. One Edit Distance](https://leetcode.com/problems/one-edit-distance/)
### 99.56 100 20mins
```java
class Solution {
    public boolean isOneEditDistance(String s, String t) {
        if(s.equals(t)) return false;
        
        // bugfix: 对于到达length的边界没有控制
        /**
        for(int i = 0; i < s.length() && i < t.length(); ++i)
            if(s.charAt(i) != t.charAt(i)) {
                s = s.substring(i);
                t = t.substring(i);
            }
        **/
        int i = 0;
        while(i < s.length() && i < t.length() && s.charAt(i) == t.charAt(i)) {
            i++;
            if(i >= s.length() || i >= t.length()) break;
        }
        s = s.substring(i);
        t = t.substring(i);
        
        
        // insert
        if(s.length() == t.length()-1 && t.substring(1).equals(s)) return true;
        // delete
        if(s.length() == t.length()+1 && s.substring(1).equals(t)) return true;
        // replace
        if(s.length() == t.length() && s.substring(1).equals(t.substring(1))) return true;
        
        return false;
    }
}
```
从左到右扫描，遇到不同的一个字分三种情况比对即可.有bug，for循环无法进入到达length的情况

## [211. Add and Search Word - Data structure design](https://leetcode.com/problems/add-and-search-word-data-structure-design/)

### 85.20 81.82 25mins
```java
class WordDictionary {
    private class Node {
        public Node[] nodes;
        public boolean end;
        public Node(){
            nodes = new Node[26];
        }
    }
    
    private Node node;
    
    /** Initialize your data structure here. */
    public WordDictionary() {
        node = new Node();
    }
    
    /** Adds a word into the data structure. */
    public void addWord(String word) {
        Node sentinel = node;
        for(char c: word.toCharArray()) {
            if(sentinel.nodes[c-'a'] == null) sentinel.nodes[c-'a'] = new Node();
            sentinel = sentinel.nodes[c-'a'];
        }
        sentinel.end = true;
    }
    
    /** Returns if the word is in the data structure. A word could contain the dot character '.' to represent any one letter. */
    public boolean search(String word) {
        return search(word, node);
    }
    
    private boolean search(String word, Node node) {
        Node sentinel = node;
        for(int i = 0; i < word.length(); i++){
            char c = word.charAt(i);
            if(c != '.')
                if(sentinel.nodes[c-'a'] == null) return false;
                else sentinel = sentinel.nodes[c-'a'];
            else {
                // bugfix boolean allNull = true;
                for(Node n: sentinel.nodes)
                    if(n != null) {
                        // bugfix allNull = false;
                        if(search(word.substring(i+1), n)) return true;
                    }
                // bugfix if(allNull) return false;
                return false;
            }
        }
        return sentinel.end;
    }
}

/**
 * Your WordDictionary object will be instantiated and called as such:
 * WordDictionary obj = new WordDictionary();
 * obj.addWord(word);
 * boolean param_2 = obj.search(word);
 */
```
经典的字典树，与之前思路不一样的是，之前我一直使用数组，而本次使用了根节点。对于空串的判断方便了很多。作为例子记住。
另外存在一个bug：不存在那个bool判断，请注意。

## [304. Range Sum Query 2D - Immutable](https://leetcode.com/problems/range-sum-query-2d-immutable/)

### 43.84 100.00 20mins
```java
class NumMatrix {
    private int[][] sum;

    public NumMatrix(int[][] matrix) {
        if(matrix.length == 0 || matrix[0].length == 0) return;
        
        
        sum = new int[matrix.length+1][matrix[0].length+1];
        
        sum[1][1] = matrix[0][0];
        for(int i = 1; i < matrix[0].length; i++)
            sum[1][i+1] = matrix[0][i] + sum[1][i];
        for(int i = 1; i < matrix.length; i++)
            sum[i+1][1] = matrix[i][0] + sum[i][1];
        for(int i = 1; i < matrix.length; i++)
            for(int j = 1; j < matrix[i].length; j++)
                sum[i+1][j+1] = matrix[i][j] + sum[i][j+1] + sum[i+1][j] - sum[i][j];
    }
    
    public int sumRegion(int row1, int col1, int row2, int col2) {
        return sum[row2+1][col2+1] + sum[row1][col1] - sum[row1][col2+1] - sum[row2+1][col1];
    }
}

/**
 * Your NumMatrix object will be instantiated and called as such:
 * NumMatrix obj = new NumMatrix(matrix);
 * int param_1 = obj.sumRegion(row1,col1,row2,col2);
 */
```

不可变的二维数组，求和。比较常规。较为普通写法改进的是，新建空间的时候多建了一行，这样在计算的时候就不需要判断0了。

## [311. Sparse Matrix Multiplication](https://leetcode.com/problems/sparse-matrix-multiplication/)

### 25.55 100.00 6mins
```java
class Solution {
    public int[][] multiply(int[][] A, int[][] B) {
        int[][] res = new int[A.length][B[0].length];
        for(int i = 0; i < res.length; i++)
            for(int j = 0; j < res[0].length; j++) {
                int n = 0;
                for(int ind = 0; ind < A[i].length; ind++)
                    n += A[i][ind] * B[ind][j];
                res[i][j] = n;
            }
        return res;
    }
}
```
不考虑稀疏矩阵，时间排名为25，后面针对这一点进行优化。

### 44.08 100.00 
```java
class Solution {
    public int[][] multiply(int[][] A, int[][] B) {
        Map<Integer, Set<Integer>> aData = new HashMap<>();
        Map<Integer, Set<Integer>> bData = new HashMap<>();
        for(int i = 0; i < A.length; i++)
            for(int j = 0; j < A[0].length; j++)
                if(A[i][j] != 0) {
                    if(!aData.containsKey(j)) aData.put(j, new HashSet<>());
                    aData.get(j).add(i);
                }
        for(int i = 0; i < B.length; i++)
            for(int j = 0; j < B[0].length; j++)
                if(B[i][j] != 0) {
                    if(!bData.containsKey(i)) bData.put(i, new HashSet<>());
                    bData.get(i).add(j);
                }
        
        
        int[][] res = new int[A.length][B[0].length];
        //for(Integer i: aData.keySet())
            // bugfix 这样只算了中轴线上的点
            //if(bData.containsKey(i))
                //for(Integer j: aData.get(i))
                    //if(bData.get(i).contains(j))
                        //res[i][j] += A[i][j] * B[j][i];
        for(Integer j: aData.keySet())
            for(Integer i: aData.get(j))
                if(bData.containsKey(j))
                    for(Integer jj: bData.get(j))
                        // bugfix res[i][jj] += A[i][j]*B[i][jj]
                        res[i][jj] += A[i][j]*B[j][jj];
                        
        return res;
    }
    
}
```
用两个map存储稀疏图的数据。然后做处理。注意最后计算的时候的坐标，很难想，绕不过来

## [438. Find All Anagrams in a String](https://leetcode.com/problems/find-all-anagrams-in-a-string/)

### 22.89 88.00 TODO
```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        Map<Character, Integer> pattern = new HashMap<>();
        Map<Character, Integer> cache = new HashMap<>();
        
        List<Integer> res = new ArrayList<>();
        if(s == null || s.length() < p.length()) return res;
        
        // make pattern
        for(char c: p.toCharArray()) {
            if(!pattern.containsKey(c)) pattern.put(c, 0);
            pattern.put(c, pattern.get(c)+1);
        }
        
        // make first p length chars
        int i = 0;
        for(;i<p.length();i++) {
            if(!cache.containsKey(s.charAt(i))) cache.put(s.charAt(i), 0);
            cache.put(s.charAt(i), cache.get(s.charAt(i)) + 1);
        }
        // iteration
        for(;i<s.length();i++) {
            // bugfix pattern.equals(cache)
            if(mapEquals(pattern, cache)) res.add(i-p.length());
            if(!cache.containsKey(s.charAt(i))) cache.put(s.charAt(i), 0);
            cache.put(s.charAt(i), cache.get(s.charAt(i)) + 1);
            cache.put(s.charAt(i-p.length()), cache.get(s.charAt(i-p.length())) - 1);
        }
        
        if(mapEquals(pattern, cache)) res.add(i-p.length());
        
        return res;
    }
    
    private boolean mapEquals(Map<Character, Integer> m1, Map<Character, Integer> m2) {
        return mapEqualsInner(m1, m2) && mapEqualsInner(m2, m1);
    }
    
    private boolean mapEqualsInner(Map<Character, Integer> m1, Map<Character, Integer> m2) {
        for(Character c: m1.keySet())
            if(m1.get(c) != 0)
                if(!m1.get(c).equals(m2.get(c)))
                    return false;
        return true;
    }
}
```

思路比较常规，一个指针从头到位即可。但是待优化。

## [426. Convert Binary Search Tree to Sorted Doubly Linked List](https://leetcode.com/problems/convert-binary-search-tree-to-sorted-doubly-linked-list/)

### 100.00 6.90 18mins TODO
```java
/*
// Definition for a Node.
class Node {
    public int val;
    public Node left;
    public Node right;

    public Node() {}

    public Node(int _val,Node _left,Node _right) {
        val = _val;
        left = _left;
        right = _right;
    }
};
*/
class Solution {
    public Node treeToDoublyList(Node root) {
        if(root == null) return null;
        Node[] nodes = trans(root);
        return nodes[0];
    }
    
    private Node[] trans(Node root) {
        if(root == null) return null;
        if(root.left == null && root.right == null) {
            root.right = root;
            root.left = root;
            return new Node[]{root, root};
        }
        if(root.left == null) {
            Node[] nodes = trans(root.right);
            root.right = nodes[0];
            nodes[1].right = root;
            root.left = nodes[1];
            nodes[0].left = root;
            return new Node[]{root, nodes[1]};
        }
        if(root.right == null) {
            Node[] nodes = trans(root.left);
            nodes[1].right = root;
            root.right = nodes[0];
            root.left = nodes[1];
            nodes[0].left = root;
            return new Node[]{nodes[0], root};
        }
        
        Node[] nodes0 = trans(root.left);
        Node[] nodes1 = trans(root.right);
        
        nodes0[1].right = root;
        root.right = nodes1[0];
        nodes1[1].right = nodes0[0];
        
        root.left = nodes0[1];
        nodes0[0].left = nodes1[1];
        nodes1[0].left = root;
        
        return new Node[]{nodes0[0], nodes1[1]};
    }
}
```

思路：divide and conquer。分治法的经典应用。使用子树的结果，拼成总体的结果即可。但是空间复杂度很高。待优化

## [670. Maximum Swap](https://leetcode.com/problems/maximum-swap/)

### FAIL
```java
class Solution {
    // bugfix: testcase 122 -> 221 not 212
    public int maximumSwap(int num) {
        int res = num;
        int i = 0, j = 0, nj = 0, ni = 0, iter = 0;
        while(num != 0) {
            // find i j
            int current = num % 10;
            num /= 10;
            if(num == 0) break;
            int next = num % 10;
            
            if(current > next) {
                i = iter+1;
                ni = next;
                if(current > nj) {
                    nj = current;
                    j = iter;
                }
            }
            // bugfix
            else if (current == next) {
                
            } else {
                
            }
            
            iter ++;
        }
        
        res -= ni * tens(i);
        
        res -= nj * tens(j);
        
        res += ni * tens(j);
        res += nj * tens(i);
        return res;
    }
    
    private int tens(int i) {
        if(i == 0) return 1;
        int res = 1;
        for(int t = 0; t < i; t++) res *= 10;
        return res;
    }
    
}
```

思路：
+ 寻找从右向左的递减的序列，找峰值，但是要考虑相等的值的情况。
+ 想起来十分复杂，不易实现。可以优化为答案的贪心算法。

## [34. Find First and Last Position of Element in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

### 100 97.16 25mins
```java
class Solution {
    public int[] searchRange(int[] nums, int target) {
        int[] res = new int[]{-1, -1};
        if(nums == null || nums.length == 0) return res;
        // l r included
        int l = 0, r = nums.length-1;
        while(l < r) {
            int mid = l + (r-l)/2;
            if(nums[mid] > target) {
                r = mid-1;
            } else if(nums[mid] < target) {
                l = mid+1;
            } else {
                res[0] = first(nums, target, l, mid);
                res[1] =  last(nums, target, mid, r);
                return res;
            }
        }
        // l = r
        if(nums[l] == target) {
            res[0] = res[1] = l;
            return res;
        } else {
            return res;
        }
    }
    
    // must have
    private int first(int[] nums, int target, int l, int r) {
        while(l < r) {
            int mid = l + (r-l)/2;
            if(nums[mid] == target) r = mid;
            else l = mid+1;
        }
        return l;
    }
    
    private int last(int[] nums, int target, int l, int r) {
        while(l < r) {
            int mid = l + (r-l)/2;
            if(nums[mid] == target) {
                //bugfix
                if(l == mid) {
                    return nums[r] == target ? r : l;
                }
                else l = mid;
            }
            else r = mid-1;
        }
        return l;
    }
}
```

思路：
+ 二分查找为了好让循环停止，所以使用**包含**边界的写法。这样只要写<就可以了。
+ 又写出了一个死循环bug。因为 写法是 mid = l+(l-r)/2，边界以左边界作为基准移动。所以在移动左边界的时候，其实跟右边界是不对等的。所以要判断一下移动左边界的特殊情况。

## [721. Accounts Merge](https://leetcode.com/problems/accounts-merge/)

### FAIL
+ 这其实是图问题
+ 一个Account就是一个图，一个mail是一个点，就看点和点之间是否存在边
+ 如果是图，可以使用dfs。TODO
+ 可以使用并查集进行数据的处理，但是没有抽象成数据模型。

### 并查集 60mins
```java
class Solution {
    public List<List<String>> accountsMerge(List<List<String>> accounts) {
        Map<String, Integer> mail2Id = new HashMap<>();
        Map<Integer, String> id2Name = new HashMap<>();
        
        DSU dsu = new DSU();
        
        int[] id = new int[1];
        for(List<String> l: accounts)
            for(int i = 1; i < l.size(); i++) {
                String mail = l.get(i);
                // must be final
                int mailId = mail2Id.computeIfAbsent(mail, x->id[0]++);
                if(i > 1) {
                    String lastMail = l.get(i-1);
                    int lastId = mail2Id.get(lastMail);
                    dsu.union(lastId, mailId);
                }
                id2Name.put(mailId, l.get(0));
            }
        
        Map<Integer, List<String>> id2Account = new HashMap<>();
        for(Map.Entry<String, Integer> entry: mail2Id.entrySet()) {
            int p = dsu.find(entry.getValue());
            id2Account.computeIfAbsent(p, x->new LinkedList<>()).add(entry.getKey());
        }
        
        List<List<String>> res = new ArrayList<>();
        for(Map.Entry<Integer, List<String>> entry: id2Account.entrySet()) {
            String name = id2Name.get(entry.getKey());
            Collections.sort(entry.getValue());
            entry.getValue().add(0, name);
            res.add(entry.getValue());
        }
        
        return res;
    }
    
    class DSU {
        int[] parent;
        public DSU() {
            parent = new int[10001];
            for (int i = 0; i <= 10000; ++i)
                parent[i] = i;
        }
        public int find(int x) {
            if (parent[x] != x) parent[x] = find(parent[x]);
            return parent[x];
        }
        // x <- y
        public void union(int x, int y) {
            parent[find(y)] = find(x);
        }
    }
}
```

已有思路的情况下，用的时间还是比较慢的。注意DSU的写法，牢记。
时间复杂度是30，并不是最佳。TODO
思路：
+ 每个list 两两union
+ 遍历mail表，寻找每个的根节点，生成结果。

## [785. Is Graph Bipartite?](https://leetcode.com/problems/is-graph-bipartite/)
### 32.82 63.42 29mins TODO
```java
class Solution {
    public boolean isBipartite(int[][] graph) {
        Set<Integer> a = new HashSet<>();
        Set<Integer> b = new HashSet<>();
        
        Set<Integer> done = new HashSet<>();
        
        for(int i = 0; i < graph.length; i++) {
            if(done.contains(i)) continue;
            if(!dfs(graph, i, a, b, done)) return false;
        }
        return true;
    }
    
    private boolean dfs(int[][] graph, int v, Set<Integer> a, Set<Integer> b, Set<Integer> done) {
        if(a.contains(v)) return true;
        a.add(v);
        for(int u: graph[v]) {
            if(a.contains(u)) return false;
            if(!dfs(graph, u, b, a, done)) return false;
        }
        done.add(v);
        return true;
    }
}
```

思路：
使用dfs，从一个点出发，依次用set标记剩下的点。除非某点存在边位于同一个set中。否则即为二分图

## [958. Check Completeness of a Binary Tree](https://leetcode.com/problems/check-completeness-of-a-binary-tree/)

### 11.74 100.00 50mins
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
    public boolean isCompleteTree(TreeNode root) {
        return checkRecord(root).height >= 0;
    }
    
    Record checkRecord(TreeNode root) {
        if(root == null) return new Record(true, 0);
        Record l = checkRecord(root.left);
        Record r = checkRecord(root.right);
        
        if(l.height == -1 || r.height == -1) return new Record(false, -1);
        if(l.full && r.full)
            if(l.height - r.height == 0) return new Record(true, l.height+1);
            else if(l.height - r.height == 1) return new Record(false, l.height+1);
            else return new Record(false, -1);
        else if(l.full && !r.full) 
            if(l.height == r.height) return new Record(false, l.height+1);
            else return new Record(false, -1);
        else if(!l.full && r.full)
            if(l.height-r.height==1) return new Record(false, l.height+1);
            else return new Record(false, -1);
        else return new Record(false, -1);
    }
    
    
    class Record {
        Record(boolean full, int height) {
            this.full = full;
            this.height = height;
        }
        
        boolean full;
        int height;
    }
}
```

需要一次返回两个数据：是否是满的二叉树 + 最大深度
分情况进行分析即可。
可以看出时间复杂度较高，但是空间复杂度不错。
TODO: bfs

## [523. Continuous Subarray Sum](https://leetcode.com/problems/continuous-subarray-sum/)

### 26.60 88.24 19mins
```java
class Solution {
    public boolean checkSubarraySum(int[] nums, int k) {
        // bugfix k = 0;
        if(nums.length <= 1) return false;
        int[] sums = new int[nums.length+1];
        sums[0] = 0;
        for(int i = 1; i < sums.length; i++)
            sums[i] = nums[i-1] + sums[i-1];
        //for(int s : sums)
            //System.out.println(s);
        for(int i = 0; i < sums.length-2; i++)
            for(int j = i+2; j < sums.length; j++)
                if(k == 0 && sums[j] - sums[i] == 0)
                    return true;
                else if(k != 0 && (sums[j] - sums[i]) % k == 0)
                    return true;
        return false;
    }
}
```

时间复杂度为 $O(n^2)$

### hashMap 做法
```java
class Solution {
    public boolean checkSubarraySum(int[] nums, int k) {
        Map<Integer, Integer> map = new HashMap<>();
        map.put(0, -1);
        int runningSum = 0;
        for(int i = 0; i < nums.length; i++) {
            runningSum += nums[i];
            if( k != 0) runningSum %= k;
            if(map.containsKey(runningSum) && i-map.get(runningSum) > 1) return true;
            else if(!map.containsKey(runningSum)) map.put(runningSum, i);
        }
        return false;
    }
}
```

利用取mod的方式可以把时间复杂度降为$O(n)$。
思路：
+ 仍然取sum，只不过这次是取 $sum\%k$
+ 如果有两个点，sum一样，则这两个点中间的和可以被k整除。

## [1027. Longest Arithmetic Sequence](https://leetcode.com/problems/longest-arithmetic-sequence/)
### 5.08 100.00 60mins
```java
class Solution {
    Map<Integer, Set<Integer>> done = new HashMap<>();
    
    public int longestArithSeqLength(int[] A) {
        if(A.length <= 2) return A.length;
        int res = 2;
        for(int i = 0; i < A.length-1; i++)
            for(int j = i+1; j < A.length; j++)
                res = Math.max(res, dfs(A, i, j, 0));
        return res;
    }
    
    // length former length
    private int dfs(int[] A, int i, int j, int length) {
        int diff = A[j] - A[i];
        //done has bug
        //if(done.computeIfAbsent(A[j], x->new HashSet<>()).contains(diff)) return -1;
        int res = 2;
        for(int k = j+1; k < A.length; k++) {
            if(A[k] - A[j] == diff) {
                res = Math.max(res, 1 + dfs(A, j, k, 1));
            }
        }
        //done.get(A[j]).add(diff);
        return res;
    }
}
```

注意dfs的时候，length是formerlength。但是这个赶紧代码不优雅，需要看看别人怎么写的 TODO
时间复杂度较高，因为有很多重复计算。需要有个方式记录做过的dfs. TODO
注释的写法有bug：TODO

## [825. Friends Of Appropriate Ages](https://leetcode.com/problems/friends-of-appropriate-ages/)

### TLE 9mins
```java
class Solution {
    public int numFriendRequests(int[] ages) {
        if(ages.length < 2) return 0;
        int res = 0;
        for(int a = 0; a < ages.length-1; a++)
            for(int b = a+1; b<ages.length; b++)
            {
                int aa = ages[a], ab = ages[b];
                if(!(ab<=0.5*aa+7 || ab > aa || (ab>100&&aa<100))) res++;
                int tmp = aa;
                aa = ab;
                ab  =tmp;
                if(!(ab<=0.5*aa+7 || ab > aa || (ab>100&&aa<100))) res++;
            }
        return res;
    }
}
```

$O(n^2)$ TLE 了。

思路：可以以age作为key，这样的话复杂度会降低到 $O(k^2)$ k为age种类

## [314. Binary Tree Vertical Order Traversal](https://leetcode.com/problems/binary-tree-vertical-order-traversal/)

### WRONG ANSWER 
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
    public List<List<Integer>> verticalOrder(TreeNode root) {
        List<List<Integer>> res = new LinkedList<>();
        if(root == null) return res;
        
        Map<Integer, List<Integer>> map = new HashMap<>();
        int leftest = traveral(root, 0, map);
        while(true) {
            List<Integer> l = map.get(leftest++);
            if(l == null) break;
            res.add(l);
        }
        return res;
    }
    
    // root != null
    int traveral(TreeNode root, int current, Map<Integer, List<Integer>> map) {
        List<Integer> l = map.computeIfAbsent(current, x->new ArrayList<>());
        l.add(root.val);
        int res = current;
        if(root.left != null)
            res = traveral(root.left, current-1, map);
        if(root.right != null)
            res = Math.min(res, traveral(root.right, current+1, map));
        return res;
    }
}
```
由于这个题目要求顺序是**从左到右，从上到下**，上面的代码可以做到从左到右，但是无法做到从上到下。（因为是先遍历左子树，后遍历右子树），需要进行修改，让代码可以记住**从上到下**
所以DFS无法完成这个任务。
我们需要BFS。

### BFS 7.02 100.00
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
    public List<List<Integer>> verticalOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        if(root == null) return res;
        
        Map<Integer, List<Integer>> map = new HashMap<>();
        int min = 0;
        
        Queue<TreeNode> nodes = new LinkedList<>();
        Queue<Integer> cols = new LinkedList<>();
        
        nodes.offer(root);
        cols.offer(0);
        
        while(nodes.size() != 0) {
            TreeNode node = nodes.poll();
            int col = cols.poll();
            map.computeIfAbsent(col, x->new ArrayList<>()).add(node.val);
            if(node.left != null) {
                nodes.offer(node.left);
                cols.offer(col-1);
                min = Math.min(min, col-1);
            }
            if(node.right != null) {
                nodes.offer(node.right);
                cols.offer(col+1);
            }
        }
        while(true) {
            List<Integer> l = map.get(min++);
            if(l == null) break;
            res.add(l);
        }
        return res;
    }
}
```
本来我的思路是，新建一个类，这个类包含col和val，但是看别人的做法，是**新建两个一样的数据结构存着两个值**，这样肯定比新建数据结构效率要高的。
但是时间复杂度仍然不是最优。待优化TODO

## [863. All Nodes Distance K in Binary Tree](https://leetcode.com/problems/all-nodes-distance-k-in-binary-tree/)

### FAIL 61.12 100.00 44mins
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
    public List<Integer> distanceK(TreeNode root, TreeNode target, int K) {
        Map<TreeNode, TreeNode> pMap = new HashMap<>();
        dfs(root, null, pMap);
        
        Queue<TreeNode> q = new LinkedList<>();
        q.offer(null);
        q.offer(target);
        Set<TreeNode> seen = new HashSet<>();
        seen.add(target);
        // bugfix
        seen.add(null);
        
        int dis = 0;
        // null acctually is a sentinel
        while(q.size() != 0) {
            TreeNode cur = q.poll();
            if(cur == null) {
                if(dis == K) {
                    List<Integer> res = new ArrayList<>();
                    for(TreeNode t: q) res.add(t.val);
                    return res;
                } else dis ++;
                q.offer(null);
            } else {
                if(!seen.contains(cur.left)) {
                    seen.add(cur.left);
                    q.offer(cur.left);
                }
                if(!seen.contains(cur.right)) {
                    seen.add(cur.right);
                    q.offer(cur.right);
                }
                if(!seen.contains(pMap.get(cur))) {
                    seen.add(pMap.get(cur));
                    q.offer(pMap.get(cur));
                }
            }
        }
        return new ArrayList<>();
    }
    private void dfs(TreeNode n, TreeNode p, Map<TreeNode, TreeNode> pMap) {
        if(n == null) return;
        pMap.put(n, p);
        dfs(n.left, n, pMap);
        dfs(n.right, n, pMap);
    }
}
```
自己实现没有想出来，所以看了答案。
思路：
+ 使用bfs，每次队列中的都是一个距离的，距离不停加1
+ 注意初始化条件。很关键。

## [416. Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/)

### FAIL
这个题目的本质是求在一个集合中，是否存在一个子集，其和等于某个数值。本质的本质是**0/1背包问题**
解法——动态规划。
dp[i][j] = 前i个数字，是否存在一个集合 = j。
状态转换方程：如果我们不pick nums[i]，则为前者；如果pick nums[i]，则为后者
$dp[i][j] = dp[i-1][j] || dp[i-1][j-nums[i]]$

通过这个状态转换方程可以看出，我们可以优化空间复杂度至一维数组

# EASY

## [283. Move Zeroes](https://leetcode.com/problems/move-zeroes/)
### 100.00% 98.60% 7.5min
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
## [157. Read N Characters Given Read4](https://leetcode.com/problems/read-n-characters-given-read4/)
###  100.00% 100.00% 17min

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
    public int read(char[] buf, int n) {
        if(n <= 0) return 0;
        
        int iter = n/4;
        
        int result = 0;
        char[] tmpBuf = new char[4];
        for(int i = 0; i < iter; i++){
            int tmpResult = read4(tmpBuf);
            result += tmpResult;
            for(int j = 0; j < tmpResult; j++){
                buf[4*i + j] = tmpBuf[j];
            }
        }
        //bugfix: 最后一次循环要特殊判断，取两者里面小的
        int left = n%4;
        if(left == 0) return result;
        
        int lastResult = read4(tmpBuf);
        int iter2 = left < lastResult?left:lastResult;
        
        for(int i = 0;i<iter2; i++){
            buf[4*iter + i] = tmpBuf[i];
        }
        //注意加的这个数值
        result += iter2;
        return result;
        
    }
}
```

消耗时间较多了，注意两个bug

## [278. First Bad Version](https://leetcode.com/problems/first-bad-version/)
### 99.48% 100.00% 20min

```java
/* The isBadVersion API is defined in the parent class VersionControl.
      boolean isBadVersion(int version); */

public class Solution extends VersionControl {
    public int firstBadVersion(int n) {
        //bugfix r = n + 1
        int l = 1, r = n, mid;
        while(l <= r){
            if(l == r) return l;
            mid = l + (r - l)/2;
            if(isBadVersion(mid)) r = mid;
            else l = mid + 1;
        }
        return l;
    }
}
```

**bugfix**:注意binary search的时候，边界是否包括结果可能对算法正确与否产生影响。比如如果认为边界不包括结果，则 r = n + 1。那么在后续判断的时候会令 r = mid+1; 那么有一种情况会造成死循环: [l][结果][r]。这时候会造成死循环。注意考虑这种边界条件.

## [125. Valid Palindrome](https://leetcode.com/problems/valid-palindrome/)

### 96.21 92.86 20mins

```java
class Solution {
    public boolean isPalindrome(String s) {
        if(s.length() == 0) return true;
        int l = 0, r = s.length() - 1;
        while(l != r){
            while(l < s.length() && !isAlphaNum(s.charAt(l))) l++;
            while(r >= 0 && !isAlphaNum(s.charAt(r))) r--;
            if(l >= r) return true;
            //bugfix ignore case
            if(getChar(s, l++) != getChar(s, r--)) return false;
        }
        
        return true;
    }
    
    private boolean isAlphaNum(char c){
        if(c >= 'a' && c <= 'z')
            return true;
        if(c >= 'A' && c <= 'Z')
            return true;
        if(c >= '0' && c <= '9')
            return true;
        return false;
    }
    
    private int getChar(String s, int i){
        char c = s.charAt(i);
        if(c >= '0' && c <= '9') return c;
        if(c >= 'a' && c <= 'z') return c - 'a';
        if(c >= 'A' && c <= 'Z') return c - 'A';
        return c;
    }
}
```
题目不难，但是被java语法卡住了
**tips**:
+ 移动指针的时候注意边界，不要越界
+ 判断边界条件及时返回
+ java中'a'+<int>编译器不能支持，但是'a'+10是支持的。（因为编译器编译时能判断后者没越界）所以需要使用另外一种方式替代ignorecase的判断。

## [1. Two Sum](https://leetcode.com/problems/two-sum/)
### 98.89 98.95 10mins
```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();
        for(int i = 0; i < nums.length; ++i){
            int n = nums[i];
            if(map.containsKey(n) && n*2 == target) return new int[]{map.get(n), i};
            if(map.containsKey(target-n)) return new int[]{map.get(target-n), i};
            map.put(n, i);
        }
        return new int[2];
    }
}
```
**tips**: 注意相等的情况，简单的题目暗藏对情况的全面考虑

## [88. Merge Sorted Array](https://leetcode.com/problems/merge-sorted-array/)
### 100.00 100.00 15min

```java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int leftLength = m + n;
        //bugfix
        while(leftLength != 0){
            if(m == 0){
                for(int i = 0; i < n; i++){
                    nums1[i] = nums2[i];
                }
                break;
            }
            if(n == 0){
                break;
            }
            
            if(nums1[m-1] >= nums2[n-1]){
                //bugfix leftLength++ indexOutOfBound
                nums1[--leftLength] = nums1[--m];
            }else{
                nums1[--leftLength] = nums2[--n];
            }
        }
    }
}
```
**tips**:
+ 最好用length 而并非index（length比index多1）
+ 注意先减后用

## [350. Intersection of Two Arrays II](https://leetcode.com/problems/intersection-of-two-arrays-ii/)
### 17.64 83.87 10mins
```java
class Solution {
    public int[] intersect(int[] nums1, int[] nums2) {
        Map<Integer, Integer> n1Map = new HashMap<>();
        Map<Integer, Integer> n2Map = new HashMap<>();
        Map<Integer, Integer> resultMap = new HashMap<>();
        for(int n : nums1) {
            if(!n1Map.containsKey(n))
                n1Map.put(n, 0);
            n1Map.put(n, n1Map.get(n)+1);
        }
        
        for(int n : nums2) {
            if(!n2Map.containsKey(n))
                n2Map.put(n, 0);
            n2Map.put(n, n2Map.get(n)+1);
        }
        
        for(Map.Entry<Integer, Integer> entry: n1Map.entrySet()) {
            if(n2Map.containsKey(entry.getKey()))
                resultMap.put(entry.getKey(), Math.min(entry.getValue(), n2Map.get(entry.getKey())));
        }
        int number = 0;
        for(Map.Entry<Integer, Integer> entry: resultMap.entrySet()) {
            number += entry.getValue();
        }
        
        int[] result = new int[number];
        number = 0;
        for(Map.Entry<Integer, Integer> entry: resultMap.entrySet()) {
            int val = entry.getValue();
            while(val-- != 0)result[number++] = entry.getKey();
        }
        return result;
    }
}
```

使用两个map进行统计，然后用一个resultMap去整理数据，最后输出。但是能看出时间复杂度很差。

### 91.22 33.87
```java
public class Solution {
    public int[] intersect(int[] nums1, int[] nums2) {
        List<Integer> res = new ArrayList<Integer>();
        Arrays.sort(nums1);
        Arrays.sort(nums2);
        for(int i = 0, j = 0; i< nums1.length && j<nums2.length;){
                if(nums1[i]<nums2[j]){
                    i++;
                }
                else if(nums1[i] == nums2[j]){
                    res.add(nums1[i]);
                    i++;
                    j++;
                }
                else{
                    j++;
                }
        }
        int[] result = new int[res.size()];
        for(int i = 0; i<res.size();i++){
            result[i] = res.get(i);
        }
        return result;
    }
}
```

**tips**:
可以看得出，虽然时间复杂度看上去较慢，达到 nlgn，但是速度很快。这是因为**对基础类型的排序java非常快**

## [953. Verifying an Alien Dictionary](https://leetcode.com/problems/verifying-an-alien-dictionary/)

### 100.00 100.00 17mins
```java
class Solution {
    public boolean isAlienSorted(String[] words, String order) {
        int[] dic = toDic(order);
        // bugfix
        for(int i = 0; i < words.length-1; i++) {
            if(compare(words[i], words[i+1], dic) == 1) return false;
        }
        return true;
    }
    
    private int[] toDic(String order) {
        int ind = 0;
        int[] result = new int[26];
        for(char c: order.toCharArray()){
            result[c-'a'] = ind++; 
        }
        return result;
    }
    
    private int compare(String s1, String s2, int[] dic) {
        int i1, i2;
        for(i1 = 0, i2 = 0; i1 < s1.length() && i2 < s2.length(); i1++, i2++) {
            char c1 = s1.charAt(i1), c2 = s2.charAt(i2);
            if(dic[c1-'a'] < dic[c2-'a']) return -1;
            else if(dic[c1-'a'] > dic[c2-'a']) return 1;
        }
        
        if(s1.length() == s2.length()) return 0;
        if(i1 == s1.length()) return -1;
        return 1;
    }
}
```

注意bugfix地方，边界写错了

## [543. Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/)

### TODO: 13.73 19.48 20mins
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
    public int diameterOfBinaryTree(TreeNode root) {
        return traversal(root);
    }
    
    private int traversal(TreeNode root) {
        if(root == null) return 0;
        int l = traversal(root.left);
        int mid = comp(root);
        int r = traversal(root.right);
        return Math.max(l, Math.max(mid, r));
    }
    
    private int comp(TreeNode root) {
        if(root == null) return 0;
        int l = dfs(root.left);
        int r = dfs(root.right);
        return l + r;
    }
    
    private int dfs(TreeNode root) {
        if(root == null) return 0;
        int l = dfs(root.left);
        int r = dfs(root.right);
        return 1 + Math.max(l, r);
    }
}
```

**思路**：
中序遍历全部的点，每个点计算一次最大值。
存在大量的重复计算。
和内存占用。（并不会分析）待优化。

## [680. Valid Palindrome II](https://leetcode.com/problems/valid-palindrome-ii/)

### TODO: 9.68 97.22 11mins
```java
class Solution {
    public boolean validPalindrome(String s) {
        if(s.length() <= 1) return true;
        
        for(int i = 0; i < s.length()/2; i ++)
            if(s.charAt(i) != s.charAt(s.length()-1-i))
                return check(s.substring(i, s.length()-1-i)) || check(s.substring(i+1, s.length()-i));
        return true;
    }
    
    private boolean check(String s){
        if(s == null || s.length() <= 1) return true;
        for(int i = 0; i < s.length()/2; i ++)
            if(s.charAt(i) != s.charAt(s.length()-1-i))
                return false;
        return true;
    }
}
```

题目不难，去掉一个字母能回文。判断去掉哪个即可。
但是时间效率貌似只有9.待优化

## [415. Add Strings](https://leetcode.com/problems/add-strings/)
### 37.64 100.00 8mins
```java
class Solution {
    public String addStrings(String num1, String num2) {
        StringBuilder sb = new StringBuilder();
        int cas = 0;
        for(int i = 0; i < Math.max(num1.length(), num2.length()); i++) {
            int c1 = num1.length()-1-i < 0 ? 0 : num1.charAt(num1.length()-1-i)-'0';
            int c2 = num2.length()-1-i < 0 ? 0 : num2.charAt(num2.length()-1-i)-'0';
            int sum = cas + c1 + c2;
            cas = sum/10;
            sb.insert(0, sum%10);
        }
        if(cas != 0) sb.insert(0, cas);
        return sb.toString();
    }
}
```

TODO 时间有待提升