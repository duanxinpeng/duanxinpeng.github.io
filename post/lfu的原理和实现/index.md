
## LFU
https://leetcode-cn.com/problems/lfu-cache/

## HashMap+TreeSet实现
在HashMap中存放键值对，在TreeSet中存放节点的使用频率、更新时间的先后关系；
### get O(logN)
1. 判断HashMap中是否存在key，若不存在返回-1；
2. 若存在则将对应的value值返回，更新其使用频率及更新时间，并在TreeSet中更新其位置（更新用到了一次删除+一次插入，都是logn的时间）
### put O(logN)
1. 判断HashMap中是否存在key，若存在，则过程和get过程类似，只不过还需要改一下对应的node的value值。
2. 若不存在，则需要插入新节点，此时需要判断 size和capacity的关系
3. 若size==capacity，需要删除（HashMap，TreeSet中都要删）其使用频率最小（时间最久）的node，也就是TreeSet.first。
4. 生成新的node，并分别将其插入HashMap与TreeSet中。
### 代码
https://leetcode-cn.com/problems/lfu-cache/submissions/
1. Comparable的使用方式
2. TreeSet的first()函数
3. get只需要改TreeSet即可，当put时无需插入新node时（已经含有原key）和get类似
4. get如果需要插入新node时，即需要改TreeSet，也需要改HashMap。

## HashMap+HashMap+双向链表实现
LFU和LRU的本质不同在于，对频率进行更改之后，并不是简单地将该节点提到链表头，而是需要更精细粒度的定位应该提前到的位置，所以不能简单地用双向链表来存储node间的优先关系，而是要用HashMap来存储；而且当频率相同的时候，LFU相当于嵌套了一个LRU，所以实现起来稍显麻烦。需要的两个HashMap分别为fmap（频率为key，双向链表为value），nmap（key为key，node为value）！还需要一个变量minFreq来表明当前最小的频率。
### get O(1)
1. 判断nmap中是否存在key，若不存在则返回-1；
2. 如果存在该节点，对该节点的频率加1之后，会影响两个地方：fmap和minFreq，并不会影响nmap；

### put O(1)
1. 首先判断存不存在与nmap，如果存在，流程和get差不多
2. 如果不存在，需要插入新的节点，此时需要判断是否还有容量，没有容量的话，需要删除一个minFreq对应的节点。
3. 分别向nmap，fmap插入新节点

### 实现
https://leetcode-cn.com/problems/lfu-cache/submissions/