---
layout:     post
title:      "Morris Traversal"
subtitle:   "LeetCode 94"
date:       2018-04-05 14:21:00
author:     "Jiupeng Zhang"
header-img: "img/in-post/coding.jpg"
tags:
    - LeetCode
    - Algorithm
---
题目出处: [Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/)

> Given a binary tree, return the inorder traversal of its nodes' values.<br><br>
> Recursive solution is trivial, could you do it iteratively?

题目非常直接，要求实现二叉树的中序遍历（左子树-根-右子树）。这里先贴出递归和非递归的两个常规解法：

题中给出的二叉树节点结构
```java
public class TreeNode {
   int val;
   TreeNode left;
   TreeNode right;
   TreeNode(int x) { val = x; }
}
```
**递归实现：**
```java
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        inorderTraversal(root, result);
        return result;
    }
    public void inorderTraversal(TreeNode p, List<Integer> result) {
        if (p == null) {
            return;
        }
        inorderTraversal(p.left, result); // 递归遍历左子树
        result.add(p.val); // 记录当前节点的值
        inorderTraversal(p.right, result); // 递归遍历右子树
    }
}
```
**非递归实现：**
```java
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        LinkedList<TreeNode> trace = new LinkedList<>(); // 使用栈记录p的行进轨迹
        TreeNode p = root;
        while (p != null || trace.size() > 0) {
            while (p != null) {
                trace.push(p); // 记录当前的根，以便后续访问其右子树
                p = p.left;
            }
            p = trace.pop(); // 从栈顶取出上一节点的根
            result.add(p.val); // 记录当前根的值
            p = p.right; // 继续访问其右子树
        }
        return list;
    }
}
```
## Morris Traversal

上述两种实现的基本思路，均是通过栈结构保存访问节点的路径，当左子树遍历完成后，便从栈中取出上一级节点，输出节点的值，然后继续访问其右子树。这两种方法唯一的区别在于前者使用了程序运行时栈来维护访问次序，而后者则是通过显式使用栈对象来实现，更不易发生溢出。基于这个思路的实现的空间复杂度为$$O\left(n\right)$$，准确地说应该是树的深度。下面来介绍一种空间复杂度仅为$$O\left(1\right)$$的解法 —— **Morris Traversal**。

Morris遍历应用了线索二叉树的思想，利用叶子节点中多余的left或right指针域，来存储按某种顺序遍历的后续节点，由于当前题目要求实现二叉树的中序遍历，所以我们考虑将左子树中遍历的最末节点和上一级根节点做**链接**，通过链接访问来代替递归解法中的回溯过程。

**算法描述：**
```
WHILE 根节点不为空

    IF 当前根有左子树
        将左子树中最右节点拉链到当前的根；
        访问左子树（更新根节点）
        断开原来的根与左子树的链接（防止陷入死循环）
    ELSE
        输出当前节点;
        访问右子树（更新根节点）
```

**算法解说：**

我们以下图这棵树为例（其中序遍历的结果应为4，7，2，8，5，9，1，3，6）

![](http://7xp1jv.com1.z0.glb.clouddn.com/18-4-5/89562510.jpg)

当前根节点的值为1，其左子树（以2为根）不为空，所以我们将该子树最右侧的节点，即9号节点的右孩子链接至当前的根1；随后更新根为2，并断开与原根的链接，如下所示

![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSVDJmAAQ4qn1JowomD9XwIvbERj2PHL5OgMN16tkD-8WYRQNBR)

现在的根节点被更新为2，其左子树不为空，所以我们找到该子树的最右侧节点7，并将其链接到了2号节点，然后我们继续把根更新为4号节点，并断开2号与其的链接，

![](http://7xp1jv.com1.z0.glb.clouddn.com/18-4-5/66991076.jpg)

当前根节点为4，我们发现它并没有左子树，所以，直接输出其值“4”。然后，我们将根更新到其右孩子指向的节点7，

![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRl6j2lg6bQSP1hU_Kcy5Auo64WDSiPaNyVI4CbuAUp1bzJcLSQDg)

我们发现，节点7并没有左子树，所以输出“7”，随后将根更新为它的右孩子，也就是节点2（相当于完成了递归法的一次回溯），

![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQuQOIY1p7bzB94SeGdl15jVdLnI8ggdkwQzTaZlh_Y3E906-c2)

同样的道理，我们把“2”也输出，随后根被更新为5。我们找到其左子树8（实际上只有一个节点）的最右侧节点（也就是8本身），将其链到当前的根5，

![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRpMZ8aL7g0oi0i7VNWrIwzUVfYGDoKbIVkAnHJqPCb8IJtW_0F)

如此循环，直到当前根节点为空时，算法终止。

![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcThpBgeAWPUBU_VF-46VNayUHoh_rOSDfDZzCyUiyeWTZF_kqs7)

**实现代码：**

```java
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> list = new ArrayList<>();
        TreeNode head = root;
        while (head != null) {
            if (head.left == null) {
                list.add(head.val);
                head = head.right;
            } else {
                TreeNode p = head.left; // 进入左子树
                while(p.right != null) {
                    p = p.right; // 访问最右侧节点
                }
                p.right = head; // 拉链到当前根节点
                p = head.left; // 缓存左子树的根
                head.left = null; // 断开与左子树的链接
                head = p; // 更新当前的根为左子树的根
            }
        }
        return list;
    }
}
```
