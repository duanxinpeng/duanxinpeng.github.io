## 0/1背包
1. 有 N 件物品和一个容量为 V 的背包。每一件物品 i 的体积和价值分别是 $C_i$ 和 $W_i$, 请问将哪些物品装入背包可使价值总和最大？
2. 状态转移方程：$dp[i,v] = \max(dp[i-1,v],dp[i-1,v-C_i]+W_i)$，对应伪代码为：
	```
	dp[0][0...V] = 0
	for i=1 to N
		for v = C_i to V
			dp[i][v] = max(dp[i-1][v],dp[i-1][v-C_i]+W_i)
	```
3. 优化空间复杂度：只需要逆序遍历v值即可，即从0~v 变为 v~0，同时将F数组中的每一个位置初始化为0，此时状态转移方程变为 $dp[v] = \max(dp[v], dp[v-C_i]+W_i)$.
	```
	dp[0...V] = 0
	for i=1 to N
		for v = V to C_i
			dp[v] = max(dp[v],dp[v-C_i]+W_i)
	```
4. 恰好装满问题：如果题目要求恰好装满时的最优解，那么在初始化时，把 dp[0]初始化为0，同时把其他所有的位置初始化为负无穷即可。这样的话，就只有在 $C_i$ 和容量相等的时候，才能恰好到dp[0]的位置，也只有当 $C_i$ 和 v 相等时，才能恰好装满背包。
	```
	dp[0] = 0
	dp[1...V] = 0
	for i=1 to N
		for v = V to C_i
			dp[v] = max(dp[v],dp[v-C_i]+W_i)
	```
5. https://www.nowcoder.com/questionTerminal/7e157ce9a8c249daa3ddafad322dbf1e

## 完全背包
1. 与0/1背包的区别在于每件物品有无穷多个，所以每件物品可重复选择
2. 转换为0/1背包
	- 考虑到第 i 件最多选 $\lfloor V/C_i \rfloor$ 件，于是可以把第 i 种物品转化为 $\lfloor V/C_i \rfloor$ 件费用及价值均不变的物品，然后求解这个0/1背包问题。
	- 把第 i 种物品拆分成费用为 $C_i2^k$, 价值为 $W_i2^k$ 的若干件物品，其中 k 取满足 $C_i2^k \leq V$ 的非负整数。（二进制思想，因为任何整数都可以拆分为若干个 $2^k$ 之和，比如 13 可以拆分为 8+4+1, 这是因为二进制可以表示出任何一个整数！）
