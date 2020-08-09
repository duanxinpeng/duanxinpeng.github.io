## 问题描述
1. 给定一棵在每个节点都存放着一个整数值的二叉树，找出路径和等于给定数值的路径总数。路径不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是从上向下的。
2. https://leetcode-cn.com/problems/path-sum-iii/
## 收获
1. 之前做过一道在数组中找满足和为某个值的子数组个数的问题，（
https://leetcode-cn.com/problems/subarray-sum-equals-k/
）这道题目就是采用了HashMap，但是二叉树的路径不同于数组，所以还需要用到回溯的思想来解决该问题。
