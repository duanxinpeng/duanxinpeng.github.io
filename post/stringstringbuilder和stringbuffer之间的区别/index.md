## String 
1. String属于对象（引用在栈中，对象在堆中），但String是不可变的，每次操作都会生成新的对象，如下图所示
![String内存图](/media/string.png)

## StringBuilder,StringBuffer
1. StringBuilder线程不安全。
2. StringBuffer线程安全。
3. StringBuffer直接通过synchronized关键字实现的线程安全。
