## 问题描述
1. 给定一个数组，数组中的每个元素代表你能向前走的最大步数，如果你当前在数组的第一个位置，判断你能否达到数组的最后一个位置。

https://leetcode-cn.com/problems/jump-game/
## 解题思路
1. 我的第一想法是一个动态模拟，模拟一下能不能走到。
2. 其实动态模拟的思路也没错，只不过如果不引入任何变量或者数据结构，动态模拟的复杂度会非常非常高。
3. 当引入最远可达位置这样的一个全局变量之后，纯粹的动态模拟就转变为了贪心算法，即每一步都按照最远可达位置的限定来走。