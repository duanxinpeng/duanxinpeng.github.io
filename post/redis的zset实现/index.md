## redis的 zset 为什么使用 skiplist 而不用 RB-tree？
1. 需要更少的指针内存，当晋升概率设为 1/4 时，一个节点平均需要 1.33 个指针。
2. 跳表的范围查询比红黑树效率更高
3. 更简单，便于实现和调试
## zset 
1. zset 的定义如下
	```
	typedef struct zset {
		dict *dict;
		zskiplist *zs1;
	} zset;
	```
2. zset 中除了跳表 zskiplist 之外，还有一个字典类型 dict 
	- dict 维护了 zset 中的 element 与 score的映射，用于快速查找 element 对应的 score，以及判断 element 是否存在。
	- 因为本质还是一个 set 嘛， 所以肯定需要有一个 dict 用于存储的！

## skiplist 实现
```
import java.util.Random;

class SkiplistLevel {
    SkiplistNode forward;
    int span;

    public SkiplistLevel() {
        this.forward = null;
        // 当前节点距离下一个节点的跨度 不包括当前节点，包括下一个节点！
        this.span = 0;
    }
}

class SkiplistNode {
    int value;
    SkiplistLevel[] levels;

    public SkiplistNode(int value, int level) {
        this.value = value;
        this.levels = new SkiplistLevel[level];
    }
}

public class Skiplist {
    // level 允许的最大值
    final int MAXLEVEL = 32;
    // 头节点，数组长度为 32；
    SkiplistNode header;
    // 跳表的长度，不包括头指针
    int length;
    // 链表的最大层数
    int level;

    public Skiplist() {
        this.length = 0;
        this.level = 1;
        //哑节点
        this.header = new SkiplistNode(-1, MAXLEVEL);
    }

    public void insert(int value) {
        //记录每一层应该插入位置的前一个节点
        SkiplistNode[] update = new SkiplistNode[MAXLEVEL];
        // 记录每一层应该插入的位置的前一个节点的排名！
        int[] rank = new int[MAXLEVEL];
        //// 必须在 for 循环之外，才能使得时间复杂度为 O(logn)
        SkiplistNode p = this.header;
        for (int i = this.level - 1; i >= 0; i--) {
            if (i == this.level - 1) {
                rank[i] = 0;
            } else {
                rank[i] = rank[i + 1];
            }
            //
            while (p.levels[i].forward != null && p.levels[i].forward.value < value) {
                rank[i] += p.levels[i].span;
                p = p.levels[i].forward;
            }
            update[i] = p;
        }

        int level = randomLevel();
        if (level > this.level) {
            for (int i = this.level; i < level; i++) {
                // 因为上一个节点是 header 哑节点！
                rank[i] = 0;
                update[i] = this.header;
                //  header 在当前 lever 之上的每一层的 span 都是 length？
                update[i].levels[i].span = this.length;
            }
            this.level = level;
        }

        SkiplistNode x = new SkiplistNode(value, level);
        for (int i = 0; i < level; i++) {
            // 将 x 插入跳表的第 i 层
            x.levels[i].forward = update[i].levels[i].forward;
            update[i].levels[i].forward = x;
            //更新 x 的第 i 层的 span
            x.levels[i].span = update[i].levels[i].span - (rank[0] - rank[i]);
            update[i].levels[i].span = (rank[0] - rank[i]) + 1;
        }
        // level 比 this.level 小的情况 span++
        for (int i = level; i < this.level; i++) {
            update[i].levels[i].span++;
        }
        this.length++;
    }

    public int getRank(int value) {
        SkiplistNode p = this.header;
        int rank = 0;
        for (int i = this.level - 1; i >= 0; i--) {
            // 这里最后变成了 <= value， 不同于前面哪些
            while (p.levels[i].forward != null && p.levels[i].forward.value <= value) {
                rank += p.levels[i].span;
                p = p.levels[i].forward;
            }
            if (p.value == value)
                return rank;
        }
        return 0;
    }

    public int getByRank(int rank) {
        SkiplistNode p = this.header;
        int traversed = 0;
        for (int i = this.level - 1; i >= 0; i--) {
            while (p.levels[i].forward != null && traversed + p.levels[i].span <= rank) {
                traversed += p.levels[i].span;
                p = p.levels[i].forward;
            }
            if (traversed == rank)
                return p.value;
        }
        return -1;
    }

    public boolean delete(int value) {
        SkiplistNode[] update = new SkiplistNode[MAXLEVEL];
        SkiplistNode p = this.header;
        for (int i = this.level - 1; i >= 0; i--) {
            while (p.levels[i].forward != null && p.levels[i].forward.value < value) {
                p = p.levels[i].forward;
            }
            update[i] = p;
        }
        //要删除额节点
        p = p.levels[0].forward;
        // 先判断是否为 null 是有可能为 null 的
        if (p != null && p.value == value) {
            for (int i = 0; i < this.level; i++) {
                // 这里update[i].levels[i].forward 也是有可能为 null 的！
                if (update[i].levels[i].forward == p) {
                    update[i].levels[i].span += p.levels[i].span - 1;
                    update[i].levels[i].forward = p.levels[i].forward;
                } else {
                    update[i].levels[i].span -= 1;
                }
            }
            // 删除了最高层数的节点，这个节点可能比其他节点高出不止一个 level！
            while (this.level > 1 && this.header.levels[this.level - 1].forward == null)
                this.level--;
            this.length--;
            return true;
        }
        return false;
    }

    public int randomLevel() {
        int level = 1;
        //[0.0, 1.0)
        while (Math.random() < 0.25) {
            level++;
            if (level == MAXLEVEL)
                break;
        }
        return level;
    }
}

```
## zset 的 skiplist 实现和普通的 skiplist 的区别
	- 底层是一个双向链表，便于反向查询数据
	- 允许重复的 score 出现，当 score 相同时，则比较 element 的字典大小。（element 不允许重复）
	- 节点中还维护了一个 span 字段，表示`当前节点距离下一个节点的跨度（跨度指的是从前驱节点到下一个节点的经过的节点的个数，比如在最底层，跨度都是1，level越高，跨度越大）`，用于快速查找某个元素的排名。
