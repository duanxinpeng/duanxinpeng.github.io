## 问题描述
1. 给定一个无序的整数数组，找到其中最长上升子序列的长度
2. https://leetcode-cn.com/problems/longest-increasing-subsequence/
## 动态规划
1. dp[i]代表以nums[i]结尾的最长上升子序列的长度，dp[i]=max(dp[j])+1,其中nums[i] > nums[j], j = 0,1,2,3,..i-1;
## 贪心+二分查找
1. 贪心：想要让上升子序列尽可能长，我们必须要让序列上升的尽可能慢！
2. 
