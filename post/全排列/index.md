
## 字母大小全排列
### 题目描述
https://leetcode-cn.com/problems/letter-case-permutation/
### 收获
1. StringBuilder的用法（append）
2. foreach的用法
3. ArrayList是有序的而且可以用过下标index获取元素。

## 全排列1
### 题目描述
给出一个不包含重复数字的序列，返回其全排列。
https://leetcode-cn.com/problems/permutations/
### 体会
1. 思路很简单，就是swap+permute+swap，难点在于如何实现，不同的语言的实现方式不同
2. java实现中，用一个List<List<Integer>> res 存储结果，用一个List<Integer> tmp代替数组，以方便做swap，然后将两者传入一个函数中。
3. 在此函数中，就是swap+permute+swap的实现，并且需要有一个出口，出口处将tmp中的数据生成新的List，加入res中，记住，一定要生成新的List。

## 全排列2
### 题目描述
给定一个可包含重复元素的数字序列，返回所有不重复的全排列。
https://leetcode-cn.com/problems/permutations-ii/
### 体会
1. 有一种简单的解法就是全排列1得到所有全排列，然后去重即可
2. 另外一种效率更高的方法是回溯加减枝
3. 全排列1的代码写法是用swap在原数组中进行操作的，不需要new新的空间，但是对于全排列2不适用，因为全排列2需要保证有序，所以不能用swap，而必须用删除并添加到一个新的Dequeue中才行。
4. visited+nums表示剩余元素，path表示未完成排列，res中存放结果
5. 之所以用visited，是因为java中没有办法将一个元素从数组中删除之后再撤销的操作，所以用visited来辅助。