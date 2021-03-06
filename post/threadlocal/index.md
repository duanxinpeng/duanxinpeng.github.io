## 强软弱虚
1. 强引用：
	- 只要强引用存在，被引用的对象就永远不会被回收
2. 软引用：
	- 只被软引用关联着的对象在系统将要发生内存溢出时就会被列入回收范进行回收
	- jdk1.2版本之后提供了SoftReference类来实现软引用
	- 一般被用于缓存
3. 弱引用
	- 只被弱引用关联的对象只能存活到下一次垃圾回收为止
	- jdk1.2 之后提供了 WeakReference类来实现弱引用
4. 虚引用
	- 它是最弱的一种引用关系
	- 一个对象是否有虚引用，完全不会对其生存时间产生影响
	- 无法通过虚引用来取得一个对象的实例
	- 唯一作用是当这个对象被回收时收到一个系统通知
	- jdk1.2 之后提供了PhantomReference类来实现虚引用
```java
import java.lang.ref.PhantomReference;
import java.lang.ref.ReferenceQueue;
import java.lang.ref.SoftReference;
import java.lang.ref.WeakReference;
import java.util.ArrayList;

public class Reference {
    public static void softReferenceTest() {
        SoftReference<String> softReference = new SoftReference<>(new String("hello"));
        System.out.println(softReference.get());
        System.gc();
        System.out.println(softReference.get());
    }
    public static void weakReferenceTest() {
        WeakReference<String> weakReference = new WeakReference<>(new String("hello"));
        System.out.println(weakReference.get());
        System.gc();
        System.out.println(weakReference.get());
    }

    public static void phantomReferenceTest() {
        ReferenceQueue<String> referenceQueue = new ReferenceQueue<>();
        // 不同于前面几种引用，该引用有两个参数
        PhantomReference<String> phantomReference = new PhantomReference<>(new String("hello"),referenceQueue);
        System.out.println(phantomReference.get());
        System.gc();
        System.out.println(referenceQueue.poll());
    }
    public static void main(String[] args) {
        //softReferenceTest();
        //weakReferenceTest();
        phantomReferenceTest();
    }
}
```
## ThreadLocal
1. 作用
	- 将变量于线程绑定，某个线程内的一系列调用保证每个调用都使用同一个变量。
	- 比如同一个事务内如果使用不同的连接，就饿没办法统一管理事务，所以需要用ThreadLocal保证同一个线程得到的连接都是同一个。
2. ThreadLocal
	- 将ThreadLcoal本身作为key，将变量作为value，放入当前线程的ThreadLocalMap中！
	- 本质上其实是将value封装到一个Entry对象中，而Entry的本质是一个指向key（ThreadLocal对象）的弱引用，但是Entry中还有一个指向value的强引用，这也是导致threadlocal内存泄露的关键（自定义的ThreadLocal是一个强引用，这个强引用断开之后，ThreadLocal（key）对象只有Entry这个弱引用，下一次垃圾回收会自动被回收，但是entry不会被回收，entry中指向value的强引用就一直存在！从而导致内存泄露！）
3. ThreadLocalMap 
	- 本质是一个散列表（一个Entry数组！），不同于HashMap的是，ThreadLocalMap采用的是线性探测法来解决哈希冲突。
	- set(): 先用哈希函数得到key（ThreadLocal对象）的哈希值，再通过取模找到数组中的位置，如果不为空，则从当前位置向后查找直到找到第一个不为空的位置
	- get(): 哈希函数得到ke的哈希值，再取模找到在数组中的位置，再比较是否和key相等，如果不等则向后寻找直到找到目标或者空桶为止
	- remove(): 除了将对应位置置空外，还需要将其后不为null的元素进行rehash！
4. 存在的问题
	- 正如上面所说，Entry是一个指向ThreadLocal的弱引用，当ThreadLocal的强引用（在代码中自己声明的）被置为空时，ThreadLocal对象会在下一次垃圾回收发生时自动被回收。
	- 但是Entry中还有一个指向value的强引用（this.value)，这将导致value一直无法被回收进而导致内存泄漏。
	- 如何避免？及时调用remove方法，去掉Entry与map的联系，那么在下一次垃圾回收时Entry对象（包括其中的value）就会被自动回收了！
5. 补充
	- get：先从当前线程Thread中取得ThreadLocalMap，再以ThreadLocal为key，从ThreadLocalMap中取得对应的value。
	- ThreadLocalMap是一个散列表，其中存放的是一个指向ThreadLocal的弱引用Entry。
	- Entry本身是一个指向ThreadLocal的弱引用，同时其中包含对应于线程的value值。
	- 指向ThreadLocal的弱引用解决了key的问题，但是value的问题没有解决，而且因为这些value很难再被访问到了
	- Map中的value之所以一直存在是因为来自Current Thread的强引用，所以只要线程结束了，map中的value自然就被回收了。
	- 而且为了减小内存泄漏的可能性，在ThreadLocal进行get和set的时候，都会清除掉Map中key为null的value。
	- 真正会发生内存泄露的情况是使用了线程池的时候，ThreadLocal设为null了，然后线程结束，线程放回线程池中，这个线程一直不被使用，或者使用了但不调用get、set方法，那么这个期间就会发生真正的内存泄漏。