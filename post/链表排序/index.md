## 链表排序
https://leetcode-cn.com/problems/insertion-sort-list/submissions/
https://leetcode-cn.com/problems/sort-list/

## 插入排序
1. 哑节点的合理运用
2. 链表遍历需要两个指针
3. 插入排序的细节：寻找插入位置有两种情况，一种是在原位置，一种需要位置变动

## 快速排序
1. 前闭后开区间
2. 快排的本质是将元素按照某个基准元素分成两部分，一部分全小于基准元素，一部分全大于基准元素，具体怎么分都是可以的，因为时间复杂度是O(n)这一点是没办法避免的；

## 归并排序
1. 利用快慢指针寻找链表的中间节点,快慢指针有讲究的，想要slow落在中间节点，fast就不能为null，所以循环的判断条件必须是fast.next!=null &&fast.next.next!=null。
1. 快慢指针的循环结束的判断条件的判断依据主要在于它的粒度够不够细，也就是说它能不能把两个节点/三个节点的链表分成两部分。
2. 在链表排序中，归并排序是最优的排序方式，时间复杂度可以稳定保持O(nlogn)
3. 归并排序在数组排序中最受诟病的空间复杂度也从O(n)降到了O(1)
4. 注意在找到中间节点之后，记得将前半部分和后半部分切开；

## 代码
https://github.com/duanxinpeng/SortAlgorithms/blob/master/src/Chain.java