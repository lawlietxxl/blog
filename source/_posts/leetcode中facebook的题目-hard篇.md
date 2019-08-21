---
title: leetcode中facebook的题目 -- hard篇
tags: []
categories: []
keywords: []
date: 2019-08-15 23:34:39
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

## 5.04 5.95 1h
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
