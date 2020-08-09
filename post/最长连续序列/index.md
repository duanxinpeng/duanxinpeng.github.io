	
## Leecode-128
https://leetcode-cn.com/problems/longest-consecutive-sequence/
## 问题描述
给定一个未排序的整数数组，找出最长连续序列的长度。

要求时间复杂度为 __O(n)__.

__示例:__

	输入：[100,4,200,1,3,2]
	输出：4
	解释：最长连续序列是[1,2,3,4].

## 解题思路
1. 若不借助数据结构，可以先排序、去重，然后再寻找其中的最长连续序列，但复杂度达不到要求。
2. 借助 __HashSet__ 类似的数据结构，此数据结构主要起到两个作用：
	- 去重 
	- __O(1)__ 的时间判断某个数字是否在数组中出现过
3. 达到 __O(n)__ 复杂度最关键的一点是：在遍历 __HashSet__ 中的元素时，先判断遍历元素 __num__ 是否为序列头，判断方式是看 __num-1__ 是否在 __HashSet__ 中。

``` java
class Solution {
    public int longestConsecutive(int[] nums) {
        if (nums.length < 2) {
            return nums.length;
        }
        HashSet<Integer> hs = new HashSet();
        for (int num :
                nums) {
            hs.add(num);
        }
        int res = 0;
        for (int num :
                hs) {
            if (!hs.contains(num - 1)) {
                int currentNum = num;
                int currentLen = 1;
                while (hs.contains(currentNum+1)) {
                    currentNum+=1;
                    currentLen+=1;
                }
                res = Math.max(res,currentLen);
            }
            }
        return res;
    }
}
```

``` python
class Solution:
    def longestConsecutive(self, nums: List[int]) -> int:
        numset = set(nums)
        ans = 0
        for num in numset:
            if num-1 not in numset:
                tmp = 0
                while num in numset:
                    tmp+=1
                    num+=1
                if tmp > ans:
                    ans = tmp
        return ans
```

``` go
func longestConsecutive(nums []int) int {
	ans := 0
	nummap := make(map[int]bool)
	for i:=0;i< len(nums);i++ {
		nummap[nums[i]] = true
	}
	for num := range nummap {
		if !nummap[num-1] {
			tmp := 0
			for nummap[num] {
				tmp++
                num++
			}
			if tmp > ans {
				ans = tmp
			}
		}
	}
	return ans
}
```