## 问题描述
1. 返回两个链表相交的起始节点。
2. https://leetcode-cn.com/problems/intersection-of-two-linked-lists/

## 思路
1. 先同时遍历，找到两个链表相差的节点数。
2. 此时就可以让两个链表从长度相等的地方同时开始向后遍历，此时一定能找到相交的节点。
