## 继承关系
### Iterable/Collection
1. List
	- CopyOnWriteList
	- Vector --- Stack
	- ArrayList
	- LinkedList
2. Set
	- HashSet --- LinkedHashSet
	- SortedSet --- TreeSet
	- EnumSet
	- CopyOnWriteArraySet
	- ConcurrentSkipListSet
3. Queue
	- Deque
		- ArrayDeque
		- BlockingDeque
	- BlockingQueue
		- ArrayBlocingQueue
		- LinkedBlockingQueue
		- SynchronousQueue
		- PriorityBlockingQueue
	- PriorityQueue

### Map
1. HashMap
2. HashTable
2. TreeMap
3. ConcurrentHashMap
4. LinkedHashMap：LRU

## 常见问题
1. Map没有实现iterable怎么遍历？
	- 通过内部生成Collection来遍历，比如keySet，values
2. ArrayList和LinkedList的区别？
	- ArrayList底层是一个数组，LinkedList底层是一个双向链表
	- ArrayList和LinkedList的区别其实就是数组和和链表的区别
		- 数组：get和set快，insert和delete慢
		- 链表：insert和delete快，get和set慢
	- LinkedList不支持高效的随机访问，ArrayList支持快速随机访问。
3. HashMap和HashTable的区别
	- HashMap线程不安全，HashTable线程安全（每个方法都被synchronized修饰）
	- HashMap的key和value都可以为null，HashTable的key和value都不可以为null。
	- 他们的底层实现是相同的，都是数组加链表。
4. vector和arraylist的区别
	- vector是线程安全的，arraylist是线程不安全的。

