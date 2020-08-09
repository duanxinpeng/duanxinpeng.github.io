## 问题描述
1. 实现一个最小栈，push,pop,getMin操作时间复杂度都是O(1)
2. https://leetcode-cn.com/problems/min-stack/
## 思路
1. 两个栈，一个queue保存所有数据，一个minQueue保存前一个栈中的最小数据。
2. push时，判断当前元素是否小于等于minQueue的栈顶元素，若小于则将该数据入两个栈，否则只入queue。
3. pop时，判断pop出的元素是否等于minQueue的栈顶元素，若等于，则minQueue也执行pop操作。
