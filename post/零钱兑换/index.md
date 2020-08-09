## 问题描述
1. 给定不同面额的硬币 coins 和一个总金额 amount。编写一个函数来计算可以凑成总金额所需的最少的硬币个数。如果没有任何一种硬币组合能组成总金额，返回 -1。
2. https://leetcode-cn.com/problems/coin-change/
## 思路
1. 这道题本质上是一个要求恰好装满背包，且目标是求最小价值的完全背包问题，但是有以下几点需要注意
	- 恰好装满背包问题在初始化时需要将dp[0]以外的值设为 impossible， 再加上目标是求最小值，所以我理所当然地用 INTEGER.MAX_VALUE 作为impossible的设定。但是INTEGER.MAX_VALUE 加1之后就是INTEGER.MIN_VALUE ！！所以这里可以用amount+1代表impossible， 而不能用INTEGER.MAX_VALUE!
	- 在考虑返回结果时，需要考虑是否存在恰好装满的可能，如果不存在，返回-1。
	- 这道题目其实本质上不是背包问题，因为每个硬币的价值都是1，所以用贪心也就足以做出来的！

