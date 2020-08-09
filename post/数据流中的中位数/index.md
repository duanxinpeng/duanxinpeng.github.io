## 问题描述
1. https://leetcode-cn.com/problems/find-median-from-data-stream/
## 实现思路
1. 利用两个堆，一个大根堆low，存储前半部分数，一个小根堆high，存储后半部分数。
2. add(num):
```
if(low.size() > high.size())
	low.offer(num)
	high.offer(low.poll())
else
	high.offer(num)
	low.offer(high.poll())
```
## 进阶
1. 如果数据流中所有整数都在 0 到 100 范围内，你将如何优化你的算法？
	- 定义一个长度为 101 的数组 arr，分别以 0-100 为下标存储每个数字出现的次数，同时用一个变量 count 记录所有数字的总数。
	- 根据数组 arr 和变量 count 计算中位数即可。
2. 如果数据流中百分之九十九的整数都在 0 到 100 范围内，你将如何优化你的算法？
	- 将上述长度为 101 的数组改为长度为 103 的数组即可，然后分别用第一个位置和最后一个位置存储小于零的数和大于100的数出现的次数即可！