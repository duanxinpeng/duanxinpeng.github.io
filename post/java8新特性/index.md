##
1. https://www.runoob.com/java/java8-new-features.html
## Lambda表达式
1. 允许把函数作为一个参数传进方法中。
2. 作用
	- 主要用来定义行内执行的方法类型接口
	- 简化部分匿名类的写法
## 方法引用
1. 通过方法的名字来指向一个方法
1. 可以使语言的构造更简介减少冗余
2. 方法引用使用一对冒号
	- 构造器引用：Class::new
	- 静态方法引用: Class::static_method
	- 特定类的任意对象的方法引用：Class::method
	- 特定对象的方法引用：instance::method
## 默认函数
1. 之前接口中的方法必须是抽象方法
2. jdk8之后在方法前面加一个  default 就可以实现默认方法！
## 函数式接口（jdk8之前就有，但jdk8新增了java.util.function）
1. 有且仅有一个抽象方法的接口；
2. 函数式接口可以被隐式转为lambda表达式；
## Stream
1. 就是对集合的迭代的一种增强
2. 以声明式的方式处理数据
3. 需要将被处理的元素集合看作一种流，流在管道中传输，并可以在管道的节点上进行处理。
4. 是一个来自数据源的元素队列并支持聚合操作
	- 数据源：流的来源，集合/数组/IO/generator等
	- 聚合操作：类似SQL语句一样的操作，比如filter、map、reduce、sorted等
5. 具体操作
	- stream() 生成串行流
	- parallelStream() 生成并行流
	- forEach： 迭代流中的每个数据
	- map 映射每个元素到对应的结果
	- filter 设置条件过滤元素
	- limit 获取指定数量的流
	- sorted 对流进行排序
6. 统计
	- summaryStatistics()
	- getMax()
	- getSum()
	- getAverage()
## 新的日期时间API
## Base64