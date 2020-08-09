## 动态规划框架
1. 状态转移方程
	- dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1] + prices[i]) 
	- dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i])
2. base case i=-1 and k = 0
	- dp[-1][k][0] = 0 : i = -1 表示第-1天，意味着还没开始
	- dp[-1][k][1] = -infinity ： 表示还没开始就已经持有股票，不可能发生。
	- dp[i][0][0] = 0 : k=0 表示不允许交易
	- dp[i][0][1] = -infinity : 表示不允许交易的情况下还持有股票，不可能发生
	
## 买卖股票的最佳时机I
1. 给定一个数组，它的第 i 个元素是一支给定股票在第 i 天的价格，只允许一笔交易，求最大利润
2. https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/
3. 暴力法
4. 记录数组当前位置之前元素的最小值
5. 动态规划框架

## 买卖股票的最佳时机II
1. 无限次交易，求最大利润。
2. https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/
3. 贪婪法
	- 波峰波谷法
	- 爬坡法
4. 动态规划框架
	- 设k为交易次数，不限制交易次数相当于$k = infinity$。
	- dp 里面的 $k$ 不应该理解为最多交易多少次，而是应该理解为已经交易了多少次，交易次数最大不能超过k！因为 k 是无穷大的，所以意思就是没有交易次数限制。
## 买卖股票的最佳时机（含冷冻期） 
1. 无限次交易，但每次交易都有一个冷冻期（1），求最大利润。
2. https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/
2. 动态规划框架	
	- dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1]+prics[i])
	- dp[i][k][1] = max(dp[i-1][k][1],dp[i-2][k-1][0]-prices[i])
	- 之所以将上述状态转移方程改为 dp[i-2][k-1][0] 是因为 dp[i-1][k-1][0]处有两种情况：1）处于冷冻状态，此时只能看dp[i-2][k-1][0]。2）dp[i-1][k-1][0]处于非冷冻状态，此时dp[i-1][k-1][0]与dp[i-2][k-1][0]的状态是一样的！

## 买卖股票的最佳时机（含手续费）
1. 无限次交易，但每次交易都有手续非，求最大利润。
2. https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/
2. 动态规划框架，只需要在每次购买股票时将手续费用减掉即可！

## 买股票的最佳时机III
1. 最多允许两次交易，求最大利润
2. https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii/
3. 动态规划框架
	- 正常的三维数组即可，此时需要注意对k进行遍历。
	
## 买卖股票的最佳时机Ⅳ
1. 最多允许K次交易，求最大利润。
2. https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iv/
3. 动态规划框架
	- 当k>n/2时，应转为无限次交易问题求解，否则容易出现超内存问题。
	- 注意base case情况：其一是i=0的情况，其二是k=0的情况。
