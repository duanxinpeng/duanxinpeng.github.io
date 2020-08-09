## 问题描述
将一个字符串数组中的字母异位词分组，字母异位词指的是含有字母相同，但排列不同的单词。

https://leetcode-cn.com/problems/group-anagrams/

## 解题思路
1. 用到了HashMap<String,List<String>>
2. 将String转为字符数组，排序之后再转为String，用以判断两个字符串是否是字母异位词。
3. 使用String.valueOf()将字符数组转换成字符串
4. 使用.toCharArray()将字符串转换成字符数组
