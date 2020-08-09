## 问题描述
1. 根据数据大小以及数据位置（前面有几个比他大的元素）来重排这些元素
2. https://leetcode-cn.com/problems/queue-reconstruction-by-height/
## 思路
1. 相同身高的人，按照其所处位置排序
2. 不同身高的人，应该先排身高高的人，然后再将身高低的依次插进去，因为身高低的人根本影响不到身高高的人。
## code
1. int[]和Integer[]不能相互转换
2. List<int[]> list 转为二维数组:list.toArray(new int[m][n])
