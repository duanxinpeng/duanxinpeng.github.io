## 适用于滑动窗口的题目的特点

## 滑动窗口模式
### 最短
1. 先移动右指针，直至满足条件
2. 再移动左指针，直至不满足条件
### 最长
1. 先移动右指针，直至不满足条件
2. 再移动左指针，直至满足条件
## 题目
### 无重复字符的最长子串
1. 给定一个字符串，请找出其中不含有重复字符的最长子串的长度。
2. https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/
### 最小覆盖子串
1. https://leetcode-cn.com/problems/minimum-window-substring/
2. 关键在于如何判断窗口内是否包含了字符串 t 中的所有字符
	- 用一个 count 记录在窗口中包含的字符串 t 的字符个数
### 找到字符串中所有字母异位词
1. https://leetcode-cn.com/problems/find-all-anagrams-in-a-string/
2. 解决思路和`最小覆盖字串`相同，唯一的不同在于还需要判断`right-left+1 == plen`.
3. 
### 长度最小的子数组
1. https://leetcode-cn.com/problems/minimum-size-subarray-sum/
2. 思路同最小覆盖子串一样，而且判断比最小覆盖字串简单多了！ 
### 滑动窗口最大值
1. https://leetcode-cn.com/problems/sliding-window-maximum/
2. 重点考察的是单调队列，而不是滑动窗口！
### 滑动窗口中位数
1. 
