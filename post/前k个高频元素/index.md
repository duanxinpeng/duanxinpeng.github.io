## 问题描述
1. 找出数组中出现频率最高的k个元素
2. https://leetcode-cn.com/problems/top-k-frequent-elements/
## 思路
1. 用HashMap统计频率
2. 用小根堆求出前K大的数

## java实现
1. 在实现最小堆时，可以对元素建堆，并通过comparator接口实现按照元素的频率排序。