## skiplist 
1. 概率
	- 当p=1/2时，每个节点所包含的平均指针数目为2；
	- 当p=1/4时，每个节点所包含的平均指针数目为1.33。这也是Redis里的skiplist实现在空间上的开销。
2. span 
	- 跨度指的是当前节点到下一个节点之间经过的节点的个数！
	- 比如header和tailer之间的跨度就是跳表中的所有的节点。
	- 层数越高，节点越稀疏，两个节点的跨度就越大。
	- 第0层包含所有节点，所以每个节点和该节点的next之间的跨度都是1！
2. insert注意事项
	- 查找每层的插入位置的前驱节点。
	- 为了方便对插入节点和其前驱节点的span进行更改，还应该查找每一层前驱节点的排名（span累积）。
	- 通过抛硬币的方式得到新插入节点的level，此时应该注意level的几种情况
		- level小于跳表的level：需要注意跳表中比新加节点的level更高的节点的span应该加1！
		- level大于跳表的level：此时对于大于跳表level的前驱节点都是header，rank都是0（header的rank），span都是length（从header到tailer，包含跳表中的所有节点）。
	- 最后将跳表length加1.
3. delete注意事项
	- 首先通过查找找到删除节点的前驱节点（注意比较高的level中可能不存在这样的节点）！
	- 所以在删除的时候需要判断被删除节点是否存在，通过`前驱节点.next==p` 来判断（注意前驱节点.next不一定就是null了，或者说大部分都不是null）。
	- 删除一个节点对每一层的前驱节点都有影响，不包含被删除节点的层：span-1； 包含被删除节点的层：span+被删除节点的sapn-1；
	- 注意删除节点对跳表level的影响。
4. getByRank 
	- 和查找过程类似
5. getRank 
	- 和查找过程类似
	
	

