## 问题描述
1. 找出数组中第k大的数
2. 

## 快速选择算法
1. 第K大的元素在nums中的下标是nums.length-k, 把k转换成index之后，就很简单了，index不用变化，因为数组没有变化，变化的只有low和high！是我之前想的太复杂，既要改变low，high，又要改变k。

## PriorityQueue实现
1. 默认小根堆，想要得到大根堆，需要传入一个Comparator接口，可用lambda表达式代替：(a,b)->{return b-a;}
2. 求第K大和求前K大是不一样的做法，第K大要用大根堆，而前K大要用小根堆！
3. 时间复杂度并不优秀，nlog(n)
