
## 有效的完全平方数
1. https://leetcode-cn.com/problems/valid-perfect-square/
2. 二分查找
	- 注意发生越界的情况，所以用 long 代替 int。

## 完全平方数
1. 返回一个正整数最少可以由几个完全平方数构成
2. https://leetcode-cn.com/problems/perfect-squares/
3. 数学解法
	- 如果满足$4^k(8m+7)$的形式，就最少要由4个完全平方数求解
	- 否则依次检查是否可以由1个完全平方数组成、是否可以由两个完全平方数组成
	- 到最后就只可能由3个完全平方数组成
4. 动态规划
	- 可看作一个背包问题
5. 贪心枚举
	- 本质上要比背包问题简单的多

