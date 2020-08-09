## 动态规划
1. 一个模型
	- 多阶段决策最优解模型
2. 三个特征
	- 最优子结构：最优解包含子问题的最优解，或者理解为“后面的状态可以通过前面的状态推导出来”。
	- 无后效性：当前阶段的状态和之前阶段经历的过程无关，此后阶段的发展变化也仅和当前阶段的状态有关，而与此前阶段的变化过程无关。也就是说动态规划不关心过程，只关心阶段结果。回溯算法会关注并记录过程，所以复杂度高。
	- 重复子问题
	
## 子串/子序列问题
### 和最大连续子数组
1. 给定一个整数数组，求其中`连续子数组`的最大和。
1. 分为不同的阶段！在每个阶段应用动态规划，同时用一个全局变量记录全局最优。
2. dp[i] = Math.max(dp[i-1]+nums[i], nums[i])
3. 还有一种利用线段树的解法，暂时先放弃了。
4. https://leetcode-cn.com/problems/maximum-subarray/
### 最长上升子序列
1. 给定一个无序的数组，找到其中`最长上升子序列`的长度
2. https://leetcode-cn.com/problems/longest-increasing-subsequence/
3. 状态应该是一个一维数组，dp, dp[i]表示以i结尾的最长上升子序列长度。
4. 也需要一个全局变量记录全局最优
5. 因为不是连续的，所以不能只看前一个状态，需要把当前状态之前的所有状态都考虑进来！
4. $dp[i] = \max(dp[j]+1 | (0\leq j < i && nums[j] < nums[i]))$ 
1. 可以通过二分查找对时间复杂度进行优化，暂不考虑。
### 乘积最大连续子数组
1. 给定一个整数数组，请找出数组中乘积最大的连续子数组，并返回该子数组所对应的乘积。
2. https://leetcode-cn.com/problems/maximum-product-subarray/
3. 由于数组中的整数有正有负，所以即需要保存最大值，也需要保存最小值。因为负值需要乘以最小值才能得到最大值。这也就是通过扩展状态数组来消除后效性。
4. 状态转移方程如下：
	- $nums[i] \geq 0$
		- $dp[i][0] = \min(nums[i], dp[i-1][0]*nums[i])$
		- $dp[i][1] = \max(nums[i], dp[i-1][1]*nums[i])$
	- $nums[i] < 0$
		- $dp[i][0] = \min(nums[i], dp[i-1][1]*nums[i])$
		- $dp[i][1] = \max(nums[i], dp[i-1][0]*nums[i])$
5. 考虑动态规划的两种思路
	- 自顶向下：递归+记忆化
	- 自底向上：效率高，但需要逆向思维。
	
### 摆动序列
1. 给定一个整数序列，返回其最长摆动子序列的长度。
2. https://leetcode-cn.com/problems/wiggle-subsequence/
3. 思路
	- 这是一个子序列问题，所以套用子序列的动态规划模板：以 i 结尾； 遍历 i 之前的所有位置求最优；
	- 另外这道题具有后效性，需要通过扩展dp数组来消除后效性！
4. 状态转移方程如下：
	- $nums[i] - nums[j] > 0 $
		- $dp[i][1] = \max(dp[j][0]+1 | 0 \leq j < i)$
	- $nums[i] - nums[j] < 0$
		- $dp[i][0] = \max(dp[j][1]+1 | 0 \leq j < i)$
### 最长回文子串
1. https://leetcode-cn.com/problems/longest-palindromic-substring/
2. dp[i][j] 代表以i开头， 以j 结尾的子串是否是回文字符串。
3. 关键在于循环的方式,需要考虑到回文字符串的特点！
4. 状态转移方程如下：
	```
	if j-i == 0 
		dp[i][j] = true 
	else if j-i == 1
		if s.charAt(i) == s.charAt(j)
			dp[i][j] = true 
	else 
		if dp[i+1][j-1] && s.charAt(i) == s.charAt(j)
			dp[i][j] = true
	```
### 最长回文子序列
1. https://leetcode-cn.com/problems/longest-palindromic-subsequence/
2. 第一眼看起来比`最长回文子串`难很多，但是其实并不是，因为这里的 dp[i][j] 并不是用于判断以 i 开头以 j 结尾的字符串是不是回文字符串，而是直接记录以 i 开头以 j 结尾的字符串内的最长回文子序列的长度！
3. 状态转移过程如下：
	```
	if j-i == 0
		dp[i][j] = 0
	else if j-i == 1
		if s.charAt(i) == s.charAt(j)
			dp[i][j] = 2
		else 
			dp[i][j] = 1
	else 
		if s.charAt(i) == s.charAt(j)
			dp[i][j] = dp[i+1][j-1] + 2
		else 
			dp[i][j] = max(dp[i+1][j], dp[i][j-1])
	```
### 最长公共子串
1. https://www.nowcoder.com/questionTerminal/210741385d37490c97446aa50874e62d
### 最长公共子序列
1. https://leetcode-cn.com/problems/longest-common-subsequence/
2. 伪代码
	```
	if i==0 || j == 0
		dp[i][j] = 0
	else if s1.charAt(i) == s2.charAt(j)
		dp[i][j] = 1+dp[i-1][j-1]
	else 
		dp[i][j] = max(dp[i-1][j], dp[i][j-1])
	```
### 最短公共超序列
1. https://leetcode-cn.com/problems/shortest-common-supersequence/
2. 思考方式和最长公共子序列相同，但需要考虑到内存问题，所以需要利用滚动数组对空间复杂度进行优化。
		
		
## 消除后效性问题（增加维度使得状态设计满足无后效性）
### 按摩师
1. https://leetcode-cn.com/problems/the-masseuse-lcci/
### 打家劫舍系列问题
1. https://leetcode-cn.com/problems/house-robber/
2. https://leetcode-cn.com/problems/house-robber-ii/
3. https://leetcode-cn.com/problems/house-robber-iii/
### 股票买卖时机系列问题
1. https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/
2. https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/
3. https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii/
4. https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iv/
5. https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/
6. https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/