3. 最简单的方法是调整0/1背包的遍历顺序即可！这是因为调整顺序之后的算法对应的状态转移方程变为：$$dp[i][v] = \max(dp[i-1][v],dp[i][v-C_i]+W_i)$$
	- 倒着来就是第 i 件物品不可以重复使用：$（V - C_i）$：$dp[v] = \max(dp[v], dp[v-C_i]+W_i)$，即 $dp[i,v] = \max(dp[i-1,v],dp[i-1,v-C_i]+W_i)$.
	- 正着来就是第 i 件物品可以重复使用：$(C_i - V$：$dp[v] = \max(dp[v], dp[v-C_i]+W_i)$，即 $dp[i][v] = \max(dp[i-1][v],dp[i][v-C_i]+W_i)$。
	```
	dp[0...V] = 0
	for i=1 to N
		for v = C_i to V
			dp[v] = max(dp[v],dp[v-C_i]+W_i)
	```
## 多重背包
1. 类似0/1背包和完全背包，只不过每种物品的件数是固定的
2. 转换为0/1背包
	- 将第 i 种物品视为 $M_i$ 件物品，即可转为0/1背包问题。
	- 二进制优化：将第 i 种物品视为容量和价值分别以 $1, 2, 2^2, ..., 2^{k-1},M-2^k+1$ 为系数的物品。此时，k 应该是满足 $M_i - 2^k +1 > 0$ 的最大整数。对应的伪代码为；
	```
	def MultiplePack(dp,C,W,M)
		if C*M >= M
			CompletePack(dp,C,W)
		k=1
		while k < M 
			ZeroOnPack(kC,kW)
			M = M-k
			k = 2k
		ZeroOnePack(C*M,W*M)
	```
3. https://www.nowcoder.com/questionTerminal/6ce78d70a25347058004691035d7540b

## 混合三种背包问题
1. 之所以能混合，是因为一个元素是否可以重复选取只和该元素有关，和其他元素无关，也就是说无需考虑外层循环之间的关系对dp的影响！
1. 0/1背包与完全背包的混合
	```
	for i=1 to N
		if 第 i 件物品属于0/1背包
			for v = V to C_i
				dp[v] = max(dp[v], dp[v-C_i)] + W_i)
		else if 第 i 件物品属于完全背包
			for v = C_i to V
				dp[v] = max(dp[v], dp[v-c_i] + w_i)
	```
2. 三种背包的混合
	```
	for i = 1 to N
		if 第 i 件物品属于 01 背包
			ZeroOnePack(dp,C_i,W_i)
		else if 第 i 件物品属于完全背包
			CompletePack(dp, C_i, W_i)
		else if 第 i 件物品属于多重背包
			MultiplePack(dp, C_i,W_i, M_i)
	```

## 二维费用的背包问题
1. 指的是该背包问题不但有容量的限制，还有重量的限制，在体积和重量的双重限制之下，找到最大的价值。
2. 费用加了一维，只需状态也增加一维即可
3. 状态转移方程：$dp[i][v][u] = \max(dp[i-1][v][u], dp[i-1][v-C_i][u-D_i] + W_i)
4. 0/1背包二维费用伪代码
	```
	for i=1 to N
		for v=V to C_i
			for u=U to D_i 
				dp[v][u] = max(dp[v][u], dp[v-C_i][u-D_i] + W_i)
	```
5. 物品总个数限制：有时“二位费用”的条件是以一种隐含的方式给出的：最多只能取 U 件物品。

## 分组的背包问题
1. 有 N 件物品和一个容量为 V 的背包。第 i 件物品的费用是 $C_i$ ，价值是 $W_i$ 。这些 物品被划分为 K 组，每组中的物品互相冲突，最多选一件。求解将哪些物品装入背包 可使这些物品的费用总和不超过背包容量，且价值总和最大。
2. 状态转移方程：$dp[k][v] = \max(dp[k-1][v], dp[k-1][v-C_i] + W_i| i \in group k)$
3. 伪代码，下面的三层循环，保证了每组中最多只有一个会被添加到背包中。
	```
	for k=1 to K
		for v = V to 0
			for all i in group k
				dp[v] = max(dp[v], dp[v-C_i]+W_i)
	```

## 有依赖的背包问题
1. 不依赖于别的物品的物品被称为“主体”，依赖于某主件的物品称为“附件”。本问题的简化条件是没有某个物品即依赖别的物品又被别的物品所依赖，且没有某件物品同时依赖多件物品。由这个问题的简化条件可知所有的物品由若干主件和依赖于每个主件的一个附件集合组成。
2. 首先将一个主件和它的附件集看作`分组背包问题`中的一个分组，在分组之中每个可用的选择策略对应于一个物品（选了主件又选了若干附件，共$2^n+1$种）；为了减少物品组种“物品（策略）”的个数，先在物品组中做一次0/1背包，得到费用依次为 $0...V-C_k$ 所有这些值对应的最大价值 $dp[0...V-C_k]$, 那么，这个主件及他的附件集合相当于 $V-C_k+1$ 个物品的物品组，其中费用为 $v$ 的物品的价值为 $dp[v-C_k]+W_k$, v 的取值范围是 $C_k\leq v \leq V$。之后就可以采用`分组背包问题`的解决方法了。
3. 较一般的问题：依赖关系以“森林”的形式给出，也就是说主件的附件仍然可以拥有自己的附件集合。限制是每个物品最多只依赖于一个物品，且不出现循环依赖。
4. 解决这个问题仍然可以用将每个主件及其附件集合转化为物品组的方式。唯一不同的是，由于附件可能还有附件，就不能将每个附件都看作一个一般的01背包中的物品了。若这个附件也有附件集合，则它必定要被先转化为物品组，然后用分组的背包问题解出主件及其附件集合所对应的附件组中各个费用的附件所对应的价值。
5. 事实上，这是一种树形动态规划，其特点是，在用动态规划求每个父节点的属性之前，需要对它的各个儿子的属性进行一次动态规划式的求值。这已经触及到了“泛化物品”的思想。
6. 这个“依赖关系树”每一个子树都等价于一件泛化物品，求某节点为根的子树对应的泛化物品相当于求其所有儿子的对应的泛化物品之和。

## 泛化物品
1. 考虑这样一种物品，它并没有固定的费用和价值，而是它的价值随着你分配给它的费用而变化。这就是泛化物品的概念。
2. 在背包容量为 V 的背包问题中，泛化物品是一个定义域为 $0...V$ 中的整数的函数 h ，当分配给它的费用为 v 时，能得到的价值就是 h(v)。
3. 一个泛化物品就是一个数组 $h[0...V ]$，给它费用 $v$ ，可得到价值 $h[v]$ 。
4. 一个费用为 $c$ 价值为 $w$ 的物品，如果它是 01 背包中的物品，那么把它看成泛化物品，它就是除了 $h(c) = w$ 外，其它函数值都为 0 的一个函数。
5. 如果它是完全背包中的物品，那么它可以看成这样一个函数，仅当 $v$ 被 $c$ 整除时有 $h(v) = w*\frac{v}{c}$，其它函数值均为 0 。
6. 如果它是多重背包中重复次数最多为 $m$ 的物品，那么它对应的泛化物品的函数有 $h(v) = w*\frac{v}{c}$ 仅当 $v$ 被 $c$ 整除且 $\frac{v}{c} \leq n$ ，其它情况函数值均为 0 。
7. 一个物品组可以看作一个泛化物品 $h$ 。对于一个 $0...V$ 中的 $v$ ，若物品组中不存在费用为 $v$ 的物品，则 $h(v) = 0$ ，否则 $h(v)$ 取值为所有费用为 $v$ 的物品的最大价值。
8. 泛化物品的和：$f(v) = \max (h(k) + l(v-k)|0 \leq k \leq v)$ , 即 $f$ 是一个由泛化物品 $h$ 和 $l$ 决定的泛化物品。
9. 一般而言，求解背包问题，即求解这个问题所对应的一个函数，即该问题的泛化物品。而求解某个泛化物品的一种常用方法就是将它表示为若干泛化物品的和然后求之。

## 背包问题问法的变化
1. 输出方案
2. 输出字典序最小的最优方案
3. 求方案总数
4. 最优方案的总数
5. 求次优解，第K优解