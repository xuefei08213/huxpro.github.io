---
layout:     post
title:      "Leetcode617-合并两个二叉树"
subtitle:   ""
date:       2019-07-01 12:00:00
author:     "xuefei"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - leetcode
---


&nbsp; &nbsp; &nbsp; &nbsp;合并给定两个二叉树，可以想象为将两个树进行重叠操作。当两颗树的节点重叠时，将两个节点值相加的结果作为新的节点值。

    Input: 
	Tree 1                     Tree 2                  
          1                         2                             
         / \                       / \                            
        3   2                     1   3                        
       /                           \   \                      
      5                             4   7                  
	Output: 
	Merged tree:
	     3
	    / \
	   4   5
	  / \   \ 
	 5   4   7

## 递归算法

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
    public TreeNode mergeTrees(TreeNode t1, TreeNode t2) {
        if (t1 == null && t2 == null) {
            return null;
        }

        if (t1 != null && t2 == null) {
            return t1;
        }

        if (t1 == null && t2 != null) {
            return t2;
        }

        if (t1 != null && t2 != null) {
            t1.val = t1.val + t2.val;
        }

        t1.left = mergeTrees(t1.left, t2.left);
        t1.right = mergeTrees(t1.right, t2.right);

        return t1;
    }
}
```

&nbsp; &nbsp; &nbsp; &nbsp;对上面代码的if判断条件优化之后的代码如下

```java
public class Solution {
    public TreeNode mergeTrees(TreeNode t1, TreeNode t2) {
        if (t1 == null)
            return t2;
        if (t2 == null)
            return t1;
        t1.val += t2.val;
        t1.left = mergeTrees(t1.left, t2.left);
        t1.right = mergeTrees(t1.right, t2.right);
        return t1;
    }
}
```