## 问题描述
1. 层次遍历
2. 层与层之间分开
https://leetcode-cn.com/problems/binary-tree-level-order-traversal/

## 分析
1. 层次遍历很简单，只需要一个队列就可以完成。
2. 层与层之间分开是参考的官方解法：每次循环之前都求一下queue的长度，该长度就是一层中节点的个数，每次循环都会遍历一整层。
